# Windbgshark #

This project includes an extension for the windbg debugger as well as a driver code, which allow you to manipulate the virtual machine network traffic and to integrate the wireshark protocol analyzer with the windbg commands.

The motivation of this work came from the intention to find a handy general-purpose way to debug network traffic flows under the Windows OS for the purposes of dynamic software testing for vulnerabilities, for reverse engineering of software and just for fun.

# Theory of operation #

The main idea is to rely on the Windows Filtering Platform capability to inspect traffic at the application level of OSI (however, the method works well on any level introduced by the WFP API). This gives us a way to intercept and modify any data, which goes through the Windows TCP/IP stack (even the localhost traffic), regardless of the application type and transport/network protocol. Modification and reinjection also work excellent: the operating systems does all the dirty work, reconstructing the transport and network layer headers, for example, as if we were sending the data from the usermode winsock application.

This tool needs a virtualized enviroment (it works fine with VMWare Workstation now) with windbg connected to the virtual machine as a kernel debugger. Installation is done in two steps: driver installation and extension loading in windbg. Driver intercepts network traffic, allows the windbg to modify it, and then reinjects packets back into the network stack. The extension on its turn implements simple interface for packet edit and also uses Wireshark to display data flows. The extension is executed on the host machine, while the driver is located on the virtual machine. To interact with its driver, windbg extension sets the corresponding breakpoints with its own callbacks right inside the driver code. Every time a packet comes in or out, a breakpoint is hit and the windbgshark extracts the app-level payload of the current packet, constructs a new pcap record and sends it to Wireshark. Before the packet is reinjected back, user may modify it, and the Wireshark will re-parse and show the modified record.

# Build #

Source code is presented as a Visual Studio 2010 solution with both projects, Windows Driver Kit http://msdn.microsoft.com/en-us/windows/hardware/gg487428 is required to build this solution. You can buld either from the command line or from the Visual Studio (Ctrl + B), all the necessary makefiles come along with the source code.

# Install #

First, you need to prepare your VMWare virtual machine to interact with the kernel debugger. This task is covered in http://www.ndis.com/ndis-debugging/virtual/vmwaresetup.htm, this tool http://visualddk.sysprogs.org/ simplifies this process a bit. You also need to set up a correct symbol path in windbg, pointing to the windbgshark\_drv.pdb (debugging symbols for the driver).

When the windbg is set up, you need to install and start the driver windbgshark\_drv.sys, .inf file is included in this project. Start the driver, for example, from the command-line:
**sc start windbgshark\_drv**

After that you can load the windbgshark library in windbg. Copy the dll to a location that can be found by your windbg, and type in the command window:
**!load windbgshark**.
The library should start the wireshark (now its path is hardcoded, you should have the executable C:\Program files\Wireshark\Wireshark.exe on the host machine).
Type !windbgshark.help to get the list of commands and start playing with the tool.

# Quickstart #
```
> !load windbgshark
> !strace on
> g
…
[packet was caught]
> !packet 100 +AAAAAAAAAAAAAAAAAAA
[look in wireshark]
> g
…
```

# Slides #

This presentation was given at [Zeronights 2011](http://zeronights.org/): http://www.slideshare.net/Sciurus/windbgshark-tool

# Screenshots #

Step-trace mode on (in windbg), wireshark shows traffic flow:

![http://windbgshark.googlecode.com/svn/trunk/screen1.png](http://windbgshark.googlecode.com/svn/trunk/screen1.png)


Trying to fuzz a web browser:

![http://windbgshark.googlecode.com/svn/trunk/screen2.png](http://windbgshark.googlecode.com/svn/trunk/screen2.png)


Localhost traffic is also processed:

![http://windbgshark.googlecode.com/svn/trunk/screen3.png](http://windbgshark.googlecode.com/svn/trunk/screen3.png)


# Restrictions #

The tool is expected to be a proof-of-concept code rather than a reliable software product. The main problems:
  * Wireshark does not reload the pcap file automatically, you still need to press Ctrl + R every time you want to look at the changes. This sucks, yeah.
> > I need the to update and re-parse the last packet by the Wireshark once it is changed by user (just before it is reinjected to the network stack). This seems to be impossible with pipes (to the best of my knowledge). On the other hand, pcap files are not updated automatically. I am now thinking of a simple solution to this issue.
  * Only IPv4/TCP is supported now. It is quite easy to add IPv6 and non-TCP transport, nevertheless, it is not done yet.
  * The extension lowers virtual machine bandwidth dramatically (to the value of about 0.6Mb/s on my machine).
  * Whatever you find

# Credits #

Thanks to [dvserikov](https://twitter.com/dvserikov) and `Honorary_Bot` for their support, helpful thoughts and advices, thanks to [CathBlake](http://katinart.ru/) for the cute logo.

# If I want to contribute or say a few words to the author #

Yes, feel free to drop me a few lines if you would like to at isciurus@gmail.com.


