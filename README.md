# MQTT-SN-Standardization

Results and lessons learned from implementing and running MQTT-SN in large scale, constrained node networks.


## Initial Notes

### Background

- We created two MQTT-SN implementations in [RIOT](https://github.com/RIOT-OS/RIOT):
  - [emcute](https://github.com/RIOT-OS/RIOT/tree/master/sys/net/application_layer/emcute): minimal, blocking implmentation optimized for extremely low ressource usage, using UDP
  - [asymcute](https://github.com/RIOT-OS/RIOT/tree/master/sys/net/application_layer/asymcute): flexible, asynchronous implementation, using UDP

- We have done some large-scale measurements protocol comparison using MQTT-SN on top of IPv6/UDP (6LoWPAN) in IEEE802.15.4-based networks. Our results are published in 

  `Gündoğan, C.; Kietzmann, P.; Lenders, M.; Petersen, H.; Schmidt, T.C.; Wählisch, M. NDN, CoAP, and MQTT: A comparative measurement study in the IoT. In Proceedings of the 5th ACM Conference on Information-Centric Networking (ICN ‘18). ACM, New York, NY, USA, 21–23 September 2018; pp. 159– 171, doi:10.1145/3267955.3267967. `



### From a implementers (developers) perspective

**1.1 Address encoding, e.g. in gateway discovery packets**

In multi-hop networks, forwarding `GWINFO` packets by clients is mandatory for a working gateway discovery. However, the standard does not say anything on how addresses (or endpoint identifiers e.g. IP + Port) should be encoded in the `GwAdd` field. This is of course heavily dependent on the used transport protocols, but some general advice and preferably even specific encodings for the most common link layers would help.

- e.g. use string-based URI notation (RFC3986) or similar?

- See packet type `GWINFO` (sec. 5.4.3)



**1.2 PDU sizes**

MQTT-SN has no notion on MTU sizes supported of by the underlying transport protocols. Right now, applications have to know on which type of network they are deployed to make sure that the used packet sizes do not exceed the link/network layers MTU (e.g. see sec. 7.4 -> max packet size for ZigBee networks should be 60 bytes).

To make applications (and MQTT-SN implementations) actually portable, it would be very helpful to include MTU information in the protocol:

- make MQTT-SN aware of the local MTU and include means to signal insufficient MTU sizes when trying to send messages

- add means to discover the max-MTU that can be send to/from the gateway

It would furthermore be preferable, if MQTT-SN would support something similar to the block-wise transfer of data in CoAP (see RFC7959). This way, it would be no problem to transfer any amount of data even on constrained networks with (very) constrained MTU sizes.



**1.3 More examples**

Implementation of MQTT-SN could be made easier, if the standard would give some more concrete examples. In particular, some more detailed diagrams like fig. 5 (sec. 6.14) on the following procedures would be helpful:

1. gateway discovery -> direct and via client forwarding
2. Publishing and subscribing using QoS level 2
3. Example on the usage of the last will feature



### From a network perspective

**2.1 Gateway discovery problems**

The gateway discovery (section 6.1) is not applicable to every link layer. It states `A gateway may announce its presence by broadcasting periodically ...`. However, not all transports/link layers do provide a broadcast domain (e.g. IPv6, Bluetooth Low Energy). Furthermore, flooding/distributing `ADVERTISE` messages in multi-hop environments can be very inefficient (`... ADVERTISE messages are broadcasted into the whole wireless network ...`). 

The standard should be more precise on the usage and limitation of this approach and give options on what to do in environments without a broadcast domain. It should also differentiate between potential differences in link layer broadcast domain and network layer broadcasts (e.g. when using IP-based multi-hop networks).

For efficiency reasons, the option for a pure, on-demand gateway discovery could be defined. In that case, gateways would not broadcast periodic `ADVERTISE` messages anymore, and clients would use and monitor `SEARCHGW`/`GWINFO` messages to find gateways only if needed.

One more option that should be considered is using capabilities of the underlying network for discovering  gateways. For example if IPv6 is used, the gateway information could be distributed in the (local) network using the neighbor discovery (ND), DHCP, or DNS (see e.g. https://tools.ietf.org/html/draft-ietf-core-resource-directory-23, sec. 4.1). This would mitigate the need of periodic `ADVERTISE` packets.



### Misc

**3.1 Clarification of terms `Broker`, `Forwarder`, `Gateway`, and `Server`** 

The description of the overall MQTT-SN architecture in section 4 is partly confusion. The term `broker` is used in figure 1 and figure 2, but never mentioned in the text. Also this term is never used anywhere in the MQTT v5.0 specification. It would be beneficial to keep these terms coherent.

**3.2 Implementation overview**

Although this is not directly tied to the standard itself, it would be helpful to have some kind of website or similar, that lists the available MQTT-SN client and gateway implementations. See http://coap.technology/impls.html as example.

**3.3 Test suite**

Also not directly concerning the standard itself, it would be nice to have a supporting document defining a test suite that can be used to qualify client and gateway implementation.
