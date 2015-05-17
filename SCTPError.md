#Getting error java.lang.UnsupportedOperationException: libsctp.so.1: cannot open when trying to start SCTP

# Introduction #

Mobicents jSS7 uses Mobicents SCTP library which leverages the SCTP API of JDK 7. JDK 7 uses the sctp native library of underlyiing OS. If sctp library is not installed you will get above error


# Details #

**Linux**
In linux OS you need to install lksctp library as linux doesn't have this installed by default.

  * _Fedora 10_: sudo yum install lksctp-tools-1.0.9-1.fc10   or
  * _Ubuntu_: sudo apt-get install lksctp-tools

**Solaris**
Solaris ships with native support for SCTP since Soalris 10. So if you are running on Solaris 10 or greater, then once you got a JDK with SCTP you are good to go.

**Running a simple SCTP application**

To ensure that you have a correctly configured machine and JDK, try compiling and running the following application:
```
    public class TestSCTP
    {
        public static void main(String[] args) throws Exception {
            com.sun.nio.sctp.SctpChannel sc = com.sun.nio.sctp.SctpChannel.open();
        }
    }
```

```
    $ jdk1.7.0/bin/javac TestSCTP.java
    $ jdk1.7.0/bin/java TestSCTP
```

If it compiles and runs without any errors, then congratulations you have a correctly configured machine and JDK. You can now start Mobicents jSS7 over M3UA