# Google-cloud-shell-hacking
Hacks for a better google cloud shell experience

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

When you start a cloud shell with the gcloud-cli or with cloud shell web page, it is assigned an ip to your google cloud shell and it seems that you can not ssh if not using the webpage or the gcloud-cli. The ssh shell listens on the port 6000 and in the sshd config the password authentication is disabled so it means that it is used the key authentication. In the cloud shell generate the key pair by using the following command:

`ssh-keygen -t rsa`

at passphrase leave empty, then go to the .ssh folder and download the id_rsa file. For downloading the id_rsa file you can use the webpage cloud shell by selecting the menu and the voice download, and then go to the path **/home/your-account-name/.ssh/id_rsa** and click on download, or use ngrok and python http module. In the second case insert the following commands in the cloud shell:

```
cd /home/your-username/.ssh
python3 -m http.server 8080 &
ngrok http 8080

```

copy the url of ngrok and then on the client execute:

`wget http://sdhakdhsaj.ngrok.eu/id_rsa`

as you have done these operations connect with ssh:

`ssh -i id_rsa your-username@ip-google-cloud-shell`

