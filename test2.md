# A Crash Course in Just Enough Administration

A> ## by James Petty

## What is JEA

Just Enough Administration (JEA for the rest of this chapter) is a way in which you can whitelist PowerShell commands that someone can run on a remote machine. It is another layer of security that you can implement in your environment. Its main purpose is to limit the number of users who have administrative privileges on machines. This is done by leveraging the built in capabilities already found in Windows PowerShell. JEA will allow you to custom tailor a PowerShell environment for someone that has all the rights that are needed to perform his/her task but nothing more. Whether that is to restart the spooler service or to kill a process. 

There are three main reasons that you should consider implementing JEA in your environment:

* Whitelist the commands and parameters that a person can use.
* Limit the number of administrators on each machine.
* Know what your users are doing on the machines.

Please note the full list of scripts will be available for download with this book. There for not all of the PowerShell code will be displayed. 

## Prerequisites

The following requirements must be meet for JEA to work correctly in your environment:

* Make sure you have the most up to date version of Windows PowerShell (WMF5.1.x) on your servers and workstations. It should be noted that although Windows Server 2008 R2 can run WMF 5.1 JEA will have slightly limited capabilities.
* PowerShell Remoting must be enabled.

   ```PowerShell
   Enable-PSRemoting
   ```

* It's suggested that you also turn on script block logging but not required. This can be acheived via Group Policy in Administrative Templates -> Windows Components -> Windows PowerShell. By enabling this Windows PowerShell will log all script blocks and write them to the ETW event log Microsoft-Windows-PowerShell/Operational ETW event log.

## How JEA Works

JEA is a mechanism that will allow you to have stricter control of capabilities on endpoints (an endpoint is simply a set of configurations on a remote computer that can be customized when a user connects to it. By default when using Enter-PsSession you connect to the 'microsoft.powershell' Endpoint. All of your endpoints can be found by running the command `Get-PsSessionConfiguration`). You will be creating and configuring endpoints by using Configuration Files (the who) and Role Capability (the what) files . The combination of these two files will allow you to manage who can do what on each machine.

### Role Capability Files (the what) - .psrc

You have to create a PowerShell module. The psm1 file can be blank and the manifest file can have all the defaults. Since JEA is nothing more than a PowerShell module, these files are are required to make a proper module. And inside your module directory you will need a folder called RoleCapabilities
`Screen shot of JEA directory structure here`

Once you have your module created you are ready to make your Role Capability File. Microsoft has provided you with a PowerShell command that we can use to make your files for you  

Below is an example of how to create a Role Capability File with all the defaults.

```PowerShell
New-PSRoleCapabilityFile -Path 'C:\Program Files\WindowsPowerShell\Modules\`
Demo\RoleCapabilities\demo.psrc'
```

Keep in mind that this file has all the power. This is where you include what the user can do. Let’s say that you wanted to let the user see all services.

```PowerShell
#Just Cmdlests
$CapabilityFile = @{
    Path = 'C:\Program Files\WindowsPowerShell\Modules\Demo\RoleCapabilities\`
    demo.psrc'
    VisibleCmdlets = "get-service"
}
New-PSRoleCapabilityFile @CapabilityFile
```

But what if you wanted the HelpDesk to be able to restart the spooler service? You also have the ability to restrict parameters to commands as well. 

```PowerShell
$CapabilityFile = @{
    Path = 'C:\Program Files\WindowsPowerShell\Modules\Demo\RoleCapabilities\`
    demo.psrc'
    VisibleCmdlets = @{Name = 'Restart-Service;
                       Paramaters = @{Name = 'Name'; ValidateSet = 'Spooler'}}
}
New-PSRoleCapabilityFile @CapabilityFile
```

Now that you have the basis down we can start adding more commands and functions. Now lets give access to some selected functions:

```PowerShell
#Adding functions
$CapabilityFile = @{
    Path = 'C:\Program Files\WindowsPowerShell\Modules\Demo\RoleCapabilities\`
    demo.psrc'
    VisibleCmdlets = "get-process","get-service","restart-computer"
    VisibleFunctions='Disable-ScheduledTask',
    'Enable-ScheduledTask',
    'Start-ScheduledTask',
    'Stop-ScheduledTask',
    'Where-Object',
    'Select-Object',
    'Get-SmbOpenFile',
    'Close-SmbOpenFile'
}

New-PSRoleCapabilityFile @CapabilityFile
```

