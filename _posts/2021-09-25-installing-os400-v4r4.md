---
title: "Installing OS/400 V4R4 on an AS/400 Model 150"
date: 2021-09-25T10:20:00+01:00
categories:
 - sysadm
tags:
 - as400
 - system
 - os400
 - software
 - ibm
---

Back in June I got a pallet of AS/400 stuff from a fellow collector who is downsizing his rather impressive collection. The haul included a pair of model 150s that were either missing this or that thing, and most importantly didn't have any DASD: since I was able to combine the two machines into a working and good-looking system, I figured that the drives that came with `44-L5517` (aka the system from [the backyard](https://jack23247.github.io/blog/hardware/core-transfer/)) could be rehoused. While the system IPLed fine with the new core, the ASP was full of <s>old payroll crud</s> sensitive business data, so I decided to give it some new life with a fresh V4R4 installation.

## The Procedure

Installing OS/400 from a CD-ROM distribution is a fairly straightforward process. Make sure you have <u>at least</u> the following disks at hand:

- `I_BASE_01` V4R4 - Licensed Internal Code for OS/400
- `B2924_01` V4R4 - Operating System/400
- `B2924_02` V4R4 - OS/400 no-charge options 1
- `B2924_03` V4R4 - OS/400 no-charge options 2
- `L2924_01` V4R4 - Priced licensed programs

I've also ended up using disk "`F2924_01` CPA Toolkit" and `C1072440_01` thru `_03` containing the cumulative PTF package up to 13/03/2001.

