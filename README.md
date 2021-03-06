# PREFACE

About CORBA, please have a look at http://en.wikipedia.org/wiki/Corba.

omniORB is one of the CORBA implementations, please have a look at http://omniorb.sourceforge.net/.

About Qt, please have a look at http://en.wikipedia.org/wiki/Qt_%28toolkit%29.

And there is one add-on, QtCorba, http://trolltech.com/products/qt/addon/solutions/catalog/4/Utilities/qtcorba/, which is for the integration Qt 4.x with ACE/TAO or MICO(another two CORBA implementations). Then main concept in QtCorba is to convert ACE/TAO or MICO events into Qt event, just because they need the main thread to run their own event loop. QtCorba is not open source(it's commercial product).

I had inquried the omniORB mailing list how to integrate omniORB with Qt 4.x. The core developer of omniORB, Duncan Grisby, gave me a reply on it.

http://www.omniorb-support.com/pipermail/omniorb-list/2008-February/029227.html
omniORB doesn't have an application-hookable event loop like Mico. it
always uses its own threads to dispatch CORBA calls. You can just use
the main thread to service Qt's event loop and it will work fine.

Then I tried some code, it works fine.

1. Install omniORB 4.1.1 and run the examples

I am using openSUSE 10.3, gcc 4.2.1 on 32bit PC.

Download: omniORB-4.1.1.tar.gz
http://sourceforge.net/project/showfiles.php?group_id=51138

There are four parts, BUILD omniORB, RUN omniORB EXAMPLES, BUILD Qt, RUN Qt&omniORB EXAMPLES in this article.

# BUILD omniORB

```
cd ~/tmp
mkdir build
tar zxvf omniORB-4.1.1.tar.gz
cd omniORB-4.1.1
./configure -prefix /user
make
sudo make install
```

Build examples:
```
cd ~/tmp/omniORB-4.1.1
cd src/examples/echo
make
```

Please read Chapter 2 in doc/omniORB.pdf at first.

# RUN omniORB EXAMPLES

(1) eg1 is an example which creates a corba object and invokes the method of it in same code.

```
cd ~/tmp/omniORB-4.1.1
cd src/examples/echo
./eg1
```

The result should be:
I said, "Hello!".
The Echo object replied, "Hello!".

(2) eg2_impl is the server, eg2_clt is the client, they are based on IOR, not need to run omniNames.

In terminal 1, run the server:
```
cd ~/tmp/omniORB-4.1.1
cd src/examples/echo
./eg2_impl
IOR:010000000d00000049444c3a4563686f3a312e3000000000010000000000000068000000010102000f0000003133342e33322e3138352e3232300000f96600000e000000fe23aab24700000562000000000000000200000000000000080000000100000000545441010000001c00000001000000010001000100000001000105090101000100000009010100
```

In terminal 2, run the client:
```
cd ~/tmp/omniORB-4.1.1
cd src/examples/echo
./eg2_clt IOR:010000000d00000049444c3a4563686f3a312e3000000000010000000000000068000000010102000f0000003133342e33322e3138352e3232300000f96600000e000000fe23aab24700000562000000000000000200000000000000080000000100000000545441010000001c00000001000000010001000100000001000105090101000100000009010100
```

Yeap, you should use the IOR:01000.... string as the parameter of client side.

The output of server side:
```
Upcall Hello!
Upcall Hello!
Upcall Hello!
Upcall Hello!
Upcall Hello!
Upcall Hello!
Upcall Hello!
Upcall Hello!
Upcall Hello!
Upcall Hello!
```

The output of client side:
```
I said, "Hello!".
The Echo object replied, "Hello!".
I said, "Hello!".
The Echo object replied, "Hello!".
I said, "Hello!".
The Echo object replied, "Hello!".
I said, "Hello!".
The Echo object replied, "Hello!".
I said, "Hello!".
The Echo object replied, "Hello!".
I said, "Hello!".
The Echo object replied, "Hello!".
I said, "Hello!".
The Echo object replied, "Hello!".
I said, "Hello!".
The Echo object replied, "Hello!".
I said, "Hello!".
The Echo object replied, "Hello!".
I said, "Hello!".
The Echo object replied, "Hello!".
```

And you also can separately run server and client on different machines.

(3) eg3_impl is the server, eg3_clt is the client, they are based on omniNames(the nameing service).

Now we need do something about config file. There is a sample.cfg in ~/tmp/omniORB-4.1.1.

We create a server3.cfg and a client3.cfg in ~/tmp/cfg/.

```
diff -c sample.cfg server3.cfg
*** sample.cfg  2008-02-13 09:34:57.000000000 +0100
--- server3.cfg 2008-02-13 09:54:42.000000000 +0100
***************
*** 329,334 ****
--- 329,335 ----
  #   corbaname URI:
  #
  #   InitRef = NameService=corbaname::my.host.name
+    InitRef = NameService=corbaname::localhost
  #
  #
  #   Specify the Trading service and the interface repository using corbaloc:
```

```
diff -c sample.cfg client3.cfg
*** sample.cfg  2008-02-13 09:34:57.000000000 +0100
--- client3.cfg 2008-02-12 14:27:59.000000000 +0100
***************
*** 329,334 ****
--- 329,335 ----
  #   corbaname URI:
  #
  #   InitRef = NameService=corbaname::my.host.name
+    InitRef = NameService=corbaname::localhost
  #
  #
  #   Specify the Trading service and the interface repository using corbaloc:
```

