# BlokSec Windows Passwordless Logon credential provider
This credential provider allows you to enjoy the security and convenience of BlokSec's passwordless authentication on your Windows workstations and servers. 

## Prerequisites

**Server**

Windows Server 2016 or forward with following components:
* Active Directory Domain Services
* Active Directory Certificate Services (This is required to trust self-signed client certificates which will be created during user enrollment)
* DNS Services

**Client Workstation**

Windows 10 / 11 Professional or Enterprise
* Client workstation must be domain joined
* Device must have access to a TPM (Trusted Platform Module) to securely store the BlokSec credentials
* Mobile device with the BlokSec authenticator installed
