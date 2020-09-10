---
layout: post
title:  "Connection NodeJS With RabbitMQ"
date:   2020-09-10 11:40:00 +0700
categories: [rabbitmq, nodeJs, socket.io, ejs]
---
### RabbitMQ를 통해 자바서버의 정보를 얻어 웹을 연다###

```javascript
var express = require('express');
var app = express();
var io = require('socket.io').listen(30002);

var amqp = require('amqp');
var connection = amqp.createConnection({
	host : 'localhost'
});

var id;
var status;
var check;

app.set('views', __dirname + '/views');

//서버가 HTML렌더링을 할 때, EJS 엔진을 사용
app.set('view engine', 'ejs');
app.engine('html', require('ejs').renderFile);


app.get('/', (req, res) => {
	  res.render('index.ejs');
});

var server = app.listen(30000, function(){
	console.log("Start ...")
});

var queueToReceiveFrom = "hello";
connection.on('ready', function() {
	connection.queue(queueToReceiveFrom, {
		autoDelete : false
	}, function(queue) {
		console.log('Waiting messages...');
		queue.subscribe(function(messageReceived) {
		var strArray = messageReceived.data.toString().split('/');
		id = strArray[0];
		status = strArray[1];
		check = strArray[2];
		console.log(id);
		console.log(status);
		console.log(check);
		io.sockets.emit('reqMsg', id, status, check);
		});
	});
});

//서버에서 읽을 수 있는 정적 파일 위치
app.use(express.static('public'));

```