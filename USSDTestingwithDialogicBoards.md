This wiki talks about how to test the Mobicents USSD Demo SLEE Application with Dialogic's MAP Test Utility (MTU) tool.

It is assumed that you have two Servers (each fitted with Dialogic card) connected to each other via cross cable. To understand how to set-up the environment for Dialogic card please download respective Programmers Manual. Since this test was carried out on Dialogc SPCI2S cards the programmers manual can be downloaded from [http://www.dialogic.com/manuals/ss7/cd/ProductSpecific/SPCI-CPM8/ProgrammersManual/SPCI-PM.pdf](http://www.dialogic.com/manuals/ss7/cd/ProductSpecific/SPCI-CPM8/ProgrammersManual/SPCI-PM.pdf)

It is recommended to understand Dialogic's MTU/MTR before proceeding further. Please download the MTU/MTR guide from [http://www.dialogic.com/manuals/ss7/cd/GenericInfo/GeneralDocumentation/U30SSS03-MTU-MTR-UG.pdf](http://www.dialogic.com/manuals/ss7/cd/GenericInfo/GeneralDocumentation/U30SSS03-MTU-MTR-UG.pdf)

The SS7 layers involved and the two applications are shown below

![http://wiki.jss7.googlecode.com/git/images/DialogicBoards-Mobi.jpg](http://wiki.jss7.googlecode.com/git/images/DialogicBoards-Mobi.jpg)


### MTU Setup ###
Please download the modified MTU code for USSD from [http://www.dialogic.com/support/helpweb/helpweb.aspx/1580/ussd\_map\_sample\_code/Signaling\_SS7](http://www.dialogic.com/support/helpweb/helpweb.aspx/1580/ussd_map_sample_code/Signaling_SS7)

You have to build the modified MTU code to create the executable.

Attached here with ussd\_mtu\_config.txt and ussd\_mtu\_system.txt is sample configuration and system file used for MTU. Rename these to config.txt and system.txt respectively and place it to corect folder (/opt/dpklnx/ for linux) before loading the driver and activating links. Its is assumed that MTP2/MTP3 is loaded onboard and Dialogic SCCP, TCAP, MAP and MTU is loaded on host system.

Command to load drivers#
```
root@abhayani:/opt/dpklnx# ./gctload -d
```

Command to activate link#
```
root@abhayani:/opt/dpklnx# ./mtpsl act 0 0
```

Once the link is activated, don't forget to execute the mtucfg.ms7 script. This script registers the MTU with Dialogic SCCP stack.#
```
root@abhayani:/opt/dpklnx# ./s7_play -f/home/abhayani/workarea/mobicents/dialogic/upd_tar_FILES/RUN/MTU/SCRIPTS/mtucfg.ms7
```


### Mobicents USSD Application Setup ###
Download the latest release of Mobicents SS7 from [Downloads](Downloads.md)
Download the latest Mobicents SLEE release from [http://www.mobicents.org/slee/downloads.html](http://www.mobicents.org/slee/downloads.html)

Attached here with mobicents\_config.txt and mobicents\_system.txt is sample configuration and system file used for Mobicents. Rename these files to config.txt and system.txt respectively and place it to corect folder (/opt/dpklnx/ for linux) before loading the driver and activating links. Use same commands as shown above to load drivers and activate link. It is assumed here that MTP2/MTP3 is loaded onboard. The Mobicents Dialogic native driver (libmobicents-dialogic-linux.so) communicates with MTP3 onboard.

Once link is active start the Mobicents JAIN SLEE (JBoss Application) Server. Remember before starting application server, place the native file in correct folder (for example incase of 32bit linux mobicents-jainslee-2.4.0.CR1-jboss-5.1.0.GA/jboss-5.1.0.GA/bin/META-INF/lib/linux2/x86/libmobicents-dialogic-linux.so) or the java.library.path should be set to point the directory containing native component.

Deploy the Mobicents SS7 Service as explained in Mobicents SS7 Docs.
Deploy the Mobicents MAP RA as explained in Mobicents JSLEE Docs. Use attached map-default-ra.properties for this setting.
Deploy [map-demo](http://code.google.com/p/mobicents/source/browse/#svn%2Ftrunk%2Fservers%2Fjain-slee%2Fexamples%2Fmap-demo)

One last step before the application can be tested is configuring the Linkset and SCCP rule. Fire the CLI and connect to Mobicents SS7 Service as explained in Mobicents SS7 Doc. Following configuration is to be done only once and it survives system restarts.

Execute bellow command to register routing rule based on DPC + SSN#
```
mobicents(127.0.0.1:3435)>sccprule create Rule4 dpc 1 ssn 8 mtpinfo name name1 opc 2 apc 1 sls 16
```

### Test ###
Execute the following MTU command from MTU machine#
```
ussdtest -d6 -g43010008 -a43020008 -i987654321 -s"1" -U*88#
```

You should see the [INFO](INFO.md) messages in the Mobicents JAIN SLEE Server console about received USSD short codes and response sent.

The /opt/dpklnx in both machines will have mtp.pcap and mtp.txt showing the USSD messages exchanged.