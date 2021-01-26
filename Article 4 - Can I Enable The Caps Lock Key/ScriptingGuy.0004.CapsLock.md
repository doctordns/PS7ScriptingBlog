# Can I Enable the Caps Lock Key??

**Q:** I have a script where users enter some information in an Input box.
The information needs to be entered in all-capital letters, so my instructions say, “Please make sure the Caps Lock key is on before entering the information.”
They don’t always do that, however.
Is there a way to turn the Caps Lock key on and off using a script?

**A:**  I don't know how to run the key off and on, but with PowerShell there is a way to mimic the effect of haiving the caps lock key on.

## Tip of the Hat
This artile is based on an earlier article here: https://devblogs.microsoft.com/scripting/can-i-enable-the-caps-lock-key/
I have re-developed the article around PowerShell.

## User Input Considered Harmful

In a tip of the hat to one of my earliest IT heros, Eesger Dijkstra, I believe that all user input is dangerous until proven otherwise.
As an aside, Dijkstra published a letter "Go To Statement Considered Harmful" in the late 1960s which began the structured programming revolution.
And this is one reason why PowerSHell has no goto statement.
The phrase "COnsidered Harmful" is also well know and even has a wikipedia entry at <https://en.wikipedia.org/wiki/Considered_harmful#:~:text=Considered%20harmful%20was%20popularized%20among,the%20day%20and%20advocated%20structured>

Is user input really harmfulo?
At university, I was a verification programmer and got paid an hourly rate plus a bonus for finding bugs.
I made far more than my hourly wage by testing out conditions that were outside what was "normal".
If an instruction said "Enter a number betwen 1 and 6", i tried -124, 0, 8, 124 - which led to several bugs (and bug bounties)
Ever since then, I have always taught my students to never accept user input unchecked.
And don't forget that unchecked input is one cause of SQL injection attacks.

You should never trust any user input without validating it first.
Although you ask the user to type her name in all upper case, I'll bet that many just won't.
So what are you to do?

## The Caps Lock Key

The point of the Caps Lock key is to turn everything you type into uppercase letters. For example, you might type this:

```console
this is my sentence.
```

But using the Caps Lock makes it appear on screen like this:

```console
THIS IS MY SENTENCE.
```

So how can we achieve the same affect in a script? 
Simple: we just accept the input as the user typed it, then use the .ToUpper() method.
This method converts any string to all upper case.

## Getting user input

There are several ways to get user input, but a common one is to use the ``Read-Host` command. 
Suppose you wanted to ask the user for their name (and wanted it to be upper case).
You could do something like this this:

```powershell
$Answer = Read-Host -Prompt "Enter Your Name In ALL Upper case"
```

From the console, you would see this:

```console
PS C:\Foo> $Answer = Read-Host -Prompt "Enter Your Name In ALL Upper case"
Enter Your Name In ALL Upper case: Thomas Lee
PS C:\Foo> $Answer
Thomas Lee
```

But that is not in upper case, I hear you say.
Yes, true - but there is just one more step.
Be patient grasshopper.

## Converting a string to upper case

WHen you use ``Read-Host``, the result is a string.
If you enter a number (say 42) PowerShell still treats this as a string containing two characters, like this:

```console
PS C:\> $Answer = Read-Host -Prompt "Enter Your Name I ALL Upper case"
Enter Your Name In ALL Upper case: 42
PS C:\> $answer.GetType().Fullname
System.String
```

This matters because the ``System.String`` .NET class has a very useful method, **ToUpper()**.
This method converts the string to all upper case.
So to convert the string you entered and stored in **$Answer**, you use the **ToUpper()** method like this:

```console
PS C:\> $Answer = Read-Host -Prompt "Enter Your Name In ALL Upper case"
Enter Your Name In ALL Upper case: Thomas Lee
PS C:\> $Answer
Thomas Lee
PS C:\> $Answer.ToUpper()
THOMAS LEE
```

Strings also have other methods, including ToLower() that change a string to all lower case.
You can always discover the methods of a string (or any other variable type) by piping the variable to ``Get-Member``
Like this

```console
PS C:\ $Answer | Get-Member -MemberType Method

   TypeName: System.String

