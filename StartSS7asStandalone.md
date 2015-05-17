Below wiki explains how Mobicents jSS7 can be started as standalone Java Project without any dependency on JBoss, Mobicents JSLEE or any other container. It also gives some details of how jSS7 layers can be configured.

# Setting up the project #

Instead of the sophisticated application, you’ll get started with a “Hello World” example. That way, you can focus on the development process without getting distracted by jSS7 details. The example is based on Apache Maven
and should be installed on your development machine.

Let’s set up the project directory first.

## Creating the work directory ##

Create a new directory on your system, in any location you like; `\home\<user>\jss7example` is a good choice if you work on linux where `<user>` is the user-name. We’ll refer to this directory as WORKDIR in future examples.
Create src/main/java subdirectories (following maven jar directory structure)

```
WORKDIR
	+src
	   +main
	      +java
```

Note : Its assumed that you have basic understanding of Apache Maven

In the “Hello World” application, we will send USSD messages and listen for incoming USSD messages.

## Creating the Application ##

The Client acts as Mobile Handset sending the short code to Server acting as USSD Gateway Server. Client and Server are simple Java program with `public static void main(String args[])` method to fire up the program.
Application using jSS7 stack should setup SS7 layers from lowest one SCTP to the top MAP

### Building SCTP ###

This is an example of a typical SCTP startup procedure, in one line of code, using automatic configuration file detection:


```

	org.mobicents.protocols.api.Management sctpManagement = null;
	sctpManagement = org.mobicents.protocols.sctp.ManagementImpl("Client");
	this.sctpManagement.setSingleThread(true);
	this.sctpManagement.setConnectDelay(10000);
	this.sctpManagement.start();
```

Wait-how did jSS7 know where the configuration file was located and which one to load?

When `this.sctpManagement.start();` is called, jSS7 searches for a file named `Client_sctp.xml` in the directory path set by user by calling `this.sctpManagement.setPersistDir("<your directory path>")`.
For example in case of linux you can pass something like `this.sctpManagement.setPersistDir("/home/abhayani/workarea/mobicents/git/jss7/master/map/load/client")`

If directory path is not set, Management searches for system property `sctp.persist.dir` to get the path for directory.

Even if `sctp.persist.dir` system property is not set, Management will look at System set property `user.dir`

Once you know SCTP layer is configured and started, next step is add the `Association` and/or `Server` depending on whether this setup will be acting as client or server or both.

  * For client side : `sctpManagement.addAssociation(CLIENT_IP, CLIENT_PORT, SERVER_IP, SERVER_PORT, CLIENT_ASSOCIATION_NAME, ipChannelType, null);`
  * For server side : `sctpManagement.addServerAssociation(CLIENT_IP, CLIENT_PORT, SERVER_NAME, SERVER_ASSOCIATION_NAME, ipChannelType);`

Before adding server side association the server should also be defined and started
```
	sctpManagement.addServer(SERVER_NAME, SERVER_IP, SERVER_PORT, ipChannelType, null);
	sctpManagement.addServerAssociation(CLIENT_IP, CLIENT_PORT, SERVER_NAME, SERVER_ASSOCIATION_NAME, ipChannelType);
	sctpManagement.startServer(SERVER_NAME);
```

Note : You should never start the `Association` programatically. `Association` will be started automatically when layer above it M3UA's `Asp` is started.

This completes the SCTP configuration and start-up


### Building M3UA ###
Configuring the M3UA layer follows the same steps as SCTP above.

```
		org.mobicents.protocols.ss7.m3ua.impl clientM3UAMgmt = null;
		this.clientM3UAMgmt = new M3UAManagement("Client");
		this.clientM3UAMgmt.setTransportManagement(this.sctpManagement);
		this.clientM3UAMgmt.start();
```


For M3UA, it should know which underlying SCTP layer to use `this.clientM3UAMgmt.setTransportManagement(this.sctpManagement);`

