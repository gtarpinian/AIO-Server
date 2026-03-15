# AIO-Server
 Plans for building a home server with a AIO PC

## Introduction

We're going to be installing Linux on the AIO PC in order to install Docker. Docker is the most useful way to install server apps because the installation can be declared as code, the docker compose stack can install dependencies such as databases as separate services and because it is isolated from other parts of the OS.

Apps installed with Docker are called Containers. Containers have their own file system, and you can mount a directory from the host file system inside the container file system. For example, you'll often mount ``~/docker/appname/config`` as ``/config`` inside the container. In this respect containers are like virtual machines, but the difference is they use the host system's kernel. This lets them take up an order of magnitude less space than a virtual machine.

Because the containers are isolated and the storage space needed is small, they can be backed up and migrated at a whim. Given the same backed up configuration file and docker compose file, the same container will be recreated every time.

### What we will do today

- Install Linux
- Install Docker
- Speedtest container
- Docker manager (Arcane)
- Tailscale for remote access
- TSBridge for Tailscale inside Docker
- Mealie
- Immich
- Home Assistant
- Automatic Ripping Machine
- Jellyfin

## Linux OS

We need to install Linux because docker needs access to the host system kernel, which Microsoft doesn't provide. Any version of Linux would work here, but because the AIO has a built-in screen, it just makes sense to install a regular desktop environment instead of a dedicated server "distribution".

That being said, we don't want to waste any resources on having a GUI that no one is going to be using most of the time. Therefore I chose one of the lightest weight desktop enviornments, called LXQt. The LXQt desktop is being installed on top of Fedora Linux, but it could have just as easily been installed on top of Debian, Ubuntu or Arch.

I happen to prefer Fedora, and use it on both Martina's gaming PC as a Windows 11 replacement and on my mini-server. Hopefully this should let be helpful and familiar with it. But this just goes to show how versatile Linux is, that the Windows 11 gaming PC and the headless server use the same underlying OS. For LXQt, it will kind of be like installing Windows XP GUI on a Windows 11 foundation. Everything will be super modern but the interface will feel a bit dated, in the name of conserving resources.

### Backing up Windows

It's always worth taking a little extra time to make sure we've copied any important files over to another device before wiping the hard drive. You haven't used this computer for much lately, but you never know if there's some important tax document or what have you from several years ago. 

The most important directories to back up are Documents and Photos. We can start out by copying those. Then we can take a look at Downloads. As a rule, downloads should only have files that could be downloaded again, but sometimes we download a file, make an edit, and save it still in the downloads folder.

The easiest way to not waste space on the back up is to sort the Downloads directory by file size, descending, and just delete the first few biggest files that we know we don't need. Then simply copy over the rest of the directory.

As an aside, let's grab a screenshot of how much RAM Windows 11 i using before we remove it and install Linux.

### Getting the install

