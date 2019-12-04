# Simple Syslog Server

[![Simple Syslog Server](syslog-sml.png)](https://pixabay.com/photo-3685581/)

A simple _evented_ syslog server and parser supporting UDP, TCP & TLS Transports

## Install

```sh
npm install simple-syslog-server
```

## Usage

Refer to the `logger` command for testing the examples included in this package.

```bash
man logger
```

### Synopsis
```js

const Syslog = require('simple-syslog-server') ;

// Create our syslog server with the given transport
const socktype = 'TCP' ; // or 'TCP' or 'TLS'
const address = '' ; // Any
const port = 20514 ;
var server = Syslog(socktype) ;

// State Information
var listening = false ;
var clients = [] ;
var count = 0 ;

server.on('msg', data => {
	console.log('message received (%i) from %s:%i\n%o\n', ++count, data.address, data.port, data) ;
	/*
	message received (1) from ::ffff:192.168.1.13:59666
	{
	  "facility": "daemon",
	  "facilityCode": 3,
	  "severity": "info",
	  "severityCode": 6,
	  "tag": "systemd[1]",
	  "timestamp": "2018-12-26T17:53:57.000Z",
	  "hostname": "localhost",
	  "address": "::ffff:192.168.1.13",
	  "family": "IPv6",
	  "port": 20514,
	  "size": 80,
		"msg": "Started Daily apt download activities.",
		"raw_msg": "<PRI>timestamp hostname tag: info"
	}	
	*/
})
.on('invalid', err => {
	console.warn('Invalid message format received: %o\n', err) ;
})
.on('error', err => {
	console.warn('Client disconnected abruptly: %o\n', err) ;
})
.on('connection', s => {
	let addr = s.address().address ;
	console.log(`Client connected: ${addr}\n`) ;
	clients.push(s) ;
	s.on('end', () => {
		console.log(`Client disconnected: ${addr}\n`) ;
		let i = clients.indexOf(s) ;
		if(i !== -1)
			clients.splice(i, 1) ;
	}) ;
})
.listen({host: address, port: port})
.then(() => {
	listening = true ;
	console.log(`Now listening on: ${address}:${port}`) ;
})
.catch(err => {
	if ((err.code == 'EACCES') && (port < 1024)) {
		console.error('Cannot listen on ports below 1024 without root permissions. Select a higher port number: %o', err) ;
	}
	else { // Some other error so attempt to close server socket
		console.error(`Error listening to ${address}:${port} - %o`, err) ;
		try {
			if(listening)
				server.close() ;
		}
		catch (err) {
			console.warn(`Error trying to close server socket ${address}:${port} - %o`, err) ;
		}
	}
}) ;

```

### TCP Transport
The above code uses the factory function `Syslog` to instantiate a server object, based on the transport protocol name given.  Alternately, a server instance can be created directly by calling the factory function property of the same name.  For example:

```js
const Syslog = require('simple-syslog-server') ;

// Create our syslog server with the given transport
const options = {} ;
const address = '' ; // Any
const port = 20514 ;
const listen = {host: address, port: port} ;
var server = Syslog.TCP(options) ;

server.on('msg', data => {
	console.log('message received from %s:%i\n%o\n', data.address, data.port, data) ;
})
.listen(listen)
.then(() => {
	console.log(`Now listening on: ${address}:${port}`) ;
}) ;


```
`options` to the constructor are as per calls to [net.createServer()](https://nodejs.org/api/net.html#net_net_createserver_options_connectionlistener "net.createServer"). 

The `listen` object passed to the `listen` method are the same options provided to [server.listen()](http://nodejs.org/api/net.html#net_server_listen "server.listen").

### TLS Transport

```js
const selfsigned = require('selfsigned') ;
const tls = require('tls') ;
const Syslog = require('simple-syslog-server') ;

let attributes = [{ name: 'commonName', value: 'localhost' }] ;
let x509 = selfsigned.generate( attributes, { days: 2 } ) ;

// Create our syslog server with the given transport
const tls_options = {
	key: x509['private'],
	cert: x509['cert'],
	ca: [ x509['cert'] ]
} ;
const address = '' ; // Any
const port = 20514 ;
const listen = {host: address, port: port} ;
var server = Syslog.TLS(options) ;

server.on('msg', data => {
	console.log('message received from %s:%i\n%o\n', data.address, data.port, data) ;
})
.listen(listen)
.then(() => {
	console.log(`Now listening on: ${address}:${port}`) ;
}) ;


```

Again, `options` relate to the [tls.createServer()](http://nodejs.org/api/tls.html#tls_tls_createserver_options_secureconnectionlistener "tls.createServer") call.  And `listen` to the [server.listen()](https://nodejs.org/api/tls.html#tls_server_listen) call of the `TLS.Server` object instance.

### UDP Transport


```js
const Syslog = require('simple-syslog-server') ;

// Create our syslog server with the given transport
const options = {type: 'udp4'} ;
const address = '' ; // Any
const port = 20514 ;
const listen = {host: address, port: port} ;
var server = Syslog.UDP(options) ;

server.on('msg', data => {
	console.log('message received from %s:%i\n%o\n', data.address, data.port, data) ;
})
.listen(listen)
.then(() => {
	console.log(`Now listening on: ${address}:${port}`) ;
}) ;


```

Finally, and predictably, `options` for the UDP transport relate to the [dgram.createSocket()](http://nodejs.org/api/dgram.html#dgram_dgram_createsocket_options_callback "dgram.createSocket") call.  For UDP sockets, rather than `listen`, they `bind` instead.  However, the principle is the same as other transport types.  Thus, a call to syslog `server.listen` method is identical to calling `bind`, and the `listen` options also relate to [socket.bind()](https://nodejs.org/api/dgram.html#dgram_socket_bind_port_address_callback).

## Acknowledgements

Like others who stand on the shoulders of giants, I'd like to acknowledge
the contributions of the following people/groups without which, more directly,
this modest Node-RED node would not be possible.

- [Mark Eschbach](https://github.com/meschbach "Mark Eschbach") and [chunpu](https://github.com/chunpu "chunpu") for their [initial work](https://github.com/meschbach/syslogd) on a node.js implementation of syslog on which [I forked](https://github.com/damoclark/simple-syslog-server "simple syslog server") to create this module.
- [rawpixel](https://pixabay.com/en/users/rawpixel-4283981/) for the [logo artwork](https://pixabay.com/en/graphic-icon-folder-data-file-3685581/) used in this module. 

## Licence

Portions created by Mark Eschbach and Chunpu are copyrighted and MIT licenced.  This module has been relicenced according to Apache 2.0

Copyright (c) 2019 Damien Clark

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at

[http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0 "Apache Licence")

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.

