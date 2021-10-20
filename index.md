---
title: Install Teradici PCoIP on AWS  EC2 Mac instance
description: Install guide for installing Teradici PCoIP on AWS Mac Instance, this is outside of AWS marketplace offering.
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

### Procure the Mac workstation

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

### Set up the connection to your workstation

In this section, you will establish a connection to your instance using SSH, to install VNC temporary GUI access to the Mac. (some Teradici prerequisites configurations that can only be accomplished within the Mac GUI) Finally, PCoIP will be installed and configured via VNC session into the Mac GUI.

1. On the **EC2 Dashboard**, select your **EC2 Mac Instance** and choose **Connect**.

1. On the **Connect to an instance** dialog, choose **SSH client**. Follow the instructions in the dialog for SSH client to connect to your mac1.metal instance

#### Configure VNC and start listener

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
    
#### Install Teradici PCoIP Agent
Note: Teradici PCoIP Agent for macOS Installation guide may change between releases. It is always best to consult the [latest deloyment guides](https://docs.teradici.com/find/product/cloud-access-software).



## Install Steam and SteamVR

In this section, you download and install [Steam](https://store.steampowered.com/) and [SteamVR](https://store.steampowered.com/steamvr). Both are free to 
download and install.

1.  On your virtual workstation, download, install, and launch [Steam](https://store.steampowered.com/about/).
1.  To install SteamVR, right-click the Steam icon on the taskbar and select **SteamVR**. The application will download and install.
1.  Launch SteamVR to initialize the software.
1.  Quit both Steam and SteamVR.

## Install the CloudXR Server

1.  On your virtual workstation, download and install [NVIDIA CloudXR SDK](https://developer.nvidia.com/nvidia-cloudxr-sdk).
1.  Extract the downloaded archive and run `CloudXR-Setup.exe`, located under the subdirectory `Installer`.

    If you are prompted with a "Windows protected your PC" warning, click **More Info**, and then click **Run anyway**.

1.  When prompted, choose components for the server installation only:

    1.  Select **CloudXR Server**.
    1.  Deselect **CloudXR Client Program**.
    1.  Ensure that the **Redistributables** checkbox is selected (required only for a first-time installation).

## Install the Android Studio SDK

To load files onto your HMD, you use the [Android Debug Bridge (ADB)](https://developer.android.com/studio/command-line/adb), which is part of the
[Android Studio SDK](https://developer.android.com/studio).

1.  On your local workstation, download and install the [Android Studio SDK Platform Tools](https://developer.android.com/studio/releases/platform-tools)
    for your operating system.

## Connect your workstation to your HMD

1.  On your local workstation, download and extract **_but do not install_** the [NVIDIA CloudXR SDK](https://developer.nvidia.com/nvidia-cloudxr-sdk).
1.  If required, enable Developer Mode on your device (typically only required on the
    [Oculus Quest 2](https://developer.oculus.com/documentation/native/android/mobile-device-setup#enable-developer-mode)).
1.  Connect your HMD to your workstation using the appropriate cable (typically USB 3.0).

    You can also [connect over WiFi](https://developer.android.com/studio/command-line/adb#connect-to-a-device-over-wi-fi-android-11+).
    
1.  If prompted, select **Allow USB Debugging** on the HMD.
1.  Verify that your HMD is connected by running the following in a command shell (terminal on Linux or Mac OS, PowerShell on Windows):  
  
        adb devices -l
  
    You should see your HMD listed, along with the status of the device, as in the following example output:

        List of devices attached
        1WMHHXXXXDXXXX         device usb:1-4.3 product:hollywood model:Quest_2 device:hollywood transport_id:1

    If your HMD isn't listed, check your cable connections, verify that your USB port is USB 3.0, and repeat the steps in this section.

## Install the sample application

In this section, you install the sample application on your HMD. Sample apps for all supported HMDs are provided with the CloudXR SDK.

**Note:** This section covers client installation only for the Oculus Quest 2. For other HMDs, see the
[NVIDIA CloudXR documentation](https://docs.nvidia.com/cloudxr-sdk/index.html).

1.  In a command shell on your local workstation, go to where you extracted the NVIDIA CloudXR SDK, and then go to the subdirectory that contains the sample app
    for the Oculus Quest 2: `Sample/Android/OculusVR`.

1.  Install the sample application on your Oculus Quest 2:  
  
        adb install -r ovr-sample.apk
  
    On the Oculus Quest 2, the CloudXR Client is located under **Apps > Unknown Sources**.

## Install the configuration file

1.  On your local workstation, create a plain-text file named `CloudXRLaunchOptions.txt` containing the following:  
  
        -s [VM-EXTERNAL-IP]

    Replace `[VM-EXTERNAL-IP]` with the external IP address of your virtual workstation. You can find the external IP address of your VM using the
    [Cloud Console](https://cloud.google.com/compute/docs/instances/view-ip-address#console), the
    [`gcloud` command-line tool](https://cloud.google.com/compute/docs/instances/view-ip-address#gcloud), or the
    [API](https://cloud.google.com/compute/docs/instances/view-ip-address#api).  
  
1.  Save the file on your local workstation.
1.  In a terminal on your local workstation, load the configuration file onto your Oculus Quest 2:  

        adb push CloudXRLaunchOptions.txt /sdcard/CloudXRLaunchOptions.txt

1.  Disconnect the cable from the HMD.

## Launch the CloudXR client

Connect to your virtual workstation using the CloudXR client on your HMD.

1.  On your local workstation, connect to your virtual workstation using a VNC client.
1.  Launch SteamVR.

    You will see the message *Headset Not Detected*:  

    ![image](https://storage.googleapis.com/gcp-community/tutorials/streaming-vr-content-from-a-virtual-workstation-using-nvidia-cloudxr/headset-not-detected.png)

1.  On your HMD, start the CloudXR Client.

    The first time you launch the application, you are prompted on your HMD to grant permissions to allow access.
    
1.  Follow the prompts to allow access.

When the connection is established, the SteamVR app shows the connection status of your HMD and controllers:

![image](https://storage.googleapis.com/gcp-community/tutorials/streaming-vr-content-from-a-virtual-workstation-using-nvidia-cloudxr/headset-connected.png)

The default CloudXR environment appears in your HMD display. You can preview this display on your virtual workstation by right-clicking the
SteamVR icon on the taskbar and selecting **Display VR View**:

![image](https://storage.googleapis.com/gcp-community/tutorials/streaming-vr-content-from-a-virtual-workstation-using-nvidia-cloudxr/display-vr-view.png)

A window will open showing a preview of the CloudXR environment:

![image](https://storage.googleapis.com/gcp-community/tutorials/streaming-vr-content-from-a-virtual-workstation-using-nvidia-cloudxr/cloudxr-env.png)

You can now launch SteamVR games or experiences on your virtual workstation, where they will be streamed to your HMD. Any application that uses the
[OpenVR SDK](https://en.wikipedia.org/wiki/OpenVR) will play over CloudXR, even ones not launched from SteamVR.

## Clean up

To avoid incurring charges to your Google Cloud account for the resources used in this tutorial, you can delete the project:

1.  In the Cloud Console, go to the [Projects page](https://console.cloud.google.com/iam-admin/projects).
1.  In the project list, select the project you want to delete and click **Delete**.
1.  In the dialog, type the project ID, and then click **Shut down** to delete the project.

## What's next

+   Install SteamVR games and experiences and play them through your HMD.
+   Learn more about [NVIDIA CloudXR SDK](https://developer.nvidia.com/nvidia-cloudxr-sdk).
+   Read the [CloudXR documentation](https://docs.nvidia.com/cloudxr-sdk/index.html).
+   Learn more about [creating a virtual workstation](https://cloud.google.com/architecture/creating-a-virtual-workstation).
