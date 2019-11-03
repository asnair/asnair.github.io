+++
date = "2019-01-02T19:05:41-07:00"
title = "Docker, React, and Security"
draft = false
tags = ["Docker", "DevOps", "Tutorial"]
categories = ["DevOps"]

+++
<a name="title"></a>
------

[Docker, React, and Security](#title)

1. [React](#react)
2. [Docker](#docker)
  - [Git](#git)
  - [Installing Docker](#install-docker)
  - [Security](#security)
  - [Using Docker Hub](#using-docker-hub)
  - [Useful Docker Commands](#useful-docker-commands)
  - [Lessons from Docker](#lessons-from-docker)
3. [Docker Machine](#docker-machine)
4. [Troubleshooting](#troubleshooting)

------

This post has been constructed over the course of roughly 4 months, and was done with my friend [Zach](https://Github.com/zkeim1997) as independent research in docker security related to dockerizing react web apps. All the code can be found in the [repo on Github](https://Github.com/asnair/react-docker). I'd like to point out that the broken branch wasn't working for some (still) unknown reason related to versioning between when we set up the system and when we upgraded some of the components including nodejs. This post will walk through the process of first creating the basic react app, setting up docker, benchmarking docker with a script, and trying to get hot-reloading working using docker machine (spoiler alert: docker machine is getting older and seems to not be as well-supported, it's recommended to use [terraform](https://www.terraform.io/) for the purpose we were trying to get docker machine to work with). We then test our docker container using common docker benchmarking tools, and make the site and container available to the local network. Note this is not a production build, as in production a heavier web server and load balancer should be used, such as apache or nginx.

Note all commands used are on Ubuntu 18.04, although these should work in any debian-based Linux environment.
<a name="react"></a>
## React

To build the basic React app, use used [create-react-app](https://Github.com/facebook/create-react-app) and [material-ui](https://material-ui.com/) to make an application that takes shipping information as input. To get started with these tools, you must first install npm and nodejs, which you can do using the following commands:

```sh
sudo apt-get update
sudo apt-get install nodejs npm
nodejs -v
npm -v
```

Note for the app we created we used nodejs v8.10.0 and npm v6.4.0, although installing newer versions shouldn't be an issue.

We then created a default application using the following command:

`npx create-react-app reactdocker`

Where 'reactdocker' is the directory name of the basic application you want to create. To test the new directory change into the new dir and start the application:

```sh
cd reactdocker
npm start
```

You can then see the application at http://localhost:3000 and should look like the following:

![Working React App](https://gooddebate.org/images/create-react-app.png)



To install material-ui, run the following command to incorporate the framework into the project:

` npm install @material-ui/core `

Components using material-ui could now be used. To create a navigation bar to the app, create the file *NavBar.js* in *../reactdocker/src/* then add the following code:

```js
import React from 'react';
import AppBar from '@material-ui/core/AppBar';
import Toolbar from '@material-ui/core/Toolbar';
import Typography from '@material-ui/core/Typography';
import IconButton from '@material-ui/core/IconButton';
import MenuIcon from '@material-ui/icons/Menu';



const NavBar = () => {
    return (
        <div>
            <AppBar position="static">
                <Toolbar>
                    <IconButton color="inherit" aria-label="Menu">
                        <MenuIcon />
                    </IconButton>
                    <Typography variant="title" color="inherit">
                        499 React App
                </Typography>
                </Toolbar>
            </AppBar>
        </div>
    )
}
export default NavBar;
```

This code imports several modules for the creation of our navigation bar. We then define a navbar with a static position and color that inherits from its parent color. 

We can then create an address form several components. A widely used module that is worth pointing out is the *Grid* module from material-ui, which is heavily used in many React applications. Instead of writing the code out in its full form, you can read through the code for the AddressForm [here](https://Github.com/asnair/react-docker/blob/master/src/AddressForm.js), and where the form is actually used in the primary App.js form [here](https://Github.com/asnair/react-docker/blob/master/src/App.js). Components that are created in their own file (such as *NavBar.js*) need to be rendered within the application. The entry point of any web application is always the index file, which in this case is [index.html](https://Github.com/asnair/react-docker/blob/master/public/index.html). The index references [index.js](https://Github.com/asnair/react-docker/blob/master/src/index.js), which returns a div element with our AddressForm and NavBar, which can be done in 18 lines due to React. The final product of the web app is shown below, hosted on localhost using nodejs built into npm: `npm start`

![Final Basic React App](https://gooddebate.org/images/final-react-app.png)
<a name="docker"></a>
## Docker

Before getting this project set up in Docker, we wanted to make sure we could roll back if we broke something. This meant setting up git and putting our project on Github, which also made it easier to share what we were doing and reproduce steps for documentation. Since we don't use git as a core part of our cirriculum in school, we really only use git for personal projects. This meant we ran into some issues with git that slowed us down because we weren't sure what to do, such as [rolling back to an older commit](https://stackoverflow.com/questions/4114095/how-to-revert-a-git-repository-to-a-previous-commit) or [reseting the head](https://stackoverflow.com/a/28354830), whatever that meant. Since this is a post about our methodology regarding this project, here is a quick summary about setting up a git repo and using it with Github for those who rarely use git.
<a name="git"></a>
### Git

To be able to share code with each other using Github, we first made an empty repo on Github (at https://github.com/asnair/react-docker) using the GUI, then ran the following commands on each of our local machines in the *reactdocker* folder after adding all users that needed access as contributors in the Github GUI:

```sh
git init
git remote add origin https://github.com/asnair/react-docker
git pull origin master
git add .
git commit -m "First commit"
git push origin master
```

The first line initializes a git repo in the current working directory. After that, we add the empty git repo on Github as the origin, so changes are committed to the online repo when we push new commits. We then pull anything from the online repo to make sure the current working directory is synchronized. After that, we add everything (really all new and modified files) in the current directory using the command `git add .` to the staging area, known as the index. We then commit the changes to the local repo in the current directory, with a description to match. Lastly, we push the changes in the local repo to the remote repo on Github, which exhibited no conflicts because it was empty.

Now that our git repo was set up and working, we could move on to Docker.
<a name="install-docker"></a>
### Installing Docker

We installed Docker CE (what most people just refer to as *Docker* unless you're an enterprise) according to the [guide on Docker's website](https://docs.docker.com/install/linux/docker-ce/ubuntu/). Their documentation and guides are some of the best, so I encourage you to read their official guide, and I'll include the summarized commands here for Ubuntu 16.04 and 18.04 using x86_64 and amd64, which is the most common linux set up. To see your specific system information try running `uname -m` and `lsb_release -a`

```sh
sudo apt-get remove docker docker-engine docker.io
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt-get update
sudo apt-get install docker-ce
```

Now that docker is installed, we can focus on building the container to run the React web app. To get started we wrote a Dockerfile, which is a basic configuration file that tells Docker how to package an application. The following Dockerfile got our application up and running (Note we got the Dockerfile from [this guide](https://mherman.org/blog/dockerizing-a-react-app/), which is a great guide to follow although the docker machine section is broken):

``` dockerfile
# base image
FROM node:8.14

# set working directory
RUN mkdir /usr/src/app
WORKDIR /usr/src/app

# add `/usr/src/app/node_modules/.bin` to $PATH
ENV PATH /usr/src/app/node_modules/.bin:$PATH

# install and cache app dependencies
COPY package.json /usr/src/app/package.json
RUN npm install --silent
RUN npm install react-scripts@1.1.1 -g --silent

# start app
CMD ["npm", "start"]
```

The nodejs and react-scripts versions you use might be different, but you should set these to provide consistency, and update them when you update the packages within your application. Note that this dockerfile is useful for testing, however in production more needs to be specified such as which web server to use that is built for scalability such as nginx or apache.

After writing a Dockerfile, an image must be built before it can be run. To build an image, execute the following command:

`docker build -t app .`

Now the container is built and can be run using the following command (you can add this to a shell script, such as *run.sh* then run `chmod +x run.sh` to be able to execute the script):

```sh
docker run -it \
  -v ${PWD}:/usr/src/app \
  -v /usr/src/app/node_modules \
  -p 3000:3000 \
  --rm \
  app
```

This script runs docker in the current directory in interactive mode with volumes in the container linking directly to */usr/src/app* and *usr/src/app/node_modules*. It also binds port 3000 on the host machine to the container, and removes all unnecessary files. The name of the web app is ‘app’. After running this script, you should be able to navigate to http://localhost:3000 and view the web app.
<a name="security"></a>
### Security

To test the security of our docker set up, we used a popular [docker security benchmark script](https://github.com/docker/docker-bench-security) to test the basic security of docker. This can be done by running a docker container against other containers as stated on the github page:

```sh
docker run -it --net host --pid host --userns host --cap-add audit_control \
    -e DOCKER_CONTENT_TRUST=$DOCKER_CONTENT_TRUST \
    -v /var/lib:/var/lib \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v /usr/lib/systemd:/usr/lib/systemd \
    -v /etc:/etc --label docker_bench_security \
    docker/docker-bench-security
```

We ran this script by downloading a local version using: 

```
git clone https://github.com/docker/docker-bench-security.git
sudo ./docker-bench-security/docker-bench-security.sh
```

We then installed auditd and added some rules:

```sh
sudo apt-get install auditd
sudo vim /etc/audit/audit.rules
```

```
-w /usr/bin/docker -p wa
-w /var/lib/docker -p wa
-w /etc/docker -p wa
-w /lib/systemd/system/docker.service -p wa
-w /lib/systemd/system/docker.socket -p wa
-w /etc/default/docker -p wa
-w /etc/docker/daemon.json -p wa
-w /usr/bin/docker-containerd -p wa
-w /usr/bin/docker-runc -p wa
```

We then restarted, auditd so the new rules would take effect, ran the benchmarking script, ensured that only non-root trusted users had access to docker, and configured the docker daemon:

```sh
sudo systemctl restart auditd
sudo ./docker-bench-security/docker-bench-security.sh
cat /etc/group | grep docker
sudo vim /etc/docker/daemon.json
```

```
{
    "icc": false,
    "userns-remap": "default",
    "log-driver": "syslog",
    "disable-legacy-registry": true,
    "live-restore": true,
    "userland-proxy": false,
    "no-new-privileges": true
}
```

The first line indicates that containers can only communicate with each other when explicitly linked. With this configuration attackers who gain control of one container will not be able to manipulate others unless they are configured to do so. The second line enables user namespace remapping allowing the root use to own the process and then the process is then remapped to a less privileged user. The third line is used to enable system logging. The live-restore is used to allow containers to run so that there is improved uptime during updates. The final line is used to prevent privilege escalation. 

Lastly we enabled content trust, which is a tool used to sign Docker images and verify images before running them. We enabled content trust using the following command:

`echo "DOCKER_CONTENT_TRUST=1" >> /etc/environment`

After completing these steps we have successfully baselined the machine used to host our docker images. It is recommended that users also create a separate partition as well as use correct key and certification configurations when deploying docker images to the public.
<a name="using-docker-hub"></a>
### Using Docker Hub

This section outlines how to explore Docker more, and see how others set up their docker containers. [The Docker Hub](https://hub.docker.com/) is a public repo where docker images are stored. You can pull and push images just like with git. If you try and run an image that isn't on your local machine, docker will automatically check the hub and if the image exists, download the image to run it. You can also just download an image and not run it using `docker pull USER/IMAGE`.

To push images to Docker Hub you must login, find an image you want to push (using `docker images` or `docker image ls`), tag the image, then push the image. The commands to do this are listed below:

```sh
docker login --username=yourUsername --email=yourEmail@email.com
docker images
docker tag IMAGE-ID yourUsername/reponame:TAG
docker push yourUsername/reponame
```

The TAG should be descriptive, like 'apache' or 'docker_web_app'.

You can then run this Docker container anywhere, using the following command:

`docker run -p 3000:3000 yourUsername/reponame`

This will download and run the image automatically.
<a name="useful-docker-commands"></a>
### Useful Docker Commands

Useful commands to use while working with Docker include the following:

List all docker containers currently running: `docker ps`

List all docker images: `docker image ls`

Kill a specific container: `docker rm -f IMAGE`

Pull image from a registry (such as Docker Hub): `docker image pull NAME[:TAG or @DIGEST]`

SSH into a running Docker Container: `sudo docker exec -it CONTAINERID /bin/bash`
<a name="lessons-from-docker"></a>
### Lessons from Docker

In our small project working with Docker, we've learned a few things to consider when working with Docker in the future:

1. Use the same versions of systems and applications outside containerization technology, meaning the operating system, management applications, and docker versions should be the same.
2. The quickest way to get Docker up and running is to use a Dockerfile, however for systems that aren’t simple, using docker-compose and management engines such as k8s or swarm is necessary to increase scalability and simplicity on larger projects.
3. Without tooling for integrating Docker into the deployment process of an app, more time may be spent setting up Docker and similar technologies than is worth the trouble, at least for small projects that don't require extreme consistency across several development teams or even extreme consistency in a single development team. The technology is really cool, but is more useful for medium to larger companies that already have scalability needs.
<a name="docker-machine"></a>
## Docker Machine

We tried to use Docker Machine to get hot-reloading of the web app to work, however we failed due to some ongoing bugs in Docker Machine itself, which is why [Terraform](https://www.terraform.io/) is the recommended way currently. Despite failing, outlining what we tried to do may help others in the future. If this doesn't apply to you, you can [move along to the Troubleshooting section](#Troubleshooting). Our set up was Ubuntu 18.04 using Virtualbox 5.2.22 and docker-machine 0.16.0.

We wanted to get hot-reloading working in Docker, meaning a new container would be spun up per instance of access to docker. This means each user would have their own container per session, greatly enhancing security. Using our current build, we could just install docker machine using:

```sh
base=https://github.com/docker/machine/releases/download/v0.16.0 && curl -L $base/docker-machine-$(uname -s)-$(uname -m) /tmp/docker-machine && sudo install /tmp/docker-machine /usr/local/bin/docker-machine
```

Then we could set our current docker image to build a container in a VM that would manage spinning up the same container for each new connection. This new startup script used the following code:

```sh
docker run -it --mount type=bind,source=${PWD},destination=/usr/src/app --mount type=bind,source=${PWD}/
node_modules,destination=/usr/src/app/node_modules,readonly -p 3000:3000 -e CHOKIDAR_USEPOLLING=true --rm app
```

The section of this code which allows for spinning up new containers is the *CHOKIDAR_USEPOLLING=true* environment variable which allows the VM to use polling to spin up new containers.

This code should have worked in theory, however we discovered that docker-machine has an [open issue from 2015](#issue5) where when mounting volumes within the container (which we must do for the app itself in */usr/src/app* and for the dependency modules in */usr/src/app/node_modules*), the volumes only get mounted to the host machine and not the virtual machine. This means when the container is created in the virtual machine, it can’t link to the volumes because the virtual machine also doesn't have access to these volumes. There are fixes for this issue involving ssh or rsync, but these only work on a single virtual machine at a time, and none of the solutions we found would provide auto-reloading. This would mean rolling our own automation by auto-mounting volumes through ssh or rsync, which was outside the scope of our research.
<a name="troubleshooting"></a>
## Troubleshooting

We ran into several issues, which are bound to happen during development. The fun part, of course, is fixing all the issues. We've outlined some of the issues we found, how we tried to solve them, and documented what finally worked for us.

1. Issue: Received error response from daemon driver. This occurred when attempting to duplicate container creation process on another machine. 

- Possible Fixes:

  - Reinstall docker
  - Ensure docker is updated

    - `sudo apt-get upgrade`

  - Ensure port isn’t being used

    - `netstat -natp`

  - Dependency issue

- Fix: Created ubuntu 16.04 machine using VirtualBox, reinstalled nodejs, npm, docker, and followed the previous steps to create docker image containing the react-app. 


2. Issue: Received several javascript errors after attempting to update versions of node, npm, react, and dependencies.

- Possible Fixes:

  - Reinstall docker
  - Revert to old version
  - Dependency issue

- Fix: Reverted to working version in git using

  - `git revert HEAD~7..HEAD` (see next issue)



3. Issue: Reverting in git continually failed

- Possible Fixes:

  - `git revert d411df6714395d692df62ce25dc509fb7e1ce4a3..cd5b9584baf2ca73a53b5f1ee163279034f52455`
  - `git revert master~7..HEAD`
  - `git cherry-pick --continue`

- Fix: Rebasing was needed rather than reverting, and we didn’t want to hard reset and lose our version history.

  - `git rebase -i fd4a0907f8b18f10686576e155fab5a49f8b0d56`



4. Issue: After following steps for docker bench security docker user had restricted access and was unable to start or stop docker containers.

- Possible Fixes:

- - Check that the user was in the docker group
  - Check that the docker user was logged in
  - Check permissions of user

- Fix: The user was not a member of the sudoers file. The user was then added to the sudoers file:

- - `su `
  - `visudo`
  - Added `zach ALL=(ALL) NOPASSWD:ALL` to */etc/sudoers* file
  - Note: This is not a best security practice and should only be used during development. 



5. <a name="issue5></a>Issue: After following steps for hot reloading in docker, the volumes couldn't be mounted within the virtual machine.

- Possible Fixes:

- - Ensure the build process was correct
  - Check where volumes were being mounted
  - Manually mount volumes into the VM using the following command: `docker-machine mount ${PWD}:/usr/src/app app`  and `docker-machine mount /usr/src/app/node_modules`

- Fix: We found mounting in docker-machine is broken in the software itself, and is an ongoing [open bug on github](https://github.com/docker/machine/issues/179) so we couldn’t move past this issue unfortunately.