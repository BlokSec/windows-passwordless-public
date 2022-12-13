# BlokSec Windows Passwordless Logon credential provider
This credential provider allows you to enjoy the security and convenience of BlokSec's passwordless authentication on your Windows workstations and servers. The credential provider acts as a virtual smart card provider, interacting with the Windows operating system using native capabilities provided by Microsoft and the secure trusted platform module (TPM) of the machine to keep all user credentials safe. This allows the BlokSec service to securely log users into the domain without requiring the installation of any agents or DLLs on your organization's domain controllers. The only software required is the BlokSec Windows Passwordless Longon credential provider on client Windows 10/11 workstations.

The BlokSec Windows Passwordless Logon credential provider creates a virtual smart card on the client workstation (communicating with the AD CS during this process), and during logon the credential provider returns a cryptographically secure response to the operating system that cannot be phished, brute-forced, or forged.

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

### Pre-requisite

To complete this step you must have a Windows Password application configured in the BlokSec administration console (this is where you will find the required **Application ID** (a.k.a. `appDID`) and **Application Secret** (a.k.a. `appSecret`)

### Configuration

Once the BlokSec installer has completed installing the BlokSec Windows Passwordless Logon credential provider, an administrator must configure the BlokSec API credentials (Application ID a.k.a. `appDID` and Application Secret a.k.a. `appSecret`) and register the user's virtual smart card before the user can logon. 

> Note: in the next version, these steps will be incorporated in the installer to avoid the requirement for these manual steps

### Create and Initialize the Virtual Smart Card
1. Logon to windows with administrative account

2. Perform the following operation to create and initialize the virtual smart card.

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