We create a log directory for omniNames.
```
cd ~/tmp
mkdir log
```

In terminal 1, run the omniNames:
```
export OMNIORB_CONFIG=~/tmp/cfg/server3.cfg
omniNames -start -log ~/tmp/log
```
(This is for first time to run.)
```
omniName -log ~/tmp/log
```
(This is for not first time to run.)

In terminal 2, run the server:
```
cd ~/tmp/omniORB-4.1.1
cd src/examples/echo
unset OMNIORB_CONFIG
./eg3_impl
```

In terminal 3, run the client:
```
cd ~/tmp/omniORB-4.1.1
cd src/examples/echo
export OMNIORB_CONFIG=~/tmp/cfg/client3.cfg
./eg3_clt
```

The output of client side is same as previous case.

If the Nameing Service(omniNames) is not running, then you will get:
```
./eg3_clt
Caught system exception TRANSIENT -- unable to contact the server.
```

And you also can separately run server and client on different machines.
Please make sure to edit the config file correctly, for example:
You can replace "localhost" with your server's ip or domain name in server3.cfg and client3.cfg.

(4) The applications are same with previous, but the config file are different, follow an old way.

```
diff -c sample.cfg server4.cfg
*** sample.cfg  2008-02-13 10:07:09.000000000 +0100
--- server4.cfg 2008-02-13 10:23:49.000000000 +0100
***************
*** 329,334 ****
--- 329,335 ----
  #   corbaname URI:
  #
  #   InitRef = NameService=corbaname::my.host.name
+    InitRef = NameService=corbaname::192.168.249.130
  #
  #
  #   Specify the Trading service and the interface repository using corbaloc:
***************
*** 1041,1044 ****
  # bootstrap agent protocol.  This enables interoperability between omniORB
  # servers and Sun's javaIDL clients. When this option is enabled, an
  # omniORB server will respond to a bootstrap agent request.
! supportBootstrapAgent = 0
--- 1042,1045 ----
  # bootstrap agent protocol.  This enables interoperability between omniORB
  # servers and Sun's javaIDL clients. When this option is enabled, an
  # omniORB server will respond to a bootstrap agent request.
! supportBootstrapAgent = 1
```

```
diff -c sample.cfg client4.cfg
*** sample.cfg  2008-02-13 09:34:57.000000000 +0100
--- client4.cfg 2008-02-13 10:24:48.000000000 +0100
***************
*** 1042,1044 ****
--- 1042,1046 ----
  # servers and Sun's javaIDL clients. When this option is enabled, an
  # omniORB server will respond to a bootstrap agent request.
  supportBootstrapAgent = 0
+ bootstrapAgentHostname = 192.168.249.130
+ bootstrapAgentPort = 2809
```

In terminal 1, run the omniNames:
```
export OMNIORB_CONFIG=~/tmp/cfg/server4.cfg
omniNames -start -log ~/tmp/log
```
(This is for first time to run.)
```
omniName -log ~/tmp/log
```
(This is for not first time to run.)

In terminal 2, run the server:
```
cd ~/tmp/omniORB-4.1.1
cd src/examples/echo
unset OMNIORB_CONFIG
./eg3_impl
```

In terminal 3, run the client:
```
cd ~/tmp/omniORB-4.1.1
cd src/examples/echo
export OMNIORB_CONFIG=~/tmp/cfg/client4.cfg
./eg3_clt
```

The output of client side is same as previous case.

# BUILD Qt 

Tested with Qt 5.4.1.

Simply qmake && make.

# RUN Qt&omniORB EXAMPLES

Based on Duncan Grisby's reply, first I tried to run the omniORB code in a QThread, it works fine. Then I just run the omniORB code in my GUI code, it also works fine.

(1) eg1.tar.gz, just like the first example in echo.

```
cd ~/tmp
tar zxvf eg1.tar.gz
cd eg1
ls
echo.cpp  echo.h  echo.idl  eg1.pro  main.cpp  omniorbthread.cpp  omniorbthread.h
```

echo.h and echo.cpp are generated from echo.idl with omniidl.

```
omniidl -bcxx echo.idl
```

It will generate echo.hh and echoSK.cc.

```
mv echo.hh echo.h
mv echoSK.cc echo.cpp
```

Then you need to modify the echo.cpp to include "echo.h", not "echo.hh".

```
qmake
make
```

```
unset OMNIORB_CONFIG
./eg1

I said, "Hello!".
The Echo object replied, "Hello!".
OmniORBThread is over!
```

(2) eg3.tar.gz, just like the third example in echo.

```
cd ~/tmp
tar zxvf eg3.tar.gz
cd eg3
qmake
make
```

```
ls
client  echo.cpp  echo.h  echo.idl  eg3.pro  guiclient  guiclient2  server
```

server is a console server application which runs the omniORB code in a thread.
client is a console client application which runs the omniORB code in a thread.
guiclient is a gui client application which runs the omniORB code in a thread.
guiclient2 is a gui client application which runs the omniORB code in main thread.

You can use server3.cfg/client3.cfg or server4.cfg/client4.cfg to run these examples application.

# Log

* 2008-02-13, Liang Qi
* 2015-06-06, Updated for Qt 5 by Johan Thelin
