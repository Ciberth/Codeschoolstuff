# Codeschool Real Time web with nodejs

## Lesson 1

Node.js is not a web framework!
Callbacks are important
Difference between blocking and non-blocking!
Event loop that checks events.

```js

var http = require('http');

http.createServer(function(request, response) {
	response.writeHead(200);
	response.write("Hello, this is dog.");
	response.end();
}).listen(8080);

console.log('Listening on port 8080...');

```

```js

var http = require('http');

http.createServer(function(request, response){
	response.writeHead(200);
	response.write("Dog is running. ");
	setTimeout(function(){
		response.write("Dog is done.");
		response.end();
	}, 5000);
}).listen(8080);

```

## Lesson 2: Events

Events (cfr. DOM)
In node objects emit events.


```js

var EventEmitter = require('events').EventEmitter;
var logger = new EventEmitter();

logger.on('error', function(message){
	console.log('ERR: ' + message);
});

logger.emit('error', 'Spilled Milk');
logger.emit('error', 'Eggs Cracked');

```

```js
http.createServer(function(request, response){ ... });

// equals

var server = http.createServer();
server.on('request', function(request, response){ ... });
//
server.on('close', function(){ ... });

```


## Lesson 3: Streams

In the http.createServer(function(request, response))

The request = readable stream
The response = writable stream


```js

http.createServer(function(request, response){
	response.writeHead(200);
	
	request.on('readable', function(){
		var chunk = null;
		while(null != (chunk = request.read())){
			//console.log(chunk.toString());
			response.write(chunk);
		}
	});

	request.on('end', function(){
		response.end();
		});
}).listen(8080);

```

can be replaced by
(think pipe | in bash)

```js

http.createServer(function(request, response) {
	response.writeHead(200);
	request.pipe(response);
}).listen(8080);

```

### Echo Server

```js

http.createServer(function(request, response) {
	response.writeHead(200);
	request.pipe(response);
}).listen(8080);

```

and we do

``curl -d 'hello' http://localhost:8080``

we can see the hello back!

### Reading and writing a file

```js

var fs = require('fs');

var file = fs.createReadStream("readme.md");
var newFile = fs.createWriteStream("readme_copy.md");

file.pipe(newFile);

```
### Upload a file

```js

var fs = require('fs');
var http = require('http');

http.createServer(function(request, response) {
	var newFile = fs.createWriteStream("readme_copy.md");
	request.pipe(newFile);

	request.on('end', function(){
		response.end('uploaded!');
	});
}).listen(8080);

```

and from client

``curl --upload-file readme.md http://localhost:8080``

### Full uploadservice

```js

var fs = require('fs');
var http = require('http');

http.createServer(function(request, response) {
	var newFile = fs.createWriteStream("readme_copy.md");
	var fileBytes = request.headers['content-length'];
	var uploadedBytes = 0;

	request.on('readable', function(){

		var chunk = null;
		while(null != (chunk = request.read())){
			uploadedBytes += chunk.length;
			var progress = (uploadedBytes / fileBytes) * 100;
			response.write("Progress: " + parseInt(progrss, 10) + "%\n");
		}
	});

	request.pipe(newFile);

	request.on('end', function(){
		response.end('uploaded!');
	});
}).listen(8080);

```



## Lesson 4: Modules

```js
var http = require('http');
var fs = require('fs');
```

Let's create our own module!

custom_hello.js:

```js
var hello = function() {
	console.log("Hello!");
}
module.exports = hello;
```

custom_goodbye.js:

```js
exports.goodbye = function(){
	console.log("Bye!");
}
```

app.js: 

```js
var hello = require('./custom_hello');
var gb = reqiore('./custom_goodbye');
hello();
gb.goodbye();
```

If you only need it once you can also use ``require('./custom_goodbye').goodbye();``.

my_module.js:

```js
var foo = function(){}
var bar = function(){}
var baz = function(){}

exports.foo = foo
exports.bar = bar
```

app.js:

```js
var myMod = require('./my_module');
myMod.foo();
myMod.bar();
// but baz is private!
```

### Making HTTP Resuests

```js

var http = require('http');

var message = "Here's looking at you, kid.";
var options = {
	host: 'localhost', port: 8080, path: '/', method: 'POST'
}

var request = http.request(options, function(response){
	response.on('data', function(data){
		console.log(data);
	});
});

request.write(message);
request.end();
```

Let's now map it in a function!

```js

var http = require('http');

var makeRequest = function(message) {
	var options = {
		host: 'localhost', port: 8080, path: '/', method: 'POST'
	}

	var request = http.request(options, function(response){
		response.on('data', function(data){
			console.log(data);
		});
	});

	request.write(message);
	request.end();
}

makeRequest("Here's looking at you, kid.");
```

Let's now shrink it and make make_request.js

```js
var http = require('http');
var makeRequest = function(message) {
	...
}

module.exports = makeRequest;
```

so in app.js

```js
var makeRequest = require('./make_request');

makeRequest("Here's looking at you, kid");
makeRequest("Hello this is kid");

```

``npm install request`` 
use -g for global
will install under home/my_app/node_modules/request
 
 my_app/package.json
 ```json
{
	"name": "My App",
	"version": "1",
	"dependencies": {
		"connect": "1.8.7"
	}
}
 ```

 and ``npm install`` to install

 ### Semantic Versioning

Connect: 1.8.7

 1 Major
 8 Minor
 7 Patch
 Ranges: "~1" will fetch bigger then 1.0.0 (latest) but NOT 2.0.0 (Dangerous)
 ~1.8.7 will fetch 1.8.7 or bigger but lower then 1.9.0 (safe);
