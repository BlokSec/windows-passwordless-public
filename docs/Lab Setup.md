# Example Lab Setup Instructions
To showcase the capabilities of BlokSec's passwordless credential provider for Microsoft Windows, you can follow these instructions to setup a lab. You will need two machines (physical or virtual) - one to act as the combined domain controller and certificate services server, and at least one workstation where you will install the BlokSec virtual smart card agent. These instructions were written and tested with Microsoft Server 2019 for the AD DS and ADCS server, and Windows 10 and Windows 11 for the passwordless workstation.

**Installation Overview**
1. [Install and configure the domain controller with Active Directory Directory Services (AD DS)](#install-and-configure-the-ad-domain-controller)
2. [Configure the domain controller with Active Directory Certificate Services (AD CS)](#configure-the-adcs-active-directory-certificate-services-role)
3. [Edit permissions on certificate templates](#edit-permissions-on-certificate-templates)
4. [Install certificate templates](#install-certificate-templates)
5. [Create group policy for "Do Not Display Last Logon"](#create-group-policy-for-do-not-display-last-logon)
6. [Install BlokSec virtual smart card provider on client workstation](#install-bloksec-virtual-smart-card-provider-on-client-workstation)
7. [Create and assign a virtual smart card to the user](#create-and-assign-a-virtual-smart-card-to-the-user)
8. [Example logon flow](#example-logon-flow)

## Install and configure the AD Domain Controller
At least one Active Directory Domain Services server is required to have an Active Directory domain. If you already have an AD DS (also knows as a Domain Controller or DC) then you can skip this part of the configuration. 

1. [Download](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2019) and install an evaluation version of Windows Server 2019. (We have used the .VHD version and are running it on [Windows Hyper-V](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v) on a Windows 10 Pro workstation)

2. Before adding the DC role, it is recommended to configure the server to have a static IP address [(sample instructions)](https://www.server-world.info/en/note?os=Windows_Server_2019&p=initial_conf&f=4) and add the localhost address (127.0.0.1) as the primary DNS

![image](https://user-images.githubusercontent.com/1026425/205295244-4719349b-4cb9-43dc-b020-baf544e69a8d.png)

3. In the Server Manager > Dashboard, click **Add roles and features**

![3zreyAbNtm](https://user-images.githubusercontent.com/1026425/205292642-80c277c1-c962-4338-82c5-4e5be158b9cd.png)

4. Choose **Role-based or feature-based installation**

![image](https://user-images.githubusercontent.com/1026425/205293055-45785c31-351d-4166-9860-d0fc2af15cec.png)

5. Select the current server

![image](https://user-images.githubusercontent.com/1026425/205293219-dffd9324-fb87-488e-a2ba-9bb1580dada5.png)

6. Check the box for the **Active Directory Domain Services** role

![image](https://user-images.githubusercontent.com/1026425/205293303-dc29bf0b-04ad-414a-a749-2fabb9f79369.png)

7. Click **Add Features**, then **Next**

![image](https://user-images.githubusercontent.com/1026425/205293390-a5dab539-a5de-4c7a-a5fe-6c45e6f80bc4.png)

8. Accept the default features by clicking **Next**

![image](https://user-images.githubusercontent.com/1026425/205293553-7098ceda-7edd-41c4-9277-c3cd21499637.png)

9. Acknowledge the notes by clicking **Next**

![image](https://user-images.githubusercontent.com/1026425/205293656-70b01b3d-b954-499d-b7c9-0682fdfe467e.png)

10. Click **Install** to proceed with the installation of the AD DC role

![image](https://user-images.githubusercontent.com/1026425/205293756-9f1096f6-d8ab-4dcb-9683-d19c102f0725.png)

*Allow the installation to complete*
11. Ensure the installation completes successfully

![image](https://user-images.githubusercontent.com/1026425/205294156-5c429cbe-d32f-468d-88e3-53fe62b7f186.png)

12. Start the DC configuration process by clicking the notification and selecting **Promote this server to a domain controller**

![image](https://user-images.githubusercontent.com/1026425/205294488-9609eeff-4c75-406c-9e7c-20a5f2e50bd1.png)

13. Select **Add a new forest** and specify a **Root domain name** (we are using `bloksec.local`)

![image](https://user-images.githubusercontent.com/1026425/205295863-3393e681-97a4-480f-a106-66a8d2753cce.png)

14. Leave the default Domain Controller Options, and provide a **Directory Services Restore Mode (DSRM)** password

![image](https://user-images.githubusercontent.com/1026425/205296163-e8b9ebe3-a429-4003-bf0f-3259d18bb2b8.png)

15. We skip DNS delegation in the DNS Options because this is a non-Internet domain

![image](https://user-images.githubusercontent.com/1026425/205296332-e1192ca5-1397-4bef-a5b3-f2c8c9d4e09d.png)

16. Provide a NetBIOS name for legacy connections

![image](https://user-images.githubusercontent.com/1026425/205296431-bf934280-bff3-4e60-bc13-1faa38964824.png)

17. Accept the default paths

![image](https://user-images.githubusercontent.com/1026425/205296569-cd2c759b-dad5-4d5e-99ff-abbf34770022.png)

18. Review the options then click **Next**

![image](https://user-images.githubusercontent.com/1026425/205296650-6f78e236-e430-4599-883d-251ac72db32a.png)

19. Ensure there are no errors (warnings may be ignored) then proceed with installation by clicking **Install**

![image](https://user-images.githubusercontent.com/1026425/205296851-45cc189b-e04e-4150-9502-bd0315c96c85.png)

*Installation will proceed and upon completion the server is automatically rebooted.* ***Note that Windows can take upwards of 5-10 minutes to complete applying settings after the reboot!***

20. Sign into the domain controller as **Administrator** and verify that the new role is showing up as expected

![image](https://user-images.githubusercontent.com/1026425/205298588-5276d643-beea-487e-b54c-ab3296daa4ba.png)

*Domain controller configuration is complete*


## Configure the ADCS (Active Directory Certificate Services) role
An Active Directory Certificate Services server provides customizable services for issuing and managing digital certificates used in software security systems that employ public key technologies. In our use case, the AD CS server is responsible for signing virtual smart cards and associating them with a user's identity in the domain. If you have an AD CS server in your domain you can use it for this purpose (skip to the **Edit permissions on certificate templates** section below) or you can configure a subordinate, dedicated AD CS server for smart card enrollment (instructions for this are outside the scope of these instructions)

1. In the Server Manager > Dashboard, click **Add roles and features**

![3zreyAbNtm](https://user-images.githubusercontent.com/1026425/205292642-80c277c1-c962-4338-82c5-4e5be158b9cd.png)

2. Choose **Role-based or feature-based installation**

![image](https://user-images.githubusercontent.com/1026425/205293055-45785c31-351d-4166-9860-d0fc2af15cec.png)

3. Select the current server

![image](https://user-images.githubusercontent.com/1026425/205293219-dffd9324-fb87-488e-a2ba-9bb1580dada5.png)

4. Check the box for the **Active Directory Certificate Services** role

![image](https://user-images.githubusercontent.com/1026425/205320516-6f7d91d4-623d-4dae-90fc-3ea26505a017.png)

5. Accept the default features by clicking **Add Features**, then click **Next** again

![image](https://user-images.githubusercontent.com/1026425/205320663-30dc064c-d791-4e93-b673-9dfa43569022.png)

6. Accept the default features by clicking **Next**

![image](https://user-images.githubusercontent.com/1026425/205293553-7098ceda-7edd-41c4-9277-c3cd21499637.png)

7. Acknowledge the notes by clicking **Next**

![image](https://user-images.githubusercontent.com/1026425/205320949-0ff40626-b597-4700-9d67-d185714e42e7.png)

8. Select the checkbox for **Certificate Enrollment Policy Web Service** and then select **Add Features**

![image](https://user-images.githubusercontent.com/1026425/205321125-8be55d45-72fb-4b74-be4b-6a8f8dc2383b.png)
![image](https://user-images.githubusercontent.com/1026425/205321322-dc965bf8-4330-45ab-a344-84b1770dac32.png)

9. Select the checkbox for **Certificate Enrollment Web Service**

![image](https://user-images.githubusercontent.com/1026425/205321531-594b591f-53c2-44f5-af86-53377d6c272f.png)

10. Select the checkbox for **Certification Authority Web Enrollment** and then select **Add Features**

![image](https://user-images.githubusercontent.com/1026425/205321986-64cf55cb-1ac3-497d-953c-21ffbc216b1d.png)
![image](https://user-images.githubusercontent.com/1026425/205322021-b16e6fb0-3fa2-48b2-b1b1-49c4a162223c.png)

11. Click **Next** after reading the notes about Web Server Role (IIS)

![image](https://user-images.githubusercontent.com/1026425/205322192-f9d3713d-f977-4e39-b120-04ceedb33c20.png)

12. Accept the default role services for IIS

![image](https://user-images.githubusercontent.com/1026425/205322293-3bd4a060-615a-4e67-a7f7-c35cc64487be.png)

13. Confirm the installation selections and then click **Install** to install the role

![image](https://user-images.githubusercontent.com/1026425/205322393-9509b035-62e9-417d-8483-fa0ffc4327cb.png)

*Allow the installation to complete*
11. Ensure the installation completes successfully

![image](https://user-images.githubusercontent.com/1026425/205322911-b8e324b6-187f-41b4-ad96-a93816e23ac5.png)

12. Under notifications, click **Configure Active Directory Certificate Services on the destination server** to begin the ADCS configuration process

![image](https://user-images.githubusercontent.com/1026425/205323182-82774635-3ef8-4e13-b151-ef7ef6006e93.png)

13. Specify the credentials being used to configure the roles services - here we have used the domain administrator

![image](https://user-images.githubusercontent.com/1026425/205323542-07cf547d-80d8-4470-beb8-06c667123509.png)

14. Select **Ceritification Authority** and **Certification Authority Web Enrollment** services to configure

![image](https://user-images.githubusercontent.com/1026425/205323778-e737809c-c173-4681-a8df-837ce4ab0868.png)

15. Choose **Enterprise CA**

![image](https://user-images.githubusercontent.com/1026425/205323864-8aff4995-ed39-4c33-9ef0-1bdbab3b1696.png)

16. Choose **Root CA**

![image](https://user-images.githubusercontent.com/1026425/205323964-63994cc8-ff6e-404c-9bb0-247675f79c44.png)

17. Choose **Create a new private key**

![image](https://user-images.githubusercontent.com/1026425/205324055-b0c74bd2-a67e-4180-a19d-fdfeba047d60.png)

18. Keep the default **RSA#Microsoft Software Key Storage Provider** and **2048** key length options

![image](https://user-images.githubusercontent.com/1026425/205324269-96710f79-1006-4429-a01b-8f26a377f3cb.png)

19. Specify the CA name - you can keep the default (auto-generated computer name), or create your own as we have done

![image](https://user-images.githubusercontent.com/1026425/205324574-af671780-77ae-4515-870e-02686336e382.png)

20. Specify the validity period (default is 5 years)

![image](https://user-images.githubusercontent.com/1026425/205324665-321eb26e-e4df-43a2-8fbb-076e0cc22f10.png)

21. Keep the default certificate database locations

![image](https://user-images.githubusercontent.com/1026425/205324740-924f0df9-d600-432f-b21c-249d3fc4bf75.png)

22. Confirm the settings and then click **Configure** to complete the configuration process

*Wait for the configuration to complete and verify that it has completed successfully*
![image](https://user-images.githubusercontent.com/1026425/205325214-b1673acc-3f19-4606-96a5-5bc293a0ef72.png)

23. Click **Yes** to configure additional services

![image](https://user-images.githubusercontent.com/1026425/205325344-2d1f7924-d07e-4dc4-ae1d-b0af483461b7.png)

24. Keep the same credentials and click **Next**
25. Select **Certificate Enrollment Web Service** and **Certificate Enrollment Policy Web Service**

![image](https://user-images.githubusercontent.com/1026425/205325755-1888b74b-3451-4f5c-a183-cabfaa660026.png)

26. Accept the default CA for Certificate Enrollment Services

![image](https://user-images.githubusercontent.com/1026425/205325998-7be398fc-15fb-4cb5-8338-316e896467e1.png)

27. Keep the default selection of **Windows integrated authentication**

![image](https://user-images.githubusercontent.com/1026425/205326112-88942a26-488c-467e-9e33-4e7e6b876b17.png)

28. Choose **Use the built-in application pool identity**

![image](https://user-images.githubusercontent.com/1026425/205326242-f51b6409-9d36-424c-a2e0-e4344f89898b.png)

29. Keep **Windows integration authentication** for the CEP authentication type

![image](https://user-images.githubusercontent.com/1026425/205326354-e679a92a-953a-49ea-a683-e37faa61286b.png)

30. Select the Root CA for SSL encryption

![image](https://user-images.githubusercontent.com/1026425/205326506-7f712bc1-830c-4b1f-b006-8151ee36d1b0.png)

31. Verify the settings and then click **Configure**

![image](https://user-images.githubusercontent.com/1026425/205326584-6d1caa8a-cf5a-43ab-9e76-e841a6b547b6.png)

*Wait for the configuration to complete and verify that it has completed successfully*

![image](https://user-images.githubusercontent.com/1026425/205326704-50cbce6f-3ca7-4f23-aaee-67031a9e0b0a.png)

## Edit permissions on certificate templates
Now we will configure permissions to allow authenticated users to self-enroll secure virtual smart cards for authentication.

1. On the AD CS server, open the **Certificate Templates Console** (<kbd>Win</kbd>+<kbd>R</kbd>, `certtmpl.msc`)

![image](https://user-images.githubusercontent.com/1026425/205383653-611271f8-fad3-45ca-b96f-07d943bf569a.png)

2. Right-click **Enrollment Agent** and choose **Properties**

![image](https://user-images.githubusercontent.com/1026425/205383878-59dc85c6-a4a5-4590-9cd3-646474e001e6.png)

3. On the **Security** tab, check the **Enroll** permission checkbox for `Authenticated Users` then click **Apply** and **OK**

![image](https://user-images.githubusercontent.com/1026425/205384064-7ec59288-0e35-4db3-97da-9843e209ff0b.png)

4. Right-click **Smartcard Logon** and choose **Properties**

![image](https://user-images.githubusercontent.com/1026425/205384219-2dfad1d4-7a25-4d1e-bb4c-715284d53b5b.png)

5. On the **Security** tab, check the **Enroll** permission checkbox for `Authenticated Users` then click **Apply** and **OK**

![image](https://user-images.githubusercontent.com/1026425/205384287-7b004199-c2d8-4f15-82e0-efb322a7e0eb.png)

6. Close the **Certificate Templates Console**


## Install certificate templates
Next we will install certificate templates that will be used by the enrollment agent and for smartcard logon.

1. On the AD CS server, select **AD CS** from the menu, then right-click the server entry and select **Certification Authority**

![image](https://user-images.githubusercontent.com/1026425/205498866-b979a64c-dd59-4456-9750-a06125d98363.png)


2. Expand the Certification Authority tree for your server, and select **Certificate Templates**
3. Right-click below the last template and select **New** > **Certificate Template to Issue**

![image](https://user-images.githubusercontent.com/1026425/205499000-cbec2e46-a134-4d5d-9d39-a0c1c6150dd9.png)

4. Select **Enrollment Agent** then click **OK** (the Enrollment Agent certificate template will show up in the Certificate Templates) 

![image](https://user-images.githubusercontent.com/1026425/205499049-bb5ae484-b364-4e40-8433-cf9204f154ca.png)

5. Once again, right-click below the last template and select **New** > **Certificate Template to Issue**; select **Smartcard Logon** then click **OK**

![image](https://user-images.githubusercontent.com/1026425/205499200-0123ad0b-8aed-4501-8f35-5790fd0ab9ba.png)

You should now see **Enrollment Agent** and **Smartcard Logon** certificate templates in the list. This step is now complete and the **Certification Authority** windows can be closed

![image](https://user-images.githubusercontent.com/1026425/205499281-27ee1f15-2bf0-4bef-a4b4-bbc8fad6b083.png)

## Create group policy for "Do Not Display Last Logon"
By default, the login screen on Windows 10/11 and Windows Server 2019/2016/2012R2 displays the account of the last user who logged in to the computer. Although this is convenient for the person authorized to use the computer, it also makes it easier for an attacker to access the computer. To access your device, they only need to find the correct password. To do this, there are various ways of social engineering, brute-force attacks, or a banal sticky piece of paper with a password on the monitor.

We will tighten the security of our lab workstation by disabling display of the last logged on user via a group policy object (GPO) on the domain controller.

1. If not already signed in, login to the AD DS server with administrative permissions
2. In the **Server Manager** > **Dashboard** click **Tools** then select **Group Policy Management**

![image](https://user-images.githubusercontent.com/1026425/205519318-07ce6816-d357-4de7-8988-1bf8a43c133d.png)

3. Expand your forest > **Domains** > your domain, then right-click your domain and select **Create a GPO in this domain, and Link it here**

![image](https://user-images.githubusercontent.com/1026425/205519455-bffe4957-6012-49eb-b469-7c96febef7d5.png)

4. Provide a policy name (we are using `BlokSec Smart Logon`) and click **OK**

![image](https://user-images.githubusercontent.com/1026425/205519521-887a8be1-7bd5-426e-9c96-ad6dd1824c9d.png)

5. Right-click the new policy, and select **Edit...**

![image](https://user-images.githubusercontent.com/1026425/205519549-bba50823-5c64-42c3-96a2-c2c8ffaa8353.png)

6. Expand the tree to **Computer Configuration** > **Policies** > **Windows Settings** > **Security Settings** > **Local Policies** > **Security Options** then double-click **Interactive logon: Don't display last signed-in**

![image](https://user-images.githubusercontent.com/1026425/205519648-fa735d0e-4c47-4c3c-af39-b2c8455f8e07.png)

7. Check the **Define this policy setting:** checkbox and choose **Enabled**, then click **Apply** and **OK**

![image](https://user-images.githubusercontent.com/1026425/205519690-7b5bccd6-0668-4c9a-a7c2-a8189a359547.png)


## Install BlokSec virtual smart card provider on client workstation
> Note that in our lab we are using Windows 10 running as a virtual machine hosted by Windows Hyper-V as the client workstation. The instructions we followed to create the virtual machine itself [can be found here](Hyper-V%20Workstation%20Configuration.MD)

1. Copy the `BlokSec Installer.exe` to your virtual machine (we temporarily enabled **Enhanced session** in the Hyper-V **View** menu to make this easier)

2. Execute the installer (administrative permissions will be required; installer requires no user interaction)

![image](https://user-images.githubusercontent.com/1026425/205635796-9d0f4f18-6987-4bec-a914-05c285453b39.png)

![image](https://user-images.githubusercontent.com/1026425/205635852-46b9ff4f-7ac7-432c-ad1e-843dfc215ee5.png)


## Create and assign a virtual smart card to the user
> **Note:** the following instructions are temporary during the private beta; for general availability release we will have a scripted version of the below instructions

### Pre-requisite 
1. (To be documented) create the Windows Passwordless Logon application in the BlokSec administrative console
2. (to be documented) Create an account for the user user under the Windows Passwordless Logon application in the BlokSec administrative console
3. On the client workstation, open a command prompt with administrative permissions

![image](https://user-images.githubusercontent.com/1026425/205642603-f69b4b04-86ca-469b-b0cd-ef2966c2d186.png)

4. Create the virtual smart card for the user (in this example, our user is `mgillan` in the `bloksec.local` domain so we register the user with their UPN, `mgillan@bloksec.local`)

```
BlokSecCardUtility.exe /an=mgillan@bloksec.local /cc
```

![image](https://user-images.githubusercontent.com/1026425/205642488-91dd10cf-f1f9-4384-9b8f-1442faa16f80.png)

5. Once created, the smart card adds a registry entry that can hold the secure credentials required to exchange information with the BlokSec secure API. Next, we need to provide the API credentials for the virtual smart card reader to communicate with the BlokSec secure API:

```
BlokSecCardUtility.exe /an=mgillan@bloksec.com /appDID=<your app DID>
BlokSecCardUtility.exe /an=mgillan@bloksec.com /appSecret=<your app secret>
```

![image](https://user-images.githubusercontent.com/1026425/205646617-46263007-6dd6-4ff6-932b-5fabbafc7c60.png)

6. Close the administrative command prompt
7. Open a new command prompt running as the user who will be logging in via BlokSec passwordless logon

![image](https://user-images.githubusercontent.com/1026425/205646978-b81e6788-8969-44cc-a941-dd68ba2434df.png)

9. Enroll the BlokSec virtual smart card

> **Note:** during this operation, the user will need to be ready to respond to the enrollment authorization request, which will be sent to their BlokSec app on their mobile device
 
```
BlokSecCardUtility.exe /an=mgillan@bloksec.local /ec
```

The user will receive an authorization prompt on their BlokSec mobile authenticator app:

![image](https://user-images.githubusercontent.com/1026425/205650162-46e5fad7-4115-46fc-9e02-318e80104050.png)

Once accepted, you will be prompted to select a certitification authority. Choose the certification authority created earlier and click **OK**

![image](https://user-images.githubusercontent.com/1026425/205650460-f52e1631-e276-40d9-b847-4732a830fcb0.png)

![image](https://user-images.githubusercontent.com/1026425/205650564-6c464df6-fad6-4ee9-84a9-d46935223e45.png)

*The user is now ready to login to the client workstation using BlokSec Windows Passwordless Logon*
