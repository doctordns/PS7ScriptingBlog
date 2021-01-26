# Discovering Changes in AD

**Q:** I have a log file in which new data is appended to the end of the file.
That means the most recent entries are at the end of the file. 
I’d like to be able to read the file starting with the last line and then ending with the first line, but I can’t figure out how to do that.

**A:**  There are loads of ways you can do this. 
A simple way is to use the power of array handling in PowerShell.

## Get-Content
Before getting into the solution, let's look at ``Get-Content``.
The ``Get-Content`` cmdlet allows you to read a file, but it always does so from front to end.
You can always get the very last line of the file like this:

```powershell
Get-Content -Path C:\Foo\BigFile.txt |
  Select-Object -Last 1
```

This is similar to the ``tail`` command in Linux.
As is so often the case, the command doesn't quite do what you want it to.
That being said, with PowerShell 7, there's _always_ a way.

## Using Arrays

In this case, we start by reading through the file from top to bottom.
But before displaying those lines to the screen we store them in an array, with each line in the file representing one element in the array.
An array is a collection of objects - in this case the collection of the lines in your file.
Once you have the lines in the array, you can work backwards to achieve your goal.

## Creating a simple file

To demonstrate it, we create a simple file, and output it, like this

```powershell
$File = @'
violet
indigo
blue
green
yellow
orange
red
'@
$File | Out-File -Path C:\Foo\SmallFile.txt
Get=ChildItem -Path C:\SmallFile.txt
```

The output of this step looks like:

```powershell
PS C:\Foo> $File = @'
>> violet
>> indigo
>> blue
>> green
>> yellow
>> orange
>> red
>> '@
PS C:\Foo> $File | Out-File -Path C:\Foo\SmallFile.txt
PS C:\Foo> ls .\SmallFile.txt

    Directory: C:\Foo

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a---          22/01/2021    20:13             44 SmallFile.txt

Once you have the file created you can get the contents and display it, like this:

```powershell
PS C:\Foo> $Array = Get-Content -Path C:\Foo\SmallFile.txt
PS C:\Foo> $Array
violet
indigo
blue
green
yellow
orange
red
```

Admittedly, all we seem to have done so far is get back to where we started.
So how do we get to where you want to go?

## Arrays vs text files

There’s an important difference between a text file and an array.
With a text file, using ``Get-Content``, you read it from top to bottom.
However, it’s actually pretty easy to read an array from the bottom to the top.

Let's start by working out how many lines there are in the array.
And, more as a sanity check, you can display how many lines there are in the file, like this:

```powershell
PS C:\Foo> $Array = Get-Content -Path C:\FOo\SmallFile.txt
PS C:\Foo> $length = $Array.count
PS C:\Foo> "There are $Length lines in the file"
There are 7 lines in the file
```
## Getting Array Members

In our sample array, ``$Array`` we have 7 lines.
In PowerShell we can get to any individual array member by using ``[<index>]`` to get the appropriate line.
In PowerShell the first item is 0 (or ``$Array[0]``).
Thus violet has an index number of 0 - ``$Array[0]``.
And  red has an index number of 6 - ``$Array[6]``.
But that doesn't help us much - just yet!

A particularly neat feature of PowerShell is that we can work backwards in an array
An index of [-1] is always the last element, [-2] is the penultimate line, and so on.
So ``$Array[-1]`` is Red, ``$Array[-2]`` is Orange, and so on.

So what we do is to look first at $Array[-1], then $Array[-2], and so on, like this:

```powershell
$Array = Get-Content -Path C:\Foo\SmallFile.txt
$length = $Array.count
"There are $Length lines in the file"
$Line = 1
1..$length | foreach {$Array[-$line]; $Line++}
```

This first sets a variable, $Line to 1.
Then you use ``foreach`` to run a script block.
Inside the script block you get the array element starting at the end.
Then you increment the line number and repeat.

Put all together you see this:

```powershell
PS C:\Foo> $Array = Get-Content -Path C:\FOo\SmallFile.txt
PS C:\Foo> $length = $Array.count
PS C:\Foo> "There are $Length lines in the file"
There are 7 lines in the file
PS C:\Foo> $Line = 1
PS C:\Foo> 1..$length | foreach {$Array[-$line]; $Line++}
red
orange
yellow
green
blue
indigo
violet

```

This may be a little confusing if you haven;t work with arrays, but once you get the hang of it, you'll see how simple it really is.

## Summary

So as you saw, ``Get-Content`` does not read backwards through a file.
If you bring the file contents into an array, you can easily read it backwards.
For more information on arrays in PowerShell, see: <https://docs.microsoft.com/powershell/module/microsoft.powershell.core/about/about_arrays>