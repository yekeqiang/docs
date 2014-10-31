# Using Elastic Beanstalk with DockerHub or a Docker Registry

---

##### Author: Colin Johnston

---

Combining DockerHub (using either public or private repositories) or your own Docker registry and Amazon’s Elastic Beanstalk creates a very simple method of delivering a versioned application into a production environment.

## The Benefit of Elastic Beanstalk with DockerHub:

For smaller engineering organizations, for organizations running lean on Operations or just staying within the walled-garden of AWS the Elastic Beanstalk <-> DockerHub/Registry combination combines low operational overhead, versioned application rollout/rollback and reliable PaaS service. Assuming you are already building Docker images, the entire setup can be completed within a half of a day.

## Overview of Elastic Beanstalk and Docker Deployment Flow:

1. Build source code and dependencies into a Docker image.
2. Push an image to DockerHub (use “docker push”)
3. Push an “Application Version” (use a Dockerfile).
4. Deploy the application version to a “QA”/”Staging”/”Pre-production” environment, if required.
5. Deploy the application version to Production

The diagram below is a visual representation of the deployment flow, from start (creating an image) to finish (deployment into Production).

![alt](http://resource.docker.cn/dockerhub-to-elasticbeanstalk.png)

## Initial Setup:

If you wish to utilize a private DockerHub repository or your own registry/repository combination and you requires authentication, you’ll need to do a bit of preparation. This preparation is described below.

1. You will need to create and upload a “.dockercfg” file to S3. This file provides authentication information to Amazon’s Elastic Beanstalk. The dockercfg file contains the following text:

````
{"https://index.docker.io/v1/":{"auth":"3srCWya6OO3y9KUkKLrNZNunIMGiB2Fc","email":"colin@cloudavail.com"}}
```

You can create a .dockercfg file by running docker login and entering your username and password and the login prompt. This will create a .dockercfg file in the format required by Elastic Beanstalk.

2. If you do use a dockercfg file, the Elastic Beanstalk hosts will need access to it through an AWS IAM profile.

## Performing a Deployment:

### Upload an Image to DockerHub

Upload Docker image(s) to DockerHub. This makes images available for Elastic Beanstalk or any other host capable of running Docker. A typical Docker push can be done through the command line:

```
docker push my_organization/my_repository:version (example: cloudavail/test_tomcat_app:0.1.0)
```

or as a target when using a build tool. For example, I’ve pushed Docker code using Transmode’s gradle-docker plugin.

### Create a Dockerrun file for the given “Application Version”

The Dockerrun file is used to describe an application to the Elastic Beanstalk platform. Practically speaking a Dockerun file creates an “Application Version” and instructs Elastic Beanstalk how to do both of the following:

- “get” an image from DockerHub
- how to configure a Docker container.
- 
A dockerrun file looks something akin to the following:

```
{
  "AWSEBDockerrunVersion": "1",
  "Authentication": {
    "Bucket": "my_bucket",
    "Key": "my_dockercfg_file"
  },
  "Image": {
    "Name": "cloudavail/my_tomcat_app:0.1.0",
    "Update": "true"
  },
 "Ports": [
    {
      "ContainerPort": "8080"
    }
  ]
}
```

The “Authentication” section is required if authentication is needed to access images that are stored in a private repository: “my_bucket” is the name of the bucket where the authentication file is stored and “Key” is the path to the given authentication file. In my example, I’d use something akin to the following

```
"Bucket": "cloudavail_releases",
 "Key": "docker/dockercfg"
```

The “Image” section contains the path to a given image and tag.
The “Ports” section contains the port that should be open – which, by default, is mapped to port 80.

### Upload the “Application Version” to Elastic Beanstalk

You’ll need to submit the previously created “Dockerrun” file to Amazon. You can do this in one of two ways:

1. Upload through the UI.
2. Upload the Dockerrun file to S3 and then issue an API call using Amazon’s CLI tools or custom tooling of your choice.

One cool thing to note – Amazon retains old application versions to allow versioned rollout and rollback. For instance, you can select an application version “0.1.2”, deploy this version to QA, test and then deploy the exact same version to Production. If you needed to rollback, you can select the application version 0.1.1 and deploy this version to Production. The screenshot below demonstrates this:

![alt](http://resource.docker.cn/elasticbeanstalk-application-versions.png)

### Deploy the newly created Application Version

1. Deploy through the UI.
2. Deploy through an API call using Amazon’s CLI tools or custom tooling of your choice.

---

Original source: [Using Elastic Beanstalk with DockerHub or a Docker Registry](http://cloudavail.com/2014/10/26/elasticbeanstalk-docker/)