Name                 MemberType Definition
----                 ---------- ----------
Clone                Method     System.Object Clone(), System.Object ICloneable.Clone()
CompareTo            Method     int CompareTo(System.Object value), int CompareTo(string strB), int IComparabl…
Contains             Method     bool Contains(string value), bool Contains(string value, System.StringComparis…
CopyTo               Method     void CopyTo(int sourceIndex, char[] destination, int destinationIndex, int cou…
EndsWith             Method     bool EndsWith(string value), bool EndsWith(string value, System.StringComparis…
EnumerateRunes       Method     System.Text.StringRuneEnumerator EnumerateRunes()
Equals               Method     bool Equals(System.Object obj), bool Equals(string value), bool Equals(string …
GetEnumerator        Method     System.CharEnumerator GetEnumerator(), System.Collections.IEnumerator IEnumera…
GetHashCode          Method     int GetHashCode(), int GetHashCode(System.StringComparison comparisonType)
GetPinnableReference Method     System.Char&, System.Private.CoreLib, Version=5.0.0.0, Culture=neutral, Public…
GetType              Method     type GetType()
GetTypeCode          Method     System.TypeCode GetTypeCode(), System.TypeCode IConvertible.GetTypeCode()
IndexOf              Method     int IndexOf(char value), int IndexOf(char value, int startIndex), int IndexOf(…
IndexOfAny           Method     int IndexOfAny(char[] anyOf), int IndexOfAny(char[] anyOf, int startIndex), in…
Insert               Method     string Insert(int startIndex, string value)
IsNormalized         Method     bool IsNormalized(), bool IsNormalized(System.Text.NormalizationForm normaliza…
LastIndexOf          Method     int LastIndexOf(char value), int LastIndexOf(char value, int startIndex), int …
LastIndexOfAny       Method     int LastIndexOfAny(char[] anyOf), int LastIndexOfAny(char[] anyOf, int startIn…
Normalize            Method     string Normalize(), string Normalize(System.Text.NormalizationForm normalizati…
PadLeft              Method     string PadLeft(int totalWidth), string PadLeft(int totalWidth, char paddingCha…
PadRight             Method     string PadRight(int totalWidth), string PadRight(int totalWidth, char paddingC…
Remove               Method     string Remove(int startIndex, int count), string Remove(int startIndex)
Replace              Method     string Replace(string oldValue, string newValue, bool ignoreCase, cultureinfo …
Split                Method     string[] Split(char separator, System.StringSplitOptions options), string[] Sp…
StartsWith           Method     bool StartsWith(string value), bool StartsWith(string value, System.StringComp…
Substring            Method     string Substring(int startIndex), string Substring(int startIndex, int length)
ToBoolean            Method     bool IConvertible.ToBoolean(System.IFormatProvider provider)
ToByte               Method     byte IConvertible.ToByte(System.IFormatProvider provider)
ToChar               Method     char IConvertible.ToChar(System.IFormatProvider provider)
ToCharArray          Method     char[] ToCharArray(), char[] ToCharArray(int startIndex, int length)
ToDateTime           Method     datetime IConvertible.ToDateTime(System.IFormatProvider provider)
ToDecimal            Method     decimal IConvertible.ToDecimal(System.IFormatProvider provider)
ToDouble             Method     double IConvertible.ToDouble(System.IFormatProvider provider)
ToInt16              Method     short IConvertible.ToInt16(System.IFormatProvider provider)
ToInt32              Method     int IConvertible.ToInt32(System.IFormatProvider provider)
ToInt64              Method     long IConvertible.ToInt64(System.IFormatProvider provider)
ToLower              Method     string ToLower(), string ToLower(cultureinfo culture)
ToLowerInvariant     Method     string ToLowerInvariant()
ToSByte              Method     sbyte IConvertible.ToSByte(System.IFormatProvider provider)
ToSingle             Method     float IConvertible.ToSingle(System.IFormatProvider provider)
ToString             Method     string ToString(), string ToString(System.IFormatProvider provider), string IC…
ToType               Method     System.Object IConvertible.ToType(type conversionType, System.IFormatProvider …
ToUInt16             Method     ushort IConvertible.ToUInt16(System.IFormatProvider provider)
ToUInt32             Method     uint IConvertible.ToUInt32(System.IFormatProvider provider)
ToUInt64             Method     ulong IConvertible.ToUInt64(System.IFormatProvider provider)
ToUpper              Method     string ToUpper(), string ToUpper(cultureinfo culture)
ToUpperInvariant     Method     string ToUpperInvariant()
Trim                 Method     string Trim(), string Trim(char trimChar), string Trim(Params char[] trimChars)
TrimEnd              Method     string TrimEnd(), string TrimEnd(char trimChar), string TrimEnd(Params char[] …
TrimStart            Method     string TrimStart(), string TrimStart(char trimChar), string TrimStart(Params c…
```

If you look carefully at this list, you can see methods that convert a string to different kinds of numbers.
These methods would help you convert the string of 2 characers (e.g. 42) into an integer number.
I plan to have another article that shows you how to acheive thie.

## Summary

Turning the Caps Lock key on is not something I know how to do.
And if you did it might confuse the user (when she sees the Caps Lock indicator light up on her keyboard).
As well, you would need to turn it off.

But rather then trying, or depending on any user to always do the right thing, you can always ensure that the input is indeed in all upper case.
