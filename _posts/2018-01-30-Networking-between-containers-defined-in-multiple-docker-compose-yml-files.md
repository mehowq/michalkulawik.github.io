---
title: Networking between containers defined in multiple docker-compose.yml files
---

One of the projects that I was involved in is source controlled in two separate repositories - AngularJS and KeystoneJS/MongoDb repos. My task was to fix certain functionality within the Angular app. I got myself up to speed with the code but remember having dependency issues when setting up the environment on my dev machine.


# Dockerization

I recently started playing with Docker and thought that I could actually solve all the mentioned dependency issues by using preinstalled angular/node/mongodb docker images.

The containers worked perfectly. I could run all project components with no dependency errors.

## Automation

I thought it would be good to use the docker-compose to script out creating the containers/volumes/network.

I've created `docker-compose.yml` for each of the projects - the Angular frontend and Keystone/MongoDb backend. All three containers are set to use exactly the same network.

![AngularJS app docker-compose.yml](/img/content/posts/2018/2018-01-30-docker-compose-yml-angularapp.png "AngularJS app docker-compose.yml")  
*AngularJS app docker-compose.yml*

![KeysoneJS cms docker-compose.yml](/img/content/posts/2018/2018-01-30-docker-compose-yml-keystonecms.png "KeysoneJS cms docker-compose.yml")  
*KeysoneJS cms docker-compose.yml*

However, the Angular app couldn't get data from the Keystone app for some reason.

I run `docker network ls` command and noticed that although I used exactly the same network name in both of the files, Docker created a separate network for each of them anyway:

![Listing docker network](/img/content/posts/2018/2018-01-30-docker-network-ls.png "Listing docker networks")  
*Listing docker networks*

As you can see, Docker **added a prefix to each network name** specified in the `docker-compose` file. The **prefix name is based on the docker-compose.yml file root folder**.

## Moving containers onto the same network

In order to put the Angular app onto the same network as Keystone, I set the Angular app network to the one that Docker created for Keystone (I run `docker-compose up` on Keystone app first):

![AngularJS app docker-compose.yml fix](/img/content/posts/2018/2018-01-30-docker-compose-yml-angularapp-fix.png "AngularJS app docker-compose.yml fix")  
*AngularJS app docker-compose.yml fix*

I checked if all three containers are on the same network by running:
```
docker network inspect keystonecms_test-network
```

![Inspecting docker network](/img/content/posts/2018/2018-01-30-docker-network-inspect.png "Inspecting docker network")
*Inspecting docker network*

The AngularJS app can now communicate with the other containers on the network.


# Wrapping Up

When you want to try something new, you start from a simple thing. The same applies to a new framework - you don’t want to mess about with the dependency issues before you even run a simple code.

I think Docker is a fantastic way to try new things without a risk of cluttering / breaking your dev machine. It's very likely that you will find the framework that you need as a choice of images on Docker hub is constantly growing. I love the fact that I can download an image, create container and start working on a tested environment instanlty. Having docker-compose file in the source control also means that anyone can pick up and run a project really quickly. Docker containers are lightweight, fast, modular and configurable - I can see myself using them a lot.