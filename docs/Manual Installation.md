# Installing the BlokSec Windows Credential Provider package manually
If there is a problem with the installer, it is possible (though not recommended) to install the BlokSec credential provider manually.

## Pre-requisite: install the Microsoft Visual C++ Redistributable
1. Download the [Visual C++ Redistributable](https://aka.ms/vs/17/release/vc_redist.x64.exe)
2. Install the downloaded package following the installer prompts

## Copy the BlokSec DLL and executable files to the System32 folder

1. Copy blokSecCP.dll and BlokSecCardUtility.exe to C:\windows\system32

## Create the required registry keys

1. Create following registry Entries

    [HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Authentication\Credential Providers\{E51E2DED-6FAA-47D3-B99E-304AC474FA53}]
    
    @="blokSecCP"

    [HKEY_CLASSES_ROOT\CLSID\{E51E2DED-6FAA-47D3-B99E-304AC474FA53}]
    
    @="blokSecCP"

    [HKEY_CLASSES_ROOT\CLSID\{E51E2DED-6FAA-47D3-B99E-304AC474FA53}\InprocServer32]
    
    @="blokSecCP.dll"
    
    "ThreadingModel"="Apartment"

    [HKEY_LOCAL_MACHINE\SOFTWARE\BlokSec]
    
    "DisablePin"=dword:00000001
    
    "LogEnable"=dword:00000000
    
    "requesturl"="https://api.bloksec.io/auth/pinRequest"
    
    "connectTimeout"=dword:0000ea60
    
    "resolveTimeout"=dword:00000000
    
    "sendTimeout"=dword:00007530
    
    "receiveTimeout"=dword:00007530
  