# Getting Yesterday's Date


**Question:** How can I get yesterday's date?

**Answer:** Yes, using a combination of ``Get-Date`` and .NET Time/Date methods. 

First, let's look at dates in PowerShell.
Then we can look at how to calcualte yesterday and use that in your scripts.

## Tip of the Hat

This article is based on an earlier Scripting Guys blog article at: https://devblogs.microsoft.com/scripting/how-can-i-get-yesterdays-date/.
Not sure who wrote it.

## Dates in PowerShell

Let's start by looking at how you can deal with dates and times.
As you probably know, PowerShell contains the ``Get-Date`` cmdlet.
Using this cmdlet, you can get today's date and either display it or store it in a variable, like this:

```powershell
# Get the current date
Get-Date
# Store the date in a variable
$Now = Get-Date
```

The output looks like this:

```console
PS C:\> Get-Date

08 January 2021 11:24:46
# Store the date in a variable
$Now = Get-Date
```

The ``Get-Date`` cmdlet returns an object whose type is ``System.DateTime``.
This .NET structure provides a rich set of properties and methods to help you manipulate the date/time object.
See <https://docs.microsoft.com/dotnet/api/system.datetime> for more details on this structure.
A date and time structure contains both a date and a time.
This means you can create an object with just a date or just a time, or both, which gives you huge flexibility in handling dates and time.

If you run ``Get-Date`` and specify no parameters, the cmdlet returns the current date and time.
To alter this behavior, there are several parameters to this cmdlet.
These parameters allow you to create an object for a particular date, like this:

```powershell
# Using the -Date Parameter
Get-Date =Date '1 August 1942'
# Using more parameters
Get-Date -Month 8  -Day 1 -Year 1942
```

The output of these two commands is the same:

```console
PS C:\> Get-Date -Date '1 August 1942'

01 August 1942 00:00:00
PS C:\> > Get-Date -Month 8  -Day 1 -Year 1942 -Hour 0 -Minute 0 -Second 0
01 August 1942 00:00:00
```

If you use the ``-Date`` parameter, you specify the date in a culturally sensitive way.
If you are using ``Get-Date`` with a culture of ``EN-GB`` (The Queen's English), you would specify the date as '1 August 1942`
But if you are using a culture of ``EN-US`` (American English), you would specify the date as "August 1 1942'.
Using cultures and changing date/time input is the subject (or soon will be) of another blog article.

## Obtaining Yesterday's Date

You can use ``Get-Date``  to return a specific date/time.
So how do you get a date of yesterday, or last month or last year?
To do this, you create a date and time object for today (using ``Get-Date`` with no parameters).
Then you use the ``AddDays`` method to add some number of days where the number is negative, like this

```powershell
# Get today's Date
$Today = Get-Date
$Yesterday = $Today.AddDays(-1)
$Yesterday
# Or more simply
$Yesterday = (Get-Date).AddDays(-1)
$Yesterday
```

The output from this snippet looks like this:
```console

PS C:\> # Get today's Date
PS C:\> $Today = Get-Date
PS C:\> $Yesterday = $Today.AddDays(-1)
PS C:\> $Yesterday

07 January 2021 12:05:20

PS C:\> # Or more simply
PS C:\> $Yesterday = (Get-Date).AddDays(-1)
PS C:\> $Yesterday

07 January 2021 12:06:05
```

## Using Yesterday's date

There are a variety use cases for getting a date in the past (or the future), including 
**Identifying files that are older/younger than a day/month/etc ago
**Determining which AD Users have not logged on in the last week
**Creating a file name for a file representing last weeks information.

Here are some examples:

```console
PS C:> # Finding files newer than yesterday
PS C:> $Yesterday = (Get-Date).AddDays(-1)
PS C:> Get-ChildItem | Where-Object LastAccessTime -gt $Yesterday

    Directory: C:\
    
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a---          29/12/2020    21:42          11053 GratefulDead Show List.txt

PS C:> # Getting users who have logged on in the past day
PS C:> Get-ADUser -Filter * -Property LastLogonDate | Where LastlogonDate -gt $Yesterday

DistinguishedName : CN=Administrator,CN=Users,DC=cookham,DC=net
Enabled           : True
GivenName         : Jerry
LastLogonDate     : 08/01/2021 12:15:53
Name              : Jerry Garcia
ObjectClass       : user
ObjectGUID        : ae31ca0d-3f01-4eb4-8593-b1d79c71f912
SamAccountName    : JerryG
SID               : S-1-5-21-2550804810-443649076-1856842782-500
Surname           : Garcia

# Creating a file with yesterday's date
PS D:\> # Creating a file with today's date
PS D:\> $Yesterday     = (Get-Date).AddDays(-1).ToString() -replace '/','-'
PS D:\> $YesterDayDate = ($Yesterday -split ' ')[0]
PS D:\> $YesterdayFN   = "Results for $YesterdayDate.Txt"
PS D:\> 
PS D:\> New-Item -Path C:\Results -Name  $YesterdayFN -ItemType File

Directory: C:\Results

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a---          08/01/2021    12:56              0 Results for 07-01-2021.Txt
```

In that last example, you need to do a bit of manipulation of the date/time returned by ``Get-Date`` in order to get a filename that Windows accepts.
This is because ``Get-Date`` returns a string that contains the "/" character which confuses ``New-Item``.
You use the ``-Replace`` operator to replace the "/" character with a "-".
Additionalh, after performing the replacement, you end up with an (unneeded) time value.
You can use the ``-Split`` operator to pull out just the date
Once you do this, you can create you can create a file name and the file itself.

Needless to say, you could do all those file name manipulations operations as a one-liner. 
I will leave that as an exercise for you!

## Summary

.NET provides a rich date and time structure (``System.DateTime``).
This structure contains a number of properties such the day, month, hour, millisecond for a given date/time.
You also get a wide range of methods that enable you to manipulate dates by adding or subtracting hours, days, etc.
You can use ``Get-Date`` cmdlet to get the current date/time or an object for a specific date/time.
Finally, you can use the methods of the ``System.DateTime` structure to get relative dates, such as yesterday, last month or 2 years and 3 days ago.
