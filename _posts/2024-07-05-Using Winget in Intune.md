---
title: "Post: Using Winget in Intune"
last_modified_at: 2024-07-05T21:30:00+02:00
categories:
  - Blog
tags:
  - Intune
---

Deploying applications on Intune is becoming increasingly straightforward, thanks to tools like Winget. But what is Winget, and how can it simplify your app management tasks? Winget, short for Windows Package Manager, is a command-line tool designed to streamline the process of discovering, installing, updating, and configuring applications on Windows 10 and Windows 11. With its extensive repository of applications and powerful automation capabilities, Winget is a game-changer for IT administrators.

In this blog post, I'll walk you through two user-friendly scripts that leverage Winget to deploy applications via Intune. Additionally, we'll explore how Winget's auto-update feature can be efficiently managed with remediation scripts, ensuring your apps remain current with minimal effort.

Join me as we delve into these practical solutions that will help you harness the full potential of Winget for seamless app deployment and maintenance on Intune.

# Prerequisites:

- Make sure that the [following Microsoft Store package](https://apps.microsoft.com/detail/9nblggh4nns1?activetab=pivot%3Aoverviewtab&hl=en-us&gl=US#activetab=pivot:overviewtab) is installed: 
- If you want to use remediation scripts to update apps, be sure to have a Windows 10/11 Enterprise E3 or E5 (included in Microsoft 365 F3, E3, or E5) license
- To find the Winget AppID, use Winget.run
- Whttps://github.com/microsoft/Microsoft-Win32-Content-Prep-Tool

# Steps:
I'll be using Firefox as an example

1. Create a new folder (e.g C:\WingetPackages\Firefox)
2. Create a new text file, name it `Install.ps1`, copy-paste the following and change the *WingetAppID*:

    ```powershell
    $packageId = "WingetAppID"
    $action = "install"

    Function Get-WingetCmd {
        $WingetCmd = $null
        # Get WinGet Path

        try {
            # Get Admin Context Winget Location
            $WingetInfo = (Get-Item "$env:ProgramFiles\WindowsApps\Microsoft.DesktopAppInstaller_*_8wekyb3d8bbwe\winget.exe").VersionInfo | Sort-Object -Property FileVersionRaw
         # if multiple versions, pick most recent one
            $WingetCmd = $WingetInfo[-1].FileName
        }
      catch {
            # Get User context Winget Location
          if (Test-Path "$env:LocalAppData\Microsoft\WindowsApps\Microsoft.DesktopAppInstaller_8wekyb3d8bbwe\winget.exe") {
            $WingetCmd = "$env:LocalAppData\Microsoft\WindowsApps\Microsoft.DesktopAppInstaller_8wekyb3d8bbwe\winget.exe"
         } else {
                Write-Host "winget not detected"
              Exit 1
         }
     }
        Write-Host "Winget location: $WingetCmd"
        return $WingetCmd
    }

    $procOutput = & $(Get-WingetCmd) $action "--id" $packageId "--source" "winget" "--silent" "--accept-package-agreements" "--accept-source-agreements" "--disable-interactivity" "--scope" "machine"
    if ($procOutput -is [array]) {
     $lastRow = $procOutput[$procOutput.Length - 1]
     if ($lastRow.Contains("installed")) {
          Write-Host "Package $packageId v$version installed successfully"
         Exit 0
      }
    }
    ```

3. Create another text file, name it `uninstall.ps1`, copy-paste the following and change the *WingetAppID*:

    ```powershell
    $packageId = "WingetAppID"
    $action = "uninstall"

    Function Get-WingetCmd {
        $WingetCmd = $null
        # Get WinGet Path

      try {
         # Get Admin Context Winget Location
         $WingetInfo = (Get-Item "$env:ProgramFiles\WindowsApps\Microsoft.DesktopAppInstaller_*_8wekyb3d8bbwe\winget.exe").VersionInfo | Sort-Object -Property FileVersionRaw
         # if multiple versions, pick most recent one
         $WingetCmd = $WingetInfo[-1].FileName
      }
     catch {
            # Get User context Winget Location
            if (Test-Path "$env:LocalAppData\Microsoft\WindowsApps\Microsoft.DesktopAppInstaller_8wekyb3d8bbwe\winget.exe") {
            $WingetCmd = "$env:LocalAppData\Microsoft\WindowsApps\Microsoft.DesktopAppInstaller_8wekyb3d8bbwe\winget.exe"
         } else {
             Write-Host "winget not detected"
             Exit 1
         }
       }
        Write-Host "Winget location: $WingetCmd"
        return $WingetCmd
    }

    $procOutput = & $(Get-WingetCmd) $action "--id" $packageId "--source" "winget" "--silent" "--accept-source-agreements" "--disable-interactivity" "--scope" "machine"
    if ($procOutput -is [array]) {
     $lastRow = $procOutput[$procOutput.Length - 1]
      if ($lastRow.Contains("uninstalled")) {
          Write-Host "Package $packageId uninstalled successfully"
          Exit 0
       }
    }
    ```
4. Open the Win32 Prep Tool and paste the path where the scripts are (in my case C:\WingetPackages\Firefox)
5. The setup file is `install.ps1`
6. You can choose the path of the output
7. There's no need to create a catalog and press enter

You have now created a Intunewin file. This can be uploaded to Intune

## Uploading to Intune

1. Go to https://intune.microsoft.com/#
2. Go to Apps, Windows and add a new Win32 app
3. Choose the Intunewin file that you've created and fill out the app information
4. Copy-paste the install command: 
    ```
    %windir%\sysnative\windowspowershell\v1.0\powershell.exe -WindowStyle Hidden -ExecutionPolicy Bypass -File install.ps1
    ```
5. Copy-paste the uninstall command:
    ```
    %windir%\sysnative\windowspowershell\v1.0\powershell.exe -WindowStyle Hidden -ExecutionPolicy Bypass -File uninstall.ps1
    ```
6. You can choose your own requirements
7. You can choose your own detection rules. I will use the detection rule "file" with the path `C:\Program Files\Mozilla Firefox\` and file or folder `Firefox.exe` - File or folder exists

**Info Notice:** I wouldn't choose a version because Winget can auto-update.
{: .notice--info}

8. Click next and assign to users and finish the deployment.


## Auto-updating with a remediation script

1. Go to https://intune.microsoft.com/#
2. Go to Devices and then Scripts and Remedations
3. Create a remedation script with the following scripts:  
**Single App upgrade**  
Detection:

    ```powershell
    $app_2upgrade = "WingetAppID"

    $Winget = Get-ChildItem -Path (Join-Path -Path (Join-Path -Path $env:ProgramFiles -ChildPath "WindowsApps") -ChildPath "Microsoft.DesktopAppInstaller*_x64*\winget.exe")

    if ($(&$winget upgrade) -like "* $app_2upgrade *") {
	Write-Host "Upgrade available for: $app_2upgrade"
	exit 1 # upgrade available, remediation needed
    }
    else {
    		Write-Host "No Upgrade available"
	    	exit 0 # no upgrade, no action needed
    }
    ``` 
Remediate:

```powershell
    app_2upgrade = "WingetAppID"

    try{
        $Winget = Get-ChildItem -Path (Join-Path -Path (Join-Path -Path $env:ProgramFiles -ChildPath "WindowsApps") -ChildPath "Microsoft.DesktopAppInstaller*_x64*\winget.exe")

        # upgrade command
        &$winget upgrade --exact $app_2upgrade --silent --force --accept-package-agreements --accept-source-agreements
       exit 0

    }catch{
        Write-Error "Error while installing upgarde for: $app_2upgrade"
       exit 1
    }
 ```

**Upgrade all apps**

Detection:

```powershell
$Winget = Get-ChildItem -Path (Join-Path -Path (Join-Path -Path $env:ProgramFiles -ChildPath "WindowsApps") -ChildPath "Microsoft.DesktopAppInstaller*_x64*\winget.exe")

if ($(&$winget upgrade) -gt 3) {
	Write-Host "Upgrade(s) available."
	exit 1 # upgrade available, remediation needed
}
else {
		Write-Host "No Upgrade available"
		exit 0 # no upgrade, no action needed
}
```

Remediate:

```powershell
    try{
      $Winget = Get-ChildItem -Path (Join-Path -Path (Join-Path -Path $env:ProgramFiles -ChildPath "WindowsApps") -ChildPath "Microsoft.DesktopAppInstaller*_x64*\winget.exe")

      # upgrade command for ALL
       &$Winget upgrade --all #--query "" --silent --accept-package-agreements --accept-source-agreements

      exit 0

    }catch{
        Write-Error "Error while installing upgarde for: $app_2upgrade"
        exit 1
    }
```

4. Assign to users, and set the interval (i.e. Daily or hourly)


# Conclusiong

Deploying applications through Intune with the help of Winget can significantly simplify your app management processes. By following the steps outlined in this post, you can easily create scripts for installing and uninstalling applications, and take advantage of Winget's auto-update feature to ensure your apps are always up-to-date with minimal manual intervention. The use of remediation scripts further enhances the automation capabilities, allowing for seamless maintenance of your application environment.  
I hope that this post helped you!