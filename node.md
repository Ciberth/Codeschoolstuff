# Codeschool Real Time web with nodejs

## Lesson 1: Introduction

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



## Lesson 5: Express

```js

```


Express = Sinatra inspired framework

Install with ``npm install --save express``

```js
var express = require('express');
var app = express();

app.get('/', function(request, response){
	response.sendFile(__dirname + "/index.html");
});

app.listen(8080);
```

Twitter example
(wont work anymore, cuz of auth)
```js

var request = require('request');
var url = require('url');

app.get('/tweets/:username', function(req, response){
	var username = req.params.username;

	options = {
		protocol: "http:",
		host: 'api.twitter.com',
		pathname: '/1/statuses/user_timeline.json',
		query: {screen_name: username, count: 10}
	}

	var twitterUrl = url.format(options);
	request(twitterUrl).pipe(response);
});

```

Template things

``npm install --save ejs``

and in package.json

```json
"dependencies": {
	"express": "4.9.6",
	"ejs": "1.0.0"
}
```


```js
app.get('/tweets/:username', function(req, response){
	...
	request(url, function(err, res, body){
		var tweets = JSON.parse(body);
		response.locals = {tweets: tweets, name: username};
		response.render('tweet.ejs');
	});
});

```

tweets.ejs looks like this

```ejs
<h1>Tweets for @<%= name %></h1>
<ul>
	<% tweets.forEach(function(tweet){ %>
	<li><%= tweet.text %></li>
	<% }); %>
</ul>
```



## Lesson 6: Socket.io

duplex websockiets

``npm install --save socket.io``

app.js

```js
var express = require('express');
var app = express();
var server = require('http').createServer(app);
var io = require('socket.io')(server);

io.on('connection', function(client){
	console.log('Client connected...');
	// client is the browser
	client.emit('messages', {hello: 'world'});
});

app.get('/', function(req, res){
	res.sendFile(__dirname + '/index.html');
});

server.listen(8080);
```

index.html

```html
<script src="/socket.io/socket.io.js"><script>
<script>
	var socket = io.connect('http://localhost:8080');
	socket.on('messages', function(data){
		alert(data.hello);
	});
</script>
```



Sending messages to server

```js

io.on('connection', function(client){
	client.on('messages', function(data){
		console.log(data);
	});	
});

```

some jquery for index.html

```html

<script>
	var socket = io.connect('http://localhost:8080');
	$('#chat_form').submit(function(e){
		var message = $('#chat_input').val();
		socket.emit('messages', message); // emit messages event on the server
	});
</script>

```


Broadcasting messages

`` socket.broadcast.emit("message", 'Hello'); `` 



```js

io.on('connection', function(client){
	client.on('messages', function(data){
		client.broadcast.emit("messages", data);
		//broadcast to all other clients connected
	});
});

```

```html

<script>
	var socket = io.connect('http://localhost:8080');
	$('#chat_form').submit(function(e){
		var message = $('#chat_input').val();
		socket.emit('messages', message); // emit messages event on the server
		socket.on('messages', function(data){insertMessage(data)});//some jquery
		
	});
</script>

```


We need names now to know who is saying what


```js

io.on('connection', function(client) {
	client.on('join', function(name){
		client.nickname = name;
	});
});

```

and on index.html

```html
<script>
	var socket = io.connect('http://localhost:8080');
	server.on('connect', function(data){
		$('#status').html('Connected to chattr');
		nickname = prompt("What is your nickname?");
		server.emit('join', nickname); // notify server of the users nickname
	});
</script>

```

```js

io.on('connection', function(client) {
	client.on('join', function(name){
		client.nickname = name; // set the nickname associated with this client
	});

	client.on('messages', function(data){
		var nickname = client.nickname; // get name before broadcasting
		client.broadcast.emit("message", nickname + ": " + message); // broadcast
		client.emit("messages", nickname + ": " + message); // send msg back to client
	})
});

```







## Lesson 7: Persisting Data


app.js

```js

var messages = [];
var storeMessages = function(name, data){
	messages.push({name: name, data: data});
	if(messages.length > 10) {
		messages.shift(); // remove the first one
	}
}

io.sockets.on('connection', function(client) {
	client.on("messages", function(message) {
		client.get("nickname", function(error, name){
			client.broadcast.emit("messages", name + ": " + message);
			client.emit("messages", name + ": " + message);
		});
	});
});

```

```js

io.sockets.on('connection', function(client){
	...
	client.on('join', function(name){
		messages.forEach(function(message) {
			client.emit("messages", message.name + ": " + message.data);
		});
	});
});

```


Now what about writing to a db.

Redis!

`` npm install redis --save ``



```js

var redis = require('redis');
var client = redis.createClient();

client.set("message1", "hello, yes this is dog");
client.set("message2", "hello, no this is spider");

//to get out of the db

client.get("message1", function(err, reply){
	console.log(reply);
})

```



```js

//add a string to the messages listen
var message = "Hello, this is dog";
client.lpush("messages", message, function(err, reply){
	console.log(reply); // list length
});

var message = "Hello, no this is spider";
client.lpush("messages", message, function(err, reply){
	console.log(reply);
});

var message = "Hello, this is dog";
client.lpush("messages", message, function(err, reply){
	client.ltrim("messages", 0, 1); // keeps first two strigs and removes the rest
});


client.lrange("messages", 0, -1, function(err, messages){
	console.log(messages); // all strings in list
});

```





```js

var redisClient = redis.createClient();
var storeMessage = function(name, data){
	var message = JSON.stringify({name: name, data: data});
	// need to turn object into string to store in redis
	
	redisClient.lpush("messages", message, function(err, response){
		redisClient.ltrim("messages", 0,9);
	});
}

```

Output from list

```js

client.on('join', function(name){
	redisClient.lrange("messages", 0, -1, function(err, messages){
		messages = messages.reverse(); // so they are emitted in correct order
		
		messages.forEach(function(message) {
			message = JSON.parse(message);
			client.emit("messages", message.name + ": " + message.data);
		});
		
	});
});

```

Chatter list

```js
client.sadd("names", "Dog");
client.sadd("names", "Spider");
client.sadd("names", "Gregg");

client.srem("names", "Spider");

client.smembers("names", function(err, names){
	console.log(names);
});
```


on app.js

```js
client.on('join', function(name){
	//notify other clients a chatter has joined
	client.broadcast.emit("add chatter", name);
	redisClient.sadd("chatters", name); // add to chatters set
});

```

on index.html

```html

socket.on('add chatter', function(name){
	var chatter = $('<li>'+name+'</li>').data('name', name);
	$('#chatters').append(chatter);
});

```


on app.js

```js
client.on('join', function(name){
	client.broadcast.emit("add chatter", name);
	redisClient.smembers('names', function(err, names){
		names.forEach(function(name){
			client.emit('add chatter', name);
		});
	});

	redisClient.sadd("chatters", name);
});

```

Removing chatters when they disconnect



on app.js

```js
client.on('disconnect', function(name){
	client.get('nickname', function(err, name){
		client.broadcast.emit("remove chatter", name);
		redisClient.srem("chatters", name);
	});
});

```

on index.html

```html

socket.on('remove chatter', function(name){
	$('#chatters li[data-name='+ name + ']').remove();
});

```


------
