---
title: Cozy Developer Documentation

language_tabs:
  - javascript: JavaScript (ES6)
  - coffeescript: CoffeeScript

toc_footers:
  - <a href="https://cozy.io">Cozy Website</a>
  - <a href='http://github.com/tripit/slate'>Documentation Powered by Slate</a>

search: true
---

# Introduction

<aside class="notice">
Do not hesitate to <a href="https://github.com/cozy-labs/cozy-dev-docs/tree/master/source/index.md" target="_blank">edit this page</a> if you want to improve it.
</aside>

Welcome to the **Cozy Developer Space**.

<p style="text-align: center"><img src="images/cozy-logo-gradient.png" style="max-width: 150px"></p>

Here you’ll find everything you need to understand Cozy technically and to develop applications and konnectors.

The **technical level required** to read and understand this documentation is not over 9,000; you’ll roughly have to:

 * Understand how the web works (both **client** and **server** side)
 * Read/Write **JavaScript**, **HTML** and **CSS**
 * Install and run a **Node.js** process
 * Be comfortable with **command-line** calls
 * Know that cookie are not made of flour

 > Code examples are available in **both JavaScript and CoffeeScript** for <s>every troll's taste</s> everyone's preference.

```javascript
console.log(" ╰(◕‿◕)╯ ");
```
```coffeescript
console.log " ╰(◕‿◕)╯ "
```

<p style="text-align: center"><img src="images/cookie-monsta.png" style="max-width: 300px"></p>


## Why & How we made Cozy

![Capture 1](images/capture-1.jpg)

Cozy is a **personal web deployment platform**, which enables you to quickly bootstrap applications and interact with your data. It stands on a server - between your application and the operating system - easying the pain of **system administration**, **web development** and **security**.

### Data-oriented (WIP)

The initial idea was to create a space where developers can **experiment and play with their data**, while remaining in control of the platform. A friendly centralized point to fetch data from Things-Of-Internet, devices and personal services.

IMAGE

More than a simple Platform-as-a-Service, Cozy puts the cloud back where it belongs: at home.

