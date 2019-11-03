+++
date = "2017-05-02T16:38:19-05:00"
title = "Failed Building Wheel for Cryptography"
draft = false

tags = ["Docker","Cryptography","Vagrant","Ansible","Python"]

categories = ["Development"]

+++

This post was originally going to be about how to set up a few different systems, including vagrant, docker, ansible, and some random python projects on debian/ubuntu systems, but I ran into a singular problem on almost everything I did. The error was always

`Failed Building Wheel for Cryptography`

I've gotten this problem several times, and it can be caused by different things. I found a few posts that helped me ([here](http://stackoverflow.com/questions/22073516/failed-to-install-python-cryptography-package-with-pip-and-setup-py) and [here](https://github.com/openwhisk/openwhisk/issues/309) and [here](https://github.com/docker-library/python/issues/60)), but here is a quick summary which should help you in almost all situations:

1. Install dependencies (including gcc, which although included by default almost always, is removed in the python-**slim** packages which are used as an example for docker getting started)

   - `sudo -H apt-get install build-essential libssl-dev libffi-dev python-dev gcc`

     If any of these packages are already installed, try removing them then running the same line again:

   - `sudo -H apt-get uninstall build-essential libssl-dev libffi-dev python-dev gcc`

     For some reason removing it then adding it again worked most for me. Also, if you don't use the -H flag it might say the program/folder don't belong to the specified user, which means it failed.

2. When setting up your *Dockerfile* for the first time, make sure not to use python 2.7 **slim**, as stated slim packages trim a lot, including gcc which is needed for the Cryptography wheel. This was one of the more frustrating issues to fix, so I'm stating it specifically.

3. Making sure docker-py is installed, which allows docker to interface with python, and is used for apps that interface with docker:

   - `sudo pip install docker-py`

4. Installing openssl would probably be a good idea, as it is used in several libraries. I've run into a few errors involving libssl, which I believe is part of openssl and can be solved by installing the full library.

More than these options, you'll have to look through the error log if you get problems. When trying to solve the ``Failed Building Wheel for Cryptography`` , make sure to look before that error, as sometimes there was a previous error that caused it which might not show up in red text. With the Dockerfile and gcc, the only error was `gcc: no file or directory found` as it didn't know gcc was a program to be installed, but this was buried in the log file, not even the error log. So make sure to look through both files if you can't figure it out online.



Thanks for reading.

![Abstract](http://gooddebate.org/images/elephant.JPG)

