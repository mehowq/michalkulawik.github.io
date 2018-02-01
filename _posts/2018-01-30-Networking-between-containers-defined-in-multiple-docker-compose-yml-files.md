---
title: Networking between containers defined in multiple docker-compose.yml files
---

One of the projects that I've been involved in was source controlled in two separate repositories - AngularJS and KeystoneJS/MongoDb repos. My task was to fix certain functionality within the Angular app. I got myself up to speed with the code but remember having some dependency issues when setting up the environment on my dev machine.


# Dockerization

I've recently started playing with Docker and thought that I could actually solve all of the above mentioned dependency issues by using preinstalled angular/node/mongodb docker images.

The containers worked perfectly. I could run all project components with no dependency errors.

## Automation

I thought it would be a good idea to use the docker-compose to script out creating the containers/volumes/network.

I've created `docker-compose.yml` for each of the projects - the Angular frontend and Keystone/MongoDb backend. All three containers are set to use exactly the same network.

![AngularJS app docker-compose.yml](/img/content/posts/2018/2018-01-30-docker-compose-yml-angularapp.png "AngularJS app docker-compose.yml")  
*AngularJS app docker-compose.yml*

![KeysoneJS cms docker-compose.yml](/img/content/posts/2018/2018-01-30-docker-compose-yml-keystonecms.png "KeysoneJS/MongoDb docker-compose.yml")  
*KeysoneJS/MongoDb docker-compose.yml*

However, I wasn't able to ping Keystone container from the Angular container for some reason. As you've probably noticed, networking between Angular container and Keystone is not really necessary in my case as the Angular app (port 4201) pulls the data through the exposed on the host machine Keystone API (port 8080). However, I really wanted to find out why they can't talk to each other internally.

I run `docker network ls` command and noticed that although I've used exactly the same network name in both of the files, Docker created a separate network for each of them anyway:

![Listing docker network](/img/content/posts/2018/2018-01-30-docker-network-ls.png "Listing docker networks")  
*Listing docker networks*

As you can see, Docker added a **prefix** to each network name specified in the `docker-compose` file. **The prefix name is based on the docker-compose.yml file root folder**.

## Moving containers onto the same network

In order to put all containers onto the same network, I set the Angular container network to the one that Docker created for Keystone (I run `docker-compose up` on Keystone app first):

![AngularJS app docker-compose.yml fix](/img/content/posts/2018/2018-01-30-docker-compose-yml-angularapp-fix.png "AngularJS app docker-compose.yml fix")  
*AngularJS app docker-compose.yml fix*

To check whether or not all three containers are on the same network, I run the following command:
```
docker network inspect keystonecms_test-network
```

![Inspecting docker network](/img/content/posts/2018/2018-01-30-docker-network-inspect.png "Inspecting docker network")
*Inspecting docker network*

Networking between containers has been fixed.



# Wrapping Up

When you want to try something new, you start with a simple thing. The same applies to a new framework - you don’t want to mess about with dependency issues before you even run a simple code.

Overall, I think Docker is a fantastic tool which allows you to try new things without a risk of cluttering/breaking your dev machine. It's very likely that you will find the framework that you are interested in as the number of images available on Docker hub is constantly growing. I love the fact that I can download an image, create a container and start working on a premade environment instantly. Having docker-compose file in the source control also means that anyone can pick up and run the project really quickly.