Find out more on the [Architecture section](#architecture-amp-components)



### Tools

Cozy also became our **daily working environment**, and we want it to be as comfortable as possible. It means that we require it flexible, extensible and usable. We use **Node.js** and **CouchDB** as our main technologies.

<p style="text-align: center">
  <a href="https://nodejs.org" target="_blank"><img src="images/nodejs.png" style="width: 30%"></a>
</p>


**Node.js** is a great and modern engine with a huge community of module creators. It has elvoved along with the GitHub's Open-Source popularization and is now wide-spread in the Web industry. Aside from being heavily maintained, it remains fun to extend and play with.

<p style="text-align: center">
  <a href="https://couchdb.apache.org" target="_blank"><img src="images/couchdb.png" style="width: 30%"></a>
</p>

**CouchDB** is surely less-known, but definetly reliable. As a proper Erlang program, it has proven its strength and stability. A consistent, well-documented API, plus wonderful replication capabilities: the recipe of what we love on a database engine.    



## Developing with Cozy


“ *Cozy has been a great opportunity for me to start coding. Now I run a decent business.* ”     
<small>Bill Getas - Startup entrepreneur</small>

“ *Thanks to Cozy, I successfully managed to build a working web application without a hassle.* ”     
<small>Tim Bernee-Lers - enthusiast web developer</small>

“ *I'm sorry Dave, I'm afraid I can't do that.* ”     
<small>HAL 9000 - CoffeeScript compiler</small>

<p style="border-bottom: 1px solid #ccc;"></p>

But to be honest, we don't need those fancy words to convince you to develop on Cozy.


### To the future, Marty!

Cozy is a **modern web platform** that abstracts a lot of complexity out of personal **data manipulation**. If you are planning on developing a tool to **fetch**, **visualize** or **mix data**, a Cozy Application or a Konnector may be a good way to start.

By developing on Cozy, you will:

 * Reach a community of **enthusiast testers** and end-users
 * Learn how to make **single-page applications** with modern JavaScript frameworks like **Angular.js** or **React.js**
 * Be guided by [Cozy mentors](#mentorship), JavaScript gurus and other contributors
 * Enjoy the **built-in security** of Cozy

And whatever the reason, you will always find a friendly team member to help you dealing with your struggles and questions on the [IRC channel](#irc).

**So let's dive into it**!

![Back to the Future](images/back-to-the-future.jpg)


# Getting Started

This section is a quick tutorial showing off the different steps to start using Cozy as a development environment, making a simple app, and interact with your data.

## Set up the Development Environment

Since Cozy is made from several pieces, we wrapped it into an **easy-to-use Virtual Machine**, so you don't clutter your local system. That also means you can use the operation system you like! Well, there are obviously issues on Windows, so we will assume here that you use a **GNU/Linux system or Mac OS X**.

The environment is made of two parts: the Virtual Machine itself - a fully installed Cozy platform - and a local Node.js tool called `cozy-dev` which will help you getting started and deploying your app to your Cozy.

### 1. Install Git

 > On GNU/Linux, just install Git with your package manager

```shell
apt-get install git
```

 > On Mac OS, download the software here: <br>
 > <a href="http://git-scm.com/download/mac">http://git-scm.com/download/mac</a>

```shell
# To check if Git is installed
git --version
```

The first step is obviously to install the development dependencies. Git is the [SCM](https://en.wikipedia.org/wiki/Version_control) we use at Cozy, and if you don't know how to use it yet, don't worry: we will give example commands in the following tutorial.


### 2. Install Node.js 0.10.x

 > On Debian GNU/Linux Jessie, the proper version of Node.js is available in the official repositories

```shell
apt-get install nodejs nodejs-legacy
```

 > On other GNU/Linux systems, you can install Node.js manually by doing

```shell
wget -q -O - http://nodejs.org/dist/v0.10.40/node-v0.10.40.tar.gz | tar xz
cd node-v0.10.40
./configure
make
make install
cd .. && rm -r node-v0.10.40
npm install -g npm
```

 > On Mac OS X, simply download the package, and install it <br>
 > <a href="https://nodejs.org/dist/v0.10.40/node-v0.10.40.pkg">https://nodejs.org/dist/v0.10.40/node-v0.10.40.pkg</a>

```shell
# To check the Node.js version
node --version
```

Installing Node.js on your local environment is necessary to run your app and use `cozy-dev`. Cozy runs well on <strong>Node.js v0.10.26 to v0.10.40</strong>, and you will have to install one of these versions.

You can find detailed instructions of installation on the [official page](https://github.com/nodejs/node-v0.x-archive/wiki/Installing-Node.js-via-package-manager).

<aside class="warning">
If you already have a Node.js version installed on your computer that is not between <strong>v0.10.26</strong> and <strong>v0.10.40</strong>, you will have to consider <strong>upgrading/downgrading</strong> it, or to use a tool like <a href="https://github.com/tj/n">N</a>.
</aside>


### 3. Install VirtualBox and Vagrant

**VirtualBox** is used to emulate a full Operating System in a Virtual Machine. You can download it from [https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads)

**Vagrant** is a Command-Line Interface to interact with VirtualBox and handle development environment easily. You can download it from [https://www.vagrantup.com/downloads.html](https://www.vagrantup.com/downloads.html)

<aside class="notice">
On Debian GNU/Linux 8, <code>virtualbox</code> and <code>vagrant</code> are available in the official repositories.
</aside>


### 4. Install <code>cozy-dev</code>

```shell
# Use the -g option to install system-wide
sudo npm install -g cozy-dev
```

As mentionned above, `cozy-dev` is a set of tools aiming to help you managing your app deployment and your development environment. It is recommended to install it system-wide.

The full documentation of `cozy-dev` can be found below on the [related section](#cozy-development-environment).


### 5. Download and start the environment

```shell
# Create a development directory
mkdir cozy && cd cozy

# Download the environment
cozy-dev vm:init

# Start the environment
cozy-dev vm:start

# Check that the environment is properly started
cozy-dev vm:status

# Update the environment (strongly recommended)
cozy-dev vm:update
```

Use `cozy-dev` to initialize the environment.    
It is recommended to **update it** at the end because we don't necessarily maintain the raw image in an up-to-date state, as a new version of Cozy can be released every couple of weeks ☺

<aside class="notice">
The command <code>cozy-dev vm:init</code> can take a few minutes to complete as it downloads the full environment. You only have to do it once though.
</aside>

<aside style="clear: both" class="success">
You should now be able to go to <a href="http://localhost:9104">http://localhost:9104</a> ! ☺
</aside>

## Hello, World!
## Interact with your Cozy's data
## Import data with a Konnector
## Debug your application
## The Cozy Way: Developing Modern Web Applications

# Going further
## Architecture & Components
## Learn CouchDB
## Authentication and permissions
## Internationalization
## Realtime events
## Encryption management

# References
## Data System API
## Cozy DB API
## Controller API
## Cozy Development Environment

# Getting help
## IRC
## Forum
## Email
## GitHub
## Mentorship
