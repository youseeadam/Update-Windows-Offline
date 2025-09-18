# Update-Windows-Offline
## How to update Windows Media Offline ##
* There are 3 types of media that must be updated
  * Operating System (install.wim)
  * WinRE (winre.wim)
  * WinPE (boot.wim)
* The Install.wim is located on the ISO in the sources directory
* The WInRE resides inside the install.wim, so the install.wim must be mounted first.
* Then mount the WinRE file located at: MounPoint\Windows]System32\Recovery\winrm.wim
* WinPE is located on the ISO in the sources directory
### Download the Files you will need ####
1. Get the version of WIndows you are using:
   ```powershell
   Get-WindowsImage sources\install.wim -index 1
   ```
1. This will return a version number, for example Version : 10.0.26100.1. The minor version is what we need, in this example 26100
1. Go to Windows 11, version 24H2 update history - Microsoft Support : https://support.microsoft.com/en-us/topic/windows-11-version-24h2-update-history-0929c747-1815-4543-8461-0160d16f15e5
1. Go to the most recent non Preview Build for the version, in this example 26100, This will typically be a KB article. Note the KB Article
1. Then go to the Microsoft Update Catalog : https://www.catalog.update.microsoft.com/
1. In the search criteria enter the KB article, make sure the download the proper version in this case 24H2 for x64 based systems. There are usually 2 updates, but these get applied dynamically.
1. Mount the install.wim file to get the build version of WinRE
1. Get the version and service version of the WINRE.wim file by running the command:
   ```powershel
   Get-WIndowsImage MounPoint\Windows\System32\Recovery\winrm.wim -index 1
   ```
1. That will return a string like the following: Version : 10.0.26100.1. You will use the minor version in the next step, the 26100
1. Go to Windows Update and search the following: "Safe OS Dynamic Update TheMinorVersion", in this example: Safe OS Dynamic Update 26100
1. Download the the latest version for Windows 11 Version, not the server version for X64
1. Get the version and service version of the boot.wim file by running the command:
   ```powershell
   Get-WIndowsImage sources\boot.wim -index 1
   ```
1. That will return a string like the following: Version : 10.0.26100.1. You will use the minor version in the next step, the 26100. In most cases the version used in boot.wim and winre.wim will be the same, but you should always check.
1. Go to Windows Update and search the following: Setup Dynamic Update TheMinorVersion, in this example: Setup Dynamic Update 26100
1. Download the the latest version for Windows 11 Version, not the server version for X64
### Update the Image ###
1. After mounting the install.wim, apply the update, referencing the latest MSU (the one with the larger KB version, for example from an elevated PowerShell Terminal. This will automatically apply both updates if needed (a Dynamic Update) and can take several minutes even after it shows the update has been applied:
   ```powershell
   Add-WindowsPackage -Path "c:\offline" -PackagePath "Windows11.0-KB5065426-x64.msu" -PreventPending
   ```
1. Mount the WindowsRE Image
1. Apply the update to the WinRE image:
   ```powershell
   Add-WindowsPackage -Path "c:\WinREoffline" -PackagePath "windows11.0-kb5064097-x64.cab" -PreventPending
   ```
1. Dismount the WinRE image
1. Dismount the install.wim
1. Mount the boot.wim Image
1. Apply the update to the winpe image (the update will fail if not applicable):
   ```powershell
   Add-WindowsPackage -Path "c:\WinPEoffline" -PackagePath "windows11.0-kb5066990-x64.cab" -PreventPending
   ```
1. Dismount the boot.wim
1. Once you have updated all the files, you can then use WIndows System Image Manager to open the WIM and build a catalog file
