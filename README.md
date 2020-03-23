# Daemon
A full rewrite of 1M5 Java in Rust. 

## Donate

[![Become a patron](https://resolvingarchitecture.io/images/become_a_patron_button.png)](https://www.patreon.com/1m5)

## Why rewrite in Rust?

* 1M5 is heavily networked and part of its targeted platforms are tiny electronics with limited resources.
* Rust has great support for threading and keeping threads safe. 
* Mozilla has a great reputation, Oracle nor Google does not. 
* Rust was open sourced from the beginning therefore not likely to cause future issues with copyright.
* Many of the platforms 1M5 is integrating with like Bitcoin, Monero, GNU Radio were written in C++ and a bit of a pain 
to integrate with Java, so ease of interoperating with them is important. 
* C, C++, and Go were considered but the first two don't have the memory protections I wish for (not entirely true with C++) 
and are often written in very terse methods and the latter uses garbage collection which I wanted to avoid and was written by Google who is slowly restricting Android development. 
* Learning Rust is exciting. Neither Clojure, Scala, or other JVM languages have produced much of that affect.
* 1M5 codebase is going to be much smaller with far less objects yet still get the benefit of object-orientation.

Notes: a re-design in routing was also necessary as the original routing across networks exposed the origination 
to the destination and every 1M5 relay in between. It's desired to change this to onion routing so that 1M5 relays
are not aware of whom is an originator nor destination. This requires building up lease sets similar to how TOR/I2P works.

## Design

This daemon runs as an OS service providing:

* a network router for routing Packets through networks to ensure censorship resistance
* network clients for each network supported (e.g. I2P, TOR, VPN) handling network interactions
* a service router for routing Envelopes through intra-/inter-services
* keychain service for interacting with keychain usb key like Nitrokey
* infovault service for persistence to/from personal usb key like Apricorn
* content distribution service for distributing information across network for backup and performance
* decentralized identification service for building reputation

Packets are used to route Envelopes across networks. 

Envelopes are routed within the destination node's 1M5 Daemon process to fulfill a request.

``` 

----------- OS Process --------------    ----------- OS Process --------------
-  -------------------------------  -    -  -------------------------------  -
-  -         SEDA Bus            -  -    -  - [API]    Browser            -  -
-  -------------------------------  -    -  -------------------------------  -
-                                   -    -------------------------------------
-  -------  ----------------------  -    
-  -     -  - Network Router     -  -    ------------ OS Process -------------
-  -     -  ----------------------  -    -  -------------------------------  -
-  -     -                          -    -  - [API]    Messenger          -  -
-  -     -  ----------------------  -    -  -------------------------------  -
-  -     -  - LiFi Client        -  -    -------------------------------------
-  -     -  ----------------------  -    
-  -     -                          -    ----------- OS Process --------------
-  -  S  -  ----------------------  -    -  -------------------------------  -
-  -  e  -  - Bluetooth Client   -  -    -  - [API]       CLI             -  -
-  -  r  -  ----------------------  -    -  -------------------------------  -
-  -  v  -                          -    -------------------------------------
-  -  i  -  ----------------------  -
-  -  c  -  - WiFi Direct Client -  -    ----------- OS Process --------------
-  -     -  ----------------------  -    -  -------------------------------  -
-  -  B  -                          -    -  - [API] External Service      -  -
-  -  u  -  ----------------------  -    -  -------------------------------  -
-  -  s  -  - HTTPS Client       -  -    -------------------------------------
-  -     -  ----------------------  -
-  -     -                          -
-  -     -  ----------------------  -
-  -     -  - VPN Client         -  -
-  -     -  ----------------------  -
-  -     -                          -
-  -     -  ----------------------  -
-  -     -  - TOR Client         -  -
-  -     -  ----------------------  -
-  -     -                          -
-  -     -  ----------------------  -
-  -     -  - I2P Client         -  -
-  -     -  ----------------------  -
-  -     -                          -
-  -     -  ----------------------  -
-  -     -  - Satellite Client   -  -
-  -     -  ----------------------  -
-  -     -                          -
-  -     -  ----------------------  -
-  -     -  - FS Radio Client    -  -
-  -     -  ----------------------  -
-  -     -                          -
-  -     -  ----------------------  -
-  -     -  - Service Router     -  -
-  -     -  ----------------------  -
-  -     -                          -
-  -     -  ----------------------  -
-  -     -  - Keychain           -  -
-  -     -  ----------------------  -
-  -     -                          -
-  -     -  ----------------------  -
-  -     -  - InfoVault          -  -
-  -     -  ----------------------  -
-  -     -                          -
-  -     -  ----------------------  -
-  -     -  - CDN                -  -
-  -     -  ----------------------  -
-  -     -                          -
-  -     -  ----------------------  -
-  -     -  - DID                -  -
-  -     -  ----------------------  -
-  -------                          -
-------------------------------------

```

### Components

External processes like the Browser, Messenger, and CLI can use the API to send/receive messages via services.
The API packs up the service request into an Envelope and transmits it to the SEDA Bus via its inbound external channel.
The SEDA Bus retrieves this message asynchronously sending it to the Service Bus.
If the message is a Packet, it sends it to the Network Router.
If the message is an Envelope, it sends it to the Service Router.

#### [SEDA Bus](https://github.com/resolvingarchitecture/seda-bus)

#### [Service Bus](https://github.com/resolvingarchitecture/service-bus)

#### [RA]

