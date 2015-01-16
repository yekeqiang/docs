# CI & CD With Docker, Beanstalk, CircleCI, Slack, & Gantree


---

Author: Amber Kaplan

---

> This is a follow-up post to a series highlighting Bleacher Report’s continuous integration and delivery methodology by Felix Rodriguez. To find out how they were previously handling their stack, visit the [first](http://sauceio.com/index.php/2014/06/continuous-delivery-through-elastic-beanstalk/), [second](http://sauceio.com/index.php/2014/06/bleacher-reports-continuous-integration-delivery-methodology-creating-an-integration-testing-server/), and [third](http://sauceio.com/index.php/2014/06/bleacher-reports-continuous-integration-delivery-methodology-test-analytics/) posts from June 2014. 

There is definitely a huge [Docker](https://www.docker.com/) movement going on in the dev world right now and not many QA Engineers have gotten their hands dirty with the technology yet. What makes Docker so awesome is the ability to ship a container and almost guarantee its functionality.

At Bleacher Report we get to play with some amazing, cutting edge tools. While [CircleCI](https://circleci.com/) has been in the game for a while, they were the first CI system to adopt [Docker support](https://circleci.com/integrations/docker). With just one line of code you have access to all of the power of docker. Bleacher Report also let me build an open source tool called [Gantree](https://github.com/feelobot/gantree) that allows us to manage and deploy our applications seamlessly.

Not only did we want to ship our containers from development to production seamlessly, we also wanted to ensure the image we built actually passed our integration and unit tests as well. While the below example only shows a small set of unit tests, it is a great sample application due to its simplicity.

Note, our Carburetor app is an [Elixir](http://elixir-lang.org/) application that uses [mix](http://elixir-lang.org/docs/master/mix/) to run its tests.

## Getting Started

- First you need to create an account with [CircleCi](https://circleci.com/) and add your Github repository.

- Then, commit a circle.yml

- **Example circle.yml**:

```
machine:
  ruby:
    version: 2.1.3
  services:
    - docker
  environment:
    SHORT_SHA: $(echo $CIRCLE_SHA1 | cut -b1-7)
    TAG_NAME: “br-$CIRCLE_BRANCH-$SHORT_SHA”
    IMAGE_PATH: “bleacher/cms”
    DOCKER_RUN_CMD: “elixir -pa _build/prod/consilidated -S mix test”
    PROD: carburetor-prod-s1
    STAG: carburetor-stag-s1
 
dependencies:
  pre:
    - docker login -e $DOCKER_EMAIL -u $DOCKER_USER -p $DOCKER_PASS
    - docker build -t $IMAGE_PATH:$TAG_NAME .
 
test:
  override:
    - docker run -tip 3000:3000 bleacher/carburetor:$TAG_NAME $DOCKER_RUN_CMD
 
deployment:
  production:
    branch: release
    commands:
      - gem install gantree
      - docker push $IMAGE_PATH:$TAG_NAME
      - gantree deploy $PROD -i $IMAGE_PATH -t $TAG_NAME
  staging:
    branch: continuous-deployment
    commands:
      - gem install gantree
      - docker push $IMAGE_PATH:$TAG_NAME
      - gantree deploy $STAG -i $IMAGE_PATH -t $TAG_NAME
```

- Change the image_path to your docker hub account/repo

- Add your [environment variables](https://circleci.com/docs/environment-variables) needed

- Remove the docker login steps if you are using an open source image.

- Change your docker run variable to match the command needed to kickoff your tests.

## Running Tests:

![alt](http://resource.docker.cn/running-tests.png)

- This should now be able to deploy your application.

## Gantree deploy:

![alt](http://resource.docker.cn/deploy1-600-487.png)

## Slack notification from ubuntu:

- This will only happen if you have a .gantreecfg with your information added in the repo.

![alt](http://resource.docker.cn/slack-600-542.png)

**A note about integration tests**

This example does not show how to run integration tests but this is dependent on your language and testing choice. One way to theoretically handle this is to have multiple docker run commands. Here is an example of a rails application:

```
test:
  override:
    - docker run -ti bleacher/carburetor:$TAG_NAME bundle exec rake test
    - docker run -tip 3000:3000 bleacher/carburetor:$TAG_NAME bundle exec rails s
    - docker run -ti bleacher/carburetor:$TAG_NAME bundle exec rake test:integration:sauce
```

The first command runs the integration tests, the second starts the web server and the third starts the integration test suite. You just have to make sure your tests are pointed at localhost:3000.

> Have a question or comment? Follow Felix on Twitter at [@feelobot](https://twitter.com/feelobot). His original post can be found [here](http://www.feelobot.com/ci/).

---

Original source:[CI & CD With Docker, Beanstalk, CircleCI, Slack, & Gantree](http://sauceio.com/index.php/2014/12/ci-cd-with-docker-beanstalk-circleci-slack-gantree/)