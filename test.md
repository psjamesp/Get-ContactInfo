# Just Enough Administration

By James Petty

## What is JEA
Just Enough Administration (JEA for the rest of this chapter) is a way to whitelist PowerShell commands that someone can run on a remote machine. It is another layer of security that you can implement in your environment. Its main purpose is to limit the number of users who have administrative prilvedges on machines. This is done by leveraging the built in capabilities already found in Windows PowerShell. JEA allows an administrator to custom taylor a PowerShell enviornment for someone that has all the rights that are needed to perform his/her task but nothing more. Whether that is to restart the spooler service or to kill a process. 

The three main reasons for using JEA are
* Whitelist the commands and pramaers that a person can use
* Limit the number of administratrs on each machine
* Know what your users are doing on the machines

Pleae note the full list of scripts will be available for download with this book. There for not all of my PowerShell code will be displayed. 

## Prerequisites
To preparte your environment you need to make sure the following items are in place
* Make sure you have the most up to date version of Windows PowerShell (WMF5.1.x) on your servers and workstations
    * It should be noted that although Windows Server 2008 R2 can run WMF 5.1 JEA will have slightley limited cabilities
* PowerShell Remoting must be enabled 

   ```powershell 
   Enable-PSRemoting 
   ```

* It's suggested that you also turn on script block logging but not required

## How JEA Works
JEA is a mechnisam that allows for stricter control of capbilities on endpoints. This is accomplished by creating endpoints on any computer and using Configuration Files (the who) and Role Capability (the what) files you can manage who can do what on each machine. 

### Session Configuration File (the who)
This file allows you to configure who can access the system by using individual Domain Accounts or Domain Global Groups. You can also set global settings such as virtual accounts and logging in this file as well. Each machine must have a configuartion file, so this allows you to configure access on a per-machine basis. The file extension is PSSC. The Microsoft default location for htis file is env:ProgramData\JEAConfiguration.

#### How to create and Configure a Configuration File
Lucky for us there Microsoft has provided us with the `New-PSSessionConfigurationFile` command that will create the file for us. 
First we need to make our JEA direcotry in ProgramData direcotry

Then we are going to set the settings that we want
*Session Type is set to Restricted. This allows us to run the following commands
    *Exit-PSSession, Get-Command, Get-FormatData, Get-Help, Measure-Object, Out-Default, and Select-Object
*TranscriptDirectory is where the transcript will live once. You can put this anywhere you wish. It is highley recommended that you enable this feature
*RunAsVirtualAccount uses a virtual administrative account that the user connects to. This account is unknown to the user and the user cannot use this account in anyother way . The virtual account destroys its self when the session is closed. 
*Role Definations is what roles the user can access
*Path is where you are going to save the pssc file

```powershell
$ConfFileData = @{
    SessionType = "RestrictedRemoteServer"
    TranscriptDirectory = "C:\ProgramData\JEATranscripts"
    RunAsVirtualAccount = $true
    RoleDefinitions = @{'Demo\User1'= @{ RoleCapabilities = 'Demo' }}
    Path = "$env:ProgramData\JEAConfiguration\Demo.pssc"
}
New-PSSessionConfigurationFile @ConfFileData
```
Another option that you can enable is the `MountUserDrive = $True` this will mount the user's drive to the remote session as a PSDrive. This can be handy for neeing to copy files to the remote server, but can come with its own security risks. You will need to decided if this is something that you would like to use in your enviornment. 

Lets come back to the Role Definitions for a little bit. In our example above User1 has the ability to connect to the 'Demo' Role. But what if you you wanted to have multiple roles? You are in luck becuase that option is built in.
`RoleDefinitions = @{'Demo\User1'= @{ RoleCapabilities = 'Demo','PrintSpooler','DNSOperator' }}` 
This way you do not have to put everything in one configuration file (we will get ot that later). But will allow you to seperate which users can access what. Again, our goal with JEA to give people the tools they need to perform their job, no more and no less than that.
We can also have multiple accounts with different roles. Lets say you have a job that you want the HelpDesk to take care of, but the server is also a DNS server so your DNS admins need to perform administrative functions as well.

`RoleDefinitions = @{'Demo\HelpDeskGroup'= @{ RoleCapabilities = 'HelpDeskFunction' }}`

`RoleDefinitions = @{'Demo\DNSAdmin'= @{ RoleCapabilities = 'DNSAdminRole' }}`

`RoleDefinitions = @{'Demo\Tier2Support= @{ RoleCapabilities = 'HelpDeskFunction','DNSAdminRole','Tier2Support'}}`

###Role Capability Files (the what)