From here the possibilities are endless on what you can allow the Helpdesk, or whoever to do. From as simple as allowing developers to restart their own services or more complicated deployments where admins have a specific endpoint to connect to so they can perform their job.


### Session Configuration File (the who)

The Session Configuration File gives you the ability to configure who can access a system by using a Domain Accounts or Domain Global Groups. You can also set global settings such as virtual accounts and logging here as well. Because each machine has to have its own Configuration File this allows you to set and configure JEA endpoints on a per machine basis. The Microsoft recommended location for this file is env:ProgramData\JEAConfiguration and the file extension for the Configuration File is PSSC. 

#### How to Create and Configure your Configuration File - .pssc

First you will need to make sure that your JEA folder exists in your ProgramData directory if not then create it. You will then need to make your Configuration File. Lucky for you Microsoft has provided you with a command to help you create your Configuration File. The `New-PSSessionConfigurationFile` command that will create the file for you. 

The only required parameter is path (where you are going to save the file). The below command will make a skeleton Configuration File with the most commonly used parameters. To create a file with all the parameters use the -full switch. This can be edited in your favorite text editor it has been created.

```PowerShell
New-PSSessionConfigurationFile -path "env:ProgramData\JEAConfiguration\demo.pssc"
```

You also have the ability to specify parameters so that you do not have to edit the file after its created. 

```PowerShell
$ConfFileData = @{
    SessionType = "RestrictedRemoteServer"
    TranscriptDirectory = "C:\ProgramData\JEATranscripts"
    RunAsVirtualAccount = $true
    Path = "$env:ProgramData\JEAConfiguration\Demo.pssc"
}
New-PSSessionConfigurationFile @ConfFileData
```

* TranscriptDirectory is where the transcript file will live once. You can put this anywhere you wish. It is highly recommended that you enable this feature.

* RunAsVirtualAccount uses a virtual administrative account that the user connects to. This account is unknown to the user and the user cannot use this account in any other way. The virtual account destroys its self when the session is closed. 

* Role Definitions is what roles the user can access (more on this later)

* Path is where you are going to save the pssc file

Another option that you can enable is the `MountUserDrive = $True` this will mount a new PSDrive that is unite to each user that connects, it is also persistent across sessions. The location for this folder is $env:LOCALAPPDATA\Microsoft\Windows\PowerShell\DriveRoots\DOMAIN_USER. This can be handy if you need to copy files to a remote server, using a JEA endpoint. To take advantage of this option you will need to use the `-ToSession` and `-FromSession` parameters for `Copy-Item`. This options comes with its own security risks. You will need to decide if this is something that you would like to use in your environment.

```PowerShell
# Connect to the JEA endpoint
$MyJea = New-PSSession -ComputerName 'srv01' -ConfigurationName 'Demo\User2'

# Copy a file in the local folder to the remote machine.
# Note: you cannot specify the file name or subfolder on the remote machine. You must type "User:"
Copy-Item -Path .\MyFile.txt -Destination User: -ToSession $MyJea

# Copy the file back from the remote machine to your local machine
Copy-Item -Path User:\MyFile.txt -Destination . -FromSession $MyJea
```

As you are completing building the Role Capability File it is time for you to decided what do you want the user to be able to do. You need to assign a role that you want them to have (as defined in your Role Capability file discussed a little further down). You accomplish this by using the `RoleDefinitions` Paramater. `RoleDefinitions = @{'Demo\User1'= @{ RoleCapabilities = 'Demo' }}`. Now your PowerShell command should look something like this

```PowerShell
$ConfFileData = @{
    SessionType = "RestrictedRemoteServer"
    TranscriptDirectory = "C:\ProgramData\JEATranscripts"
    RunAsVirtualAccount = $true
    RoleDefinitions = @{'Demo\User1'= @{ RoleCapabilities = 'Demo' }}
    Path = "$env:ProgramData\JEAConfiguration\Demo.pssc"
}
New-PSSessionConfigurationFile @ConfFileData
```



In the example above User1 has the ability to connect to the 'Demo' Role. But what if you wanted to have multiple roles? You are in luck because that option is built in.
`RoleDefinitions = @{'Demo\User1'= @{ RoleCapabilities = 'Demo','PrintSpooler','DNSOperator' }}`
This way you do not have to put everything in one configuration file (we will get to that later). But will allow you to separate which users can access what. Again, your goal with JEA is to give people the tools they need to perform their job, no more and no less than that.
You can also have multiple accounts with different roles. Let’s say you have a job that you want the HelpDesk to take care of, but the server is also a DNS server so your DNS admins need to perform administrative functions as well.

