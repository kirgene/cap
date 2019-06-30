
Description
===========

Network packet decoders (extracted from [cap](https://github.com/mscdex/cap) module)


Requirements
============

* [node.js](http://nodejs.org/) -- v4.0.0 or newer


Install
============

    npm install cap


Examples
========

* Capture and decode all outgoing TCP data packets destined for port 80 on the interface for 192.168.0.10:

```javascript
var Cap = require('cap').Cap;
var decoders = require('cap-decoders');
var PROTOCOL = decoders.PROTOCOL;

var c = new Cap();
var device = Cap.findDevice('192.168.0.10');
var filter = 'tcp and dst port 80';
var bufSize = 10 * 1024 * 1024;
var buffer = Buffer.alloc(65535);

var linkType = c.open(device, filter, bufSize, buffer);

c.setMinBytes && c.setMinBytes(0);

c.on('packet', function(nbytes, trunc) {
  console.log('packet: length ' + nbytes + ' bytes, truncated? '
              + (trunc ? 'yes' : 'no'));

  // raw packet data === buffer.slice(0, nbytes)

  if (linkType === 'ETHERNET') {
    var ret = decoders.Ethernet(buffer);

    if (ret.info.type === PROTOCOL.ETHERNET.IPV4) {
      console.log('Decoding IPv4 ...');

      ret = decoders.IPV4(buffer, ret.offset);
      console.log('from: ' + ret.info.srcaddr + ' to ' + ret.info.dstaddr);

      if (ret.info.protocol === PROTOCOL.IP.TCP) {
        var datalen = ret.info.totallen - ret.hdrlen;

        console.log('Decoding TCP ...');

        ret = decoders.TCP(buffer, ret.offset);
        console.log(' from port: ' + ret.info.srcport + ' to port: ' + ret.info.dstport);
        datalen -= ret.hdrlen;
        console.log(buffer.toString('binary', ret.offset, ret.offset + datalen));
      } else if (ret.info.protocol === PROTOCOL.IP.UDP) {
        console.log('Decoding UDP ...');

        ret = decoders.UDP(buffer, ret.offset);
        console.log(' from port: ' + ret.info.srcport + ' to port: ' + ret.info.dstport);
        console.log(buffer.toString('binary', ret.offset, ret.offset + ret.info.length));
      } else
        console.log('Unsupported IPv4 protocol: ' + PROTOCOL.IP[ret.info.protocol]);
    } else
      console.log('Unsupported Ethertype: ' + PROTOCOL.ETHERNET[ret.info.type]);
  }
});
```

Static methods
-----------------------

The following methods are available off of `require('cap-decoders')`. They parse the relevant protocol header and return an object containing the parsed information:

* Link Layer Protocols

    * **Ethernet**(< _Buffer_ buf[, < _integer_ >bufOffset=0])

* Internet Layer Protocols

    * **IPV4**(< _Buffer_ buf[, < _integer_ >bufOffset=0])

    * **IPV6**(< _Buffer_ buf[, < _integer_ >bufOffset=0])

    * **ICMPV4**(< _Buffer_ buf, < _integer_ >nbytes[, < _integer_ >bufOffset=0])

* Transport Layer Protocols

    * **TCP**(< _Buffer_ buf[, < _integer_ >bufOffset=0])

    * **UDP**(< _Buffer_ buf[, < _integer_ >bufOffset=0])

    * **SCTP**(< _Buffer_ buf, < _integer_ >nbytes[, < _integer_ >bufOffset=0])
