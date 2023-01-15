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

2. Head to <a href="https://www.freenom.com/en/index.html?lang=en" target="_blank"> Freenom </a> and begin searching for a domain. Type the name you want to use and hit "check avaliability". 

3. You will be prompted with a bunch of free endings (Common ones are .tk .ml and .ga). Choose one that you like and follow the steps to checkout and claim your domain. The free version lets you go up to 12 months on a single domain before it expires. You probably want to choose 12 months! (You will need to create a free account). **You should not have to pay for anything!**

4. Do not close this page 

## Create a Cloudflare Account

1. First, head to <a href="https://cloudflare.com" target="_blank"> cloudflare's website </a> and sign up for a free account. We will be coming back to this dashboard later.

2. After creating an account look for the blue **add site** button. Enter your sites domain from above and continue with the setup steps. Skip the add dns records step. (We will come back to this at the end).

3. The next page should give you Cloudflare's nameservers. Copy these and head back to your Freenom page. On this page click  
**Manage Domain > Management Tools > Nameservers**. 

4. Select the option to ***Use custom nameservers*** and enter Cloudflare's nameservers and confirm by clicking *Change Nameservers* at the bottom of the page. 

5. After this, head back to your Cloudflare dashboard and click the **Check nameservers** button. This can either be instant or take some time. In my expereince it takes anywhere between 5-20 minutes. Once this is done you will be greeted with a different page.


Now we have our website setup, so we can head to our linux device to link our local website with the custom domain name. 




## Install Cloudflared on your device

The install script depends on the architecture of your system. Check what it is with the following command: 

```bash
uname -a
```

### arm64 architecture:
```bash
sudo wget -O cloudflared https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64
sudo mv cloudflared /usr/local/bin
sudo chmod +x /usr/local/bin/cloudflared
cloudflared -v
```

### AMD64 architecture:
```bash
sudo wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo apt-get install ./cloudflared-stable-linux-amd64.deb
cloudflared -v
```

### armhf architecture:
```bash
sudo wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-armhf.deb
sudo apt-get install ./cloudflared-linux-armhf.deb
cloudflared -v
```

Once clouflared has been installed login using your credentials. It will give you a url to paste into a browser for you to login. It will ask you to select the domain name you want to link. 

```bash
cloudflared login
```



## Create a Cloudflare Tunnel: 
This step will link your local website with the custom url. 


```bash 
cloudflared tunnel create <NAME>
```
<h6>Replace NAME with any name you'd like</h6>  

<br>

One the tunnel is created you will be presented with its unique uuid. Copy this for later use. 

___

Next we will need to create our config file.

```bash 
sudo nano ~/.cloudflared/config.yml
```
Now paste the following into the file
```bash
tunnel: <the UID you saved earlier>
credentials-file: /root/.cloudflared/<UID>.json

ingress:
  - hostname: mytunnel.ml 
    service: http://localhost:80 
  - service: http_status:404
```

{: .note }
>1. **UID** = The UID you saved earlier
>2. **Hostname** = Your custom url
>3. **Service** = localhost should stay the same >however the port number should change to the port you would use to connect to your service on your local network.  

## Create DNS recods to route tunnel traffic
At this part in the process you may decide you want to add more than one service to your new custom domain. You can add as many subdomains as you would like.  
(*Ex) test.mytunnel.ml, printer.mytunnel.ml, klipper.mytunnel.ml*)

### Single Service

```bash 
cloudflared tunnel route dns <The name of your tunnel> <url you are routing to> 
```

{: .example } 
>```bash
cloudflared tunnel route dns myklippertunnel kprinting.ml
```


### Multiple Services 
For every service you want to link you will need to make two additional entries to the ingress section of your config file. 

```bash
  - hostname: <subdomain>.mytunnel.ml 
    service: <address of local server>
```

{: .example }
>```bash
>ingress:
>  - hostname: test.mytunnel.ml 
>    service: http://localhost:80 
>  - hostname: klipper.mytunnel.ml 
>    service: http://localhost:5000
>  - service: http_status:404
>```  

For each subdomain you will have to run the following command to make the connection. 

```bash
cloudflared tunnel route dns <The name of your tunnel> <url you are routing to> 
```

{: .example }
>```bash
cloudflared tunnel route dns myklippertunnel klipper.mytunnel.ml
cloudflared tunnel route dns myklippertunnel test.mytunnel.ml
```


## Run the Tunnel as a service: 
If you don't want to start the tunnel everytime your reboot your device with the following command: 
```bash 
 cloudflared tunnel --config /root/.cloudflared/config.yml run <NAME of your Tunnel> or <UUID>
```  

Then you can create a service so it will restart automatically for you with the following steps: 
1. First move the config file from ```.cloudflared``` to a place where linux can use it at boot. 
> ```sudo mv /home/p2/.cloudflared/config.yml /etc/cloudflared/ ```
2. Now start the service
> ```sudo cloudflared service install```

{: .note}
> You can check the status of the tunnel with:
```sudo systemctl status cloudflared```  
>You can restart the service with:
```sudo systemctl restart cloudflared```


