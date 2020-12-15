# Create Backups with Borg and WSL on Windows

!!! summary
    This article explains how to create a backup on windows with borg.

* install WSL Version 2.0 [more](https://docs.microsoft.com/de-de/windows/wsl/install-win10#step-4---download-the-linux-kernel-update-package)
    * can be checked with `PS> wsl -l -v`
* install as Distribution [Debian](https://www.microsoft.com/store/productId/9MSVKQC78PK6)
* install `Borg` in `Debian`

* create repository
`debian$ borg init ----encryption=none /path/to/Repository/`   

## ToastNotifications
* For giving the enduser a feedback, ToastNotifications are used

```Powershell
# Install Module BurnToast
PS> Install-Module -Name BurntToast
# Allow foreign Modules to be installed
PS> Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
# Import Module BurnToast
PS> Import-Module BurntToast
# Test ToastNotification
PS> New-BurntToastNotification -Text "Test",'This is a ToastNotification'
```

## Creating the Backup Script

borg.sh
```bash
#!/bin/bash

/mnt/c/windows/system32/windowspowershell/v1.0/powershell.exe "New-BurntToastNotification -AppLogo \path\to\image.png -Text "Borg",'creating Backup'"

REPOSITORY=/path/to/Repository

#export BORG_PASSPHRASE=PW von eben

borg create -v --stats                                  \
    --filter AME                                        \
    --list                                              \
    --show-rc                                           \
    --compression lz4                                   \
    --exclude "/path/to/exclude"                        \
    --exclude-caches                                    \
    $REPOSITORY::'{hostname}-{now:%Y-%m-%d_%H:%M}'      \
    /mnt/c/Users/User/Downloads                         \
    /mnt/c/Users/User/Documents                         \
    /mnt/c/Users/User/Desktop
borg list $REPOSITORY

/mnt/c/windows/system32/windowspowershell/v1.0/powershell.exe "New-BurntToastNotification -AppLogo \path\to\image.png -Text "Borg",'Backup was created'"

# shutdown computer
/mnt/c/Windows/system32/windowspowershell/v1.0/powershell.exe "shutdown /s /t 60 /c 'Backup has been created and Computer will shutdown in 60 seconds'"
```
* test script before creating Task with Task Scheduler

## Create Task with Task Scheduler
* create new task
* check option `Run whether user is logged on or not` to run task in background
* add a Trigger, when to create the backup
* add Action of type `Start a Programm` with the Parameter
```
C:\Windows\System32\wsl.exe -d debian /path/to/borg.sh
```
!!! warning
    Path to wsl.exe or powershell.exe could be different

## Resources
* [Borg](https://borgbackup.readthedocs.io/en/stable/quickstart.html)
* [WSL Windows Toast](https://codelearn.me/2019/01/13/wsl-windows-toast.html)
* [BurnToast](https://github.com/Windos/BurntToast)
* [Windows Task Scheduler](https://www.wintips.org/how-to-run-a-scheduled-task-in-background-or-in-foreground/)