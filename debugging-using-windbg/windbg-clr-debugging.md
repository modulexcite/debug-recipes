
Debugging .NET using WinDbg
===========================

CLR debugging setup
-------------------

### Loading SOS ###

You may use a great WinDbg plugin (**procdumpext**) by Andrew Richards for loading SOS:

```
0:000> .load c:\tools\diagnosing\Debugging Tools for Windows\_winext-x64\ProcDumpExt.dll
=========================================================================================
 ProcDumpExt v6.4 - Copyright 2013 Andrew Richards
=========================================================================================
0:000> !loadsos
```

However, recently I usually use any command from **netext** (eg. **!wver**) and make it load sos automatically, eg.:

```
0:000> .load netext
netext version 2.1.2.5000 Jan 21 2016
License and usage can be seen here: !whelp license
Check Latest version: !wupdate
For help, type !whelp (or in WinDBG run: '.browse !whelp')
Questions and Feedback: http://netext.codeplex.com/discussions
Copyright (c) 2014-2015 Rodney Viana (http://blogs.msdn.com/b/rodneyviana)
Type: !windex -tree or ~*e!wstack to get started

0:000> !wver
Runtime(s) Found: 1
0: Filename: mscordacwks_Amd64_Amd64_4.6.1080.00.dll Location: C:\Windows\Microsoft.NET\Framework64\v4.0.30319\mscordacwks.dll
.NET Version: v4.6.1080.00
NetExt (this extension) Version: 2.1.2.5000
0:000> .chain
Extension DLL search Path:
...
Extension DLL chain:
    netext: image 2.1.2.5000, API 1.0.0, built Thu Jan 21 17:33:00 2016
        [path: C:\tools\diag\Debugging Tools for Windows\_winext-x64\netext.dll]
    C:\Windows\Microsoft.NET\Framework64\v4.0.30319\SOS.dll: image 4.6.1080.0, API 1.0.0, built Tue Apr 12 03:02:38 2016
        [path: C:\Windows\Microsoft.NET\Framework64\v4.0.30319\sos.dll]
...
```

### Get help for a SOS command ###

SOS commands sometimes get overriden by other extensions help files. In such a case just use `!sos.help <cmd>` command, eg.

    0:000> !sos.help !savemodule
    -------------------------------------------------------------------------------
    !SaveModule <Base address> <Filename>
    ...


Usage examples
--------------

### Save a module from a dump (!savemodule) [SOS] ###

    !SaveModule <Base address> <Filename>

This command allows you to save a module from a memory dump to a disk. The base address can be found by using `lm` command. Example:

    0:000> lm
    start    end        module name
    00d50000 00d96000   log4net    (deferred)
    00e70000 00e7e000   Products_Service   (deferred)
    ...
    0:000> !SaveModule 00e70000 c:\temp\Products.Service.dll
    3 sections in file
    section 0 - VA=2000, VASize=66e4, FileAddr=200, FileSize=6800
    section 1 - VA=a000, VASize=3b0, FileAddr=6a00, FileSize=400
    section 2 - VA=c000, VASize=c, FileAddr=6e00, FileSize=200

To save all modules in a given dump you may use **!for\_each\_module** command:

    !for_each_module !savemodule @#Base C:\path_to_save\${@#ModuleName}.dll

### Break when specific exception occurs [SOS] ###

Break when `NullReferenceException` is thrown:

    !sxe -c "!soe System.NullReferenceException 1;.if (@$t1 != 1) { g; }" clr

### Find a value of a static field [SOSEX,Netext] ###

