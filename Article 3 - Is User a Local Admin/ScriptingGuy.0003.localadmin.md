# Discovering Local Administrators

**Q:** Some of the things we do in our logon scripts require the user to be a local administrator. How can the script tell if the user is a local admin or not?

**A:**  Easy using PowerShell 7 and its LocalAccounts module


This article was originally a VBS based solution as described here: <https://devblogs.microsoft.com/scripting/how-can-i-determine-if-a-user-is-a-local-administrator/>
Not sure who the author was - but thanks!


## Local Users and Groups

Before looking at the answer to this great question, let's review local users and groups.
Every Windows system, except for Domain Controllers, maintains a set of local users and groups. 
These local users and groups are in addition to domain users and domain groups.
On Domain Controllers you just have the Active Directory users;

As with AD groups, Local groups and Local users each have a Security ID (SID).
If you give a local user or group access to a file or folder, Windows adds tha SID to the object's Access Control List.
A cool feature of Local Groups is that they can contain both local users AND domain users
This is the same way Windows enables you to give permissions to a local file or folder to any user or group in your AD>.

In a work group environment, you can use a local group or local user when setting all permissions to any file and folder.
This approach works but does not scale very well.
For example, the local user JerryG might exist on one host - but may not on others.
for cross-workgroup use, you would nee the same users defined on each system.
But even so, you are left wth the interesting issue of keeping passwords etc all in sync.
As a word of advice, just don't go there if you can possibly avoid it - use a domain and domain users/gorups.
But there are times when you need to know if the logged on user IS an Administrator.

Traditionally, you might have used the ``Wscript.Network`` COM object, in conjunction with ADSI. 
You can, of course, use the older approach in side PowerShell, but why bother?
The good news is that you do not have to - there is an easier way.
With PowerShell 7, you can use a new module with great cmdlets, the ``Microsoft.PowerShell.LocalAccounts`` module.

## The Microsoft.PowerShell.LocalAccounts module

In PowerShell 7 for Windows, you can use the ``Microsoft.PowerShell.LocalAccounts`` module to manage local users and group.
This module contains 15 cmdlets, which you can view like this:

```console
PS C:\> Get-Command -Module Microsoft.PowerShell.LocalAccounts

CommandType     Name                       Version    Source
-----------     ----                       -------    ------
Cmdlet          Add-LocalGroupMember       1.0.0.0    Microsoft.PowerShell.LocalAccounts
Cmdlet          Disable-LocalUser          1.0.0.0    Microsoft.PowerShell.LocalAccounts
Cmdlet          Enable-LocalUser           1.0.0.0    Microsoft.PowerShell.LocalAccounts
Cmdlet          Get-LocalGroup             1.0.0.0    Microsoft.PowerShell.LocalAccounts
Cmdlet          Get-LocalGroupMember       1.0.0.0    Microsoft.PowerShell.LocalAccounts
Cmdlet          Get-LocalUser              1.0.0.0    Microsoft.PowerShell.LocalAccounts
Cmdlet          New-LocalGroup             1.0.0.0    Microsoft.PowerShell.LocalAccounts
Cmdlet          New-LocalUser              1.0.0.0    Microsoft.PowerShell.LocalAccounts
Cmdlet          Remove-LocalGroup          1.0.0.0    Microsoft.PowerShell.LocalAccounts
Cmdlet          Remove-LocalGroupMember    1.0.0.0    Microsoft.PowerShell.LocalAccounts
Cmdlet          Remove-LocalUser           1.0.0.0    Microsoft.PowerShell.LocalAccounts
Cmdlet          Rename-LocalGroup          1.0.0.0    Microsoft.PowerShell.LocalAccounts
Cmdlet          Rename-LocalUser           1.0.0.0    Microsoft.PowerShell.LocalAccounts
Cmdlet          Set-LocalGroup             1.0.0.0    Microsoft.PowerShell.LocalAccounts
Cmdlet          Set-LocalUser              1.0.0.0    Microsoft.PowerShell.LocalAccounts
```
The names of these cmdlets is a clue to what they do, as with all cmdlets.
The cmdlets allow you to add, remove, change, enable and disable a local user or local group
And they allow you to add, remove and get the local group's members.
These cmdlets are broadly similar to the ActiveDirector Cmdlet but work on local users.
And as noted above, you can use domain users/groups as a member of a local group should you wish to.

The best way to do that is to check the membership of the local Administrator’s group and see if the user is in there. 
You use the ``Get-LocalGroupMember`` command to view the members of a local group, like this:

```console
PS C:\> Get-LocalGroupMember "Administrators"

ObjectClass Name                     PrincipalSource
----------- ----                     ---------------
Group       COOKHAM\Domain Admins    ActiveDirectory
User        COOKHAM24\Administrator  Local
User        COOKHAM\Bobby            ActiveDirectory
User        COOKHAM24\Dave           Local
```
As you can see in this output fragment, the local Administrators group contains domain users and groups as well as local users. 

## Is the User an Administrator?

It's easy to get membership of any local group, as you saw above.
But what if you want to find out if a given user is a member of some group?
Or, determine whether the user running a given script is a member of the administrator's group?
That too is pretty easy and take a couple of steps.
You get the name of the current user, using ``whoami.exe``.
Then you get the members of the Administrator's group.
Finally, you check to see if the currently logged on user is a member of the group, which looks like this:

```console
# Get who I am
$Me = whoami.exe

# Get members of administrators group
$Admins = Get-LocalGroupMember -Name Administrators | Select-Object -ExpandProperty name

# Check to see if this user is an administrator and act accordingly
if ($Admins -Contains $me) {
  "$Me is a local administrator"} 
else {
  "$Me is NOT a local administrator"}
Cookham\JerryG is a local administrator
```

If the user running the script, then $Me is a user in the local administrator's group.
And if the current user is in the group, that that means that the logged-on user must be a local administrator; otherwise he or she wouldn’t be a member of the Administrators group.

In this sample snippet, we just echo the fact that the user is, oir is not, a local administrator.
If this was a logon script, for example, then once you know that the user is a local admin, you go ahead and carry out any tasks that require admin privileges.
And if the user is not a local admin, you could echo that fact, and avoid using a cmdlet that requires the user to be an admin.

## Summary
Using the Local Accounts module, it's easy to manage local groups! 