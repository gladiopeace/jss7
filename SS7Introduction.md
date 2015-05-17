This wiki is meant to give a very small introduction on SS7. There are many books available which can provide complete details of SS7. Below are few of my recommendations
  * [Signaling System No. 7 (SS7/C7): Protocol, Architecture, and Services (Networking Technology)](http://www.amazon.com/Signaling-System-No-SS7-Architecture/dp/1587050404)
  * [Signaling System #7, Fifth Edition (McGraw-Hill Computer Communications Series)](http://www.amazon.com/Signaling-System-McGraw-Hill-Computer-Communications/dp/007146879X/ref=sr_1_1?s=books&ie=UTF8&qid=1339062422&sr=1-1)

### SS7 Introduction ###
Common Channel Signaling System No. 7 (i.e., SS7 or C7) is a global standard for telecommunications defined by the [International Telecommunication Union](http://www.voip-info.org/wiki/view/ITU) (ITU) Telecommunication Standardization Sector (ITU-T). The standard defines the procedures and protocol by which network elements in the public switched telephone network (PSTN) exchange information over a digital signaling network to effect wireless (cellular) and wireline call setup, routing and control. The ITU definition of SS7 allows for national variants such as the American National Standards Institute (ANSI) and Bell Communications Research (Telcordia Technologies) standards used in North America and the [European Telecommunications Standards Institute](http://www.voip-info.org/wiki/view/ETSI) ( ETSI ) standard used in Europe.


#### SS7 Protocol Stack overview ####
The hardware and software functions of the SS7 protocol are divided into functional abstractions called "levels". These levels map loosely to the Open Systems Interconnect (OSI) 7-layer model defined by the International Standards Organization (ISO).

![http://wiki.jss7.googlecode.com/git/images/ss7-fig3.gif](http://wiki.jss7.googlecode.com/git/images/ss7-fig3.gif)


#### Message Transfer Part ####
The Message Transfer Part (MTP) is divided into three levels. The lowest level, **MTP Level 1**, is equivalent to the OSI Physical Layer. MTP Level 1 defines the physical, electrical, and functional characteristics of the digital signaling link. Physical interfaces defined include E-1 (2048 kb/s; 32 64 kb/s channels), DS-1 (1544 kb/s; 24 64kb/s channels), V.35 (64 kb/s), DS-0 (64 kb/s), and DS-0A (56 kb/s).

**MTP Level 2** ensures accurate end-to-end transmission of a message across a signaling link. Level 2 implements flow control, message sequence validation, and error checking. When an error occurs on a signaling link, the message (or set of messages) is retransmitted. MTP Level 2 is equivalent to the OSI Data Link Layer.

**MTP Level 3** provides message routing between signaling points in the SS7 network. MTP Level 3 re-routes traffic away from failed links and signaling points and controls traffic when congestion occurs. MTP Level 3 is equivalent to the OSI Network Layer.


#### ISDN User Part (ISUP) ####
The ISDN User Part (ISUP) defines the protocol used to set-up, manage, and release trunk circuits that carry voice and data between terminating line exchanges (e.g., between a calling party and a called party). ISUP is used for both ISDN and non-ISDN calls. However, calls that originate and terminate at the same switch do not use ISUP signaling.


#### Telephone User Part (TUP) ####
In some parts of the world (e.g., China, Brazil), the Telephone User Part (TUP) is used to support basic call setup and tear-down. TUP handles analog circuits only. In many countries, ISUP has replaced TUP for call management.


#### Signaling Connection Control Part (SCCP) ####
SCCP provides connectionless and connection-oriented network services and global title translation (GTT) capabilities above MTP Level 3. A global title is an address (e.g., a dialed 800 number, calling card number, or mobile subscriber identification number) which is translated by SCCP into a destination point code and subsystem number. A subsystem number uniquely identifies an application at the destination signaling point. SCCP is used as the transport layer for TCAP-based services.


#### Transaction Capabilities Applications Part (TCAP) ####
TCAP supports the exchange of non-circuit related data between applications across the SS7 network using the SCCP connectionless service. Queries and responses sent between SSPs and SCPs are carried in TCAP messages. For example, an SSP sends a TCAP query to determine the routing number associated with a dialed 800/888 number and to check the personal identification number (PIN) of a calling card user. In mobile networks (IS-41 and GSM), TCAP carries Mobile Application Part (MAP) messages sent between mobile switches and databases to support user authentication, equipment identification, and roaming.