---
title: "PoC P2P Mesh C2 Framework"
date: 2025-08-24T22:14:52+05:30
draft: false
author: "Pratik Singh"
tags: ["Red Teaming", "C2", "P2P", "Decentralisation"]
categories: ["Projects"]
series: []
summary: "So I made a Peer-to-Peer Mesh C2 Framework for resillient red team ops (PoC), here what you need to know about this project."
cover:
    image: "/images/covers/mesh-c2-cover.png"
    alt: "A network mesh of implants"
    caption: ""
    relative: false
    hidden: false
ShowToc: true
TocOpen: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowShareButtons: true
searchHidden: false
---

## Introduction 🙂‍↕️
Hi there lawful-evil nerds, I made a **Proof-of-Concept C2 Framework**, but not the *usual* one.

In this C2 Framework, the implants use a "**Gossip System**" to *talk to each other* and share data, so every implant can stay updated with stuff like peer list and commad list. With no single point of failure in command.

This isn’t a production-grade framework, but a PoC I built to learn how P2P syncing and gossip protocols could power a  decentralized C2*.

> **this project is not completely decentralised but more like "hybrid-decentralised" like a torrent system with tracker*

## The Components and "Mesh" 🧩
Let me tell you about some organs of the project.

If you go to [the github repo of the project](https://github.com/pratiksingh94/mesh-c2) you will see 2 folders:

1. `C2/`: The C2 Server which contains the **API** of C2 and a **Dashboard** made in **Python using Flask** and **SQLite3** as database.
2. `implant/`: The implant or **agent** or **backdoor** whatever you want to call it, made in **C** which **lives inside affected systems** and executes commands.

**But what is this "mesh" thing?**

See, this is just a PoC (Proof-of-Concept) C2, one might even call this a *"toy"* C2, so I am not going to test this on real systems used by real humans in daily life, and thats why i decided to go with **Docker**

We can use Docker to start **multiple instances of implants** in **"containers"**, which are **lightweight isolated environments** that package code and dependencies together. Thats going to be **our sandbox** so we dont need to mess with host system.

And Docker has its own network interface (`172.17.0.1/16`) so all the implants will be on **the same network** so they can connect easily.

Not *realistic* simulation of "network mesh of implants" but good enough to learn about **P2P Syncing** and play around with our PoC C2, we can also use this in VMs by **building the implants inside the VM** so we can also use it in Blue Team practice in **detection of C2 traffic**.

## Demo and Hands-on 💥
You are not going anywhere without seeing stuff in action.
<br/>
Busy? Dont want to read text? Dont want to watch a video? No excuses, I got something for everyone.

### Run it yourself!
If you are one of those people that like to just dive in and see stuff work in real time, here's what you need to do:

#### Installing
First, run the install script or just git clone the project:

```bash
curl -fsSL https://raw.githubusercontent.com/pratiksingh94/mesh-c2/refs/heads/master/install.sh \
  | bash
```

<details>
<summary> Configuration (optional) </summary>

1. Navigate to the implant directory:
```sh
cd implant
```

2. Change the configuration

    Make a copy of `/includes/config.example.h` and rename it to `config.h`
    Now edit the content of the the file according to your setup anc choice

now you are ready
</details>


#### Starting the whole thing
Spin up the mesh of implants and C2 (3 implants by default)

```
cd mesh-c2
./start.sh
```

#### Viewing logs
Now pull up the logs of everything if you want to see the inside work

Open a new terminal window or tab, and run this (uses tmux windows to show logs of C2 server and all implants):
```
./attach-logs.sh
```

#### Extra options
To spawn more implants or enable verbose mode:

```bash
./start.sh 5 -v    # 5 implants, verbose
```

To use the installer script with custom destination name:
```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/pratiksingh94/mesh-c2/refs/heads/master/install.sh)" -- my-mesh-c2-dir

# it will install the project in my-mesh-c2-dir instead of mesh-c2 directory
```


### Full walkthrough video

Dont want to try by yourself?

I've made a demo video of this project so which goes through from the starting of implants to command broadcast, execution, gossiping, everything!

You can watch that if you have a couple minutes to spare and want to know see it in action:

<div align="center">
<iframe width="555" height="312" src="https://www.youtube.com/embed/CYCQuMdeyTY" title="I Made a P2P Mesh C2 Framework Proof-of-Concept (POC) | Demo Video" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
</div>

### Important parts in action

If you dont want to try by yourself or open youtube, you can go through the texts below and you can see some important part of projects in action (showing everything will make it too big):

#### Implants registerng

Once we start the spin up the C2 and a couple implants (using `./start.sh`) we can see them registerng by sending **Heartbeats** to C2, you can see it in the flask logs of C2 and also in the Dashboard at `localhost:8000` or whatever your port is.

But before registerng they also get an *"initial"* peer list from C2 server so they can gossip with each other, and this list will get updted through syncing too!

<div align="center">
<a href="https://ibb.co/JWd2z6Nc"><img src="https://i.ibb.co/9mv2qKJp/mpv-shot0001.jpg" alt="Screenshot of heartbeat requests in flask logs" border="0"></a>
<p align="center"><em>You can see the heartbeat requests in logs</em></p>
</div>

<div align="center">
<a href="https://ibb.co/Dxnf6h1"><img src="https://i.ibb.co/MYLkjrg/image.png" alt="Dashboard showing implants online and their details" border="0"></a>
<p align="center"><em>Dashboard gets updated with details every 10 seconds</em></p>
</div>

#### Sending commands

You can go to dashboard and send commands from there, there are two target modes, `All` and `Singular`.

1. `All` => Command will be broadcasted to every implant in mesh and they all will share with newer implans too those who join after the coommand was sent
2. `Singular` => Command to a single implant, wont be shared with other implants

<div align="center">
<a href="https://imgbb.com/"><img src="https://i.ibb.co/GvpmJGkf/image.png" alt="Dashboard screenshot of broadcasting whoami command" border="0"></a>
</div>

And now after we hit send payload button we can see the implants receving the commands in a couple second:


<div align="center">
<a href="https://imgbb.com/"><img src="https://i.ibb.co/p6k3YXvk/image.png" alt="Implant logs of receiving the command and executing it" border="0"></a>
<p align="center"><em>Implant 1 has received it and executed it too!</em></p>
</div>

#### Seeing results

Every command in command log table has a result button, press it when its there (if its not there you gotta wait for implants to send them, just a couple seconds)

<div align="center">
<a href="https://ibb.co/fzbrqRQ2"><img src="https://i.ibb.co/Rk1QYn79/image.png" alt="results from 3 implants" border="0"></a>
</div>

Got 'em!

If you dont see results from all implants just wait a couple more seconds they all different result timing (due to extra jitter time added to avoid showing a pattern to defense system)

#### "Gossiping" (syncing)

When new implant joins the network it will receive the broadcasted commands automatically through syncing

<div align="center">
<a href="https://imgbb.com/"><img src="https://i.ibb.co/7x3wBNMZ/image.png" alt="new implant receiving command through syncing" border="0"></a>
</div>

As we can see the new implant got the command we sent before through gossiping with other implants. You can also check that command's result, it will get updated too!

THIS IS SO CUTE LOL

Now the older implants will update their **Peer List** through syncing with new implant

<div align="center">
<a href="https://ibb.co/twW2KKkK"><img src="https://i.ibb.co/35LFddGd/image.png" alt="older implant got updated peer list through gossiping" border="0"></a>
<p align="center"><em>Check first line and last line of logs</em></p>
</div>


## Conclusion

This project took a long time, because when I started it I did not even know what is C2 and how it works, I was also beginner in C.

But it was a really good learning experience, and this is **my first *real* cybersecurity project** 🥹

If you see any issues in it or got any questions, you can [contact me](/contact) :)

*using "gossiping" inside of "syncing" is so fancy lol*

## Links

GitHub: [https://github.com/pratiksingh94/mesh-c2](https://github.com/pratiksingh94/mesh-c2)

YouTube Demo: [https://www.youtube.com/watch?v=CYCQuMdeyTY](https://www.youtube.com/watch?v=CYCQuMdeyTY)


### Tech stack:

**Implant:**
- [C](https://en.wikipedia.org/wiki/C_(programming_language))
- [Mongoose Web Server](https://github.com/cesanta/mongoose)
- [cURL](https://github.com/curl/curl)
- [cJSON](https://github.com/DaveGamble/cJSON)
- [make (GNU)](https://www.gnu.org/software/make/)

**C2 Server:**
- [Python](https://en.wikipedia.org/wiki/Python_(programming_language))
- [SQLite](https://www.sqlite.org/)
- [Flask](https://github.com/pallets/flask)

**Others:**
- [Docker](https://www.docker.com/)













