# Example Lab Setup - Creating a Windows 10 Virtual Machine with Windows Hyper-V
If you do not have a physical machine that can be used for testing the BlokSec Windows Credential Provider, you can test with a virtual machine. We have used Windows Hyper-V to create our lab for developing and local testing of the BlokSec Windows passwordless logon capability.

>  **Warning**: When using Windows Hyper-V, the IP address of the virtual switch created by Hyper-V will change each time you reboot the host machine. This will cause the IPs of the guest VMs to also change and interrupt the client workstation's ability to communicate with the DC. Ensure you update the static DNS IP used by the client workstation to point to the new DC IP each time the host machine is rebooted.

## Create the Virtual Machine in Hyper-V

1. Start the **New Virtual Machine Wizard** in Windows Hyper-V

![image](https://user-images.githubusercontent.com/1026425/205500195-ccc4a04c-3020-41c9-8e04-d5395092c699.png)

2. Give the virtual machine a name and select a location

![image](https://user-images.githubusercontent.com/1026425/205500246-7ef63379-6352-47b8-adac-0a6899f6ea1c.png)

3. Ensure you select **Generation 2** for the virtual machine

![image](https://user-images.githubusercontent.com/1026425/205500273-f468c85d-fdd9-4cc9-96f0-781301055649.png)

4. Assign memory to the virtual machine

![image](https://user-images.githubusercontent.com/1026425/205500302-5872b561-dcd9-4722-83ab-6ef9824358b5.png)

5. Ensure the virtual machine has access to the network or switch where the AD domain controller can be contacted

![image](https://user-images.githubusercontent.com/1026425/205500350-1790461a-09fe-482d-b2ea-637171c4a41d.png)

6. Create or connect a virtual hard disk *note that the author tried to skimp on disk space, selecting only 12GB and then promptly ran out of disk space; 15GB+ is recommended*

![image](https://user-images.githubusercontent.com/1026425/205500428-d7fb226e-2ac3-4b9f-af47-d0386ed28fac.png)

7. Select the location of your installation media

![image](https://user-images.githubusercontent.com/1026425/205500450-b101fd65-85c9-4d99-a55d-2ffb48d04409.png)

8. Review your settings the click **Finish** to complete creation of the virutal machine

![image](https://user-images.githubusercontent.com/1026425/205500470-b1b004a8-620a-4131-84e3-44366d5a0382.png)

## Install Windows 10

1. Start the virtual machine and install Windows (the installation process is not covered in this document)

![image](https://user-images.githubusercontent.com/1026425/205500904-5ddcd37f-0034-4880-9f5d-f9a58bd30581.png)

2. Once in account setup, choose **Set up for an organization** then instead of signing in with Microsoft, choose **Domain join instead**

![image](https://user-images.githubusercontent.com/1026425/205501920-cb090421-b60f-4958-b7f3-287eaf959b4f.png)

![image](https://user-images.githubusercontent.com/1026425/205501848-c5ba3e39-7f60-4dec-b9d7-a31b52876a63.png)

3. Provide an account name, create a password, and configure PVQs

![image](https://user-images.githubusercontent.com/1026425/205501941-0fdd3344-5de1-485d-aade-dd2a74c819e7.png)

4. Finish the installation by selecting your preferences for Microsoft services (for our lab, we select the negative option for each)
5. Shut down the virtual machine to allow for hardware settings changes

## Configure Hyper-V to allow access to the virtual TPM
1. In Hyper-V, click **File** > **Settings** (note VM must be powered-off)

![image](https://user-images.githubusercontent.com/1026425/205502244-26d6249c-9401-4f75-b35a-e1ed4830f6c6.png)

2. Under **Settings** check the checkbox for **Enable Trusted Platform Module** then click **Apply** and **OK**

![image](https://user-images.githubusercontent.com/1026425/205502302-b937c020-3b50-4ab1-986f-b89559d172ba.png)

3. Start the virtual machine

## Disable Enhanced Mode for Hyper-V
In our testing we discovered that Hyper-V's **Enhanced Mode** causes the guest operating system to see the user as connecting via RDP rather than being physically present at the machine (i.e., console connection) and this prohibits the ability to sign in with a smart card. Therefore we will disable **Enhanced Mode** in Hyper-V for our lab 

1. In the main Hyper-V window, select your Hyper-V Manager then choose **Hyper-V Settings** in the **Actions** menu

![image](https://user-images.githubusercontent.com/1026425/205502771-ac694602-68aa-408c-80bf-5bb81b2787fd.png)

2. Under **User** > **Enhanced Session Mode**, delesect the checkbox for **Use enhanced session mode** then click **Apply** and **OK**

![image](https://user-images.githubusercontent.com/1026425/205502878-644aedb0-e89b-4cff-b74b-5ffdcab45724.png)


## Join the computer to the domain
Reference - [Join a Computer to a Domain](https://learn.microsoft.com/en-us/windows-server/identity/ad-fs/deployment/join-a-computer-to-a-domain) from microsoft.com

> ☝ **Tip** in a lab environment where the DHCP server doesn't automatically associate the AD DS server as the DNS server, you will need to manually assign the DNS in the IPv4 settings. Here we are setting the primary DNS server to the IP of the AD DS / DNS server, and the secondary DNS to the IP of the default gateway (which can be found by executing `ipconfig` from the command line)
> 
> ![image](https://user-images.githubusercontent.com/1026425/205520486-3db58edb-20c9-44e2-a274-70d1207e2441.png)
> 
> Ensure you can `ping` or `nslookup` the domain
> 
> ![image](https://user-images.githubusercontent.com/1026425/205520590-46270817-d3ed-4a97-a54c-8f2f69ca4bc1.png)


1. Sign into the virtual machine with the administrative account created during installation
2. Click **Start** > **Settings**

![image](https://user-images.githubusercontent.com/1026425/205520146-42a3b933-338c-4505-aaae-999082d1e2ea.png)

3. Click **System**

![image](https://user-images.githubusercontent.com/1026425/205520210-b76bd786-99ad-412d-8f04-300405dd81bf.png)

4. Choose **About** then scroll down and click **Rename this PC (advanced)**

![image](https://user-images.githubusercontent.com/1026425/205520289-dc824332-93d1-4af9-ad76-d8a62d91e8bb.png)

5. Click **Change...* next to "To rename this computer or change its domain..."

![image](https://user-images.githubusercontent.com/1026425/205520343-be3e9c4e-bc6b-437d-b09a-fae3b73ec318.png)

6. Rename the computer if desired, then choose **Member of:** > **Domain** and enter your domain name

![image](https://user-images.githubusercontent.com/1026425/205520689-0dbb3efe-8641-42c7-85cc-62260bedcd1a.png)

7. Provide administrative credentials to join the domain

![image](https://user-images.githubusercontent.com/1026425/205520758-bfea5385-d14c-41d2-b2aa-b3b5c9bb84e9.png)

8. If successful, you will receive a confirmation message

![image](https://user-images.githubusercontent.com/1026425/205520777-cad1ca73-d287-48a7-aab4-533522a8bdba.png)

9. Restart the computer to complete joining the domain

*The virtual machine is now ready to have the BlokSec Windows Credential Provider installed*

