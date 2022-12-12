# Overcoming Windows Defender Warnings
In some environments, Windows Defender incorrectly identifies the BlokSec installer as a trojan (malware) due to missing provider signature on the installer executable. Until BlokSec resolves this with Microsoft, the work-around is to create an exclusion for the installer in Windows Defender. This can be done locally on the machine with PowerShell, or globally for the domain via a Group Policy Object. See [this Microsoft article](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/configure-extension-file-exclusions-microsoft-defender-antivirus?view=o365-worldwide) for more information.

## Create an exclusion locally via PowerShell
1. Start a PowerShell with administrative prileges
2. Create a directory for the installer (in this example, we create `C:\Users\Administrator\BlokSec`)
3. Execute the command 
```powershell
Add-MpPreference -ExclusionPath "C:\Users\Administrator\BlokSec\BlokSec Installer.exe"
```
![image](https://user-images.githubusercontent.com/1026425/204521524-a61ebbed-fddf-40a8-bed0-9d26c5b93b82.png)

1. Copy `BlokSec Installer.exe` to the above folder and execute - Windows Defender should no longer quarantine the file and execution should be permitted.

## Create an exclusion via Group Policy Object (GPO)
1. On your Group Policy management computer, open the Group Policy Management Console, right-click the Group Policy Object you want to configure (or create a new one), and then select Edit. In this example, we will edit the `BlokSec Smart Card Logon` policy created while configuring the AD Certificate Server.
2. In the Group Policy Management Editor go to Computer Configuration > Policies > Administrative Templates.
![image](https://user-images.githubusercontent.com/1026425/204523686-665dbe74-d492-4b6e-a929-e338dad7fa73.png)

3. Expand the tree to Windows Components > Windows Defender Antivirus > Exclusions.
![image](https://user-images.githubusercontent.com/1026425/204523801-2973f8fe-9e13-4722-b75c-2362366a0e91.png)

4. Open the Path Exclusions setting for editing, and add your exclusions.
   - Edit `Path Exclusions` and set the option to Enabled; under the **Options** section, select **Show...**
   ![image](https://user-images.githubusercontent.com/1026425/204524463-e4462446-6b14-4d73-a639-50fb2dc8e120.png)

   - Enter the fully qualified path to the `BlokSec Installer.exe` file, including the drive letter, folder path, file name, and extension. (e.g., `C:\Installers\BlokSec Installer.exe`)
   - Enter `0` in the Value column.
     
     ![image](https://user-images.githubusercontent.com/1026425/204528203-f7f134a4-0d56-4978-beb6-d74d337de305.png)
   
   - Choose **OK**.
   - Choose **Apply** then **OK**

5. Copy `BlokSec Installer.exe` to the folder specified in step 4 above and execute - Windows Defender should no longer quarantine the file and execution should be permitted.

*Note* Group policies can take some time to propagate to all machines on the domain. If you are testing, you can ensure the policy change has been picked up by running
```winbatch
gpupdate /force
```
![image](https://user-images.githubusercontent.com/1026425/204527305-aed37bf5-96f7-4f59-98b4-c5cd9d7adb4d.png)