Linux is free of course, and is usually downloaded directly from the website of the distribution being used. We'll therefore go to Fedora's website and download the [LXQt version](https://fedoraproject.org/spins/lxqt/). We want the Live ISO for x86_64 (Intel and AMD processors). We will also grab the Fedora Media Writer software, which will turn a thumbdrive into something the computer's BIOS can boot from.

Download both of these, then install the Fedora Media Writer software. Once that's installed, it will explain how to flash the downloaded ISO to a USB stick. Just trust the process.

### Installing Linux

This will be pretty fast and then really slow, so let's keep up the pace on the fast parts. Turn off the AIO for the last time as a Windows machine. Plug the USB into the computer, turn it back on, and start mashing the delete key. Look for some text in the first screen that pops up that says "Press this key for boot menu." If we're lucky it's the delete key and we'll already be pressing it, but otherwise just be quick and press the button it says. If you miss it and Windows starts back up, no worries, we'll just restart.

In the boot menu there should be some settings we need to change. First we will need to disable Secure Boot, which is some probably smart but in practice limiting system of certifying that the hard drive hasn't been tampered with in any way between start ups. Then we need to go to the boot order settings and tell the BIOS to boot from the Fedora USB instead of Windows. Set this and exit the BIOS to restart the computer once again.

We'll then boot into the "live" version of Fedora. This means that the USB is the hard drive and nothing has been changed yet to the original hard drive. We can play around a little and see what we think of how the Linux looks but we will want to start the installation process soon since it takes a while.

Click the install shortcut on the desktop, follow the instructions and get the installation going. We're going to be wiping the whole Windows disk but we have already backed up any important files. **Resist the urge to hoard!**

We're not letting this computer talk to the outside internet much so we can use an easy and memorable password, no Dashlane goobly-gook. But do remember the password, we'll need to use it often to use admin overrides.

### Exploring

While the install is happening, we can take the time to explore the desktop environment because it is the same as what we will be installing. Check out the start menu, open up a few apps and see what you think.

### Updating

This step and the installation are two to of the longest steps, so hopefully we can keep up the pace and get to the fun stuff. The ISO we downloaded was written a few months ago, so there have been many software updates since then. Therefore, the first thing we will do in our new system is run an update.

Find the terminal from the start menu and open it up. Then to run the update simply type ``sudo dnf update`` and hit enter.

Let's break this command down:

- ``sudo`` means do the next command as a super user (admin). We need to use this for updates and a lot of what we're setting up today.
- ``dnf`` is the name of the package manager in Fedora. Ubuntu and Debian use ``apt`` which is very similar.
- ``update`` is the command we want ``dnf`` to do. If we were installing a new app, we'd instead write ``dnf install appName``.

When we enter this command, it will first figure out what all needs to be updated. Then, before beginning the update, it will ask if you want to proceed and ask you to respond ``(y/N)``. The capital N is the default, meaning that if we typed nothing and hit enter the update would not happen. Instead type y and hit enter to start the update, which will take a fair bit of time.

### Install something

Just for demonstration, let's install an app with dnf. Type ``sudo dnf install libreoffice`` and hit enter. This will ask if you want to install a free and open source alternative to Microsoft office. In one command we can install the entire package with no extra effort needed from our end.

## Docker

Now that we're set up we can install Docker. Docker is a little particular in that they want us to add their own website as a source for Docker, instead of wherever ``dnf`` would normally look. This isn't normally something you should trust an app with, but we're actually going to do it twice today.

For Docker, we'll go to their website using the Falkon browser and find the [install instructions for Fedora](https://docs.docker.com/engine/install/fedora/). There's a couple of steps we *could* do but further down they have what they call the convenience script.

``` ./bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh
```

Let's do that, seems easier.

Once that has done it's thing, we'll make it so we don't need to use ``sudo`` to use docker. 

``` .bash
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```

These commands create a group called Docker, modify our user to become a member of the docker group and finally refresh the group so that the changes are loaded.

We can then create our first container to make sure everything is working. Type ``docker run hello-world`` and run it in the terminal.

Oops, that didn't work. Looks like we need to start the docker "service". We can also set it to always start on reboot in one command:

``sudo systemctl enable --now docker``

Let's try hello-world again and see the fruits of our labor. 

Not very impressed? Let's grab our first container instead.

## Open speed test

In the web browser let's seach for openspeedtest docker and open the link to docker hub. This is an official directory of docker "images" that can be installed.

Scroll down to the self hosted installation section and we will find two different ways to install, Docker Run and Docker Compose. Docker run can be copied and pasted directly into the terminal, which is very convenient.Let's try that out: 

``` ./bash
sudo docker run --restart=unless-stopped --name openspeedtest -d -p 3000:3000 -p 3001:3001 openspeedtest/latest
```

This installs a self hosted internet speed test for within the local network. The app is a server that uses port 3000 to be accessed. This means we can go to ``localhost:3000`` in the browser and would you look at that. Our own little website.

Click the speed test though and you'll notice that this is super fast, faster than the wifi ought to be. That's because since we're testing over localhost, we aren't going out over the internet at all, and we're basically seeing how fast our system is.

We need to test from another computer to see how fast the network is. But what would we put in the address bar? Localhost won't work because it's only for that device specifically. Therefore we need to get the ip address of our new server using the command ``ip addr``. Look for the one beginning 192.168.x.xxx. This should be our local network (LAN). Now on your phone or something, try going to to the address [192.168.1.3:3000](192.168.1.3:3000).

Let's run the test again. Ouch, not crazy fast, huh? One thing I'll be pedantic about is using a wired connection for server computers. Let's go plug in with an ethernet cable and run the test again.

Wow, what a difference! But really, it is worth having it full plugged in. Handy that there is wifi as a backup, though.

Back to the docker process. Didn't I say that this should be repeatable? However, if we tried to run docker run with the same settings, we would end up with a second container openspeedtest-1. The ease of updating that I want to acheive comes with docker compose. 

Looking at the docker hub page, though, what would we do with that? Can't just paste that into the terminal. But if you look closely you can see the similarity to the docker run command. These will actually make the same container, it just has to be set up the right away.

Let's get organized and keep all our docker folders together.. We should have your USB SSD plugged in and mounted permanently. Next let's go there and create a new folder. We could use a file browser, but we're learning today! Let's use the command line.

When we start the terminal, we generally start out in our home directory. This is located at ``/home/userName``. We can verify we're at the home directory by running the ``ls`` command. This lists the files and folders in our current directory, and we should see the same as if we were in the file browser. Now we should switch to where want to put the docker folders, which is on the ssd. We set it up at ``/mnt/fast/``. To get there simply run ``cd /mnt/fast/``. cd stands for change directory.

Now let's make a docker folder. Run ``mkdir docker`` to make a directory called docker, and let's ``cd`` into that directory. Here's a couple tricks:

- When you want to change into a directory that sits in your current directory, you don't need to type anything but that directory's name
- If you start typing something in the terminal, you can often press tab to see if it can be autocompleted

Therefore let's type ``cd`` and then press tab. Bam, the only possible option is docker. Enter the directory, make a new directory called openspeedtest and change into it. Let's copy this docker compose text from the docker hub. Now create a new document by running the terminal text editor, ``nano``. 

Linux doesn't use all of the same shortcuts as windows, and it's not even consistent between apps. ``ctrl+v`` works fine in the web browser but in the terminal we need to use ``ctrl+shift+v``. Paste the docker compose text in nano. Now to exit, we need to use the shortcut listed at the bottom ``ctrl+x``. It will prompt us to save the file, very polite. Docker compose files always need to be saved as docker-compose.yml. Therefore it's good to keep them all in separate directories.

Now we can exit nano and the terminal is back in the ``.../docker/openspeedtest`` directory. ``ls`` shows us that the file we just created is here and accounted for. Now the reason that the file always needs the same name is because from this directory we can just run ``docker compose up -d``. Woah, what just happened???

Docker tried to create a container using the instructions from the yml file, but it didn't want to create a second container that's the same as the one we already have with docker run. Therefore we need to run ``docker stop openspeedtest`` and ``docker rm openspeedtest`` to stop and remove the container, respectively.

Another terminal shortcut: simply use the up arrow to back in the history of commands you've done, and keep going until the docker compose command. Run it, and now we have the exact same container. The handy bit is we can run updates on this container, and also create multiple containers at once. Notice a small warning that the attribute version is obsolete. We can keep in mind in the future that we don't need to include this.

## Arcane

Now it was kind of a pain faffing around with that nano thing, and I am a merciful tech guy. Let's use something a little easier and friendlier. I like this look of [arcane](getarcane.app). Let's check out this docker compose generator they have to install it.

This starts a little wizard that should give some insight as to what a docker compose file *is*. Checking out these defaults, it looks good to me. Let's hit next.

Now it asks if we want an external PostgreSQL database. Sounds complicated, skip it and head to Authentication. OIDC is super cool but outside today's scope, so hit Generate Docker Compose.

Wow this one has a little bit more going on than the speed test. Let's hit Download so we can skip working with nano again. This will save the file to our Downloads folder. But we want this in our organized Docker folder. First we need to ``cd ..`` to our docker folder. Then we can ``mkdir arcane`` and finally we can do ``mv ~/Downloads/docker-compose.yml arcane/docker-compose.yml``. ``~`` is a shortcut for the user home directory. ``mv`` is funny because you use it really is a renaming command. You're just changing the full government name of the file, including it's parent folders, so it "moves" in that respect.

I lied, we do need to use nano one more time. I didn't realize you need to tell Arcane where your current docker compose folders are. We will ``cd arcane`` and ``nano [tab for the yml file]`` and scroll down to the volumes section. Several folumes will be there already and we will add a third line in the same format. It should read ``/mnt/fast/docker:/app/data``. This tells Arcane to build its own container with the folder where we keep our compose files mounted in the container at ``/app/data``. Finally ``docker compose up -d`` to launch our new friend.

Now we can check out Arcane through a web browser. We can use ``localhost:3552`` if you want to use this computer but you can also access this from any computer on the wifi network at the ip address, like the speed test.

However we get there, the first thing we'll do on Arcane is login with ``arcane`` and ``arcane-admin`` as the username and password. Then we will need to change the password which will allow us to take in the glory of a dashboard. I'll see this as kind of a home page for what's running on this server, although there are specialized home page docker containers that can be customized.

To make sure we imported our current compose files correctly we'll go to the projects section. We should see Arcane and openspeedtest. If we click on them we can see the docker-compose files and even edit them. There's more to see here, but let's learn by doing and move on to the next container.

## Tailscale and TSBridge

I don't know about you but I'm getting sick of these localhost ip addresses and port numbers mumbo jumbo. We should have nice clean website URLs like all the other kids. There are several ways to go about this, but keep in mind that making a URL that can be accessed from anywhere means opening a door from the whole wide internet into the home network. Everything would probably be fine because firewalls would prevent any mixing of external and internal traffic, but it seems unnecessary. 

What if we could just pretend like we were home, all the time? That's what a VPN can accomplish, and one of the easiest ways to manage a personal VPN is Tailscale. You install Tailscale on your devices and they form a mesh network of VPNs to each other. The really cool part, though, is that they only use the VPN when they need to, i.e. it something they can't get on the regular world wide web. Tailscale will even fix up these URLs for us.

Hopefully you've already created a Tailscale account and added me as a second admin user with matthew.tarpinian@gmail.com. You should set it Tailscale up on your regular Windows computer and iPhone. If you want, we can even set it up on this all-in-one, for consistency.

To get those individual URLs for the docker containers, though, we will use another container called TSBridge. It will be authorized to create new Tailscale nodes for every Docker container you have running. Neat!

First, make sure MagicDNS is enabled in Tailscale. Then we can go to the [Github repository for TSBridge](https://github.com/jtdowney/tsbridge). Apps that turn random ports into URLs are called reverse proxies, and TSBridge is modeled after a reverse proxy called Traefik. Instead of forwarding out to the internet, though, it will forward to the mesh network of VPNs.

Scroll down on the Github page to the Docker labels section and click the link to the documentation. If we read this page, it explains how we need to mark our Docker compose projects to be picked up by the TSBridge service. It also explains how we need to format the TSBridge compose file, but I've set that up for us here:

``` .yaml
services:
  tsbridge:
    image: ghcr.io/jtdowney/tsbridge:latest
    container_name: tsbridge
    command: ["--provider", "docker"]
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - tsbridge-state:/var/lib/tsbridge
    environment:
      # Pass the actual OAuth credentials to the container
      - TS_OAUTH_CLIENT_ID=${TS_OAUTH_CLIENT_ID}
      - TS_OAUTH_CLIENT_SECRET=${TS_OAUTH_CLIENT_SECRET}
    labels:
      # Tell tsbridge which env vars contain the credentials (not redundant - both are needed)
      - "tsbridge.tailscale.oauth_client_id_env=TS_OAUTH_CLIENT_ID"
      - "tsbridge.tailscale.oauth_client_secret_env=TS_OAUTH_CLIENT_SECRET"
      - "tsbridge.tailscale.state_dir=/var/lib/tsbridge"
      - "tsbridge.tailscale.default_tags=tag:containers" # Must match or be owned by your OAuth client's tag
    networks:
      - tsbridge-network
    restart: unless-stopped

networks:
  tsbridge-network:
    external: true

volumes:
  tsbridge-state:
```

This is the most complicated one yet. This container expects a network, a volume, client secrets, oh my! Let's take it one step at a time. First, we can create the network and volume that are expected in Arcane. On the left side tool bar under resources, we can click on Networks and we'll see some default networks that have already been created.

Let's create a new network and call it tsbridge-network, as it says in the compose text above. Then we can go to Volumes and create one called tsbridge-state. No need to change any other settings for either of these.

Then we can go to projects and create a new project. Call it tsbridge and delete the template text that is inserted in the boxes. Copy and paste the compose text I wrote above into the Docker Compose file box. 

We also see an Environment box. This creates a special hidden file next to the compose.yml file where we can put those secrets that are referenced in the compose file. Let's get the ball rolling by copying the ``TS_OAUTH_CLIENT_ID=`` and ``TS_OAUTH_CLIENT_SECRET=`` into the env box.

To generate those secrets we need to go to the [admin page](https://login.tailscale.com/admin/machines) for Tailscale. Click on Settings and the Trust credentials page. Generate a new credential and name it tsbridge.

Next we need to define what the credentials can do. TSBridge documenation says it needs the scope of Auth Keys - Read and Write. Hit Generate credential and then be still. The credentials are only going to be shown until we press Done, and will never be visible again. If we goof up, we'll have to delete the credentials and create new ones, but I believe in us.

All we need to do is copy the client id to after the = sign in the .env file in Arcane, then do the same for the client secret. Easy peasey. Lastly, we see something about tags. This is a setting in Tailscale and is required for these nodes created by TSBridge.

In Tailscale admin, go to Access Controls and Tags. Make a new tag called containers and make it owned by ``Autogroup:admins``. Save the tag, then we can go ahead and create the project in Arcane. We get some new buttons which we will explore by pressing Up. We now have a tsbridge container running, but need to tell it about our other containers.

We'll skip openspeedtest for now, but we can fix up Arcane, using Arcane. Meta! We'll go to Configuration, and then under the environment section put our cursor at the end of the last line, and then copy and paste the below text from the end of the first line.

```.yaml
    - blah blah last line of env variables, copy from end of this line
    labels:
    - "tsbridge.enabled=true"
    - "tsbridge.service.port=8080"
    networks:
      - tsbridge-network

networks:
  tsbridge-network:
    external: true
```

All of the indenting is important, but locally Arcane will point out any formatting errors. Docker networks are like mini home networks inside a docker compose file, so that they're isolated from each other. In our case, though, we want them all on the same network that TSBridge is listening on to turn them into URLs.

Now we can press save, and then normally we would press redeploy. However, things go a little funny when you use Arcane to redeploy itself, so we'll jump back to the terminal, ``cd /mnt/fast/docker/arcane``, ``docker compose down`` to take down the old version and then ``docker compose up -d``. This tells Arcane to shut down, rebuild itself with the new compose information and start back up. This is why we are using Compose instead of Run.

Now we need to head back to Tailscale admin Machines. We should see a new machine listed called Arcane. What fun! That worked and definitely didn't take an hour of troubleshooting.

The Arcane machine in tailscale has an IP address, but if you click on that IP, it should dropdown the other names it could be called, including the fully qualified domain name, that looks a real URL.

Let's keep going!

## Mealie

This next one is a recipe organizer and meal planning app. By default it does a pretty good job of importing online recipes. If you want to play with AI, you can set it up to take scans of physical recipes and parse them into its databse.

Let's take a look at [Mealie's installation instructions](https://docs.mealie.io/documentation/getting-started/installation/sqlite/). Mealie has instructions for this version, with the database built into the application or for using a separate database container. This version should be fine for family use but it's cool that they think about scale.

Before we create a new project in Arcane for Mealie, let's set it up to use our TSBridge labels as a template. Click on Customizations > Templates, and then let's edit the default. Delete the text in the Docker Compose Template and paste:

```.yaml
services:
  XXXX:
    labels:
      - "tsbridge.enabled=true"  
      - "tsbridge.service.port=XXXX"
    networks:
      - tsbridge-network

networks:
  tsbridge-network:
    external: true
```

This will let us copy over the xxxx with whatever Docker Compose file we like. Click save, then delete the template .env file and save again. Now we can copy Mealie's docker compose text and bring it over to Arcane.

Create a new project, highlight from the xxxx: to the top and paste in our docker compose. Something might look a bit off with the indenting, though. The ``volumes`` section looks like it should be at the bottom. However, we're actually going to delete this section entirely as we will be storing the Mealie app and database on the ssd.

Now that we've deleted the volumes section, we still need to tell docker where to put the app files. In the main body, under volumes, delete ``mealie-data`` and replace it with ``./config. This tells docker to create a new directory called config in the directory where the docker-compose.yml file is located. Arcane takes care of creating the parent directory and compose file, so we don't have to manually create any directories or documents!

Before we create this, the last thing we need to do is fix the TSBridge labels, namely the port number. Look at the ports section of the compose file. Which side of the colon do you think we use? Hint: it's the same format as the volumes section.

That's right, the right side of the colon is for inside the docker container, and left side is what exists outside the container. Since the TSBridge lives inside the container, we will use 9000.

Now we can press up to create this app. To get the url, we can just copy the Arcane url and replace arcane with mealie. Let's log in, create an admin account and take a look around. Do you have a recipe that you've found online that you want to go into Mealie? Let's test it out.

Something I've talked about is scanning your physical recipe cards. To do this, we'll need to add AI into the mix, which is a bit outside of today's scope. Basically, we'll change the Docker compose file a bit with the account information for an OpenAI account. We'll need to load up some money but Mealie has the requests optimized to only cost a few pennies per recipe. Like I said, we can try that another time.

## Immich

This is what we've been working towards. To get this far we've had to put in the work, the hours, separate the boys from the men. Now we are finally looking at the Google Photos replacement, Immich.

