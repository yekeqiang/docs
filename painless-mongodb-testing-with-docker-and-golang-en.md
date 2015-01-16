# PAINLESS MONGODB TESTING WITH DOCKER AND GOLANG

---

##### Author: Niilo Ursin

---

![alt](http://resource.docker.cn/docker-mongodb-golang.jpg)

## BACKGROUND

We’re constantly looking for new technologies to ease the pain of our developers. Our heritage is in Java+Spring with small exceptions. Java8 and especially Spring Boot bring some most needed updates and changes to typically monolithic Java applications. And when you have proper api’s you just need a nice frontend framework to replace jsp’s and tons of jQuery: in our case we chose [AngularJS](https://angularjs.org/). The first project using Angular was done almost two years ago and currently all projects are done using it.

## OVER 10 YEARS OF JAVA LEAVES A MARK ON YOUR SOUL

For couple a years I have been looking for something better. Best thing in this profession is that you have great variety of choices. We have built couple projects with NodeJS and have learned how horrible Ruby ecosystem can be with Chef. Of course we have some Scala, but with other languages we have done only some experiments: Yes, I’m talking about you Clojure, Haskell and Rust. Then we have Go (golang). Which we’ve used only on couple of smallish services, but I’m really impressed about language, standard library, tooling and community around it. There’s a great deal of blog posts explaining why different companies have chosen Golang, this isn’t one of those posts, I might write that some day. Meanwhile get familiar with Golang with the interactive [A tour of Go](http://tour.golang.org/) if you learn by coding, [Effective Go](http://golang.org/doc/effective_go.html) if you prefer reading or A tour of [Go 34:40](http://research.swtch.com/gotour) if video’s is you thing.

## TESTING BURDEN

That was quite long intro to the actual topic. With all programming languages some unit testing for coding is needed, others go all the way with TDD and target for 100% test coverage. Dynamic languages require more testing for types. When your application is a decent sized you end up with hundreds of tests. Then comes the pain, depending on the language, your test runs start to take more time: What used to be couple of seconds will become minutes and in worst case tens of minutes. Team starts to run only unit tests and let CI handle integration tests. So you start mocking repository (database) calls. And build preload & cleanup methods for development database for integration tests. From time to time builds might fail for integration tests due to timeouts or just because two builds are running parallel with the same database.

## TESTING WITH GOLANG AND DOCKER

Golang will not make exception with this, but with support of Golang’s exceptionally fast build & test cycles and some Docker magic you can start MongoDB Docker container and run all tests with it in seconds! That really is seconds from start to finish, with exception for first run that will download and provision MongoDB Docker container.

I got actual inspiration from this tweet and ever since have been looking for excuse to check if this is true:

<blockquote class="twitter-tweet" lang="en"><p>More fun: <a href="https://twitter.com/Camlistore">@Camlistore</a> starting to use <a href="https://twitter.com/docker">@Docker</a> for unit tests. Here&#39;s our full 0.7 second MongoDB test: <a href="https://t.co/t1qcEJpQBW">https://t.co/t1qcEJpQBW</a> <a href="https://twitter.com/hashtag/golang?src=hash">#golang</a></p>&mdash; Brad Fitzpatrick (@bradfitz) <a href="https://twitter.com/bradfitz/status/432727349784612864">February 10, 2014</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

![alt](http://resource.docker.cn/tweet-snap.png)


## LET’S BUILD SOMETHING NICE WHERE WE CAN EXPERIMENT WITH DOCKER

I’ve been playing with my spare time toy Golang+AngularJS project [Inkblot](https://github.com/niilo/inkblot) for some time and now was perfect time to checkout if Docker is as magical as it has been hyped to be. There’s a small annoyance for OS X user when it comes to Docker: it runs on Linux only. Yes, you can install it to OS X with Boot2Docker that actually runs Docker daemon on virtualized (Virtualbox) Linux. I was using Vagrant with Ubuntu already as development environment for Inkblot so I just installed Docker on that.

First I got familiar with Camlistore implementation and copied it with pride, special thanks goes to [Brad Fitzpatrick](https://twitter.com/bradfitz), you have done exceptional work with Camlistore and Golang standard library. Thanks!

The actual test can be found here [story_test.go](https://github.com/niilo/inkblot/blob/master/api/server/story_test.go). For those who can’t read Golang, I added extra comments to most intresting parts of the code.


### Setup test environment

```
func TestStoryCreateAndGet(t *testing.T) {

  // Start MongoDB Docker container
  //
  // One of the most powerful features in Golang
  // is the ability to return multiple values from functions.
  // In this we get:
  // - containerID (type=ContainerID struct)
  // - ip (type=string)
  containerID, ip := dockertest.SetupMongoContainer(t)

  // defer schedules KillRemove(t) function call to run immediatelly
  // when TestStoryCreateAndGet(t) function is done,
  // so you can place resource clenup code close to resource allocation
  defer containerID.KillRemove(t)

  app := AppContext{}

  // Connect to Dockerized MongoDB
  mongoSession, err := mgo.Dial(ip)

  // Golang favors visible first hand error handling.
  // Main idea is that Errors are not exceptional so you should handle them
  if err != nil {
    Error.Printf("MongoDB connection failed, with address '%s'.", Configuration.MongoUrl)
  }

  // close MongoDB connections when we're finished
  defer mongoSession.Close()

  app.mongoSession = mongoSession

  // create test http server with applications route configuration
  ts := httptest.NewServer(app.createRoutes())
  defer ts.Close()

  storyId := testCreate(ts, t) // run create test
  testGet(ts, storyId, t) // run get test for created story
}
```

### Post json document to http handler

```
func testCreate(ts *httptest.Server, t *testing.T) string {

  postData := strings.NewReader("{\"text\":\"tekstiä\",\"subjectId\":\"k2j34\",\"subjectUrl\":\"www.fi/k2j34\"}")

  // create http POST with postData JSON
  res, err := http.Post(ts.URL+"/story", applicationJSON, postData)

  // read http response body data
  data, err := ioutil.ReadAll(res.Body)
  res.Body.Close()
  if err != nil {
    t.Error(err)
  }

  id := string(data)

  // verify that we got correct http status code
  if res.StatusCode != http.StatusCreated {
    t.Fatalf("Non-expected status code: %v\n\tbody: %v, data:%s\n", http.StatusCreated, res.StatusCode, id)
  }

  // verify that we got valid lenght response data
  if res.ContentLength != 5 {
    t.Fatalf("Non-expected content length: %v != %v\n", res.ContentLength, 5)
  }
  return id
}
```

### Test that previously created story exists

```
func testGet(ts *httptest.Server, storyId string, t *testing.T) {

  // create http GET request with correct path
  res, err := http.Get(ts.URL + "/story/" + storyId)
  data, err := ioutil.ReadAll(res.Body)
  res.Body.Close()
  if err != nil {
    t.Error(err)
  }

  body := string(data)

  // validate status code
  if res.StatusCode != http.StatusOK {
    t.Fatalf("Non-expected status code: %v\n\tbody: %v, data:%s\n", http.StatusCreated, res.StatusCode, body)
  }

  // validate that response has correct storyId
  if !strings.Contains(body, "{\"storyId\":\""+storyId+"\",") {
    t.Fatalf("Non-expected body content: %v", body)
  }

  // validate that content leght is what is should be
  if res.ContentLength < 163 && res.ContentLength > 165 {
    t.Fatalf("Non-expected content length: %v < %v, content:\n%v\n", res.ContentLength, 160, body)
  }

}
```

## SEEING IS BELIEVING

So it starts MongoDB Docker container, configures it to application then creates http server with built in testing support. Then we setup same routes to server that actual server has and run two request against test server, first creates story comment and another tries to get it. All data is stored and fetched from MongoDB. And how long all of this takes time?


![alt](http://resource.docker.cn/mongodb-time-takes.png)


## DOCKER IS FOR ALL DEVELOPERS NOT JUST GOLANG USERS

And for those that are not that fortune that they can use Golang Docker can help all you too. It wouldn’t be so super fast as with Golang but as fast as using external MongoDB server without extra cleanup hassle. No doubt that Docker is game changer in virtualization business and all hype that it has is well earned. There’s just no excuses to write any mock tests for MongoDB functions.

---

Original source: [PAINLESS MONGODB TESTING WITH DOCKER AND GOLANG](http://developers.almamedia.fi/painless-mongodb-testing-with-docker-and-golang/)