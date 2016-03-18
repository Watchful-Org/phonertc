> [Wiki](Home) ▸ **Installation**

### Overview

In order to use PhoneRTC in your app, you will need to set up 3 things:

1. PhoneRTC Cordova plugin
2. TURN server
3. Signaling server

### Plugin Installation

Install Cordova:

    npm install -g cordova ios-deploy --unsafe-perm=true

Create a new Cordova project:

    cordova create <name>
    cordova platform add ios android
    cordova platform add browser --usegit

Add the plugin:

    cordova plugin add https://github.com/mbanting/phonertc

For iOS, follow these steps:

1. Go `platforms/ios` and click on [ProjectName].xcodeproj to open it with XCode
2. Click `Convert` when `Convert to Latest Swift Syntax` modal appears
3. Go to your project settings
4. In General, change Deployment Target to 7.0 or above
5. Go to Build Settings and change:

    a. `Valid Architectures => armv7`

    b. `Build Active Architecture Only => No`

    c. `Runpath Search Paths => $(inherited) @executable_path/Frameworks`

    d. `Objective-C Bridging Header => [ProjectName]/Plugins/com.dooble.phonertc/Bridging-Header.h`

    e. `Embedded content contains Swift Code => yes`

6. Repeat steps 5a. - 5c. for the CordovaLib project

7. Make sure your build target is an actual iPhone or iPad running on the arm7 architecture. The iPhone and iPad simulators are not emulators, and only run on i386. The compiled RTC libraries for ios have been built for arm7.

### Setting up a Signaling server

When starting a call, PhoneRTC tries to establish a peer-to-peer connection between 2 users. It uses the **signaling server** to exchange network-related information between the 2 users.

The signaling server is simply a **way to exchange messages between 2 users**. It's something that you need to provide that works with the user system of your app. You don't care about the *content* of the messages, you just need to provide a way to exchange messages.

We recommend using [socket.io](http://socket.io/) or [SignalR](http://signalr.net/) for this purpose.

A SocketIO-based signaling server is provided in the [demo/server](https://github.com/alongubkin/phonertc/tree/master/demo/server) directory as an example.

### Setting up a TURN server

In case the peer-to-peer connection fails, PhoneRTC will use your TURN server. The TURN server is simply a proxy.

To set up a TURN server, create an Amazon EC2 instance with the latest Ubuntu. Open the following ports in the instance security group:

    TCP 443
    TCP 3478-3479
    TCP 32355-65535
    UDP 3478-3479
    UDP 32355-65535

Open SSH and run:

    sudo apt-get install rfc5766-turn-server

Next, edit `/etc/turnserver.conf` and change the following options:

    listening-ip=<private EC2 ip address>
    relay-ip=<private EC2 ip address>
    external-ip=<public EC2 ip address>
    min-port=32355
    max-port=65535
    realm=<your domain>

Also uncomment the following options:

    lt-cred-mech
    fingerprint

Next, open `/etc/turnuserdb.conf` and add a new user at the end of the file. The format is:

    username:password

To start the TURN server, run the following command:

    sudo /etc/init.d/rfc5766-turn-server start