In SOS you need to first find the EEClass address for a given type (by calling for instance `!DumpMT`) and then use `!DumpClass`:

    0:000> !DumpClass /d 00c1af5c
    Class Name:      LowLevelDesign.Diagnostics.Musketeer.Config.SharedInfoAboutApps
    mdToken:         02000011
    File:            c:\tools\musketeer\Musketeer.exe
    Parent Class:    698239a4
    Module:          002c2edc
    Method Table:    00be97c0
    Vtable Slots:    9
    Total Method Slots:  b
    Class Attributes:    100101
    Transparency:        Critical
    NumInstanceFields:   4
    NumStaticFields:     2
          MT    Field   Offset                 Type VT     Attr    Value Name
    689a45c8  4000044        4 ...derWriterLockSlim  0 instance           lck
    0359001c  4000045        8 ...Info, Musketeer]]  0 instance           appPathToAppInfoMap
    03590440  4000046        c ...eer]], mscorlib]]  0 instance           workerProcessIdToAppInfoMap
    035907fc  4000047       10 ...eer]], mscorlib]]  0 instance           logsPathToAppInfoMap
    69c23e18  4000042       40        System.String  0   static 00e21598 MachineName
    69be0cbc  4000043       44      System.Object[]  0   static 00e215b0 NoApps

If you have an instance of a given type it will be probably faster to simply use netext `!wdo` which displays static fields automatically when dumping an object.

### Find a type or method (Name2EE) [SOS] ###

**Name2EE** searches through all the domains and lists matching methods and types, eg.

    0:000> !Name2EE * System.Diagnostics.EventLog.SourceExists
    ...
    --------------------------------------
    Module: 000007fc0d741000 (System.Configuration.dll)
    --------------------------------------
    Module: 000007fbff201000 (System.dll)
    Token: 0x00000000060038fd
    MethodDesc: 000007fbff308070
    Name: System.Diagnostics.EventLog.SourceExists(System.String)
    JITTED Code Address: 000007fbff996ec0
    -----------------------
    Token: 0x00000000060038fe
    MethodDesc: 000007fbff308090
    Name: System.Diagnostics.EventLog.SourceExists(System.String, System.String)
    JITTED Code Address: 000007fbff996d70
    --------------------------------------
    ...

### Show GC roots for all objects found with !wfrom [Netext] ###

Example for dumping GC roots for IP address objects:

    .foreach (t { !wfrom -nofield -nospace -type System.Net.IPAddress select $addr(); }) { !GCRoot ${t} }

Work with types
--------------

### Decode a GUID value ###

The `!wdo` command from netext nicely decode Guids. If necessary you may use the `!weval` and `$toguid` function.

### Decode a DateTime value ###

The `!wdo` command from netext prints readable date for fields of date type. If you need to decode a specific DateTime object, dump it:

```
0:019> !do 09443a74
Name: System.DateTime
MethodTable: 79308184
EEClass: 790e0564
Size: 16(0x10) bytes
 (C:\WINDOWS\assembly\GAC_32\mscorlib\2.0.0.0__b77a5c561934e089\mscorlib.dll)
Fields:
      MT    Field   Offset                 Type VT     Attr    Value Name
79309428  40000f4        4        System.UInt64  1 instance 634915192800000000 dateData
79332cc0  40000f0       30       System.Int32[]  0   shared   static DaysToMonth365
    >> Domain:Value  0015f7c8:01082d10 <<
79332cc0  40000f1       34       System.Int32[]  0   shared   static DaysToMonth366
    >> Domain:Value  0015f7c8:01082d50 <<
79308184  40000f2       28      System.DateTime  1   shared   static MinValue
    >> Domain:Value  0015f7c8:01082cf0 <<
79308184  40000f3       2c      System.DateTime  1   shared   static MaxValue
    >> Domain:Value  0015f7c8:01082d00 <<
```

Copy the `dateData` and use `!weval` function:

```
0:023> !weval $tickstodatetime(0n634915192800000000)
calculated: 2012-12-19 13:08:00
```

or powershell

```powershell
> new-object System.DateTime 634915192800000000

19 grudnia 2012 13:08:00
```

### Decode a timespan value ###

By default netext shows the value of a timespan field when calling `!wdo`, eg.

