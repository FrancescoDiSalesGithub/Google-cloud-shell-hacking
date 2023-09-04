# Google-cloud-shell-hacking
Hacks for a better google cloud shell experience

![alt text](https://github.com/FrancescoDiSalesGithub/Google-cloud-shell-hacking/blob/main/cludshell.jpeg)

## Donation

If you want to support me, or if this guide helped you, you can donate me some monero coins at: `4B9WQivaHfd3miDfPKEfCianocGpBx9d8FXycz2vmNW3aBDVKHgkBd9Gmapt4RBVEpTwnehujsiUBBehUiLvnEHs7VFstCC`

## Sponsor

Exciting News: Introducing Hack The Box Academy! lock

fire Calling all cybersecurity enthusiasts and aspiring hackers! fire

I'm thrilled to announce an incredible opportunity for you to take your skills to the next level. Today, I proudly sponsor Hack The Box Academy, an innovative online platform dedicated to cybersecurity education and practical training.

Hack The Box Academy has earned a stellar reputation for its cutting-edge approach to teaching real-world hacking techniques and cybersecurity principles. With a mission to empower individuals with the knowledge and skills necessary to succeed in the ever-evolving world of cybersecurity, this platform is an absolute game-changer.

Start now: https://referral.hackthebox.com/mzwyliz



# Summary

* Donation
* Sponsor
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
* Getting oauth token on a running instance of a google cloud shell
* Change location and zone of the google cloud shell
* Use the google cloud shell as proxy
* Running another operative system at the login in the google cloud shell
* Using the postgres database
* Autorun the Google Cloud shell at login
* Containers stored locally in google cloud shell
* Persistent postgresql data
* Enabling systemctl on google cloud shell
* Cockpit interface on google cloud shell
* Connect external drives to google cloud shell
* Connect with rdp protocol to Google cloud shell
* Windows Server on google cloud shell
* Removing bloat from google cloud shell
* Using dbeaver on a google cloud shell database
* Gitlab on google cloud shell

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
  "key": "ssh-rsa content of the public key"
}

where content of the public key = abkjdksajkgajkdhkhksda  
no username should be pasted after the rsa encryption

Example:

{
  "key":"afabasjdgnjadkjadhaksdsajkdsjakd"
}

```
And the header **Authorization**. It should has value Bearer value-of-the-access-token.

The content of the public key can be obtained by run the following line:
`cat .ssh/id_rsa.pub`

## Start the google cloud shell instance

Use the following api:
https://content-cloudshell.googleapis.com/v1/users/me/environments/default:start 

Add the header **Authorization**. It should has value Bearer value-of-the-access-token. The json request is the following:
```
{
  "accessToken": "access-token-value",
  "publicKeys": [
    "content-of-the-local-public-key"
  ]
}
```
Then run the api

## Check the ip of the google cloud shell

Run:
```
 https://content-cloudshell.googleapis.com/v1/users/me/environments/default
```

Like the previous one it has the **Authorization header**. Run it and you will have the IP of your google cloud shell.

## Run the google cloud shell in ssh

Running the api:
```
https://content-cloudshell.googleapis.com/v1/users/me/environments/default:start
```
As response of the rest api, search for the json keys:

* sshHost
* sshPort

you can connect in ssh with the terminal by doing:

```
ssh -i my_key_rsa -p value_of_sshPort my_google_username@value_of_sshHost
```

If everything goes well you will have a ssh session with the google cloud shell without using the webpage or the gcloud-cli.


# Connect a google cloud shell to another google cloud shell

Let's suppose we have two users:
* UserA
* UserB

UserA wants to connect to UserB's cloud shell.

UserA has to retrieve the oauth token, and register on his google cloud shell the public key of UserB's cloud shell. Then download from UserB's cloud shell the private key of the UserB. Finally he can run the following command: `ssh -i userb_rsa -p 6000 UserB@IP-CLOUD_SHELL_USERB`

# Getting oauth token on a running instance of a google cloud shell

If you want an oauth token while logged in your google cloud shell, you can run the following command:
```
gcloud config set compute/region us-east1
gcloud auth application-default print-access-token
```

# Change location and zone of the google cloud shell

If you want that your google cloud shell starts to a different country, then you have to run the following commands:

```
gcloud config set compute/region REGION
gcloud config set compute/zone ZONE

