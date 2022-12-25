# Google-cloud-shell-hacking
Hacks for a better google cloud shell experience

![alt text](https://github.com/FrancescoDiSalesGithub/Google-cloud-shell-hacking/blob/main/cludshell.jpeg)


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

