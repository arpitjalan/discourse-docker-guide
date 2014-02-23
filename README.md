Ok, so you want to deploy your own instance of Discourse, and looking from where to start? You've come to the right place, here I'll walk you through the installation steps with the help of screenshots, so you won't find yourself lost, let's begin this wonderful journey.

Unlike many other Rails Apps the deployment of Discourse is extremely simple thanks to awesome [Docker Image][1] by @sam, all you need is a ssh access to cloud server. In this guide I'll assume that you are using Digital Ocean, while the steps will work fine on other cloud servers as well.

The below guide assumes that you have no knowledge of Ruby/Rails, Shell, so it will be detailed. Feel free to skip steps which you think, you are comfortable with.

# Create new Digital Ocean Droplet

Discourse Team recommends a minimum of 1 GB Ram, so that's what we are gonna go with. For the sake of simplicity we will name the Hostname as discourse.

<img src="https://meta-discourse.r.worldssl.net/uploads/default/2997/c453053824cef71a.png" width="690" height="457"> 

We will install Discourse on Ubuntu 12.04.3 x64 as this is [recommended][2] in [Official Documentation][3].

<img src="https://meta-discourse.r.worldssl.net/uploads/default/2998/0084fb4e84c1d812.png" width="690" height="404"> 

Once you will complete with above steps you will receive a mail from Digital Ocean, providing root users password. (In case you have entered your SSH keys, then you don't require password to login).

# Access your newly created Droplet

To access the droplet, type in following command in your terminal:

    ssh root@162.243.201.40

Replace `162.243.201.40` with the IP address you got from Digital Ocean.

<img src="https://meta-discourse.r.worldssl.net/uploads/default/2999/0934a0158459ec3f.png" width="571" height="130"> 

It will ask your permission to connect, type `yes`, and then it will ask you for root's password. The root's password is in your mail which Digital Ocean sent you. Type in the password and you will be welcomed by newly installed Ubuntu Server.

<img src="https://meta-discourse.r.worldssl.net/uploads/default/3000/8209c1e40c9d70a8.png" width="570" height="278"> 

# Install Git

To install Git, you just need to type in the command:

    sudo apt-get install git

and you are good to go.

<img src="https://meta-discourse.r.worldssl.net/uploads/default/3002/eafbf14df8eee832.png" width="572" height="263"> 

#Generate SSH Key

**It is highly recommended to set SSH key, because chances are, you may need to access rails console for debugging purposes, and it's only possible if you have SSH access preconfigured. It can't be done after bootstrapping the app.**

You will need to generate SSH key on **Server** (Droplet), type in following commands

    ssh-keygen -t rsa -C "your_email@example.com"
    ssh-add id_rsa

(We want the default settings so when asked to enter a file in which to save the key, just press enter. Taken from [this guide][7])

# Install Docker

Run following commands:

    sudo apt-get update
    sudo apt-get install linux-image-generic-lts-raring linux-headers-generic-lts-raring

<img src="https://meta-discourse.r.worldssl.net/uploads/default/3001/e94722e882f28994.png" width="566" height="339"> 

Now we will perform a system reboot:

    sudo reboot

<img src="https://meta-discourse.r.worldssl.net/uploads/default/3003/d3cc759ced335d25.png" width="532" height="155"> 

Above command will log you out from ssh session, ssh in again:

    ssh root@IP_ADDRESS

replace `IP_ADDRESS` with your IP Address.

Type in following commands:

    sudo sh -c "wget -qO- https://get.docker.io/gpg | apt-key add -"
    sudo sh -c "echo deb http://get.docker.io/ubuntu docker main\
    > /etc/apt/sources.list.d/docker.list"
    sudo apt-get update
    sudo apt-get install lxc-docker

<img src="https://meta-discourse.r.worldssl.net/uploads/default/3004/e75967a1a8e27ea3.png" width="567" height="307"> 

# Install Discourse

If you have reached this step, Congratulations! You have already done all the hard work, now you have a brand new Ubuntu Server with Docker installed. All you need to do now is install Discourse itself, which is the most easy step. Keep Calm.

Create a `/var/docker` folder where all the docker related stuff will reside:

    mkdir /var/docker

Now we will clone the [Official Discourse Docker Image][4] in `/var/docker` folder:

    git clone https://github.com/SamSaffron/discourse_docker.git /var/docker

*Make sure to copy and run the above command as is, otherwise you will face [problem][5] which I faced.*

Let's switch to `/var/docker` directory:

    cd /var/docker

Now we will copy the `samples/standalone.yml` file and place it inside `containers` folder by name `app.yml`, so the path will become `containers/app.yml`, type in command:

    cp samples/standalone.yml containers/app.yml

<img src="https://meta-discourse.r.worldssl.net/uploads/default/3005/5c253f4657e2133f.png" width="571" height="56"> 

Now we need modify the newly copied `app.yml` with our default variables:

    nano containers/app.yml

*Nano is suggested by @riking  because it works like a text editor, just use your arrow keys. Hit `Ctrl-O` to save and `Ctrl-X` to exit. In below screenshot I am using Vim.*

You will see something like:

<img src="https://meta-discourse.r.worldssl.net/uploads/default/3006/ed9f51b3a44f2b86.png" width="572" height="451"> 

You may modify the file as per your requirement, but for the sake of simplicity I will only modify two variables `DISCOURSE_DEVELOPER_EMAILS` and `DISCOURSE_HOSTNAME`.

<img src="https://meta-discourse.r.worldssl.net/uploads/default/2979/e6fedbde9b471880.png" width="565" height="172"> 

Notice that I renamed the `DISCOURSE_HOSTNAME` to `discourse.techapj.com`, this means that I want to host my instance of Discourse on http://discourse.techapj.com/, for this to work properly you will need to modify DNS Records (I will post a separate guide to configure DNS Records).

#Mail Setup

**This step is required to successfully set up mail settings for Discourse.**

It is recommended to set mail settings before bootstrapping your app, if you are advanced user, you need to put your mail credentials in `app.yml` file.

If you are a beginner, now is the good time to create a free account on [**Mandrill**][6] (*Discourse recommended*), and put your Mandrill credentials (available on Mandrill Dashboard) in above file. The required information are `DISCOURSE_SMTP_ADDRESS`, `DISCOURSE_SMTP_PORT`, `DISCOURSE_SMTP_USER_NAME`, `DISCOURSE_SMTP_PASSWORD`.

#Add SSH Key

If you successfully generated the ssh key as described in step: **Generate SSH Key**, get the generated key using following command

    cat ~/.ssh/id_rsa.pub

Copy the entire output and paste it in `ssh_key` param, in `app.yml` file.

That's it, you're all set to bootstrap your Discourse instance.

# Bootstrap Discourse

Save the `app.yml` file, and run following command:

    sudo ./launcher bootstrap app

<img src="https://meta-discourse.r.worldssl.net/uploads/default/3007/c0596ad3d330ae71.png" width="567" height="138"> 

This command may take some time, but it's doing all the hard work for you. Go drink some coffee, while this command is *automagically* configuring the Discourse environment for you.

When this command executes, type in the following command to start instance of your Discourse app:

    sudo ./launcher start app

<img src="https://meta-discourse.r.worldssl.net/uploads/default/3008/ced00cf4782f020c.png" width="568" height="137"> 

Congratulations! You have your own live instance of Discourse running on the host you provided in `app.yml` file at the time of setup.

<img src="https://meta-discourse.r.worldssl.net/uploads/default/3009/5c7b0accf602dcca.png" width="689" height="246"> 

*You can also access your instance of Discourse by visiting your `IP_ADDRESS`.*

# Access Admin

If you configured `DISCOURSE_DEVELOPER_EMAILS` and mail settings properly, your email will be auto activated and will be made Admin by default.

In case your account is not flagged as admin (which is rare, but is apparently being reported by some users), try ssh'ing in your container (**to follow these steps, you are required to provide SSH key in `app.yml` file**):

    ./launcher ssh my_container
    sudo -iu discourse
    cd /var/www/discourse
    RAILS_ENV=production bundle exec rails c
    u = User.last
    u.admin = true
    u.save

Voilà, you are now the admin of your own Discourse installation!

Okay, that's it for this guide. In my next post I'll talk in detail about more advanced topics like *Configuring mail*, *Tweaking admin*, *SSH access to running containers*, etc.

Please provide your feedback, if anything needs to be improved, don't hesitate. Also the fine folks here at [Discourse][8] are always ready to help you, in case you face any problem.

**Note**: *Please go through below posts first, before asking problems, chances are someone else might have ran into exact same problem, and answer was available right here, all along. If you still can't find the answer, please do ask,  someone else might be facing the exact same problem. :sweat_smile:  *

**I'll try to update this guide as frequently as possible.**

**[Last Update: February 18, 2014]**


  [1]: https://github.com/discourse/discourse_docker
  [2]: https://github.com/discourse/discourse_docker#important-before-you-start
  [3]: https://github.com/discourse/discourse_docker#about
  [4]: https://github.com/discourse/discourse_docker
  [5]: https://meta.discourse.org/t/error-while-deploying-discourse-to-digital-ocean-using-docker/12126/27
  [6]: https://mandrillapp.com
  [7]: https://help.github.com/articles/generating-ssh-keys
  [8]: https://meta.discourse.org