Once M3UA is configured and started, next step is to add the `As`, `Asp` and routing rules for M3UA. These depends on whether stack acts as Application Server side or Signaling Gateway side or just peer-to-peer (IPSP) client/server side.

Below is example of IPSP peer acting as client

```
		RoutingContext rc = factory.createRoutingContext(new long[] { 100l });
		TrafficModeType trafficModeType = factory.createTrafficModeType(TrafficModeType.Loadshare);
		this.clientM3UAMgmt.createAs("AS1", Functionality.AS, ExchangeType.SE, IPSPType.CLIENT, rc, trafficModeType, null);

		// Step 2 : Create ASP
		this.clientM3UAMgmt.createAspFactory("ASP1", CLIENT_ASSOCIATION_NAME);

		// Step3 : Assign ASP to AS
		Asp asp = this.clientM3UAMgmt.assignAspToAs("AS1", "ASP1");

		// Step 4: Add Route. Remote point code is 2
		clientM3UAMgmt.addRoute(SERVET_SPC, -1, -1, "AS1");
```

This completes the M3UA configuration and start-up. Once M3UA is configured depending on whether you are trying to build voice application that depends on ISUP or advanced network features such as those offered by supplementary services that depends on MAP, you would configure ISUP or SCCP


### Building SCCP ###
Configuring the SCCP layer follows exactly same architecture of persisting configuration in xml file.

```
		org.mobicents.protocols.ss7.sccp.SccpStack sccpStack = null;

		this.sccpStack = new SccpStackImpl("MapLoadClientSccpStack");
		this.sccpStack.setMtp3UserPart(1, this.clientM3UAMgmt);
		this.sccpStack.start();
```

Before starting SCCP stack all it needs to know is underlying MTP3 layer. Above sections explained building SCTP and M3UA, however if you are using Dialogic boards or dahdi based boards (Diguim/Sangoma), you need to build and configure respective MTP3 layers depending on hardware used and set those in SCCP Stack `this.sccpStack.setMtp3UserPart(1, this.clientM3UAMgmt);`.

One of the best features of jSS7 is it supports multiple MTP3 layers and hence you can have combination of many MTP3 layers (each of different or same type like M3UA, Dialogic and Dahid; all used at same time )

Once SCCP stack is started, it should be configured for local and remote signaling point-code, network indicator, remote sub system number and routing rules.

```
		RemoteSignalingPointCode rspc = new RemoteSignalingPointCode(SERVET_SPC, 0, 0);
		RemoteSubSystem rss = new RemoteSubSystem(SERVET_SPC, SSN, 0, false);
		this.sccpStack.getSccpResource().addRemoteSpc(0, rspc);
		this.sccpStack.getSccpResource().addRemoteSsn(0, rss);

		Mtp3ServiceAccessPoint sap = new Mtp3ServiceAccessPoint(1, CLIENT_SPC, NETWORK_INDICATOR);
		Mtp3Destination dest = new Mtp3Destination(SERVET_SPC, SERVET_SPC, 0, 255, 255);
		this.sccpStack.getRouter().addMtp3ServiceAccessPoint(1, sap);
		this.sccpStack.getRouter().addMtp3Destination(1, 1, dest);
```


Once SCCP is configured and started, next step it to build TCAP layer

### Building TCAP ###
There is no configuration to persist in case of TCAP.

```
		org.mobicents.protocols.ss7.tcap.api tcapStack = null;
		this.tcapStack = new TCAPStackImpl(this.sccpStack.getSccpProvider(), SSN);
		this.tcapStack.setDialogIdleTimeout(60000);
		this.tcapStack.setInvokeTimeout(30000);
		this.tcapStack.setMaxDialogs(2000);
		this.tcapStack.start();
```

Configuring TCAP is probably very simple as config reamins same irrespective of whether its used on client side or server side.

### Building MAP ###
There is no configuration to persist in case of MAP; however MAP stack can take `TCAPProvider` from `TCAPStack` which is already configured for specific SSN as shown below
```
		this.mapStack = new MAPStackImpl(this.tcapStack.getProvider());
```

