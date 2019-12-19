# MQTT-SN-Standardization
Results and lessons learned from implementing and running MQTT-SN in large scale, constrained node networks


## Initial Notes

### Background
- two implementations in [RIOT](https://github.com/RIOT-OS/RIOT)
  - [emcute](https://github.com/RIOT-OS/RIOT/tree/master/sys/net/application_layer/emcute): minimal, blocking implmentation optimized for extremely low ressource usage, using UDP
  - [asymcute](https://github.com/RIOT-OS/RIOT/tree/master/sys/net/application_layer/asymcute): flexible, asynchronous implementation, using UDP

### From a implementers (developers) perspective
- address encoding, e.g. in gateway discovery packets
  - how to encode IP endpoints (IP + port)
  - give examples in the Standard!

- state is ambiguous: is it allowed to have more than a single request/transaction pending?
  - quote part in the end where it denies this
  - quote part in beginning that leaves this open

- data packet size (block wise or similar)
  - protocol has no means of determining MTU

- examples!
  - gateway discovery for IPv6/UDP
  - QoS0-2 publish, subcribe examples
  - last will example
  
### From a network perspective
- TBD
