﻿# PSAutoLab

## about_PSAutoLab

# SHORT DESCRIPTION

This project serves as a set of "wrapper" commands that utilize the [Lability](https://github.com/VirtualEngine/Lability) module which is a terrific tool for creating a lab environment of Windows based systems.
The downside is that it is a difficult module for less experienced PowerShell users.
The configurations and control scripts for the Hyper-V virtual machine's are written in PowerShell using Desired State Configuration (DSC) and deployed via Lability.
If you feel sufficiently skilled, you can skip using this project and use the Lability module on your own.

# LONG DESCRIPTION

## SETUP

The first time you use this module, you will need to configure the local machine or host.

Open an elevated PowerShell session and run:

```powershell
Setup-Host
```

This will install and configure the Lability module and install the Hyper-V feature if it is missing.
By default, all Autolab files will be stored under C:\Autolab, which the setup will create.
If you prefer to use a different drive, you can specify it during setup.

```powershell
Setup-Host -destinationpath D:\Autolab
```

You will be prompted to reboot, which you should do especially if setup had to add Hyper-V.

## CREATING A LAB

Lab information is stored under the Autolab Configurations folder, which is C:\Autolab\Configurations by default.
Open an elevated PowerShell prompt and change location to the desired configuration folder.
View the `Instructions.md` and/or readme files in the folder to learn more about the configuration.

The first time you setup a lab, Lability will download evaluation versions of required operating systems in ISO format.
This may take some time depending on your Internet bandwidth.
The downloads only happen when the required ISO is not found.
When you wipe and rebuild a lab it won't download files a second time.

Once the lab is created you can use the module commands for managing it.
Or you can manage individual virtual machines using the Hyper-V manager or cmdlets.

*It is assumed that you will only have one lab configuration created at a time.*

### MANUAL SETUP

Most, if not all, configurations should follow the same manual process.
Run each command after the previous one has completed.

* `Setup-Lab`
* `Run-Lab`
* `Enable-Internet`

To verify that all virtual machines are properly configured you can run `Validate-Lab`.
This will invoke a set of tests and loop until everything passes.
Due to the nature of DSC and complexity of some configurations this could take 60-90 minutes.
You can use `Ctrl+C` to break out of the testing loop at any time.
You can manually run the test one time to see the current state of the configuration.

```powershell
Invoke-Pester VMValidate.test.ps1
```

This can be useful for troubleshooting.

### UNATTENDED SETUP

As an alternative, you can setup a lab environment with minimal prompting.

```powershell
Unattend-Lab
```

Assuming you don't need to install a newer version of nuget, you can leave the setup alone.
It will run all of the manual steps for you.

### STOPPING A LAB

To stop the lab VM's, change to the configuration folder in an elevated Windows PowerShell session and run:

```powershell
Shutdown-Lab
```

You can also use the Hyper-V manager or cmdlets to shut down virtual machines.
If your lab contains a domain controller such as DOM1 or DC1, that should be the last virtual machine to shut down.

### STARTING A LAB

The setup process will leave the virtual machines running.
If you have stopped the lab and need to start it, change to the configuration folder in an elevated Windows PowerShell session and run:

```powershell
Run-Lab
```

You can also use the Hyper-V manager or cmdlets to start virtual machines.
If your lab contains a domain controller such as DOM1 or DC1, that should be the first virtual machine to start up.

### LAB CHECKPOINTS

You can snapshot the entire lab very easily.
Change to the configuration folder in an elevated Windows PowerShell session and run:

```powershell
Snapshot-Lab
```

To quickly rebuild the labs from the checkpoint, run:

```powershell
Refresh-Lab
```

### TO REMOVE A LAB

To destroy the lab completely, change to the configuration folder in an elevated Windows PowerShell session and run:

```powershell
Wipe-Lab
```

This will remove the virtual machines and DSC configuration files.
If you intend to rebuild the lab or another configuration, you can keep the LabNat virtual switch.

## UPDATING

As this module is updated over time, new configurations may be added, or bugs fixed in existing configurations.
There may also be new Lability updates.
Use PowerShell to check for new versions:

```powershell
Find-Module PSAutoLab
```

And update:

```powershell
Update-Module PSAutoLab
```

If you update, it is recommended that you update the computer running Autolab.

```powershell
Refresh-Host
```

This will update Lability if required and copy all new configuration files to your Autolab\Configurations folder.
It will NOT delete any files.

## TROUBLESHOOTING

The commands and configurations in this module are not foolproof.
During testing a lab configuration will run quickly and without error on one Windows 10 desktop but fail or take much longer on a different Windows 10 desktop.
Most setups should be complete in under an hour.
If validation is failing, manually run the validation test in the configuration folder.

```powershell
Invoke-Pester VMValidate.test.ps1
```

Take note of which virtual machines are generating errors.
Verify the virtual machine is running in Hyper-V.
On occasion for reasons still undetermined, sometimes a virtual machine will shutdown and not reboot.
This often happens with the client nodes of the lab configuration.
Verify that all virtual machines are running and manually start those that have stopped using the Hyper-V manager or cmdlets.

Sometimes even if the virtual machine is running, manually shutting it down and restarting it can resolve the problem.
Remember to wait at least 5 minutes before manually running the validation test again when restarting any virtual machine.

As a last resort, manually break out of any testing loop, wipe the lab and start all-over.

If you *still* are having problems, wipe the lab and try a different configuration.
This will help determine if the problem is with the configuration or a larger compatibility problem.

At this point, you can open an issue in this repository.
Open an elevated PowerShell prompt and run `Get-PSAutolabSetting` which will provide useful information.
Copy and paste the results into a new issue along with any error messages you are seeing.

## KNOWN ISSUES

Due to what is probably a bug in the current implementation of Desired State Configuration in Windows, if you have multiple versions of the same resource, a previous version might be used instead of the required on.
You might especially see this with the xNetworking module and the xIPAddress resource.
If you have any version older than 5.7.0.0 you might encounter problems.
Run this command to see what you have installed:

```powershell
PS C:\> Get-DSCResource xIPAddress
```

If you have older versions of the module, uninstall them if you can.

```powershell
PS C:\> Uninstall-Module xNetworking -RequiredVersion 3.0.0.0
```

It is recommended that you restart your PowerShell session and try the lab setup again.

# SEE ALSO

[https://github.com/pluralsight/PS-AutoLab-Env](https://github.com/pluralsight/PS-AutoLab-Env)

# KEYWORDS

* autolab
* psautolab