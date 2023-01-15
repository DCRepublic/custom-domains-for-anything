---
layout: default
title: Home
nav_order: 1
---

# Installation
{: .no_toc }

{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Register a free domain with Freenom
1. I've been using freenom for free domains for a while now and there hasn't been any issues. If you want to pay for a domain or if you already have a domain you pay for then you can skip this step. 

2. Head to [Freenom](https://www.freenom.com/en/index.html?lang=en) and begin searching for a domain. Type the name you want to use and hit "check avaliability". 

3. You will be prompted with a bunch of free endings (Common ones are .tk .ml and .ga). Choose one that you like 


## Create a Cloudflare Account

First, head to [cloudflare's website](https://cloudflare.com) and sign up for a free account. We will be coming back to this dashboard later.

## Install Cloudflared on your device

The install script depends on the architecture of your system. Check what it is with the following command: 

```console
uname -a
```

### arm64 architecture:
```console
sudo wget -O cloudflared https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64
sudo mv cloudflared /usr/local/bin
sudo chmod +x /usr/local/bin/cloudflared
cloudflared -v
```

### AMD64 architecture:
```console
sudo wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo apt-get install ./cloudflared-stable-linux-amd64.deb
cloudflared -v
```

### armhf architecture:
```console
sudo wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-armhf.deb
sudo apt-get install ./cloudflared-linux-armhf.deb
cloudflared -v
```

Once clouflared has been installed login using your credentials. It will give you a url to paste into a browser for you to login

```console
cloudflared login
```

