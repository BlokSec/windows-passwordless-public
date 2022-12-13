# BlokSec Windows Passwordless Logon Credential Provider
This credential provider allows you to enjoy the security and convenience of BlokSec's passwordless authentication on your Windows workstations and servers. The credential provider acts as a virtual smart card provider, interacting with the Windows operating system using native capabilities provided by Microsoft and the secure trusted platform module (TPM) of the machine to keep all user credentials safe. This allows the BlokSec service to securely log users into the domain without requiring the installation of any agents or DLLs on your organization's domain controllers. The only software required is the BlokSec Windows Passwordless Longon credential provider on client Windows 10/11 workstations.

The BlokSec Windows Passwordless Logon credential provider creates a virtual smart card on the client workstation (communicating with the AD CS during this process), and during logon the credential provider returns a cryptographically secure response to the operating system that cannot be phished, brute-forced, or forged.

**Table of Contents**
1. [Prerequsites](#prerequisites)
2. [Client Installation](#client-installation)
3. [Virtual Smart Card Registration](#virtual-smart-card-registration)
4. [User Creation](#user-creation)
5. [Example Logon Flow](#example-logon-flow)

## Prerequisites

**Server**

Windows Server 2016 or forward with following components:
* Active Directory Domain Services
* Active Directory Certificate Services (This is required to trust self-signed client certificates which will be created during user enrollment)
* DNS Services

Please see our helpful [Lab Setup Guide](docs/Lab%20Setup.md) for detailed step-by-step instructions on how to setup a stand-alone lab to demonstrate the concept and value of this offering; these instructions can easily be adapted to setting up passwordless login in your own domain.

**Client Workstation**

Windows 10 / 11 Professional or Enterprise
* Client workstation must be domain joined
* Device must have access to a TPM (Trusted Platform Module) to securely store the BlokSec credentials
* Mobile device with the BlokSec authenticator installed

## Client Installation
In order to enable passwordless Windows logon, the BlokSec credential provider for Windows must be installed on each client workstation. 

> **Warning**: In some environments, Windows Defender blocks the installation due to the installer not being properly signed (it can be incorrectly identified as malware). To overcome this difficulty, you must [provide an exclusion for Windows Defender](docs/Windows%20Defender%20Exclusion.md).

1. Execute the `BlokSec Installer.exe` executable - administrative permission will be required
2. Click **Install**
   
   ![image](https://user-images.githubusercontent.com/1026425/204528540-0d4473f7-ad25-46bc-a31c-34077aa81d80.png)
3. Click **Finish** 

## Virtual Smart Card Registration

>**Pre-requisite**
>
>To complete this step you must have a [Windows Password application configured in the BlokSec administration console](/docs/BlokSec%20Administration.md#create-the-windows-passwordless-logon-application) (this is where you will find the required **Application ID** (a.k.a. `appDID`) and **Application Secret** (a.k.a. `appSecret`)

Once the BlokSec installer has completed installing the BlokSec Windows Passwordless Logon credential provider, an administrator must configure the BlokSec API credentials (Application ID a.k.a. `appDID` and Application Secret a.k.a. `appSecret`) and register the user's virtual smart card before the user can logon. 

> Note: in the next version, these steps will be incorporated in the installer to avoid the requirement for these manual steps

### Create and Initialize the Virtual Smart Card
1. Logon to windows with administrative account

2. Perform the following operation to create and initialize the virtual smart card - note that **Account Name** shall be the user's Active Directory [User Principal Name (UPN)](https://learn.microsoft.com/en-ca/windows/win32/adschema/a-userprincipalname)

    a.  Create a virtual smart card for the account name.
    
        BlockSecCardUtility.exe /an=<Account Name> /cc
    
    b.  Register Application DID to the local machine for the account name.
    
        BlockSecCardUtility.exe /an=<Account Name> /appDID=<Application DID>
     
    c.  Register Application Secret to the local machine for the account name.
    
        BlockSecCardUtility.exe /an=<Account Name> /appSecret=<Application Secret>
     
### Enroll the Smart Card Certificate to the BlokSec Virtual Smart Card
1. Logon to windows with user credential for which you want to enable passwordless windows logon.

2. Run the following command to enroll the Windows logon certificate. Make sure Certificate Authority is configured for the domain and issuance of smart card logon certificates are enabled for logged-in users.
    
        BlockSecCardUtility.exe /an=<Account Name> /ec

<hr>

## User Creation 
The last step, if it has not already been completed, is to [create the user's account in BlokSec](/docs/BlokSec%20Administration.md#create-user-accounts) and have the user complete the registration on their mobile device. This is a one-time process required to link the user's mobile device to their virtual smart card on the Windows client workstation.

## Example Logon Flow

* Log out of the workstation to return to the logon screen
* Click the newly created BlokSec virtual smart card icon

![image](https://user-images.githubusercontent.com/1026425/205650947-cd325ff9-bba3-448f-959e-fcb7cbe9a766.png)
 
 * Do not type in a PIN (no PINs or passwords are required!) just click the ➡ button to submit

![image](https://user-images.githubusercontent.com/1026425/205651595-c967826b-7853-479a-8700-169252a8ec0b.png)

* A push notification will be sent to the user's BlokSec mobile authenticator to authenticate and authorize the logon to the client workstation; the user accepts the request and provides their authentication to the mobile device (i.e., facial recognition, fingerprint, or PIN as supported by the user's device)

![image](https://user-images.githubusercontent.com/1026425/205651651-746b07d9-1cb6-4475-bf74-7d86536cb61a.png)

![image](https://user-images.githubusercontent.com/1026425/205651992-4ed45775-01bc-4602-b8c0-51a3e9d6523a.png)

*The user is then successfully authenticated and logged into the client workstation*

![image](https://user-images.githubusercontent.com/1026425/205652072-8b1d47c6-39ab-4a01-8bb1-efad3ff0245e.png)

![image](https://user-images.githubusercontent.com/1026425/205652297-62c370c4-b1cb-4312-8bf0-599ce9443819.png)
