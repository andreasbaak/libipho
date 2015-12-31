---
layout: page
title: Installation
permalink: /installation/
---

TODO: image showing router, back end, and front end.

In order to install and configure Libipho,
you have to install and configure our
three main building blocks: a Wi-Fi router, a
back end running on a miniaturized computer,
and a frontend, running on a tablet computer.
In the following, we describe the parts that you
have to install.

# Router Configuration

Our Libipho front end needs to communicate with the back end running on
a miniaturized computer, see [hardware]({{ '/hardware/' | prepend: site.baseurl }})
for a description of the different pieces of hardware.
The front end expects to obtain the IP address of the back end by using
its host name "photobooth". In order to make this work flawlessly,
we suggest to set up a static lease for the host name "photobooth"
to the miniaturized computer based on its MAC address.
If your router does not support this feature, I suggest to
use a router that is able to run the open source operating system
[OpenWRT](https://openwrt.org/).

In addition, you need to set up a Wi-Fi. This Wi-Fi is required to let
the tablet computer communicate with the back end running on the miniaturized
computer. The local Wi-Fi can be used for another purpose.
With the Libipho project, you can allow your gueses to
have access to the gallery of pictures.
To this end, your guests can connect to the local Wi-Fi of the router
and access the gallery of pictures using the URL
[http://photobooth/](http://photobooth/).
Therefore, I recommend to set up
an encrypted Wi-Fi with a simple password that can easily be distributed
among the guests.

{::options parse_block_html="true" /}
<div class="note">
If you want, you can also connect the router to the Internet.
This will allow your guests to access the content of the photobooth
over the Internet. However, since the Libipho project
does not implement any access protection at the moment, we
do not recommend to open the access to the content for the
Internet.
However, if you really want to open the access for the Internet,
you can set up a simple authentication using .htaccess files
on the webserver running on the back end.
</div>

# Back End

The back end of Libipho runs on a miniaturized computer that sits within
the photobooth. The computer runs a custom Linux operating system that is
build with [yocto](https://www.yoctoproject.org/).
In the operating system, we autostart all Libipho software
that turns the miniaturized computer into a server for the photobooth.

In the next subsection, we explain how the operating system is structured
and how you can build and deploy it.

### Libipho Operating System

Our custom operating system (Libipho OS) is build with the
[yocto](https://www.yoctoproject.org/) tools, version Jethro.
We base our operating system on the yocto reference operating system called
"poky". In addition to the basic poky Linux distribution, our Libipho OS
adds software that is needed to turn your embedded hardware into a server
for the photobooth.

{::options parse_block_html="true" /}
<div class="note">
The yocto project makes it easy to build the Libipho operating system
for embedded hardware regardless of the corresponding hardware architecture.
For example, the Libipho OS can easily be build for a Pandaboard,
Raspberry Pi, MinnowBoard MAX or other miniaturized computers.
However, we implemented and tested the Libipho OS on the
[BeagleBone Black](http://beagleboard.org/BLACK) only and you might
have to make minor adjustments if you want to run Libipho on the
hardware of your choice.
</div>


The yocto tools assemble an operating system from scratch.
To this end, the complete operating system is described by a set of
metadata files. For our Libipho OS, we provide the collection of
required metadata files in the repository
[https://github.com/andreasbaak/libipho-platform](libipho-platform).
This repository also contain scripts for building the operating system
from scratch and for deploying the operating system to
a BeagleBone Black board.

### Building the Libipho OS

First of all, you need to prepare your Linux host operating system
for cross compiling another operating system from scratch with yocto.
To this end, follow the instructions in Section 3.2 (The Build Host Packages)
in the [Yocto Mega Manual](http://www.yoctoproject.org/docs/2.0/mega-manual/mega-manual).
Also, make sure that you have got about 10 GB of disk space available for temporary files
created during the build process.

Next, clone the repository that holds the metadata and initialize the build
environment:

{% highlight bash %}
mkdir ~/libipho
cd ~/libipo
git clone https://github.com/andreasbaak/libipho-platform
cd libipho-platform
source bootstrap.sh
{% endhighlight %}
The command *source boostrap.sh* initializes all git submodules. This
can take a while since a lot of metadata files and scripts will be downloaded.
In addition, the yocto build environment is initialized.

The next command,
{% highlight bash %}
bitbake libipho-image
{% endhighlight %}
will compile the complete Libipho OS from scratch.
On modern hardware, this step completes in under two hours (make sure
you have got something else to do meanwhile).

### Deploying the Libipho OS

After the build process has finished successfully, the build artifacts
reside in the folder *libipho-platform/deploy/image/beaglebone/*.
We have set up two scripts that allow you to easily deploy the
build artifacts to an SD card with a capacity of at least 8 GB.
After this deployment step, the SD card can be used to boot your BeagleBone Black board.

Connect the SD card to your host operating system.
Find the corresponding device that Linux assigns to your SD card
using, e.g., *dmesg | tail*. Note the device (e.g., /dev/sdc)
and use it as parameters for the scripts below.

{% highlight bash %}
cd ~/libipho/libipho-platform/
./format-sd-card.sh $device
{% endhighlight %}
Now, depending on your operating system, you have to detach and re-attach
the SD card to your host computer.
Then, execute the command
{% highlight bash %}
./deploy-beaglebone.sh $device
{% endhighlight %}
in order to extract the build artifacts onto your prepared SD card.

### Testing the Libipho OS

Insert the SD card into your BeagleBone Black board
and boot the board from the SD card. Depending on the configuration
of your board, you might have to press and hold the "boot" button while you
power up the board in order to boot from the SD card.

The Libipho OS boots up a web server that hosts a website showing
all pictures taken during the latest session of the photobooth.
Connect your device to the router (using your Wi-Fi or using
a network cable) and direct your browser to [http://photobooth/](http://photobooth/).
If this shows an empty gallery website, Libipho is set up correctly.


# Front End

The front end of Libipho serves the main purpose of a photobooth:
to display the most recently taken picture so that
the audience can immediately see the picture they have taken.

The front end of the photobooth runs on a tablet computer.
Currently, we offer two different implementations of the front end:
a browser-based front end and an implementation in an Android app.

### Browser-based Front End

The Libipho back end runs a web server that hosts a special web page
that represents a front end. On your tablet computer device,
you can display this special web page using the following URL:
[http://photobooth/screen.html](http://photobooth/screen.html)

We recommend to use a web browser that uses the whole screen of your
tablet computer by removing all status-, action-, navigation- and other bars.
For Android, we can recommend the following free app:
[Full Screen Browser](https://play.google.com/store/apps/details?id=tk.klurige.fullscreenbrowser).
We suppose that for iOS there will be similar apps.

This web page is set up in a way that it automatically reloads
as soon as the content changes. To this end, the browser polls
the web server for changes of this html page.
This can lead to slight delays (e.g., about 0.5 seconds)
in displaying the most recent image due to the polling interval.
For a minimal delay between taking the image and displaying it,
we recommend to use our Android front end app as explained in the
following subsection.

### Android Front End

In order to provide the smallest possible delay between taking the picture
and displaying feedback on the front end, we implemented an Android
app. This app connects to the back end and receives push notifications
with the image data of the most recent image, exhibiting virtually
no delay between downloading the image from the camera and showing
the image on the front end.

The source code of the Android app is hosted in the github repository
[libipho-screen-android](https://github.com/andreasbaak/libipho-screen-android).
Download the source code using git:
{% highlight bash %}
cd ~/libipho
git clone https://github.com/andreasbaak/libipho-screen-android
{% endhighlight %}

There are two different ways how you can compile and deploy
the Android app. First of all, you can download and install
[Android Studio](http://developer.android.com/sdk/index.html).
Then import the libipho-screen-android project into
Android Studio and compile and deploy using the IDE.

As an alternative, you can use [gradle](http://gradle.org/)
(version >= 1.7) to build and deploy the app.
Before compiling the app with gradle,
make sure that you installed the Android SDK (follow
the [installation guide](http://developer.android.com/sdk/installing/index.html?pkg=tools)
and a recent JDK (at least version 1.8.0).
Also, do not forget to set the environment variable ANDROID_HOME to the root directory
of your Android SDK.
The, run the following commands in order to build and deploy the app to the
attached Android device:
{% highlight bash %}
cd ~/libipho/libipho-screen-android
gradle build
gradle installDebug
{% endhighlight %}



