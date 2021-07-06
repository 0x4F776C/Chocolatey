# CIFS STUFF - LINUX

I've created a samba share folder to host executable(s) and nupkg(s)

```console
sudo apt install samba
sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.bak
sudo vi /etc/samba/smb.conf

[global]
workgroup = WORKGROUP
server string = Samba Server %v
netbios name = ubuntu
security = user
map to guest = bad user
name resolve order = bcast host
dns proxy = no
bind interfaces only = yes

[Public]
path = /home/cape/samba
writable = yes
guest ok = yes
guest only = yes
read only = no
create mode = 0777
directory mode = 0777
force user = nobody

mkdir ~/samba
sudo chmod 0777 ~/samba
sudo chown -R nobody:nogroup ~/samba
mkdir ~/samba/exe
sudo chmod 0777 ~/samba/exe
```

# CHOCOLATEY STUFF

## INSTALL CHOCOLATEY

```console
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```

## PREPARE FOR PACKAGE CREATION

```console
Set-ExecutionPolicy Bypass

$Path = "C:\ProgramData\chocolatey\helpers\functions"

Get-ChildItem -Path $Path -Filter *.ps1 | Where-Object { $_.FullName -ne $PSCommandPath } | ForEach-Object { . $_.FullName }
```

## PACKAGE CREATION

### Method 1

```console
*create <package id name>.nuspec
	<?xml version="1.0" encoding="utf-8"?>
	<package xmlns="http://schemas.microsoft.com/packaging/2015/06/nuspec.xsd">
	  <metadata>
	    <id><package id name></id>
	    <title><package name> (Install)</title>
	    <authors>NA</authors>
	    <version><package version> </version>
	    <description>NA</description>
	  </metadata>
	</package>

Install-ChocolateyInstallPackage -packageName 'package id name' -fileType 'exe' -file '<full path to exe file>'

choco pack

*transfer *.nupkg file
```
### Method 2

```console
choco new <package id name> silentargs"'/sAll /rs /msi EULA_ACCEPT=YES'" url="'\\where\executable\file\is\at'"
cd <package id name>
Remove-Item _TODO.txt
Remove-Item ReadMe.md
Remove-Item <package id name>.nuspec
notepad <package id name>.nuspec
	<?xml version="1.0" encoding="utf-8"?>
	<package xmlns="http://schemas.microsoft.com/packaging/2015/06/nuspec.xsd">
  	  <metadata>
    	    <id><package id name></id>
            <version><package version></version>
            <title><package id name> (Install)</title>
            <authors>NA</authors>
            <description>NA</description>
          </metadata>
          <files>
            <file src="tools\**" target="tools" />
          </files>
        </package>
	
cd tools
Remove-Item LICENSE.txt
Remove-Item VERIFICATION.txt
Remove-Item chocolateybeforemodify.ps1
Remove-Item chocolateyinstall.ps1
ise chocolateyinstall.ps1
	$ErrorActionPreference = 'Stop';
	$toolsDir   = "$(Split-Path -parent $MyInvocation.MyCommand.Definition)"
	$url        = '\\where\executable\file\is\at'

	$packageArgs = @{
  	  packageName   = $env:ChocolateyPackageName
  	  unzipLocation = $toolsDir
  	  fileType      = 'EXE'
  	  url           = $url

   	  softwareName  = '<package id name>*'

  	  silentArgs    = "/sAll /rs /msi EULA_ACCEPT=YES"
  	  validExitCodes= @(0, 3010, 1641)
	}

	Install-ChocolateyPackage @packageArgs
	
Remove-Item chocolateyuninstall.ps1
ise chocolateyuninstall.ps1
	$ErrorActionPreference = 'Stop';
	$packageArgs = @{
  	  packageName   = $env:ChocolateyPackageName
  	  softwareName  = '<package id name>*'
  	  fileType      = 'EXE'
  	  silentArgs    = "/qn /norestart"
  	  validExitCodes= @(0, 3010, 1605, 1614, 1641)
	}

	$uninstalled = $false
	[array]$key = Get-UninstallRegistryKey -SoftwareName $packageArgs['softwareName']

	if ($key.Count -eq 1) {
  	  $key | % { 
    	    $packageArgs['file'] = "$($_.UninstallString)"
    
            if ($packageArgs['fileType'] -eq 'MSI') {
              $packageArgs['silentArgs'] = "$($_.PSChildName) $($packageArgs['silentArgs'])"
      
      	      $packageArgs['file'] = ''
    	    } else {
     	    }

    	    Uninstall-ChocolateyPackage @packageArgs
          }
	} elseif ($key.Count -eq 0) {
  	  Write-Warning "$packageName has already been uninstalled by other means."
	} elseif ($key.Count -gt 1) {
  	  Write-Warning "$($key.Count) matches found!"
  	  Write-Warning "To prevent accidental data loss, no programs will be uninstalled."
  	  Write-Warning "Please alert package maintainer the following keys were matched:"
  	  $key | % {Write-Warning "- $($_.DisplayName)"}
	}

cd ..
choco pack
```

## INSTALLATION

```console
cinst <package id name> -s <source nupkg file directory> -y

*go to C:\ProgramData\chocolatey\bin\
*install one by one LOL
```

### REFERENCES

[Adobe Reader Silent Install*](https://silentinstallhq.com/adobe-reader-dc-silent-install-how-to-guide/)
