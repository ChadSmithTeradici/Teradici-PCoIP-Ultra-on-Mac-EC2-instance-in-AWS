---
title: Install Teradici PCoIP on AWS  EC2 Mac instance
description: Install guide for Teradici PCoIP on AWS Mac Instance. This is outside of an AWS marketplace offering.
author: chad-m-smith
tags: Teradici, AWS, Mac, EC2
date_published: 2021-10-20
---

Chad Smith | Technical Alliance Architect at Teradici | HP

<p style="background-color:#CAFACA;"><i>Contributed by Teradici employees.</i></p>

This guide shows you how to install Teradici PCoIP agent on a Mac Instance running in AWS. Also this guide is inended for customers that have Teradici annual subcription and are interested in transfering licensed seats to a AWS EC2 MAC instance(s). There is an alternative option for a AWS marketplace hourly subscription which doesn't coincide with EC2 Mac instance 24hr minimum allocation peroid, AWS marketplace offering is NOT apart of this deployment guide. 

EC2 Mac instances are available for purchase as Dedicated Hosts through On Demand and Savings Plans pricing models. Billing for EC2 Mac instances is per second with a 24-hour minimum allocation period to comply with the Apple macOS Software License Agreement. Through On Demand, you can launch an EC2 Mac host and be up and running within minutes. At the end of the 24-hour minimum allocation period, the host can be released at any time without further commitment. 

More Information on EC2 MAC Instance can be found [here](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-mac-instances.html).

## Objectives

+ Allocate a AWS EC2 Mac Instance from AWS Console.
+ Configure Security Groups to allows access to instance (SSH,VNC & PCoIP ports).
+ Install supporting software and configure security parameters within Mac OS.
+ Connect to EC2 Mac Instance via PCoIP client

## Costs

This tutorial uses billable components of AWS Cloud and assumes Teradici subscription, including the following:

