---
title: "Router Space HTB Walkthrough "
author:
  name: Miguel Guerrero
  link: https://mikemighthack.me
date: 2023-06-09 11:33:00 +0105
categories: [HTB, Walkthrough]
tags: [apk, command injection, CVE-2021-3156]
math: true
mermaid: true
image:
  src: /assets/images/HTB/RouterSpace/routerspace.png
  width: 800
  height: 500
---

## Initial Foothold

We start with a basic nmap. We see that we have port 22 (ssh) and 80 (http) open.
![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\HTB\RouterSpace\20220309160318.png){: .align-center}


On port 80, we find the following web application:
![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\HTB\RouterSpace\20220309160409.png){: .align-center}


The download button will download an apk file.

If we install this program in an emulator, in our case anbox, we can see the following application:

![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\HTB\RouterSpace\20220309160633.png){: .align-center}


To see if the application makes any http requests, we can set burp as a proxy and intercept the requests.

To do this, we can configure the burp as follows:
![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\HTB\RouterSpace\20220309160837.png){: .align-center}

  
In the address field, we use the IP of our HTB vpn, in my case the tun0 interface IP address. Using adb, we configure the anbox emulator proxy:

```sh
adb shell settings put global http_proxy <IP>:8080
```

Once configured, we will see the following request made by the application:

![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\HTB\RouterSpace\20220309161100.png){: .align-center}

We send this request to the repeater.
If we try to launch it again, it will give us the following error:

**Unknown host: routerspace.htb**

To fix this, we add **routerspace.htb** to **/etc/host**. Once added, the request will work for us.

If we modify the IP parameter, to try to inject a command, we see in the response that it is simply reflected:

![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\HTB\RouterSpace\20220309161318.png){: .align-center}

But if we try to bypass the filter with the following characters, for example: ``;, |, \n,`` we see in the response the result of the command.

![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\HTB\RouterSpace\20220309161526.png){: .align-center}

If we try to send ourselves a reverse shell, it will not reach us. Surely there is some rule in the firewall.
  
## Gain access

Remember that we had an ssh service running on the victim machine.
Check if the /home/paul/.ssh/authorized_keys file exists:

![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\HTB\RouterSpace\20220309161751.png){: .align-center}

We can generate an rsa key for ssh and write the public key to the authorized_keys file to gain access to the server via ssh.

To generate the keys:

```sh
ssh-keygen -t rsa
```

To upload the public key:

```sh
; echo 'public key'
```

![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\HTB\RouterSpace\20220309162249.png){: .align-center}

Once uploaded, we can connect with the following command: 

```sh
ssh -i id_rsa paul@10.10.11.148
```

![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\HTB\RouterSpace\20220309163123.png){: .align-center}

In paul's home, we have the user.txt.

## Privilege Escalation

To escalate privileges, we will use [LinPEAS.sh](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS).

To transfer the file to the victim machine, we can do it through scp:

```sh
scp -i id_rsa ~/tools/PrivEsc/linux/linpeas.sh paul@10.10.11.148:.
```

Execute:

```sh
./linpeas.sh
```

In the output of the tool, ee see that it is vulnerable to CVE-2021-3156 (Sudo Baron Samedit) as is is shown in the image below:

![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\HTB\RouterSpace\20220309163418.png){: .align-center}

To exploit the vulnerability we will use the following exploit: 
- [Exploit (Worawit)](https://github.com/worawit/CVE-2021-3156/blob/main/exploit_nss.py)

Transfer the exploit using scp and execute it:

![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\HTB\RouterSpace\20220309163613.png){: .align-center}

And finally, we are root! 🔥