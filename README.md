# Google-cloud-shell-hacking
Hacks for a better google cloud shell experience

![alt text](https://github.com/FrancescoDiSalesGithub/Google-cloud-shell-hacking/blob/main/cludshell.jpeg)

# Summary

* Introduction
* Do and don't
* Cons of Google Cloud Shell
* Hacks of Google Cloud Shell
* SSH on the google cloud shell using the private key
* Putting the public key on google cloud shell
* Start the google cloud shell instance
* Check the ip of the google cloud shell
* Run the google cloud shell in ssh
* Connect a google cloud shell to another google cloud shell
* Using the postgres database
* Autorun the Google Cloud shell at login
* Containers stored locally in google cloud shell
* Persistent postgresql data
* Enabling systemctl on google cloud shell
* Cockpit interface on google cloud shell
* Connect external drives to google cloud shell
* Connect with rdp protocol to Google cloud shell
* Windows Server on google cloud shell
* Donation

# Introduction

When you create a google account, you obtain a shell for google cloud. These are the features:

* 5GB hard-drive
* 8GB RAM
* devops tools installed
* git installed
* Debian linux distribution

# DO AND DON'T

Do:
* develop
* trying other programs
* trying google apis
* using your cloud shell as proxy (google will still know about your data...)
* having a personal vps

Don't:
* mining
* unethical hacking

# Cons of Google Cloud Shell
Google cloud shell has the following problems:

* The vm is ephimeral that means that after 1 Hour of inactivity all the content outside the $HOME folder will be lost
* The google cloud shell is interactive so the crontab will not work
* You have 50 hours available to use the google cloud shell, after Google will reset the time at a specific date.

# Hacks of Google Cloud Shell
Here there's the interesting part of this repository:

* The google cloud shell has an hidden drive (sda1) which is 60GB you can mount it in the home folder but after the session is lost everything is lost
* When you start for the first time the google cloud shell it could track your commands. Try the following command: `gcloud config set disable_usage_reporting true`

# SSH on the google cloud shell using the private key

When you start a cloud shell with the gcloud-cli or with cloud shell web page, it is assigned an ip to your google cloud shell and it seems that you can not ssh if not using the webpage or the gcloud-cli. The ssh shell listens on the port 6000 and in the sshd config the password authentication is disabled so it means that it is used the key authentication. Generate on your machine a keypair:

`ssh-keygen -t rsa`

then get the oauth token from google:

* go to https://developers.google.com/oauthplayground/
* search for Cloud Shell api v1
* after selecting, choose https://www.googleapis.com/auth/cloud-platform
* Login and authorize the application
* On step 2 ask for auth token
* On step 3 copy the access token

After doing these steps you need to call the following api:

* https://content-cloudshell.googleapis.com/v1/users/me/environments/default:addPublicKey [POST] (will make you add your public key you created locally)
* https://content-cloudshell.googleapis.com/v1/users/me/environments/default:start [POST] (will make you start the instance)
* https://content-cloudshell.googleapis.com/v1/users/me/environments/default [GET] (will tell you the ip of the google cloud shell)

## Putting the public key on google cloud shell

The api has a POST method. Run the api with the following body:
```
{
  "key": "content of the public key"
}
```
And the header **Authorization**. It should has value Bearer value-of-the-access-token.

The content of the public key can be obtained by run the following line:
`cat .ssh/id_rsa.pub`

After doing these operations run the rest api.

## Start the google cloud shell instance

Same url as the previous paragraph, and same header but the json must have this body:
```
{
  "accessToken": "access-token-value",
  "publicKeys": [
    "content-of-the-local-public-key"
  ]
}
```
And run the api

## Check the ip of the google cloud shell

This api has a HTTP GET method, but the url is the same. Like the previous one it has the **Authorization header**. After checking that you can run it and you will have the IP of your google cloud shell

## Run the google cloud shell in ssh

Since as the beginning of the chapter we know that it listens on the 6000 port, (and also the last api tell us this information) then:

`ssh -i id_rsa -p 6000 myusername@ip-google-cloud-shell`

If everything goes well you will have a ssh session with the google cloud shell without using the webpage or the gcloud-cli.


# Connect a google cloud shell to another google cloud shell

Let's suppose we have two users:
* UserA
* UserB

UserA wants to connect to UserB's cloud shell.

UserA has to retrieve the oauth token, and register on his google cloud shell the public key of UserB's cloud shell. Then download from UserB's cloud shell the private key of the UserB. Finally he can run the following command: `ssh -i userb_rsa -p 6000 UserB@IP-CLOUD_SHELL_USERB`


# Using the postgres database

In the cloud shell run these commands:

```
sudo service postgresql start 
sudo su
su postgres
psql

```

At the first command you start the postgresql database service, and then you need to be the postgresql user so you use the su command and finally launching psql you have the interactive shell for postgresql. 

# Autorun the Google Cloud shell at login

In the Google Cloud shell session edit in your home folder the .bashrc file. At the end of the file write the commands you want to run at login, then save and close the files. At a new login the cloud shell will run the commands you have written in the .bashrc file.

# Containers stored locally in google cloud shell 

If you're using the docker you will see that, after the session is over you will lose all the content in your containers. There are two possible solutions:

* using volumes
* export and import of the container

In this paragraph we will discuss about how to export and import a container. Start with pulling a container such as ubuntu:
`docker pull ubuntu`

Run the container by doing:
`docker run --name=ubuntu -i -t ubuntu /bin/bash`

exit from the shell container, and run the following command:
`docker export -o ubuntu_container.tar`

this command will create a tar archive with all the content in the container. Doing that remove the ubuntu image:
`docker rmi ubuntu'

Now load your container instance:
`docker import ubuntu_container.tar ubuntu_mycontainer:latest`

In the import command is important to pass as argument:
* the tar archive where all the informations of the container is stored
* the name to give to the imported container and the tag (usually should be latest)

To check if the container has been imported, run again **docker images**, it should have the new name you gave to the imported container.
Run your new container by doing:
`docker run --name=mypersonalcontainer -i -t ubuntu_mycontainer:latest /bin/bash`

This command would run your container with the informations you have inside it


## Persistent postgresql data
If you want to persist your data in your google cloud shell, you need to do the following:
* create a directory in your home folder
* `mv /var/lib/postgres/15/main /home/your_google_account/database/` 
* edit the following file **/etc/postgresql/15/main/postgresql.conf** and at the voice **data-directory** add the path **/home/your_google_account/database/**
* start the postgresql service with `sudo service postgres start` if everything is ok postgresql will be up and running

When a new instance of the google cloud is running, you only need to edit the file **/etc/postgresql/15/main/postgresql.conf** and edit the **data-directory** voice

## Enabling systemctl on google cloud shell
Since the instance of the google cloud shell works on a docker container, systemctl is not enabled. It is suggested to build a docker image as follow:
```
from ubuntu

run apt update && apt install -y systemd systemd-sysv sudo
run useradd -ms /bin/bash myuser
run echo 'myuser:password' | chpasswd 
run usermod -aG sudo myuser
 
cmd ["/lib/systemd/systemd"]

```

after saving the dockerfile run the following command:
`docker build -t mysubsystem .`

wait for the complete build.

After the build has done, run the docker run command:
`docker run --privileged --name=mysubsystem-container -i -t mysubsystem`

Docker will run the container, and it will present you the ubuntu login screen. At the login credentials enter the credentials you wrote in the dockerfile and then you will have a linux container with systemctl enabled. If you want to exit from the container run the **shutdown** command:
`sudo shutdown now`

## Cockpit interface on google cloud shell

if you want to manage the google cloud shell through cockpit as prerequisite you need to read the previous paragrah about how to install systemctl on google cloud shell. When you have installed the container in which there is systemctl run this procedure:

```

sudo apt update
sudo apt install -y cockpit wget

mkdir ngrok
cd ngrok

sudo systemctl start cockpit

wget https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-linux-amd64.tgz
tar -xzvf ngrok-v3-stable-linux-amd64.tgz
rm ngrok-v3-stable-linux-amd64.tgz

```

then go to your ngrok account and copy your authentication token. You can find it in the ngrok account page at the voice **getting started -> setup and installation**. In that page you can find an instruction like the following:

`ngrok config add-authtoken` 

and there will be near this instruction your authentication token. Paste all the instruction into the terminal where the google cloud shell is running like this:

`./ngrok config add-authtoken YOUR_AUTH_TOKEN`

Now you only need to run ngrok:

`./ngrok tcp 9090`

There should be an url like the following: **tcp://0.tcp.eu.ngrok.io:10230**, please copy the address after tcp:// and paste in your browser. As you reach the webpage, enter as credentials the credentials in your linux container.

## Connect external drives to google cloud shell

If you want to add your drives to the google cloudshell you need the following requirements:

* sshfs
* ngrok
* (optional) if you have other mass storage such as microsd or hard-drives use a hdd docking station

first on your system install openssh-server if you don't have it:
`sudo apt intall openssh-server -y`

After the installation on your local machine run the ssh service:
`sudo service ssh start`

Then with ngrok listen on port 22:
`./ngrok tcp 22`

On your google cloud shell install sshfs:
`sudo apt install -y sshfs`

After installing it, always in your google cloud shell run the following command line command:
`sshfs -p NGROK_PORT USER@NGROK_TCP_ADDRESS:LOCAL_PATH PATH_GOOGLE_CLOUD_SHELL`

where:
* NGROK_PORT is the port ngrok gives on your local machine
* USER is your user in your local machine
* NGROK_TCP_ADDRESS is the address that ngrok gives you
* LOCAL_PATH is the path where you have locally your mass storage device
* PATH_GOOGLE_CLOUD_SHELL is the path where do you want that your local storage device must be mount on the google cloud shell

After that if you are already in the folder where the device has to be mount, go back one directory and then go back to your mount folder. When you are done you can unmount the device on your google cloud shell by doing the following:

`umount PATH_GOOGLE_CLOUD_SHELL`

Remember that PATH_GOOGLE_CLOUD_SHELL is the path where do you want that your local storage device must be mount on the google cloud shell.

## Connecting using rdp on Google cloud shell

If you have windows remote desktop and you want to connect to the Google cloud shell, you have to do the following steps:
* Run `sudo apt install -y xrdp dbus-x11 xfce4 xfce4-goodies`
* Start xrdp service: `sudo service xrdp start`
* Create and user and give him a password: `adduser dev`
* Assign the user to the sudo group: `usermod -aG sudo dev`
* Start ngork: `ngrok tcp 3389`
* Copy the address and port after the tcp://
* Open the remote desktop app for windows and connect
* Login with the user created at the third point and you have your Google cloud shell environment on rdp

## Windows Server on Google

As prerequisite you have to add in your google drive a qcow image of your windows server, also you have to follow the steps of the previous paragraph.
Then you have to install qemu:

`sudo apt install -y qemu-system-x86`

After that install firefox:

`sudo apt install firefox-esr`

On firefox, change the download location to **/root** and then go to **google.com** and log in as your google user, then, go to your google drive and download the qcow image of windows server. After downloading go to /root and type the following command:

`qemu-system-x86_64 -img windowserver.qcow -m 2048 -boot c`

## Donation

If you want to support me, or if this guide helped you, you can donate me some monero coins at: `4B9WQivaHfd3miDfPKEfCianocGpBx9d8FXycz2vmNW3aBDVKHgkBd9Gmapt4RBVEpTwnehujsiUBBehUiLvnEHs7VFstCC`