```


The possible values for the REGION enviroment variable are:

* asia-east1 (Changhua County, Taiwan, APAC)
* asia-east2 (Hong Kong, APAC)
* asia-northeast1 (Tokyo, Japan, APAC)
* asia-northeast2 (Osaka, Japan, APAC)
* asia-northeast3 (Seoul, South Korea)
* asia-south1 (Mumbai, India)
* asia-south2 (Delhi, India)
* asia-southeast1 (Jurong West, Singapore)
* asia-southeast2 (Jakarta, Indonesia, APAC)
* australia-southeast1 (Sydney, Australia)
* australia-southeast2 (Melbourne, Australia)
* europe-central2 (Warsaw, Poland)
* europe-north1 (Hamina, Finland)
* europe-southwest1 (Madird, Spain)
* europe-west1 (St. Ghislain, Belgium)
* europe-west10 (Berlin, Germany)
* europe-west12 (Turin, Italy)
* europe-west2 (London, England)
* europe-west3 (Frankfurt, Germany)
* europe-west4 (Eemshaven, Netherlands)
* europe-west6 (Zurich, Switzerland)
* europe-west8 (Milan, Italy)
* europe-west9 (Paris, France)
* me-central1 (Doha, Qatar)
* me-west1 	(Tel Aviv, Israel)
* northamerica-northeast1	(Montréal, Québec, North America)
* northamerica-northeast2 (Toronto, Ontario, North America)
* southamerica-east1 (Osasco, São Paulo, Brazil, South America)
* southamerica-west1 (Santiago, Chile, South America)
* us-central1 (Council Bluffs, Iowa, North America)
* us-east1 	(Moncks Corner, South Carolina, North America)
* us-east4 	(Ashburn, Virginia, North America)
* us-east5 (Columbus, Ohio, North America)
* us-south1 (Dallas, Texas, North America)
* us-west1 (The Dalles, Oregon, North America)
* us-west2 (Los Angeles, California, North America)
* us-west3 (Salt Lake City, Utah, North America)
* us-west4 (Las Vegas, Nevada, North America)

The possible values for the ZONE enviroment variable are:

Asia:
* asia-east1-a
* asia-east1-b
* asia-east1-c
* asia-east2-a
* asia-east2-b
* asia-east2-c
* asia-northeast1-a
* asia-northeast1-b
* asia-northeast1-c
* asia-northeast2-a
* asia-northeast2-b
* asia-northeast2-c
* asia-northeast3-a
* asia-northeast3-b
* asia-northeast3-c
* asia-south1-a
* asia-south1-b
* asia-south1-c
* asia-south2-a
* asia-south2-b
* asia-south2-c
* asia-southeast1-a
* asia-southeast1-b
* asia-southeast1-c
* asia-southeast2-a
* asia-southeast2-b 
* asia-southeast2-c

Australia:
* australia-southeast1-a 
* australia-southeast1-b 
* australia-southeast1-c 
* australia-southeast2-a
* australia-southeast2-b
* australia-southeast2-c

Europe:
* europe-central2-a
* europe-central2-b
* europe-central2-c
* europe-north1-a
* europe-north1-b
* europe-north1-c
* europe-southwest1-a
* europe-southwest1-b
* europe-southwest1-c
* europe-west1-b
* europe-west1-c
* europe-west1-d
* europe-west10-a
* europe-west10-b
* europe-west10-c
* europe-west12-a
* europe-west12-b
* europe-west12-c
* europe-west2-a
* europe-west2-b
* europe-west2-c
* europe-west3-a
* europe-west3-b
* europe-west3-c
* europe-west4-a
* europe-west4-b
* europe-west4-c
* europe-west6-a
* europe-west6-b
* europe-west6-c
* europe-west8-a
* europe-west8-b
* europe-west8-c
* europe-west9-a
* europe-west9-b
* europe-west9-c

Israel:
* me-central1-a
* me-central1-b
* me-central1-c
* me-west1-a
* me-west1-b
* me-west1-c

America:
* northamerica-northeast1-a
* northamerica-northeast1-b
* northamerica-northeast1-c
* northamerica-northeast2-a
* northamerica-northeast1-b
* northamerica-northeast1-c
* southamerica-east1-a
* southamerica-east1-b
* southamerica-east1-c
* southamerica-west1-a 
* southamerica-west1-b
* southamerica-west1-c
* us-central1-a
* us-central1-b
* us-central1-c
* us-central1-f
* us-east1-b
* us-east1-c
* us-east1-d
* us-east4-a
* us-east4-b
* us-east4-c
* us-east5-a
* us-east5-b
* us-east5-c
* us-south1-a
* us-south1-b
* us-south1-c
* us-west1-a
* us-west1-b
* us-west1-c
* us-west2-a
* us-west2-b
* us-west2-c
* us-west3-a
* us-west3-b
* us-west3-c
* us-west4-a
* us-west4-b
* us-west4-c

# Use the google cloud shell as proxy

If you want to use your google cloud shell instance as proxy you need to run the following commands (or insert them in the .bashrc file):

```
sudo apt install -y squid
```
Just for let you know Squid is a http proxy server. Create a **squid.conf** file with the following settings:

```
http_port 3128
cache_dir /var/cache/squid 100 16 256
acl all src 0.0.0.0/0
http_access allow all