Or it can also directly take `SccpProvider` and pass SSN to MAP Stack as shown below. In this case `MAPStack` itself creates the `TCAPStack` and leverages `TCAPProvider`
```
		this.mapStack = new MAPStackImpl(this.sccpStack.getSccpProvider(), SSN);
```

Before `MAPStack` can be started, the Application interested in particualr MAP Service should register it-self as listener and activate that service
```
		this.mapProvider = this.mapStack.getMAPProvider();

		this.mapProvider.addMAPDialogListener(this);
		this.mapProvider.getMAPServiceSupplementary().addMAPServiceListener(this);
		this.mapProvider.getMAPServiceSupplementary().acivate();
		this.mapStack.start();
```

Below is how the Application code looks like
```
	public class Client extends MAPDialogListener, MAPServiceSupplementaryListener  {
		//Implemet all MAPDialogListener methods here


		//Implement all MAPServiceSupplementaryListener methods here
	}
```

### Common code ###

All above snippet of code refers to below defined constants
```

	// MTP Details
	protected final int CLIENT_SPC = 1;
	protected final int SERVET_SPC = 2;
	protected final int NETWORK_INDICATOR = 2;
	protected final int SERVICE_INIDCATOR = 3; //SCCP
	protected final int SSN = 8;

	protected final String CLIENT_IP = "127.0.0.1";
	protected final int CLIENT_PORT = 2345;

	protected final String SERVER_IP = "127.0.0.1";
	protected final int SERVER_PORT = 3434;

	protected final int ROUTING_CONTEXT = 100;
	
	protected final String SERVER_ASSOCIATION_NAME = "serverAsscoiation";
	protected final String CLIENT_ASSOCIATION_NAME = "clientAsscoiation";
	
	protected final String SERVER_NAME = "testserver";

.....
.....
```


You can directly download the example from [here](http://code.google.com/p/jss7/source/browse?repo=examples#git%2Fussd)

Note : You need to have GIT installed on your development machine to clone code

# Running and testing the application #

To run the application, you need to compile it first.

Maven is a powerful build system for Java.

## Compiling the project with maven ##

You'll now add a pom.xml file as shown below

```
WORKDIR
	+src
	+pom.xml
```

The content of this file is shown [here](http://code.google.com/p/jss7/source/browse/ussd/pom.xml?repo=examples)

The libraries defined in `<dependencies>` are required for a typical SS7 project. Keep in mind that some of the libraries you’re seeing here may not be required for the
particular version of jSS7 you’re working with, which is likely a newer release than we used when writing this wiki. To make sure you have the right set of libraries, always check the
`mobicents-ss7-X.Y.Z/ss7/mobicents-ss7-service/lib/` in the jSS7 distribution package. This directory contains an up-to-date list of all required and optional libraries for jSS7. `X.Y.Z` is the binary version of
jSS7 distribution. Depending on which version of jSS7 you work with, the version of libraries also changes.

Call `mvn clean install` from WORKDIR to compile the project. For first time, it will download all the dependencies from repository. Make sure you have access to internet at-least for first time.


### Start Server ###

Call `mvn test -Pserver` to start the server. If everything is right you should see below message without any exceptions shown on console

```
DEBUG org.mobicents.jss7.standalone.example.ussd.SctpServer  - [[[[[[[[[[    Started SctpServer       ]]]]]]]]]]
```

Observer the configuration files `Server_sctp.xml`, `Server_m3ua.xml`, `MapLoadServerSccpStack_sccprouter.xml`, `MapLoadServerSccpStack_sccpresource.xml` created in WORKDIR


### Start Client ###

Call `mvn test -Pclient` to start the client. If everything is right you should see USSD messages getting exchanged.

Its best to start the wireshark and capture the packets to analyze how SCTP, M3UA and MAP messages are getting exchanged.