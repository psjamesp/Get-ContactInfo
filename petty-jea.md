#J ust Enough Administration

By James Petty

## What is JEA

Just Enough Administration (JEA for the rest of this chapter) is another layer of security that you can implement in your environment. Its main purpose is to limit the number of administrators that you have. This is done by leveraging the built in capabilities already found in Windows PowerShell. JEA allows an administrator to custom taylor a PowerShell enviornment for someone that has all the rights that are needed to perform his/her task but nothing more. Whether that is to restart the spooler service or to kill a process. 

*Whitelist the commands and pramaers that a person can use
*Limit the number of administratrs on each machine
*Know what your users are doing on the machines

## Prerequisites

To preparte your environment you need to make sure the following items are in place
*Make sure you have the most up to date version of Windows PowerShell (WMF5.1.x) on your servers and workstations
    *It should be noted that although Windows Server 2008 R2 can run WMF 5.1 JEA will have slightley limited cabilities
*PowerShell Remoting must be enabled ```Enable-PSRemoting

## How to implement JEA