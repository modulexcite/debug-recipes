
Debug Recipes
=============

This is a repository of my notes collected while debugging various .NET and Windows problems. You can find here commands with example usages, scripts and other debugging materials.  It is still being constructed so some notes might not be finished - use on your own responsibility. Try using **the project search box** while looking for a particular subject.

I hope you will find them useful. Any contribution is welcome.

---------------------

## General advice

Make sure you have [valid symbols configuration](windows-debugging-configuration.md) in your system. You may also have a look at a list of my [debugging tips](howto.md). If you are debugging a .NET application you may first [check the .NET debugging tips](clr-general-debugging-tips.md).

---------------------

## Troubleshooting

### .NET applications

Choose a recipe corresponding to your application problem (if a recipe requires WinDbg please have a look [how to setup .NET in WinDbg](debugging-using-windbg/windbg-clr-debugging.md) first):

- [Unknown exceptions thrown](exceptions/exceptions.md)
- [Memory leaks](memory/managed-memory-leaks.md)
- [High CPU usage (IN PROGRESS)](cpu/analyzing-high-cpu-usage.md)
- [I/O issues (IN PROGRESS)]()
- [Networking problems](network/network-tracing.md)
- [Deadlocks](threading/analysing-locks-in-net.md)
- [Assembly not found](assemblies/clr-assemblies.md)
- Slow database queries:
  - [Using ADO.NET](ado.net/ado.net-debugging.md)
  - [Using MySql connector](databases/mysql/mysql.net-connector-usage.md)

Problems in **.NET Web Applications**:

- Slow requests / tracing: [ASP.NET](asp.net/asp.net-profiling.md), [Nancy](nancy/nancy-diagnostics.md)
- Unknown errors: [ASP.NET](asp.net/asp.net-debugging.md)

### Databases

|    | MS SQL Server | MySQL |
| --- | --- | --- |
| Slow queries | [X](databases/mssqlserver/mssqlserver-querying.md) | [X](databases/mysql/mysql-querying.md) |
| Blocked requests | [X](databases/mssqlserver/mssqlserver-concurrency.md) | [X](databases/mysql/mysql-concurrency.md) |
| Indexes problems | [X](databases/mssqlserver/mssqlserver-indexes.md) | [X](databases/mysql/mysql-indexes.md) |
| I/O problems | [X](databases/mssqlserver/mssqlserver-troubleshooting-io.md) |  |
| Server problems | [X](databases/mssqlserver/mssqlserver-troubleshooting-server.md) | [X](databases/mysql/mysql-troubleshooting-server.md) |

### [Web servers](iid/README.md)

- [Troubleshooting IIS7+](iis/iis7up.md)
- [Troubleshooting IIS6](iis/iis6.md)
- [Troubleshooting IIS Express](iis/iisexpress.md)

### Native applications

TBD

### Windows system

- [Event Tracing for Windows](etw/README.md)
- [Debugging Windows kernel - setup](debugging-kernel/windows-kernel-debugging-setup.md)
- [Debugging Windows kernel - basics](debugging-kernel/windows-kernel-debugging.md)

---------------------

## Tools

- [Visual Studio (debugging)](debugging-using-vs/README.md)
- [mdbg (debugger)](debugging-using-mdbg/mdbg.exe.md)
- [WinDbg (debugger)](debugging-using-windbg/windbg-debugging.md)
  - [.NET in WinDbg](debugging-using-windbg/windbg-clr-debugging.md)
  - [Interesting WinDbg extensions](debugging-using-windbg/windbg-extensions.md)
- [DebugDiag (debugger)](debugdiag/debugdiag.md)
- [Adplus (debugger)](exceptions/adplus/adplus.md)
- [PerfView (profiler)](perfview/perfview.exe.md)
- [Windows built-in tools](windows-system-perftools.md)

