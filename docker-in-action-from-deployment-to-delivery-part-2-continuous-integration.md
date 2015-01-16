# Docker in Action - Development to Delivery, Part 2

---

Author: Michael Herman

---

This three part series will teach you everything you need to know about developing with Docker - from setting up your environments and utilizing Flask on Docker to detailing a powerful development workflow that covers setting up a fully functional development environment, on your Mac, and managing continuous integration and delivery.

1. Part 1: Local Docker Setup
2. **Part 2: Continuous Integration (current)**
3. Part 3: Continuous Delivery

**END GOAL:**

![alt](http://resource.docker.cn/steps-dec-15.jpg)

Last [time](https://blog.rainforestqa.com/2014-11-19-docker-in-action-from-deployment-to-delivery-part-1-local-docker-setup/) we set up our local environment, detailing the basic process of building an *image* from a *Dockerfile* and then creating an instance of the image called a container, which runs our Flask app. This time, let’s look at a nice continuous integration workflow powered by [CircleCI](https://circleci.com/).

> Services used: Docker Hub, Github, CircleCI

## Docker Hub

Thus far we’ve worked with Dockerfiles, images, and containers. If you’re familiar with the Git workflow, then images are like Git repositories while containers are similar to a cloned repository. Sticking with that metaphor, [Docker Hub](https://hub.docker.com/), which is repository of Docker images, is akin to Github.

1. Signup [here](https://hub.docker.com/account/signup/), using your Github credentials.

2. Then add a new automated build. Find your Github repo that you pushed to in the first tutorial. Once added, this will trigger an initial build. Make sure the build is successful.

### Docker Hub as a CI server

Docker Hub, in itself, acts as a continuous integration server since you can configure it to create a build every time you push a new commit to Github. In other words, it ensures you do not cause a regression that completely breaks the build process when the code base is updated.

> Keep in mind by using an [automated build](https://docs.docker.com/userguide/dockerrepos/#automated-builds), you cannot use the `docker push` command. Builds must be triggered by committing code to your GitHub or BitBucket repository.

Let’s test this out. Update the `test_data()` function in test.py:

```
def test_data(self):
    tester = app.test_client(self)
    response = tester.get('/data')
    self.assertEqual(response.status_code, 200)
    self.assertEqual(response.content_type, 'application/json')
```

> Need the code? Grab it from the [repo](https://github.com/realpython/flask-docker-workflow).

Commit and push to Github to generate a new build.

Bottom-line: It’s good to know that if a commit does cause a regression that Docker Hub will catch it, but since this is the last line of defense before deploying you ideally want to catch any breaks before generating a new build on Docker Hub. Plus, you also want to run your unit and integration tests from a *true* continuous integration server.

Enter CircleCi.

## CircleCI

[CircleCI](https://circleci.com/) is a continuous integration and delivery platform, which supports Docker. Given a Dockerfile, CircleCI builds an image, starts a new container, and then runs tests against that container.

The process to follow is simple:

1. Code locally on a [feature branch](https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow)
2. Open a pull request on Github against the master branch
3. Run automated tests against the Docker container
4. If tests pass, manually merge the pull request into master
5. Once merged, the automated tests run again
6. If tests pass, a build is created on Docker Hub
7. Once the build is created, it is then automagically deployed to production

Let’s take a look…

### Setup

Sign up with your Github account, then add a new project, and select your repo. At this point, CircleCI automatically-

- Adds a webhook in the repo so that anytime you push to Github, the tests are triggered to run. (You should receive an email about this.)
- Starts running a new build.

This build should pass, but we need to configure CircleCI specifically for Docker. So, let’s add a configuration file.

***circle.yml***

Add the following build commands:

```
machine:
  services:
    - docker

dependencies:
  override:
    - docker info
    - docker build -t mjhea0/flask-docker-workflow .

test:
  override:
    - docker run -d -p 80:80 mjhea0/flask-docker-workflow; sleep 10
    - curl --retry 10 --retry-delay 5 -v http://localhost:80
    - pip install -r requirements.txt
    - python app/tests.py
```

> Make sure you replace `mjhea0` with your Docker Hub username.

Essentially, we create a new image, run the container, then test - first that the app is live (e.g., the web process is running) and then that our unit tests pass. With the *circle.yml* file created, push the changes to Github to trigger a new test. *Remember: this will also trigger a new build on Docker Hub*.

> CircleCI does not support the caching feature discussed in [Part 1](https://blog.rainforestqa.com/2014-11-19-docker-in-action-from-deployment-to-delivery-part-1-local-docker-setup/), so by default the entire image is rebuilt from scratch each time. Check out the official CircleCI [documentation](https://circleci.com/docs/docker#caching-docker-layers) for an alternative way to speed up builds.

If all went well, that should have passed. Before we call it quits, we need to change our workflow since we won’t be pushing directly to the master branch anymore.

### Feature Branch Workflow

For these unfamiliar with the Feature Branch workflow, check out [this](https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow) excellent introduction.

Here’s the basic workflow that we’ll utilize:

1. Create a feature branch from master.
2. Write your code and tests on the feature branch.
3. Issue a pull request to merge your feature branch back to the master branch.
4. Run the tests from CircleCI against the feature branch.
5. If the tests pass, manually merge the commit into Master.
6. Run the tests from CircleCI against the Master branch.

Let’s run through a quick example…

### Create the feature branch

```
$ git checkout -b circleci-test master
Switched to a new branch 'circleci-test'
```

### Update the app

Add a new assert to `test_data()` in *tests.py*:

```
self.assertIn('Seattle', response.data)
```

### Issue a Pull Request

```
$ git add app/tests.py
$ git commit -am "circleci-test"
$ git push origin circleci-test
```

Even before you create the actual pull request, CircleCI runs the automated tests. While the tests are running, go ahead and create the pull request, then once the tests pass, press the Merge button (with confidence!). Once merged, the build is triggered on Docker Hub.

### Refactoring the workflow

If you jump back to the overall workflow at the top of this post, you’ll see that we don’t actually want to trigger a new build on Docker Hub until the tests pass against the master branch. So, let’s make some quick changes…

1. Open your repository on Docker Hub, and then under Settings click Automated Build.
2. Uncheck the Active box: “When active we will build when new pushes occur”.
3. Save.
4. Click Build Triggers under Settings
5. Change the status to on.
6. Copy the example curl command - i.e., `$ curl --data "build=true" -X POST https://registry.hub.docker.com/u/mjhea0/flask-docker/trigger/488f6652-6e9d-11e4-9a92-b6e30c63109a/`

Now add the following code to the bottom of your circle.yml file:

```
deployment:
  hub:
    branch: master
    commands:
      - $DEPLOY
```

Here we fire the `$DEPLOY` variable after we merge to master and the tests pass. We’ll add the actual value of this variable as an environment variable on CircleCI:

1. Open up the Settings, and click Environment variables.
2. Add a new variable with the name “DEPLOY” and paste the example curl command (that you copied) from Docker Hub as the value.

Now let’s test.

```
$ git add -A
$ git commit -am "circleci-test"
$ git push origin circleci-test
```

Open a new pull request, and then once the tests pass, merge to master. Now once the tests pass, the new build will trigger on Docker Hub via the curl command. Nice.

## Conclusion and Next Steps

Next time we’ll look at delivery. See you then!

---

Original source: [Docker in Action - Development to Delivery, Part 2](https://blog.rainforestqa.com/2014-12-08-docker-in-action-from-deployment-to-delivery-part-2-continuous-integration/)