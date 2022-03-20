---
title: "Starting with Active Directory and Bloodhound"
author:
  name: Miguel Guerrero
  link: https://mikemighthack.me
date: 2022-03-20 20:00:00 +0105
categories: [Tutorials, AD, BloodHound]
tags: [BloodHound, AD, FakeAd]
math: true
mermaid: true
---

Hi all! 
In this post, I want to show you how to mount a simulated Active Directory and how to install and use the BloodHound tool.


## Requirements
 - A Windows machine.
 - Virtualization software (in this case, VMware)
 - A Kali linux machine.


## BloodHound set-up

BloodHound is a data analysis tool which uses graph theory to reveal the hidden and often unintended relationships within an Active Directory environment.  Attackers can use BloodHound to easily identify highly complex attack paths that would otherwise be impossible to quickly identify. Defenders can use BloodHound to identify and eliminate those same attack paths. Both blue and red teams can use BloodHound to easily gain a deeper understanding of privilege relationships in an Active Directory environment.

Before installing BloodHound, we need to install Neo4j. In my case, I will do it on my windows machine, but also you can do it on macOS or Linux. 

Steps for install:
1.  Download the Windows installer for Oracle JDK 11 from [https://www.oracle.com/java/technologies/javase-jdk11-downloads.html](https://www.oracle.com/java/technologies/javase-jdk11-downloads.html)
2.  Use the installer to install Oracle JDK. The default options work fine.
3.  Download the neo4j Community Server Edition zip from [https://neo4j.com/download-center/#community](https://neo4j.com/download-center/#community)
4.  Unzip the neo4j zip file.
5.  Open a command prompt, running as administrator. Change directory to the unzipped neo4j folder.
6.  Change directory to the bin directory in the Neo4j folder.
7.  Run the following command:
```shell
C:\> neo4j.bat install-service
```
8.  Open a web browser and navigate to [http://localhost:7474/](http://localhost:7474/). You should see the neo4j web console.
9.  Authenticate to neo4j in the web console with username neo4j, password neo4j. You’ll be prompted to change this password.
10.  Now, we can install BloodHound GUI from [https://github.com/BloodHoundAD/BloodHound/releases](https://github.com/BloodHoundAD/BloodHound/releases)
11.  Unzip the folder and double click BloodHound.exe
12. Authenticate with the credentials you set up for neo4j

 if you need more details about the installation or if you are using another OS, [Here](https://bloodhound.readthedocs.io/en/latest/installation/windows.html) you have the oficial installation guide. 


## Build a simulated AD

### Download an image of windows server and build a virtual machine. 

In this post, I will use Windows server 2016. Click [here](https://isofiles.bd581e55.workers.dev/Windows%20Server/Windows%20Server%202016/) to download the ISO image.

Then, build a virtual machine with the downloaded ISO file.


### Configuring the AD

You can follow the next steps to build your AD:

1. Firstly, I will change the name of the machine (System -> System Properties -> Computer Name):
![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\BloodHoundAD\20220317135031.png){: .align-center}

2. In order to see the name changed, we need no restart the computer

3. Now click on add roles and features and click on next:
![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\BloodHoundAD\20220317152946.png){: .align-center}

4. Click next again.
![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\BloodHoundAD\20220317153039.png){: .align-center}

5. Select a server from the server pool, select you computer and click next.
![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\BloodHoundAD\20220317153120.png){: .align-center}

6. Click next.
![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\BloodHoundAD\20220317153205.png){: .align-center}

7. In Features, select Active Directory Domain services and next.
![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\BloodHoundAD\20220317153321.png){: .align-center}

8. Next.
![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\BloodHoundAD\20220317153416.png){: .align-center}

9. Next.
![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\BloodHoundAD\20220317153445.png){: .align-center}

10. Click on install and wait.

11. Now, promote this server to domain controller:
![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\BloodHoundAD\20220317153953.png){: .align-center}

11. a new pop-up is open, select add new forest and write your domain name (ins this case Mylab.Internal).
![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\BloodHoundAD\20220317160036.png){: .align-center}

12. Now choose a new password for DSRM
![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\BloodHoundAD\20220317154912.png){: .align-center}

13. In DNS options, click next.
![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\BloodHoundAD\20220317155000.png){: .align-center}

14. Choose a NetBios name (leave the default name that appears):
![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\BloodHoundAD\20220317155203.png){: .align-center}

15. Leave the default paths:
![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\BloodHoundAD\20220317155234.png){: .align-center}

16. Click next on review options. Now check if all prerequisites are met. (You need to set a password for the Administrator account in order to meet all prerequisites).
![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\BloodHoundAD\20220317160309.png){: .align-center}

17. The server will be restarted at the end of the installation process. 


### Populate the AD
Now we are going to populate the AD with [BadBlood](https://github.com/davidprowe/BadBlood).
BadBlood fills a Microsoft Active Directory Domain with a structure and thousands of objects. The output of the tool is a domain similar to a domain in the real world. After BadBlood is ran on a domain, security analysts and engineers can practice using tools to gain an understanding and prescribe to securing Active Directory. Each time this tool runs, it produces different results. The domain, users, groups, computers and permissions are different.

To run BadBlood, clone the repository and run the following command:
```shell
./badblood/invoke-badblood.ps1
```

Then , we can see an output like this one:
![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\BloodHoundAD\20220318140046.png){: .align-center}


## BloodHound
Now is time to extract all this information for bloodhound. We will use [BloodHound.py](https://github.com/fox-it/BloodHound.py), from our Kali. We can install it using pip:
```shell
pip install bloodhound
```

To use the ingestor, at a minimum we will need credentials of the domain we are logging in to. You will need to specify the `-u` option with a username of this domain. In this case we have an administrator user of the domain controller, so we will see al data of the AD.

To run the ingestor: 
```shell
bloodhound-python -u micky -p Test1234 -ns 192.168.164.130 -d Mylab.Internal -c all
```

Here you can see the output:
![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\BloodHoundAD\20220319193140.png){: .align-center}

After the execution of the ingestor, we will see the following output files:
![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\BloodHoundAD\20220319193248.png){: .align-center}

Now is time to use these files to feed our bloodhound.

### Import the Database
We can import the json files, using the import data button:
![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\BloodHoundAD\20220319193754.png){: .align-center}

After the import, we will see the following window that shows the import progress:
![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\BloodHoundAD\20220319194002.png){: .align-center}

Now we can see the Database updated:
![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\BloodHoundAD\20220319194113.png){: .align-center}

In the menu showed above, we can see three tabs. The first one, shows information about the Database, how many users we have, groups, computers, etc. The second one, shows the information of a node. And finally we have the Analysis tab, where we can make requests to database to see relevant information about the AD.

Let's see one interesting query we can do if we are trying to get access to a domain controller.
![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\BloodHoundAD\20220319195530.png){: .align-center}

The image above shows us the users who are domain administrators. If we wanted to have control of the AD, we could, for example, try to find the credentials of these users.

If we select a node of the graph:
![image-center]({{ site.url }}{{ site.baseurl }}\assets\images\tutorials\BloodHoundAD\20220319195941.png){: .align-center}

We can see the information related to that user in the second tab of the menu. Here, it is possible to see the last logon of the user, if it is enabled, if it has admin rights etc. All that kind of information is very useful. 

Feel free to play with this database and try to find paths or misconfigurations, in order to gain access to the domain controller. 

Bloodhound is a very useful tool, which will help us in our red team exercises, to find the most feasible way to achieve our goals. 

This blog tries to explain how to set up a fake Active Directory, and how to install and start using bloodhound in our exercises.

In future blogs, I hope to write about the main vulnerabilities and their exploits and how to identified them using BloodHound.

I hope to see you soon,
See ya!