The manual titled "AS/400e Software Setup - Version 4" (or similar) will definitely come in handy during this process. The Italian version of this manual (`SC13-2695-04`) can be found [here](http://publibfp.boulder.ibm.com/epubs/pdf/c1326954.pdf).

Ready? Go!

#### 1. Preparing the new Load Source

1. Use function 02 on the front panel to select the `D`-side IPL in manual mode and press OK. You can now use function 01 to check the IPL mode: the display (on a 9401) should read: `01  D M`.

   > If your system is on, power it down.

2. Power on the system and put disk `I_BASE_01` in the drive: the IPL procedure will take a bit longer than usual, then it will present you with a screen asking you for a *Natural Language Version Feature*: this is the ID of the primary system language, which can't be changed later (I think). Refer to [this page](https://www.ibm.com/docs/en/i/7.1?topic=information-national-language-version-feature-codes) for a list of possible codes (U.S. English is 2924).

3. Follow the system prompts: the installer will ask you if you want to install the system anew, answer affirmatively. This will destroy all data present on your load source. Wait for the load source to be initialized.

4. Initially, the load source will be the only disk in your ASP: the installer will ask you if you want to add all available disks to the ASP.

   This is down to your preference: I answered no, as I wanted to keep a spare disk. I only have three 6607-050 DASD so I could not configure a protected ASP anyway (it requires four). This will drop you to the Dedicated Service Tool: use DST to add the required DASD to the storage pool.

   > In my case, I added disk unit `DD002`: this will take a while. Be sure to jot the serial number (an alphanumeric string that looks like this: `XX-XXXXX`) down if you need to remove unused disk units from the card cage later on.

   When the process is complete, select option *2. Install the O/S* in DST to continue the procedure.

5. When prompted, I chose `1` to keep my current ASP configuration. If you wish to change your ASP configuration, follow the on-screen prompts and go back to point 4.

6. When prompted, I loaded disk `B2924_01` in the drive and replied `1`.

7. At this point, the installer will ask you if you want to customize the default install options. This is down to your preference: I replied `2` to see if there was anything worth changing but I ended up using the defaults.
   After choosing an option and following the on-screen prompts, the system will start copying objects to the ASP: in my case, the procedure took quite a while, and the installer ended up restoring 9322 programs and 2890 language objects.

8. The system will reboot and ask you to log on to complete the setup procedure: log on as user `QSECOFR`. No password is required at this time.

   > You might notice that the front panel IPL side has changed from `D M` to `A M`: this is normal, as we're now booting from the newly installed load source.

9. The system will ask you for time, date and other setup values, then will ask you to change the password for user `QSECOFR`.

   Now you can customize the system: when prompted, I changed the system name to `ANUBIS`, then issued a couple of commands to make sure everything was in working order.

   > At this point, the system is in a very limited state: you need to install the remainder of the base packages from menu `LICPGM`, option 11. I did this the long way, but you can probably do it from here as well.

10. When you're done, shut the system down.

#### 2. Installing the software

1. Select IPL side `B` in normal (unattended) mode. Function 01 should read `01  B N` on a 9401.

2. Let the system boot and log in as `QSECOFR`.

3. Put the system in a *restricted state* by issuing the following commands:

   - `CHGMSGQ QSYSOPR *BREAK SEV(60)`: this will bring up the System Operator Message Queue. Answer any outstanding message, then press Enter to close the prompt.

     > This step is important because unanswered messages might hold up a job queue preventing subsystems from stopping.

   - `ENDSBS *ALL *IMMED`: this will stop every running subsystem, thus bringing the system to a restricted state.

     When the message "System in a restricted state" (or similar) appears, you can proceed.

     > Be careful when you are in restricted state because **ATTN won't work**! If you fail to answer an outstanding message and exit the Operator Message Queue panel with F3, you might get stuck. If you know a solution to this problem, please send me an e-mail, I'd be glad to fix this paragraph (and not waste hours of precious disk life recovering my journals).

   - `CHGMSGQ QSYSOPR SEV(95)`: this will restore the usual System Operator Message Queue behavior.

   Refer to [this document](http://www.audentia-gestion.fr/IBM/PDF/wmrstate53.pdf) and [this guide](https://www.ibm.com/docs/en/i/7.4?topic=introduction-putting-your-system-in-restricted-state) for more informations on the restricted state.

   > You can exit a restricted state by restarting subsystem `QBASE`, but I recommend just rebooting via `GO POWER`, `12`.

4. Use command `GO LICPGM` to jump to the *Work with Licensed Programs* menu, choose option *11. Install licensed programs* to bring up the licensed program panel.

5. Prepend the licensed programs you want to add with a `1`, I chose the following:

| What                                     | Why                                        |
| ---------------------------------------- | ------------------------------------------ |
| Library QGPL                             | Mandatory                                  |
| Library QUSRSYS                          | Mandatory                                  |
| Extended Base Support                    | Mandatory                                  |
| Online Information                       | Quality of life                            |
| Extended Base Directory Support          | Mandatory                                  |
| *PRV CL Compiler Support                 | Programming and Development Environment    |
| Host Servers                             | Client Access/400 Dependency               |
| System Openness Includes                 | Programming and Development Environment    |
| Common Programming APIs Toolkit          | Programming and Development Environment    |
| QShell Interpreter                       | Programming and Development Environment    |
| ILE COBOL for AS/400                     | Programming and Development Environment    |
| *PRV ILE COBOL for AS/400                | Development Environment Retrocompatibility |
| ILE C for AS/400                         | Programming and Development Environment    |
| IBM HTTP Server for AS/400               | Fun                                        |
| Job Scheduler for AS/400                 | Fun                                        |
| App Dev ToolSet for AS/400 - SEU         | Programming and Development Environment    |
| App Dev ToolSet for AS/400 - Others      | Programming and Development Environment    |
| Query for AS/400                         | Programming and Development Environment    |
| ILE RPG for AS/400                       | Programming and Development Environment    |
| RPG for AS/400                           | Programming and Development Environment    |
| *PRV ILE RPG for AS/400                  | Development Environment Retrocompatibility |
| DB2 Query Mgr and SQL DevKit for AS/400  | Programming and Development Environment    |
| TCP/IP Connectivity Utilities for AS/400 | TCP/IP Servers                             |
| Client Access/400 Express for Windows    | Client Access/400                          |

6. When you are satisfied of your choices, press Enter. The system will ask you for a device name: if you're installing from optical media, it should be `OPT01`. Insert the `B2924_02` CD-ROM in the drive and wait for the media access light to stop blinking, then press Enter.

   The system will start loading the required objects from the CD-ROM: when a new disk is required, the system will prompt you for it. Insert the required CD-ROM and and wait for the media access light to stop blinking, then answer `G` to the prompt to continue installing the software.

   > There is no indication of which CD-ROM you have to insert next, but I have found that inserting them in the following order yields the best results: `B2924_02`, `B2924_03`, `L2924_01`, `F2924_01 (CPA Toolkit)`. This is based on guesswork, so YMMV.

   If you want to interrupt the procedure, answer `X` to the prompt: doing so could leave one or more software packages in an inconsistent state, which can be resolved by installing said software again. The offending packages will be marked in the licensed program panel with an inverted field.

   > For a detailed description of this procedure refer to the aforementioned manual or to: *[Using the Work with Licensed Programs menu to install IBM licensed programs](https://www.ibm.com/docs/en/i/7.3?topic=ialp-using-work-licensed-programs-menu-install-licensed-programs)*.


#### 3. Setting up the system

##### 3.1. Adding an user

###### A digression on OS/400's Access Control System

Adding an user is an important step in any system's configuration: if on *nix systems you don't use  `root` for mundane tasks, and you never log on as `Administrator` on NT, then you should treat `QSECOFR` _almost_ in the same way.

> You might realize we've been seeing a lot of `Q`s prepended to command or object names: this is because the AS/400 team at Rochester figured that very little English words effectively start with Q, and they decided to use this letter to denote objects, variables, users and libraries provided by the OS. As a rule of thumb, I'd refrain from creating user accounts and, thus, libraries that start with the letter Q.

You see, `QSECOFR` (as in *Security Officer*) is the closest thing OS/400 has to a superuser account: this account has complete authority over the system and can modify any system object. For systems that are not accessible from the Internet, the `QSECOFR` user is usually protected by a "no-logon" company policy: the default password is left unchanged (`QSECOFR`/`QSECOFR`) and users simply refrain from using this user to avoid dire consequences. Of course, a maligned entity that somehow gains access to the intranet or to a physical terminal could wreak havoc: for the former scenario there are some mitigations we'll discuss later, while if the latter happens... well, you might have bigger problems.

When creating a user profile you should keep in mind that access is regulated by your _User Typer_ and your _User Group_: OS/400 has a very rich authorization system that ensures no user without the right privileges can access files they don't have control over. Probably the most useful user type and group combination for an hobbyist is the _Programmer_ (`*QPGMR`): this is an user class which is applied to users who will use the development environment and should be put in the same group as `QPGMR`. This kind of user is not as powerful as `QSECOFR` or `QSYSOPR` (System Operator, we won't discuss this one just yet) so it can't cycle power to the system or perform administration tasks, but it can operate on all source objects and can use SEU (Source Entry Utility), PDM (Program Development Manager) and other tools to create and run programs on the system.

###### Creating an user profile

In any case, whether or not you decide to change `QSECOFR`'s password, you should create a regular user account: this can be accomplished via the `WRKUSRPRF USRPRF(*ALL)` or, more specifically, the `CRTUSRPRF` commands.

`CRTUSRPRF`'s panel is almost self-explainatory: just keep in mind that you need to specify a group and an user class. Other noteworthy items are:

- The _Assistance level_, which is used to give the user more or less information when using system information commands such as `WRKSYSSTS`

- The _Current library_, which is used to change the user's default library to something other than `QGPL` (General Purpose Library).

  > I strongly recommend creating a library for each of your users and work using local copies of your source objects to keep the system tidy. This can be accomplished by using the `WRKLIB` (Work with Libraries) menu as user `QSECOFR`, creating a library for your user, and then specifying its name in the current library field when creating the user. At this point, the library can be added to your user's `LIBL` (Library List) via the `EDTLIBL` command and it can be used as some kind of "home directory" for the user. To copy objects, make sure you are in the right authorization group and use the the Work with Libraries menu, option `12` (Work with Objects, aka `WRKOBJ`).

##### 3.2. Configuring TCP/IP

Configuring TCP/IP is a rather straightforward process, and excellent articles have already been written on the topic: when doing so, I always keep a copy of [Easy400.net's guide](https://www.easy400.net/tcpcfgh/page02.mbr) handy, as it gets me going in no time.

> Fun fact: I found a printed copy of this guide along with one of my systems' documentation bundle, it must have been a real hit!

While configuring TCP/IP itself is nothing special, you should take a few extra steps to customize the system and improve usability:

###### Disabling unwanted servers

From the `TCPADM` menu, choose option _2. Configure TCP/IP Applications_. Look at the attributes (usually "Change XYZ Attributes") of any option you don't need and change the  _Autostart servers_ field to `*NO` (`AUTOSTART(*NO)`): this will ensure that the servers won't be started automatically by `STRTCP` , thus reducing startup time consiferably.

###### Changing the Telnet server's behavior

By default, you'll notice that you are not able to connect to the AS/400 via TN5250: this is due to the fact that the `QAUTOVRT` system value is set to 0, so no virtual terminal will be automatically created because the system thinks no more slots are available, and the connection will be terminated. Change it using option 10 in the `CFGTCPTELN` (Configure TCP/IP TELNET) menu or by issuing `CHGSYSVAL SYSVAL(QAUTOVRT) VALUE(xxx)` to a number that suits your needs or use `*NOMAX` if you don't want to set a limit to the amount of concurrent Telnet connections: you'll notice right away that now you can connect.
As stated earlier, there is a mechanism in place that restricts `QSECOFR` by only allowing it to connect via a physical console: this is regulated by the `QLMTSECOFR` system value, which by default is set to 1 (Explicit device access needed). You can set it to 0 by using option 11 in `CFGTCPTELN` or by issuing `CHGSYSVAL SYSVAL(QLMTSECOFR) VAL(0)` to allow the Security Officer to log on via TN5250 sessions.

###### Configuring an IP Printer

I think this process is worth documenting, as it's not particularly difficult but information is all over the place. To configure the printer, you need at least three "components":

- an IP printer (duh) supporting the JetDirect protocol, I'm using an HP LaserJet 1320n,

- a _Device Description_, used to specify the type and parameters of the printer and to attach a driver to it,

- a _Printer Writer_, a special job that watches a specific printer queue and pipes outstanding documents to the printer.

> Apparently, you could skip the device description and just point a printer queue to an IP address: I've been discussing this process with a fellow hobbyist but I didn't delve into it yet.

The Device Description can be created by using `CRTDEVPRT` and specifying several parameters. An outline of the required parameters is available [here](https://www.ibm.com/docs/en/i/7.1?topic=printing-configuring-snmp-printers). Keep in mind that in the _System Driver Program_ field you should specify `*HPPJLDDRV` instead of `*IBMSNMPDRV` (`SYSDRVPGM(*HPPJLDRV)`), see [this mailing list thread](https://archive.midrange.com/midrange-l/201908/msg00362.html
) for details.

Moreover, IBM suggests that you specify a _Manufacturer Type and Model_ (`MFRTYPMDL`) for your printer so that the host-based EBCDIC-to-ASCII transformer knows what format to output: there are several printer profiles supplied with your operating system and they are listed [on IBM's website](https://www.ibm.com/support/pages/information-printers-various-manufacturers).

> Italian readers might find the following article helpful as well:
> [https://muflone.wordpress.com/2009/05/04/configurare-una-stampante-di-rete-su-iseries/](https://muflone.wordpress.com/2009/05/04/configurare-una-stampante-di-rete-su-iseries/)

##### 3.3. Installing Cumulative PTFs

Installing PTFs is a straightforward, automated process, and it's described quite clearly [on IBM's website](https://www.ibm.com/docs/en/i/7.2?topic=scenario-installing-cumulative-ptf-packages): as is the case with most OS/400 topics, the instructions listed here (targeting V7R2) apply to V4R4 as well.

> If you have a slow machine, you're definitely gonna have a bad time: on my 150, this process ended up taking several hours.  

## Conclusions

Installing OS/400 might seem a daunting process at first, but the excellent documentation and the easy to use panel-driven interface makes it pretty easy for anyone familiar with basic midrange terminology to dive into. Of course, as with any computer system, there are some pitfalls and caveats that might make your life a bit harder: I hope this guide may help you detecting and avoiding them, or at the very least that it's been a good read!



