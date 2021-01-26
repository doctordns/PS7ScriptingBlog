# Discovering Changes in AD

**Q:** Is there an easy way to detect when someone makes changes to an AD group or user?

**A:**  There are loads of ways you can do this. You could take a dump of all the users, then create a second dump and do a diff.
I do not recommend it, even it could work.
I am told there are other ways, but I really love WMI!

## Tip of the Hat

As is so often the with PowerShell, posts like this one would probably not been possible without work done by others.
The inspiration for this post came from a Tweet by Jeff Hicks.
Jeff published (and tweeted) a script you can see at: https://gist.github.com/jdhitsolutions/845b4f84117f6145cdda1babe2d81298
If you are into PowerSHell, you should follow @JeffHicks.
You can also read his most excellent blog at <https://jdhitsolutions.com/blog/>.
Thanks Jeff for yet another great tweet and post!

## Windows Management Instrumentation - WMI

Windows Management Instrumentation (WMI) is a feature of Windows
WMI first saw the light of day as an add-in for WIndows NT 4.0 and has been improved greatly since then.
I assume you are broadly familiar with WMI, including classes, namespaces, and the like.
If not, you could take a look at my latest PowerShell book which contains a chapter on WMI.
Alternatively, use your favorite search engine - there is literally tonnes of great content on WMI.

WMI contains classes - each class represents one specific bit of information.
The ``Win32_Bios`` class contains details of your host's BIOS.
You can use the ``Win32_Share`` class to view information on SMB shares on your host.

In WMI, classes are organized under hierarchical namespaces.
Most of the interesting classes exist in the ``root\cimv2`` namespace, but there are others.
If you issue a simple ``Get-CimInstance`` command and specify class, WMI defaults to looking into that namespace.
If the class exists in another WMI Namespace, you just specify the namespace when you use ``Get-CimInstance``

## WMI and AD

In WMI, MIcrosoft provides an entire WMI namespace that instruments AD operations.
The WMI Namespace is ``root\directory\ldap`` and it contains over 500 classes
In this namespace, you can find classes that contain the same data as in AD as well as event classese.
You can use WMI event classes to subscript to any event and get a notification when the event occurs.

One particularly cool feature of this namespace is that it exists on a domain-joined Windows 10 PC. 
Which means that you can query WMI directly without needing to load the RSAT tools.

You can discover the classes using the ``Get-CimClass`` cmdlet, like this:

```console
PS C:\> Get-CimClass -Namespace 'root\directory\ldap'

   NameSpace: ROOT/directory/ldap

CimClassName                        CimClassMethods      CimClassProperties
------------                        ---------------      ------------------
__SystemClass                       {}                   {}
__thisNAMESPACE                     {}                   {SECURITY_DESCRIPTOR}
__Provider                          {}                   {Name}
__Win32Provider                     {}                   {Name, ClientLoadableCLSID, CLSID, Concurrency, DefaultMachin…
__ProviderRegistration              {}                   {provider}
... and a whole lot more!
```

There are a LOT of classes, so don't be overwhelmed!

So, I can almost hear you ask, "_How can you use these classes to find out when someone adds or changes an object in AD_"
Turns out it is pretty easy!