```

copy the **squid.conf** file to **/etc/squid**
```
sudo cp squid.conf /etc/squid
```

Finally run the squid service:

```
sudo service squid start
```

Use ngrok to let the proxy be available from outside:
```
./ngrok tcp 3128
```
After running copy the tcp:// url. If you want to run the proxy from a browser it is suggested to remove the tcp:// part and the port and put the port in the port field of your browser proxy settings (squid is a http proxy server).

For better use at startup the .bashrc file should have the following lines:

```

sudo apt install -y squid
sudo cp squid.conf /etc/squid/
sudo service squid start
cd ngrok;./ngrok tcp 3128

```

# Running another operative system at the login in the google cloud shell

If you want to run another operative system instead of debian, you can try to create an operative system container. Create in your home folder a path like:
```
/home/your_google_account_name/your_favourite_operative_system
```
In the **your_favourite_operative_system** folder create a root folder:
```
mkdir root
```
Always in this directory (/home/your_google_account_name/your_favourite_operative_system), create a docker-compose.yml with the following informations:
```
version: '3.6'
services:
  kali:
    image: "ubuntu" # you can try other operative system such as alpine kali arch and so on
    tty: true
    stdin_open: true
    command: bash # if alpine or unknown operative system use sh
    volumes:
      - "/home/your_google_account_name/your_favourite_operative_system/root:/root"
```
Before editing the **.bashrc** file, run:
```
docker-compose up -d
```
then check the name of the container:
```
docker ps -a
```
At the column NAME save somewhere the name of the container. After this check, go to your home folder and edit the **.bashrc** file appending the following line of code:

```
cd /home/your_google_account_name/your_favourite_operative_system; docker-compose up -d && docker start -i your_container_name
```

Wait for the current google cloud shell to end and start a new one. On boot, you will have a running instance of your favourite system, but you will lose all your installed apps on the container.


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

`qemu-system-x86_64 -img windowserver.qcow -m 4096 -boot c`

## Removing bloat from google cloud shell

If you want to remove bloating software from your google cloud shell download the file in this repository called **remove-bloat.sh**. If you want to have always free space on your google cloud instance, write the **bash.rc** file in your google cloud home folder and write:
`chmod +x remove-bloat.sh; ./remove-bloat.sh`

At the next google cloud startup instance it would took some minutes until the removing procedure is done.

## Using dbeaver on a google cloud shell server

If you want to connect with dbeaver with the postgres database you need to do the follow:

* start the postgres service
* let ngrok listen to the port 5432 by running it as a tcp connection: `ngrok tcp 5432`
* copy the url and port and past it somewhere
* open a new instance of google cloud shell with the tmux and elevate privileges as root by doing sudo su and then go as postgres user by doing su postgres
* connect to the database doing psql -u postgres
* connect to the default postgres database by doing \c
* change the default credentials of postgres by doing: `alter user postgres password mysupersecurepassword`
* open dbeaver and create a postgresql connection
* as the voice host choose the ngrok hostname (something like 0.tcp.eu.ngrok.io) and the relative port
* insert the credentials you have edit before with the alter user command
* test the connection. If everything goes well a message box will alert that the connection works

## Gitlab on google cloud shell

To install Gitlab on google cloud shell you first need to edit your .bashrc file appending the following lines:

```
export GITLAB_HOME =/home/your_google_account_username/your_gitlab_folder
cd /home/your_google_account_username/your_gitlab_folder; docker-compose --force-recreate up -d
/home/your_google_account_username/ngrok/./ngrok http 80

```

After editing the file .bashrc, go to the your_gitlab_folder folder and make the following folders:

* config
* logs
* data

in this your_gitlab_folder folder, create a docker-compose.yml file and add these lines:

```
version: '3.6'
services:
  web:
    image: 'gitlab/gitlab-ce:latest'
    restart: always
    hostname: 'localhost'
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - '$GITLAB_HOME/config:/etc/gitlab'
      - '$GITLAB_HOME/logs:/var/log/gitlab'
      - '$GITLAB_HOME/data:/var/opt/gitlab'
    shm_size: '256m'

```

Save the file. Exit from the google cloud shell instance and start a new one. When a new google cloud shell sessions starts a gitlab docker instance will start. It may take 4 to 7 minutes for the container to boot. When done copy and paste the ngrok http link to visit the web interface of your gitlab instance.

NOTE: at first installation on gitlab it may be required to insert a root password. To find it first run `docker ps -a` and search the name of the container. Then run: `docker exec name_of_the_container cat /etc/gitlab/initial_root_password` after the output of the cat command paste in the web login interface and log in.



