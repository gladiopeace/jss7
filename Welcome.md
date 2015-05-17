Mobicents provides a open source software solution implementing MTP2,3, ISUP, SCCP, TCAP, CAMEL, MAP, INAP protocols for a dedicated equipment and also M3UA (SIGTRAN) over IP.  It is the first and only open source SS7 stack completely on Java.

Below diagram explains protocols that are already implemented in jSS7, in progress and planed to be implemented

![http://wiki.jss7.googlecode.com/git/images/MobicentsjSS7Layers.jpg](http://wiki.jss7.googlecode.com/git/images/MobicentsjSS7Layers.jpg)

## Supported equipment ##

The Mobicents SS7 implementation supports following TDM equipment:

  * Intel SS7 family board - Dialogic Boards
  * Zaptel/Dahdi compatible TDM device (Digium, Sangoma)
  * TeleStax SS7 Boards

### Intel SS7 family board ###

Dialogic SS7 boards are designed to meet the needs of telecommunications equipment manufacturers, systems integrators, and service providers deploying solutions worldwide. Two families of SS7 products are available to enable affordable, high-performance, signaling applications.

### Zaptel/Dahdi compatible board ###

There are hardware TDM devices wich share common driver suite called Zaptel/Dahdi Telephony Driver Suite. Most devices sold by Digium are members of the Zaptel/Dahdi family of hardware devices. Zaptel/Dahdi driver allows access timeslots directly from Java VM without native interface and that is why such these devices are interesting. For immediate term What we want to achieve is user can use any SS7 card (hardware like Sangoma, Digium etc) supported by Zaptel on Mobicents Media Server (MMS) side for media and Mobicents Signalin Gateway (SGW) for signaling. The signaling can be forwarded to JAIN SLEE server on remote machine over SCTP (MTP3 layer or SCCP layer). On JAIN SLEE server we will have Resource Adaptor implementation for various high-level protocols like TCAP, INAP ISUP, MAP for call control.

### TeleStax SS7 Boards ###

TeleStax SS7 Boards are specially designed to work with Mobicents jSS7. These are intelligent cards with MTP2/3 and M3UA onboard so as to act as SGW interacting with Mobicents jSS7 stack over M3UA (IP).