In my recent Wiley book on PowerSHell (cheap plug: <https://smile.amazon.co.uk/PowerShell-Professionals-Manage-Windows-Systems-ebook/dp/B08PJCLTNZ/>) I devote an entire chapter to WMI.
In that chapter, I have two scripts that deal with WMI eventing. 
You can get these scripts at https://github.com/doctordns/Wiley20/tree/master/09%20-%20WMI

The "Managing WMI Events" script, you can see how to register for WMI events and get event data. 
One downside to this script is that you have to be running PowerShell to use WMI eventing.
You can adapt this script to make use of WMI Permanent Event Handling which enables your host to use WMI eventing even if you are not even logged in.
I show this in "Implement permanent WMI Event Handling"

A word of caution - WMI permanent event handling is just that - permanent.
If you add a permanent event handler, it keeps running even after a reboot
The first thing I teach with this technology - know how to turn it off!

So let's take a look at how to use ordinary event handling.
.

## WMI AD Eventing

In a post today, Jeff pointed to a GitHub gist (<https://t.co/oeMr6915Eq>).
This gist does the job, but the directions contained a minor typo in the instructions.

Here is an updated version of the script:

```powershell
Function Get-WmiADEvent {
    Param(
       [string]$Query
     )
  
    $Path="root\directory\ldap"
    $EventQuery  = New-Object System.Management.WQLEventQuery $query
    $Scope       = New-Object System.Management.ManagementScope $Path
    $Watcher     = New-Object System.Management.ManagementEventWatcher $Scope,$EventQuery
    $Options     = New-Object System.Management.EventWatcherOptions
    $Options.TimeOut = [System.Timespan]"0.0:0:1"
    $Watcher.Options = $Options
#    cls
    Write-Host ("Waiting for events in response to: {0}" -F $EventQuery.QueryString)  -backgroundcolor cyan -foregroundcolor black
    $Watcher.Start()
    while ($true) {
       trap [System.Management.ManagementException] {continue}
  
       $Evt=$Watcher.WaitForNextEvent()
        if ($Evt) {
           $Evt.TargetInstance | select *
        Clear-Variable evt
        }
    }
  }
```

To keep this post short, this function contains no comment based help or other production oriented additions.
Adding that is a task for another day!

## How does this work

After defining the function, you call it with a particular WMI query to let the function know you what you want to detect.
You can use it like this:

```powershell
# To detect adding a new user
$Query="Select * from __InstanceCreationEvent Within 10 where TargetInstance ISA 'DS_USER'"
Get-WmiADEvent $Query

# To detect creating a new group
$Query="Select * from __InstanceCreationEvent Within 10 where TargetInstance ISA 'DS_GROUP'"
Get-WmiADEvent $Query

# To detect changing a user
$Query="Select * from __InstanceModificationEvent Within 10 where TargetInstance ISA 'DS_USER'"
Get-WmiADEvent $Query

# To detect changing a computer object
$Query="Select * from __InstanceModificationEvent Within 10 where TargetInstance ISA 'DS_COMPUTER'"
Get-WmiADEvent $Query
```

## What is the output

When you invoke this function with a suitable WMI query, the function subscribes to the event and waits for the event to fire.
Upon the addition or update of an AD object (that is the one you specify in the query), the function outputs what it knows.

To test this, you run the snippet above on one console window. 
From a separate PowerShell console you make a change to a user, like this:

```console
Get-AdUser -Identity jerryg -Properties Office | Set-AdUser -Office "Marin"
```

The console you are monitoring from looks like this:

```console
PSH [C:\Foo]: $Query="Select * from __InstanceModificationEvent Within 10 where TargetInstance ISA 'DS_USER'"
PSH [C:\Foo]: Get-WmiADEvent -Query $Query

Waiting for events in response to: select * from __InstanceModificationEvent within 10 where TargetInstance ISA 'DS_USER'

PSComputerName                                           : COOKHAM24
ADSIPath                                                 : LDAP://CN=Jerry Garcia,OU=CookhamHQ,DC=cookham,DC=net
DS_accountExpires                                        : 9223372036854775807
DS_badPasswordTime                                       : 0
DS_badPwdCount                                           : 0
DS_cn                                                    : Jerry Garcia
DS_description                                           : {Jerome Rocks!}
DS_displayName                                           : Jerry Garcia
DS_distinguishedName                                     : CN=Jerry Garcia,OU=CookhamHQ,DC=cookham,DC=net
DS_givenName                                             : Jerry
DS_instanceType                                          : 4
DS_lastLogoff                                            : 0
DS_lastLogon                                             : 0
DS_logonCount                                            : 0
DS_memberOf                                              : {CN=AAGlobal,OU=CookhamHQ,DC=cookham,DC=net}
DS_name                                                  : Jerry Garcia
DS_physicalDeliveryOfficeName                            : Marin
DS_primaryGroupID                                        : 513
DS_sAMAccountName                                        : JerryG
DS_sn                                                    : Garcia
DS_userPrincipalName                                     : JerryG@cookham.net
```

**NOTE:**
The output above is trimmed to show key user information.
The actual output generated is a LOT longer and contains mostly empty properties.

## What about permanent event handling

You could certainly update this script to be use permanent event handling.
Above, you see a reference to my script to do permanent event handling which you could easily adapt.
To detect al the activities, you would need to add several permanent event handlers.
But be careful and learn how to remove the event handlers before you implement this on a production server!!
I leave this both as an exercise for the reader and as a topic for a future blog pos.

## Summary

With WMI, it's simple to detect any change to AD users, groups, computers, or more.
One thing this script does NOT tell you is the userid of he user making the change.
WIth that said, this ia good solution and I commend it to the community.