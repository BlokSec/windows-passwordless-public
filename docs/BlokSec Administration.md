# BlokSec Administration for Windows Passwordless Logon
This document walks through the steps required to create and manage the Windows Passwordless Logon app and user accounts in the BlokSec administration console. In order to perform these steps, you will need an account with admin access to your client's BlokSec tenant.

**Table of Contents**
1. [Create the Windows Passwordless Logon Application](#create-the-windows-passwordless-logon-application)
2. [Create User Accounts](#create-user-accounts)

## Create the Windows Passwordless Logon Application
* As an administrator for your tenant, from the dashboard, click on **Applications** > **+ Add Application** > **Create From Template**

![image](https://user-images.githubusercontent.com/1026425/207306569-05c8abb8-638c-43a5-b7c6-a923a80d98ab.png)

* Select the **Windows Passwordless Logon** template

![image](https://user-images.githubusercontent.com/1026425/207306872-226a978d-ca60-41db-b32d-d353a142233a.png)

* Customize the application name (this will be shown to users on their mobile authenticator during logon) and logo URL if desired, then click **Submit**

![image](https://user-images.githubusercontent.com/1026425/207307094-6367da36-0c54-496f-9e62-943c2b125744.png)

* Your newly created application will the displayed; click the **Generate App Secret** button to generate the first application secret - this is used in combination with the application ID to authenticate and authorize the Windows credential provider during API calls. 

* Make note of the **Application ID** (a.k.a. `appDID`) and **Application Secret** (a.k.a. `appSecret`) as these are required when configuring the BlokSec Windows credential provider

**⚠Warning!** be sure to make note of the **Application Secret** when it is displayed as it cannot be retrieved after creation except by contacting BlokSec support!

![image](https://user-images.githubusercontent.com/1026425/207307473-437d42aa-50b1-4e26-862a-4486eebdee9d.png)

## Create User Accounts

###Pre-requisite** 

User has the BlokSec mobile authenticator (**yuID** on the [App Store](https://bloksec.io/images/appstore.png) or [Google Play](https://play.google.com/store/apps/details?id=com.bloksec) or if applicable, the branded authenticator for your organization - reach out to sales@bloksec.com for more details on getting your own branded authenticator)

### Instructions 

* From the BlokSec administration console dashboard, click on your Windows Passwordless application under **Applications** to load the application's view
* From the application view, click on the ⚙ icon and select **Create Account**

![image](https://user-images.githubusercontent.com/1026425/207308508-0c0d4c9a-e355-49b7-be54-71783289017b.png)

* Enter the required information as follows:
  * **First Name** and **Last Name** - the user's name as you want it to appear in the email invitation
  * **Email** - the email address where the invitation will be sent; please ensure the user has access to this email during registration time
  * **Mobile Number** - optionally add the user's mobile number to the account; this will not be used during this flow
  * **Account Name** - enter the user's fully qualified Active Directory domain [User Principal Name (UPN)](https://learn.microsoft.com/en-ca/windows/win32/adschema/a-userprincipalname), e.g., `mgillan@bloksec.com`

![image](https://user-images.githubusercontent.com/1026425/207308920-bcd72613-63c8-4683-b5d3-ed56baec6b20.png)

* Submit the form
* The user will receive an email with an invitation to register their account with the BlokSec service. The user must open the email on their mobile device, or (coming soon) scan the QR code with their mobile