+   [Teradici PCoIP](https://connect.teradici.com/contact-us), Teradici PCoIP subscriptions
+   [AWS EC2 Mac Instance](https://aws.amazon.com/ec2/instance-types/mac/), including vCPUs, memory, disk, and GPUs as a dedicated host.
+   [Internet egress and transfer costs](https://aws.amazon.com/blogs/architecture/overview-of-data-transfer-costs-for-common-architectures/), for PCoIP and other applications communications

Use the [AWS pricing calculator](https://calculator.aws/#/) to generate a cost estimate based on your projected usage.

## Before you begin

In this section, you set up some basic resources that the tutorial depends on.

1. Instructions in this guide assume that you have a [AWS account](https://aws.amazon.com/free/) 

1. Ensure you have [Service Quotas](https://console.aws.amazon.com/servicequotas) for **'Running Dedicated mac1 Hosts'**.

1. Familiarize yourself with [AWS network](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Networking.html) topology and best practices.

1. Obtain a [Teradici PCoIP registation](https://connect.teradici.com/contact-us) code that has at least one un-assigned seat available.

1. Have a [Teradici registerd login](https://help.teradici.com/s/login/SelfRegister) credentials in order to obtain **Graphics Agent for macOS**. 

## Set up the virtual workstation

In this section, you create and configure a virtual workstation, including setting up networking and installing utilities. 

### Procure the EC2 Mac Instance

In this section, you procure a mac1 type dedicated host in your region

1. Select a AWS region that has [EC2 Mac Instances available](https://aws.amazon.com/ec2/instance-types/mac/) with a understanding of hourly consumption rate.

1.  Allocate a Mac Dedicated Host within the [EC2 Dashboard](https://console.aws.amazon.com/ec2). Choose **Dedicated Hosts**, then choose **Allocate Dedicated Host**.
    
1.  On the **Allocate Dedicated Host** page, make the following selections:
    + For **Name tag**, type EC2 Mac Dedicated Host
    + For **Instance family**, choose **mac1**
    + For **Support multiple instance** types, clear the **Enable** check box
    + For **Instance type**, choose **mac1.metal**.
    + For **Availability Zone**, choose any zone in your Region.

    Keep the remaining default selections and choose **Allocate**.
    
1. Once allocated, the Dedicated Host appears with a status of **Available**. 

1.  Launch a mac1.metal instance, On the [EC2 Dashboard](https://console.aws.amazon.com/ec2), choose **Launch Instance**.

1. On the **Choose AMI** page, select the **macOS Catalina (10.15.7)** or **macOS Big Sur (11.4)** AMI(s)

1. On the **Choose Instance Type** page, keep the default selection of **mac1.metal instance** and choose **Next: Configure Instance Details**.

1. On the **Configure Instance Details** page, for **Host**, choose the **Dedicated Host** you allocated earlier. For the remaining configuration details, make any selections you prefer. Then, choose **Next: Add Storage**.

1. On the **Add Storage** page, choose the Size (GiB) cell and increase the volume based on your requirements. Then, choose **Next: Add Tags**.

1. On the **Add Tags page**, optionally add any Key:Value tags to your instance. Then, choose **Next: Configure Security Group**.

1. On the Configure Security Group page, make the following selections:

    + For **Assign a security group**, choose **Create a new security group**.
    + For **Security group name**, type a descriptive name, such as *pcoip ssh into mac1.metal*.
    + For **Description**, optionally add a description.
    + For **Type**, choose **SSH**
    + For **Source**, choose **My IP**
    + Select **Add rule**
    + For **Type**, choose **Custom TCP Rule**
    + For **Port Range** choose **HTTPS**
    + For **Source**, choose **0.0.0.0/0**
    + Select **Add rule**
    + For **Type**, choose **Custom TCP Rule**
    + For **Port Range** choose **4172**
    + For **Source**, choose **0.0.0.0/0**
    + Select **Add rule**
    + For **Type**, choose **Custom UDP Rule**
    + For **Port Range** choose **4172**
    + For **Source**, choose **0.0.0.0/0**
    + Select **Add rule**
    + For **Type**, choose **Custom TCP Rule**
    + For **Port Range** choose **5900**
    + For **Source**, choose **My IP**
    +  Select **Add rule**
    + For **Type**, choose **Custom TCP Rule**
    + For **Port Range** choose **5800**
    + For **Source**, choose **My IP**

    Then, choose **Review** and **Launch**.

1. On the **Review page**, review your selections and verify that the **Host ID** matches the Dedicated Host you created earlier. Then, choose **Launch**.

1. On the **Select an existing key pair or create a new key pair** dialog, verify your existing key pair (if you do not have a key pair, select the option to create a new key pair). Then, select the acknowlegement check box and choose **Launch Instances**.

1. On the **Instances** page, wait for the **Status Check** column of your instance to show 2/2 checks passed before continuing.

## Set up the connection to your EC2 Mac Instance

In this section, you will establish a connection to your instance using SSH, to install VNC temporary GUI access to the Mac. (some Teradici prerequisites configurations that can only be accomplished within the Mac GUI) Finally, PCoIP will be installed and configured via VNC session into the Mac GUI.

1. On the **EC2 Dashboard**, select your **EC2 Mac Instance** and choose **Connect**.

1. On the **Connect to an instance** dialog, choose **SSH client**. Follow the instructions in the dialog for SSH client to connect to your mac1.metal instance

### Configure VNC and start listener

1. Run the following command in SSH session to install and start VNC (macOS screen sharing SSH) from the Mac instance:
        
        sudo defaults write /var/db/launchd.db/com.apple.launchd/overrides.plist com.apple.screensharing -dict Disabled -bool false
        sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.screensharing.plist
    
1. Run the following command to set a password for ec2-user:

        sudo /usr/bin/dscl . -passwd /Users/ec2-user
        
1. Create an SSH tunnel to the VNC port. In the following command, replace keypair_file with your SSH key path and x.x.x.x with your instance's IP address:

        ssh -i keypair_file -L 5900:localhost:5900 ec2-user@x.x.x.x
        
    Note: The SSH session should be running while you're in the remote session.
    
1. Using a VNC client, connect to instaceIP:5900

    Note: macOS has a built-in VNC client. For Windows, you can use RealVNC viewer for Windows. For Linux, you can use Remmina. Other clients, such as TightVNC         running on Windows don't work with this resolution.
    
### Install Teradici PCoIP Agent
Once a VNC connection has been established, you can install and configure Teradici PCoIP within the macOS GUI.

**Note**: Teradici PCoIP Agent for macOS Installation guide may change between releases, consult the [latest guides](https://docs.teradici.com/find/product/cloud-access-software) before continuing.

+ **Firewall** - Becuase the AWS EC2 Mac Instance has a security group assoicated and configured, it is recommended to disable it (firewall is off)
+ **Energy Saver** - Energy Saver features can cause the remote system to go to sleep or become unresponsive. To prevent this, open **System Preferences > Energy Saver**, and configure the settings as follows:

     ![image](https://github.com/ChadSmithTeradici/TeradiciPCoIPonMACinAWS/raw/main/images/Energy_saving.jpg)
     
+ **Create a user account for PCoIP Connections** - The user name cannot contain spaces, and cannot be the root user account (the root user is an administrative account with elevated permissions, and is disabled by default in macOS).

#### Downloaing, installing, registering PCoIP
1.  [Download the agent installer](https://docs.teradici.com/find/product/cloud-access-software/current/graphics-agent-for-macos/21.07) to the machine you'll be using as the PCoIP host. You will need a Teradici registed login to gain access.

1. Run the `pcoip-agent-graphics_21.07.4.pkg`.

1. Click through the installer steps and accept the End User License Agreement. The agent application will be installed.

    ![image](https://github.com/ChadSmithTeradici/TeradiciPCoIPonMACinAWS/blob/main/images/PCoIP-Intro-1.jpg)
    
    ![image](https://github.com/ChadSmithTeradici/TeradiciPCoIPonMACinAWS/blob/main/images/PCoIP-ELUA-2.jpg)
    
    ![image](https://github.com/ChadSmithTeradici/TeradiciPCoIPonMACinAWS/blob/main/images/PCoIP-EULA-Agree-3.jpg)
    
    ![image](https://github.com/ChadSmithTeradici/TeradiciPCoIPonMACinAWS/blob/main/images/PCoIP-Dest-4.jpg)
    
    ![image](https://github.com/ChadSmithTeradici/TeradiciPCoIPonMACinAWS/blob/main/images/PCoIP-Perm-5.jpg)
    
    ![image](https://github.com/ChadSmithTeradici/TeradiciPCoIPonMACinAWS/blob/main/images/PCoIP-Trash-6.jpg)
    
1. Open the PCoIP Agent application, found in:

        /System/Volumes/Data/Applications/PCoIP Agent.app

1. You will be prompted to allow **Accessibility** permission. Grant and confirm privacy permissions for Accessibility.

1. Open the PCoIP Agent application again.

1. You will be prompted to allow **Screen Recording** permission. Grant and confirm privacy permissions for Screen Recording.

1. Next, provide the license registration code you received from Teradici.

    open a terminal window and type the following command:
    
        sudo pcoip-register-host --registration-code={REGISTRATION_CODE}
        
    The Graphics Agent for macOS must be assigned a valid PCoIP session license before it will work. Until you've registered it, you can't connect to the desktop using a PCoIP client. You receive a registration code when you purchase a pool of licenses from Teradici. Each registration code can be used multiple times; each use consumes one license in its pool.
    
      Registration codes look like this: ABCDEFGH12@AB12-C345-D67E-89FG
        
1. **Restart** the EC2 Instance


## Install PCoIP Client and connect to EC2 Mac Instance
In this section, you will establish a connection to your instance using PCoIP. You will need to install a PCoIP client on your client system that will be used to initiate the session to the EC2 Mac Instance in AWS. Depending on your network topology, use will either connect to the local IP (or) ephemeral/elastic Public IP (or) Fully Qualified Domain Names (FQDN)

1. [Download the client installer](https://docs.teradici.com/find/product/software-and-mobile-clients) based on your client OS. You don't need a login credentials to download client software and can have as many copys of various client OS as you need.

1. Install the PCoIP client software per the OSs Administration Guides installation instructions.

1. Locate the **IP address** or **FQDN** of the AWS EC2 Mac Instance via the [EC2 Dashboard](https://console.aws.amazon.com/ec2)

1. Identify the Mac Instance within the list of **Running Instances** in the EC2 Dashboard, check the **box** near the instance name, if it was named.

1. Under the **Details** tab you will see **Public IPv4 Address** (or) **Private IPv4 Address** (or) **Private IPv4 DNS** (or) **Public IPv4 DNS**

1. From the client system, start your PCoIP client per OS. Typically the PCoIP client will have a icon:

    ![image](https://github.com/ChadSmithTeradici/TeradiciPCoIPonMACinAWS/blob/main/images/PCoIP-icon.jpg)

1. When the PCoIP client starts, it will ask for a **Host Address or Code**. Enter in your **IP address or FQDN** previously identified in previous section. (optionally) enter a name to **Connection Name** field then **SAVE**, if you want to save connection.

    ![image](https://github.com/ChadSmithTeradici/TeradiciPCoIPonMACinAWS/blob/main/images/PCoIP-Client.jpg)
    
1. Next, you will get a Cannot verify your connection to IP warning. This error is becuase a 3rd party trusted certificate has not been install on the host. You can select the **Connect Insecurely** option.
    
    ![image](https://github.com/ChadSmithTeradici/TeradiciPCoIPonMACinAWS/blob/main/images/PCoIP-Trusted.jpg)
    
1. Finally, enter in the macOS login credentials(**ec2-user**, if not changed)that you used easier in your VNC session to log into the instance.

    ![image](https://github.com/ChadSmithTeradici/TeradiciPCoIPonMACinAWS/blob/main/images/PCoIP-Auth.jpg)


## Clean up

To avoid incurring charges to your AWS account for the resources used in this tutorial, you can simply delete the instance:

1.  In the [EC2 Dashboard](https://console.aws.amazon.com/ec2) , go to the Mac **Instance State** scroll to **Terminate**
1.  You can repurpose PCoIP floating seat, allow up to 24hrs for Teradici Cloud Licensing server to flush assoication to EC2 Mac Instance.
1.  Note: if you delete instance before 24hr peroid. AWS will still charge you the remaining hours until 24hr peroid expires

## What's next

+   [Configure and optimize](https://www.teradici.com/web-help/pcoip_agent/graphics_agent/macos/21.07/admin-guide/configuring/configuring/) the PCoIP expereince on the EC2 Mac Instance. 
+   Learn more about [Teradici](https://www.teradici.com/) products and offerings.
+   Learn more about [AWS EC2 Mac Instances](https://www.youtube.com/watch?v=d0FulqrjHkk&ab_channel=AmazonWebService)
