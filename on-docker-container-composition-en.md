# On Docker Container Composition

---

Author: Jeff Nickoloff

---

Docker containers are composed via links. A container’s interface is effectively expressed by expected link names (aliases), volume mount points, and exposed network ports. This expression is left to implementation, and that requires users to inspect the images they use and the software inside of them. Since one of the best reasons to use Docker is abstraction of technical detail this seems like a great candidate for improvement.

Currently if a user wants to compose containers, they would use commands like these:

```
docker run -d — name=“web” webimage
docker run -d — name=“monitor” — link web:http monitorimage
```

Arguments of the “link” flag takes the form <container_name>:<alias> where alias is the name that the software running inside the consumer container uses. The problem is that a new user has no idea what alias the software inside of monitorimage is expecting, and monitorimage cannot anticipate the alias that will be specified by the user. The best the image author can do is code to a specific alias and document that alias somewhere. But this still requires the user to dig into the image docs, or inspect the image itself.

This same issue applies to the `—volumes-from` flag that allows a user to map the volumes in one container into a new container. While the mechanism works, it requires that the author of the second container anticipates the mount points of any volumes that should be wired in.

## Interfaces and Dependencies

Today Docker lacks interface definitions. A container type system would eliminate the research required to build links between producer and consumer containers.

Consider an example. An HTTP interface is defined as follows:

```
interface Http { exposes: 80 }
```

This defines an interface named HTTP that can be used to describe any container that exposes port 80 on its network stack. Most web server images I’ve seen already expose port 80, and so there are already several image implementations of this interface. For the remainder of this article, suppose we have one such image named, “webimage.” These interface definitions would need their own repositories and distribution mechanism, similar to the infrastructure in place for images today.

Now consider a consumer. Maybe this is a simple monitor that runs periodically to check the state of a linked web server. Today, an image built for such a purpose would have to assume what name the linked image will be bound to, and then somewhere in its code or configuration note that it should connect to that host on port 80. These assumptions are currently difficult to communicate without any form of dependency declaration. Typical class declarations in typed object oriented programming languages include a set of typed dependencies. For example the Java Monitor class depends on a HttpService member:

```
class Monitor { private HttpService service; }
```

Applying the same dependency declaration style to Docker containers would create a few specific benefits. Lets see what such a declaration might look like. We’ll use the following Dockerfile to create an image called, “monitorimage.”

```
FROM busybox:latest
ADD webmonitor.sh /webmonitor.sh
INJECT Http
CMD /webmonitor.sh
```

Using the proposed INJECT directive, an image author can clearly communicate to users what assumptions the software contained will make. In this case, the containers built from, “monitorimage” will depend on a linked image that will have the http alias and expose port 80.

The second benefit is that it would enable validation. On instantiation of such a container, the linking system can validate that the referenced target conforms to the Http interface, and then link that target to the, “http” name inside of the container.

With interface modeling and typed dependency description in place, the user might build such links with the following commands:

```
docker run -d —name=“web” webimage
docker run -d —name=“monitor” —inject web monitorimage
```

The user can know to build their second run command in this way with minimal knowledge of, “monitorimage.” With all of this metadata in place, Docker could provide rich error messages on instantiation rather than making a user wait for a runtime failure. Today if I were to launch a container from monitorimage without wiring in the required webimage on the correct alias, Docker would allow it and I would discover my error because the software wouldn’t work. With this injection system in place, Docker could provide helpful errors like:

I failed to inject a container:

```
docker run -d —name=“monitor” monitorimage
[ERROR] failed to start: monitor; missing required Http dependency.
```

I injected a container that does not implement Http:

```
docker run -d —name=“database” databaseimage
docker run -d —name=“monitor” —inject database monitorimage
[ERROR] failed to start: monitor; database does not implement Http
```

This would avoid and trivialize triage of a whole class of problems that users encounter when trying to wire together containers.

## Thinking Bigger: Type Based Autowiring

With interface definition and dependency declaration in place, you are free to build further wiring simplifications. While there is power and flexibility in explicit wiring of containers, often there will only be a single container that implements a specific interface running on a single box. In these cases, it would be possible to anticipate the link that must be built.

```
docker run -d —name=“web” webimage
docker run -d —name=“monitor” —wired monitorimage
```

In fact, with autowiring in place the helpful names become less important so we could simplify to:

```
docker run -d webimage
docker run -d —wired monitorimage
```

Now that is a powerful and simple expression. What’s better is the great error messages that you can produce in the event that there are multiple candidates for wiring.

```
docker run -d webimage
docker run -d webimage
docker run -d — wired monitorimage
[ERROR] failed to start from: monitorimage; there are multiple candidates for the required Http dependency.
```

## The Proposal

I proposed these enhancements in [https://github.com/docker/docker/issues/8981](https://github.com/docker/docker/issues/8981). I’ve yet to begin hacking out a patch because I’ve been focusing on writing Docker in Action. But I hope that doesn’t come across as lack of interest. The interface based dependency declaration and wiring patterns have been used in several places. It is my hope that someone reading this article likes the idea so much that they might take up the banner to co-champion the enhancement.

There are a few other competing ideas in the proposal bin, but I think this one is worth investigating because it is a pattern that has been applied so successfully elsewhere.

---

Original source: [On Docker Container Composition](https://medium.com/@allingeek/on-docker-container-composition-a98788f1aa3c)
