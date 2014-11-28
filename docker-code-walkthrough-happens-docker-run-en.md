# Docker Code Walkthrough – What Happens During a Docker Run?

----

Author: Frank Scholten

---

In this blog I will answer the following question: What happens inside Docker during a docker run command?

## Getting Started

To get started let’s clone the [Docker Github repo](https://github.com/docker/docker) and inspect the code.

``
$ git clone https://github.com/docker/docker
```

## Looking around

Open the project using your favorite editor or IDE and look around in the main tree. Docker is written in [Golang](https://golang.org/) and consists of many packages. For instance, from top to bottom you see api, builder, builtins, contrib and so on. Some folders contain subfolders with more packages.

## Inside the Docker executable

First things first, let’s find func main, the Golang function that is executed when we run the Docker executable. There are actually more than 30 main funcs in the code tree. These are for all sorts of utilties which I won’t go into right now. Let’s continue with the main we are looking for: the one that lives in docker/docker.go. Let’s look at it more closely.

## Parsing flags

See the snippet below, showing the first dozen lines of the main func. When Docker starts it runs any initializers via reexec, if any, then it parses the flags for the Docker executable via the [mflag](https://github.com/docker/docker/tree/master/pkg/mflag) package. This packages lives under pkg/mgflag and is aliased as flag. At this point it can print out version information if necessary or enable debug mode and debug logging.


```
func main() {
  if reexec.Init() {
    return
  }
 
  flag.Parse()
  // FIXME: validate daemon flags here
 
  if *flVersion {
    showVersion()
    return
  }
 
  if *flDebug {
    os.Setenv("DEBUG", "1")
  }
 
  initLogging(*flDebug)
 
  ... SNIP
 
}
```

## Dispatch Docker command & error handling

After option parsing Docker captures hosts settings and performs [TLS verification](https://docs.docker.com/articles/https/) for the server, if necessary. This happens between line 40 and 107. See the snippet below. The flags that were parsed earlier are passed to the **Cmd** method from the **DockerCli** type in **api/client/cli.go**. If an error occurs it is logged and the program is exited.


```
func main() {
 
  ... SNIP
 
  if err := cli.Cmd(flag.Args()...); err != nil {
    if sterr, ok := err.(*utils.StatusError); ok {
      if sterr.Status != "" {
        log.Println("%s", sterr.Status)
      }
      os.Exit(sterr.StatusCode)
    }
    log.Fatal(err)
  }
}
```

## The cli package

Let’s dive into the **cli** package to see how the Docker commands are being handled. To be able to run the subcommands we look at 3 things:

- DockerCli struct
- Cmd method
- GetMethod method

### DockerCli

The DockerCli struct contains datastructures each Docker command requires such as the protocol that is used, in-, output- and error writers as wel TLS specific data structures.


```
type DockerCli struct {
    proto      string
    addr       string
    configFile *registry.ConfigFile
    in         io.ReadCloser
    out        io.Writer
    err        io.Writer
    key        libtrust.PrivateKey
    tlsConfig  *tls.Config
    scheme     string
    // inFd holds file descriptor of the client's STDIN, if it's a valid file
    inFd uintptr
    // outFd holds file descriptor of the client's STDOUT, if it's a valid file
    outFd uintptr
    // isTerminalIn describes if client's STDIN is a TTY
    isTerminalIn bool
    // isTerminalOut describes if client's STDOUT is a TTY
    isTerminalOut bool
    transport     *http.Transport
}
```

### Cmd method

The Cmd func’s responsibility is to translate the command arguments to a function using the **getMethod** func. It already supports multiple commands for the future, possibly [docker groups create](https://github.com/docker/docker/issues/8637), although as far as I know no such commands have been implemented yet.


```
func (cli *DockerCli) Cmd(args ...string) error {
    if len(args) &gt; 1 {
        method, exists := cli.getMethod(args[:2]...)
        if exists {
            return method(args[2:]...)
        }
    }
    if len(args) &gt; 0 {
        method, exists := cli.getMethod(args[0])
        if !exists {
            fmt.Println("Error: Command not found:", args[0])
            return cli.CmdHelp(args[1:]...)
        }
        return method(args[1:]...)
    }
    return cli.CmdHelp(args...)
}
```

### getMethod

Notice the getMethod is lowercased. This means it is not [exported](https://golang.org/ref/spec#Exported_identifiers) to outside packages so it’s only available from within **cli**. So how does this method find the correct func? See the snippet below. It first builds up a string starting with **Cmd** concatenated with the capitalized arguments. In case of a Docker run the **methodName** variable will be **CmdRun**. Using the **MethodByName** func from Golang’s [reflect](http://golang.org/pkg/reflect/) package it retrieves a function pointer and returns it.


```
func (cli *DockerCli) getMethod(args ...string) (func(...string) error, bool) {
    camelArgs := make([]string, len(args))
    for i, s := range args {
        if len(s) == 0 {
            return nil, false
        }
        camelArgs[i] = strings.ToUpper(s[:1]) + strings.ToLower(s[1:])
    }
    methodName := "Cmd" + strings.Join(camelArgs, "")
    fmt.Println(methodName)
    method := reflect.ValueOf(cli).MethodByName(methodName)
    if !method.IsValid() {
        return nil, false
    }
    return method.Interface().(func(...string) error), true
}
```

## CmdRun

Finally we arrive at the func responsible for running a container: **CmdRun** at api/client/commands.go. This file contains all Docker commands. Arguments for the run itself are now parsed, such as the image, the command and other arguments. Since we have already been through that I won’t show that code. Instead I show something more interesting: starting a new container to run the command in.

## Creating the container

The snippet below shows how the container is created. The configuration of the container is merged from the run config, and the host config.
The actual call to create the container is a HTTP POST to the Docker server.


```
func (cli *DockerCli) CmdRun(args ...string) error {
 
  SNIP ...
 
  runResult, err := cli.createContainer(config, hostConfig, hostConfig.ContainerIDFile, *flName)
    if err != nil {
      return err
    }
  SNIP ...
 
}
 
func (cli *DockerCli) createContainer(config *runconfig.Config, hostConfig *runconfig.HostConfig, cidfile, name string) (engine.Env, error) {
    containerValues := url.Values{}
    if name != "" {
        containerValues.Set("name", name)
    }
 
    mergedConfig := runconfig.MergeConfigs(config, hostConfig)
 
    var containerIDFile *cidFile
    if cidfile != "" {
        var err error
        if containerIDFile, err = newCIDFile(cidfile); err != nil {
            return nil, err
        }
        defer containerIDFile.Close()
    }
 
    //create the container
    stream, statusCode, err := cli.call("POST", "/containers/create?"+containerValues.Encode(), mergedConfig, false)
    //if image not found try to pull it
    if statusCode == 404 {
        fmt.Fprintf(cli.err, "Unable to find image '%s' locally\n", config.Image)
 
        // we don't want to write to stdout anything apart from container.ID
        if err = cli.pullImageCustomOut(config.Image, cli.err); err != nil {
            return nil, err
        }
        // Retry
        if stream, _, err = cli.call("POST", "/containers/create?"+containerValues.Encode(), mergedConfig, false); err != nil {
            return nil, err
        }
    } else if err != nil {
        return nil, err
    }
 
    var result engine.Env
    if err := result.Decode(stream); err != nil {
        return nil, err
    }
 
    for _, warning := range result.GetList("Warnings") {
        fmt.Fprintf(cli.err, "WARNING: %s\n", warning)
    }
 
    if containerIDFile != nil {
        if err = containerIDFile.Write(result.Get("Id")); err != nil {
            return nil, err
        }
    }
 
    return result, nil
}
```

This covers what’s happening inside the Docker client. There is of course a lot more code to be explored in the Docker server and in [libcontainer](https://github.com/docker/libcontainer) but that will be left to a future blog post.

---

Original source: [Docker Code Walkthrough – What Happens During a Docker Run?](http://container-solutions.com/2014/11/docker-code-walkthrough-happens-docker-run/)