* The role name must be the same as the Role Capability File you created earlier

    * HelpDeskFunction.psrc

```PowerShell
RoleDefinitions = @{'Demo\HelpDeskGroup'= @{ RoleCapabilities = `
'HelpDeskFunction' }}
RoleDefinitions = @{'Demo\DNSAdmin'= @{ RoleCapabilities = 'DNSAdminRole' }}
RoleDefinitions = @{'Demo\Tier2Support= @{ RoleCapabilities = `
'HelpDeskFunction','DNSAdminRole','Tier2Support'}}
```

## Deploy Jea

Now that you have all of your Configuration Files and Cability Files set up, how do you get these out to all of your servers? There are numerous ways to do this, but I suggest you use a GPO startup scripts to copy these files to their correct location.

The first thing you need to do is copy your JEA module to each computer on your network.

```PowerShell
$RemoteModule = \\PATH_TO_YOUR_JEA_MODULE_HERE
$MachineModule = 'C:\Program Files\WindowsPowerShell\Modules\MyJeaModule'

$moddest = 'C:\Program Files\WindowsPowerShell\Modules'

if (-NOT (Test-Path $MachineModule)) {Copy-Item -Path $RemoteModule `
-Destination $moddest -Recurse -Force}

$RemoteModuleHash = Get-ChildItem $RemoteModule -Recurse | Get-FileHash |
select @{Label="Path";Expression={$_.Path.Replace($RemoteModule,"")}},Hash
$MachineModuleHash = Get-ChildItem $MachineModule -Recurse | Get-FileHash |
select @{Label="Path";Expression={$_.Path.Replace($MachineModule,"")}},Hash
$moddiffEM = ''
 
Compare-Object $RemoteModuleHash $MachineModuleHash -Property Path,Hash `
-IncludeEqual -PassThru | Where-Object {$_.SideIndicator -ne '==' } |
ForEach-Object -process {
$moddiffEM = 'y'
}

if ($moddiffEM) {
Copy-Item -Path $RemoteModule -Destination $moddest -Recurse -Force
}
```

Next you need to copy the endpoint configuration files (you’re Capability File and your Session Configuration File).

```PowerShell
$RemoteConfiguration = "Path_To_My_Jea_Configuration"
$LocalConfiguration = 'C:\ProgramData\JEAConfiguration'

$JeaDestination = 'C:\ProgramData'

if (-NOT (Test-Path $LocalConfiguration)) {
Copy-Item -Path $RemoteConfiguration -Destination $JeaDestination -Recurse `
-Force
Register-PSSessionConfiguration -Path `
"$LocalConfiguration\MyJeaConfiguration.pssc" -name EM -Force
}

if (-NOT (Test-Path $LocalConfiguration\*.pssc)) {
Copy-Item -Path $RemoteConfiguration -Destination $JeaDestination -Recurse `
-Force
Register-PSSessionConfiguration -Path `
"$LocalConfiguration\MyJeaConfiguration.pssc" -name EM -Force
}

$RemoteConfigurationHash = Get-ChildItem $RemoteConfiguration -Recurse |
Get-FileHash | select @{Label="Path";
Expression={$_.Path.Replace($RemoteConfiguration,"")}},Hash
$LocalConfigurationHash = Get-ChildItem $LocalConfiguration -Recurse |
Get-FileHash | select @{Label="Path";
Expression={$_.Path.Replace($LocalConfiguration,"")}},Hash
$jeadiffEM = ''

Compare-Object $RemoteConfigurationHash $LocalConfigurationHash `
-Property Path,Hash -IncludeEqual -PassThru | 
Where-Object {$_.SideIndicator -ne '==' } |
ForEach-Object -process {
$jeadiffEM = 'y'
}

if ($jeadiffEM) {
Copy-Item -Path $RemoteConfiguration -Destination $JeaDestination -Recurse `
 -Force
Register-PSSessionConfiguration `
-Path "$LocalConfiguration\MyJeaConfiguration.pssc" -name EM -Force
}
```

## Summary 
Now you have JEA Sessions Configuration Files (Who) and Role Capability files (What) deployed to your servers. The possibilities are endless from here. When implementing  JEA in your environment start small in your lab environment and work your way up through DEV before setting this up in your production environment. If you do not take this is small chucks then you will not succeed in setting JEA for success. 