```
000007feef993d68               System.Runtime.Caching.ObjectCache +0000                                   _cache 00000000ffb3ab50
000007fef84d9360                                  System.TimeSpan +0008   <DefaultExpirationTime>k__BackingField -mt 000007FEF84D9360 000000042B700DA0 01:00:00
000007fef84d9360                                  System.TimeSpan +0010 <DefaultRefreshTresholdTime>k__BackingFi -mt 000007FEF84D9360 000000042B700DA8 00:05:00
000007fef7e0ccb8 Static  System.Collections.Concurrent.Concurrent +0000                                    Locks 00000001ffb38a10
```

But if only have the address of the timestamp instance, dump it:

```
0:023> !wdo -mt 000007fef84d9360 000000042b700da8
...
Assembly Name: C:\Windows\Microsoft.Net\assembly\GAC_64\mscorlib\v4.0_4.0.0.0__b77a5c561934e089\mscorlib.dll
Inherits: System.ValueType System.Object (000007FEF84D3390 000007FEF84D13E8)
000007fef84eb350                                     System.Int64 +0000                                   _ticks b2d05e00 (0n3000000000)
...
```

Copy the value of the `_ticks` field and use `!weval`:

```
0:023> !weval $tickstotimespan(0xb2d05e00)
calculated: 00:05:00
```

### Dump MemoryCache content ###

select memory cache stores: `!wfrom -type System.Runtime.Caching.MemoryCache select _stores`

for each store choose buckets: `!wselect _entries.buckets from {store-addr}`

for each found bucket: `!wselect key._key,val._value.m_value from {bucket-addr}`

Links
-----

- [Identifying Specific Reference Type Arrays with SOS](http://blogs.microsoft.co.il/sasha/2014/05/01/identifying-specific-reference-type-arrays-sos/)
- [A Motivating Example of WinDbg Scripting for .NET Developers](http://blogs.microsoft.co.il/sasha/2014/08/05/motivating-example-windbg-scripting-net-developers/)
- [How to display a DateTime in WinDbg using SOS](http://blogs.iis.net/carlosag/archive/2014/10/24/how-to-display-a-datetime-in-windbg-using-sos.aspx)
- [Andrew Richard's onedrive](https://onedrive.live.com/?authkey=!AJeSzeiu8SQ7T4w&id=DAE128BD454CF957!7152&cid=DAE128BD454CF957)
- [Using SoS to debug 32-bit code in a 64-bit dump with WinDbg](https://poizan.dk/blog/2015/10/15/using-sos-to-debug-32-bit-code-in-a-64-bit-dump-with-windbg/)

### loading mscordacwks.dll ###

- [Obtaining mscordacwks.dll for CLR Versions You Don't Have](http://blogs.microsoft.co.il/blogs/sasha/archive/2012/05/19/obtaining-mscordacwks-dll-for-clr-versions-you-don-t-have.aspx)
- [Automatically Load the Right SOS for the Minidump](http://www.wintellect.com/blogs/jrobbins/automatically-load-the-right-sos-for-the-minidump)
- <http://blogs.msdn.com/b/dougste/archive/2009/02/18/failed-to-load-data-access-dll-0x80004005-or-what-is-mscordacwks-dll.aspx>
- <http://blogs.msdn.com/b/asiatech/archive/2010/09/10/how-to-load-the-specified-mscordacwks-dll-for-managed-debugging-when-multiple-net-runtime-are-loaded-in-one-process.aspx>
- <http://blogs.microsoft.co.il/blogs/sasha/archive/2012/05/19/obtaining-mscordacwks-dll-for-clr-versions-you-don-t-have.aspx?utm_source=feedburner&utm_medium=feed&utm_campaign=Feed%3A+sashag+%28All+Your+Base+Are+Belong+To+Us%29&utm_content=Google+Reader>
- [Rozwiązywanie problemów z mscordacwks](http://zine.net.pl/blogs/mgrzeg/archive/2014/01/15/rozwi-zywanie-problem-w-z-mscordacwks.aspx)

- [Mscordacwks/SOS debugging archive - a list of all versions of mscordacwks and SOS](http://www.sos.debugging.wellisolutions.de/)
