---
title: "Dockerizing Springboot Elasticsearch Application"
layout: post
date: 2021-05-27
tag: 
    - Springboot
    - Elasticsearch
    - Docker
    - Docker compose
description: "This blog outlines my learnings with containerizing springboot elasticsearch application using docker compose"
image: https://upload.wikimedia.org/wikipedia/commons/f/f4/Elasticsearch_logo.svg
headerImage: https://upload.wikimedia.org/wikipedia/commons/f/f4/Elasticsearch_logo.svg
category: blog
author: pranjal
---
## Dockerizing Springboot Elasticsearch application with docker compose

So all this started when I decided to create an application that recommends food recipes based on the ingredients you have at your home. 

This is the tech stack that I want to use:
* Spring Boot for a RESTful API Server
* React Native with Expo for frontend and
* Elastic Search for full text search queries on the recipes. 

I have already found a [dataset of around 7k Indian recipes on kaggle](https://www.kaggle.com/kanishk307/6000-indian-food-recipes-dataset). So I am all set for development.

Now, this application is not meant to be an application for the general public. Its meant to be run on my private server in a private network for some limited number of people. Hence, I want to keep the deployment and maintainance overhead to a minimum and not manually do a full fledged for elastic search cluster deployment.

I am aiming for a single command deployment. 

Obviously, I am going the docker route. I am going to dockerize my application and run it with `docker run` command, or at least thats the plan.

This is my development environment:
* Mac Mini M1 with 16GB Memory
* Java 11
* Spring Boot 2.5.0
* Maven

So the following is my learning from setting all this up. 

1. Docker now has support for Apple Silicon 🤩🤩🤩

However, it still requires `Rosetta 2` as some binaries are yet to make the transition to Apple Silicon. If you need, you can use this command to install Rosetta 2.

![enter image description here](https://i.imgur.com/YUropvW.png)

2. Use docker compose

I quickly realised that for a setup like this, I will have to use `docker compose`. If you are new to docker or docker compose, let me quickly clarify this.

Docker is used to containerize your application while docker compose is a discriptor about how multiple containers behave in each others presence. Do they share a network? Do they share a volume? What about environment variables? In what order should they be run? Do you need to scale them? If yes, how many containers would you need? 

So now, the plan has been modified a bit, I am going to have two containers and I will run them using `docker compose up`. Great, I am still aiming for a single command deplyment. 

[Getting started with docker compose](https://docs.docker.com/compose/gettingstarted/) is the best place to start if you are curious.

3. Getting elasticsearch to run is amazingly easy with docker

Just go to the [elastic search website](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html) and you will find a `docker run` command for a single node dev deployment and a `docker compose` for a more elaborate 3 node cluster setup. I did the best of both and selected single node deployment with docker compose.

4. Dockerizing Spring Boot is easy once you know what not to do

Spring has a [great tutorial](https://spring.io/guides/gs/spring-boot-docker/) about how to dockerize spring boot apps and how to divide them in layers for efficiency. But I still screwed up mine and had to waste a day figuring out what was the issue. 

> The springboot tutorial says there is a [limitation with Docker for Mac](https://www.docker.com/products/docker#/mac). So it might be a good idea to avoid using container IP as far as possible.

The weirdest thing was, my app would run perfectly fine in development mode. But it would fail when I ran it in container.

There is a reason for it (or thats what I thought at the time). To keep the image size to a minimum, I was using a distroless java 11 base image. On the other hand, on my dev machine I was using `OpenJdk 11`. This java 11 distroless image was buggy as far as I can tell. 

Finally, after wasting one day trying to fix this, I gave up and shifted to good old `openJdk:11`. Although I had to compromise the image size (which increased 3x), atleast I got the service running.

(Un)fortunately, I am unable to reproduce the same error anymore so I can not upload a screenshot. (Murphy's Law here 😅) However, if you are feeling curious, this is what I was using `gcr.io/distroless/java:11`. And here is my observation

|Base Image| Final Image Size  |
|--|--|
|`gcr.io/distroless/java:11`|257 MB|
|`openjdk:11`|695 MB|

Now, I am by no means an expert in this. I am not even aware of the differences in those two base images, but that is a striking difference for me. 

5. Docker containers don't connect to other containers on their own. Or do they? 🤔

When I started off, I did not know that you need special configuration to enable the containers to talk to each other. 

So I ended up creating a bridge network in my `docker-compose.yml`. Now everything works as expected. Both my containers (springboot and elasticsearch) get connected to the new bridge network and they can discover each other. However, there is a caveat.

When running locally, you can specify `localhost:9092` for springboot to connect to the elasticsearch docker. However, its a different story in the container. 

When running in a container, the hostname has to be replaced with the container name. So, the new address becomes `<elasticsearch-container-name>:9092`

I don't want to remember this caveat every time I need to deploy something. So I created an environment variable, called it `ELASTIC_HOST`. I am setting its value in my `docker-compose.yml`. And I am reading it in my `application.yml` file.

That felt nice. Earlier my application was not running but now my application can run and connect to the elastic docker sometimes. Thats progress!

By the way, I just found out that docker compose creates a `default` network. So maybe I unnecessarily created a bridge network and maybe the containers can actually talk to each other on their own. Who knows?! Well, not me. 

Anyhow, the problem I face now, is that I need to ask spring boot to wait until elasticsearch cluster is up and running before spring boot tries to connect to it. Docker compose has a `depends_on` option to express the startup and shutdown order.  More about this [here](https://docs.docker.com/compose/startup-order/).

However, this wont work the way you expect it to. And its understandable. 

So `depends_on` will wait for the container to run before it starts the dependant containers. However, a container in running state does not mean that the application is in running state. For example, in case of a database, the database container might be in running state but that does not mean that the database is accepting connections yet. 

Docker compose can not know the application readiness and hence this needs to be handled by the developer. The recommended way is to retry on connection failure. However, the more widely used approach seems to be a busy waiting scripts ([wait-for-it](https://github.com/vishnubob/wait-for-it), [dockerize](https://github.com/jwilder/dockerize) etc) that keep checking if the service is up. 

I am planning to go the recommended route of retrying the connection on failure. This functionality is not yet available in spring boot. There is an open issue about this [#4779](https://github.com/spring-projects/spring-boot/issues/4779). Meanwhile, I am planning to implement my own.

As always, the source code is available on [github](https://github.com/jalispran/springboot-elasticsearch-docker). Check that out for the code and stay tuned for the retry mechanism.
