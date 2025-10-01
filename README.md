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

### Figure out your current build version ###
1. Get the version of WIndows you are using:
   ```powershell
   Get-WindowsImage sources\install.wim -index 1
   ```
1. This will return the Version Number, for example Version: 26100.6584, the number that is important is the first part, the major build number, the 26100

### Find the current release date ###

1. Go to https://support.microsoft.com/en-us/topic/windows-11-version-24h2-update-history-0929c747-1815-4543-8461-0160d16f15e5
2. Find the current release Build date
When it comes to Windows udpates, there are three types
* Preview, for example: _September 29, 2025—KB5065789 (OS Build 26100.6725) Preview_
* Release, for example: _September 9, 2025—KB5065426 (OS Build 26100.6584)_
* Out-of-band, for example: _September 22, 2025—KB5068221 (OS Build 26100.6588) Out-of-band_
Notice in the above examples that the September 9, 20205 is the most recent stable build, we will use that as the example. Depending on when you read this, the dates and articles will change.

### Download the Files you will need The Easy Way ####

1. Go to Windows Update Catalog: https://www.catalog.update.microsoft.com/
1. Type the following into the search window: **Dynamic Updates Windows _BuildNumber_**, for example Dynamic Updates Windows 26100
2. In the list it returns, you will want to focus on the Last udpate date corrisponding to what Relase date above
3. This will return a long list, from newest to oldest.
4. Within that search look in the title for the following two files, make sure to choose the architecutre you are using and the build date, for example September 9 2025, for example amd64
   * Setup Dynamic Updates for [Server|arm64|amd64]: These are for the Boot.wim, and are CAB files
   * Safe OS Dyanmic Update for [Server|arm64|amd64]: These are for the WinRE.wim and are CAB files
5. Click on the link for the files you need, this will tell you if they are Windows udpates, winre updates, or setup file updates
   * For example if I click on _2025-09 Safe OS Dynamic Update for Microsoft server operating system version 24H2 for x64-based Systems (KB5064097)_ in the description it says Description: This update makes improvements to the Windows Recovery Environment (WinRE) for Microsoft server operating system version 24H2.  That means this download will apply to the WinRE.wim file, and is a pretty small download.
   * If I click on _2025-09 Setup Dynamic Update for Windows 11 Version 24H2 for x64-based Systems (KB5066990)_ in the description it says: Description: This update makes improvements to Windows setup binaries or any files that setup uses for feature updates in Windows. That means this is applied to the boot.wim file, and is a pretty small download
6. To download the Windows update (for install.wim), Using the KB number that was from the Find enter that into the search bar.
7. If I click on the link _2025-09 Cumulative Update for Windows 11 Version 24H2 for x64-based Systems (KB5065426) (26100.6584)_, notice the version matches the version from the update history, and it referneces Security Updates, and is a larger download size.

### Update the Image ###

1. After mounting the install.wim, apply the update, referencing the latest MSU (the one with the larger KB version, for example from an elevated PowerShell Terminal. This will automatically apply both updates if needed (a Dynamic Update) and can take several minutes even after it shows the update has been applied:
   ```powershell
   Add-WindowsPackage -Path "c:\offline" -PackagePath "Windows11.0-KB5065426-x64.msu" -PreventPending
   ```
1. Mount the WindowsRE (WinRE) Image (you must update the base image before mounting the WinRE image)
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
