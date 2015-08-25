# An Article about [Sails.js](http://sailsjs.org) Socket system

## Introduction
Sails.js is a good framework and has Socket.io implementation out of the box. Awesome !

But when you read several articles about Socket.io + Express.js, tested something and install new Sails.js application. You realize that you don't understand anything. 

Let I try to explain.

## Socket.io in Sails.js
Actually Sails.js has a bit different Socket system that you expects. 
No... Of course it has all Socket.io functions and this functions works good. And Sails.js uses Socket.io.
**But!**
Normally you creates events listeners on server and on client code. And send events (using `emit` etc.) between them. In Sails.js your event listeners are your Controller actions.

So if you will create a new controller `MessageController.js` with action `list` - it will became accessible via Socket (and HTTP of course).

Example: 
`MessageController.js`: 

```javascript
module.exports = {

    list: function(req, res) {
        // ... Some logic here
        return res.json({
            success: true
        });
    }
};
```

Now you will be able to access this action using `http://localhost:1337/message/list` and will get JSON from method.

And exact same action will be accessible using Socket `io.socket.get()` method.
**Note that action will have a callback with data returned form action**
Client side code:
```javascript
io.socket.get('/message/list', function(data, req) {
   // ... data will contain exact same JSON
});
```

You might think that it's not Socket this is simple AJAX request. You are wrong. All data will be sent and reseived using WebSocket protocol. So this is Socket.

Example above is equal to this in Socket.io implementation on server side:
```javascript
// ...Some initial code
// Emit welcome message on connection
io.on('connection', function(socket) {
    // Note that /message/list is just an event name that will be handled.
    socket.on('/message/list', function() {
        socket.emit('/message/list', {
            success: true
        });
    });
});
```

**Note that I don't talking about lot of other stuff that Sails.js brings, like request simulation, session, etc. We are talking only about sockets**

So Sails.js implementation wouldn't require you to use `emit()` method on frontend. It will provide you with list of methods that will represend HTTP requests. Documentation for all methods is [here](http://sailsjs.org/documentation/reference/web-sockets/socket-client)

And of course all this methods available only using Sails.js client side socket implementation. (It will be located into `assets/js/dependencies/sails.io.js`) [Repo](https://github.com/balderdashy/sails.io.js)

## Where to use `emit()` `broadcast()` in Sails.js ?
You will be able to use it in any Controller, Action, Service you will use in your sails.js application. 

All this methods will be available globaly in `sails.sockets` object. Just use `sails.sockets.emit()` or any other method inside your application.

**Note that it's not exact same methods as in Socket.io**
For example:
- Sails.js `emit()` method require a `socket` instance where to send data.
- Sails.js `broadcast()` method will send event to **Room** (including sender). While in Socket.io it will send event to everybody except you.
- In Sails.js there is a method `blast()` that will send event to all listeners.

There are more diferences in Sails.js sockets with Socket.io in backend. You could read documentation [here](http://sailsjs.org/documentation/reference/web-sockets/sails-sockets)

## Sails.js request object
You are using one code for HTTP requests and for Scokets. And it also brings some restrictions. Again restrictions... restrictions... restrictions...

Actually when you try to send data to action using WebSocket Sails.js mocking request object for you. So some methods wouldn't be available for you. 

But don't worry you will have additional methods and properties that will be available for you **only** in WebSocket requests. 
[Here is a table where you will be able to find all of them](http://sailsjs.org/documentation/reference/request-req)

## How to determinate WebSocket request in action ?

It's very simple. Sails.js authors did it for you just use `req.isSocket` property.

Example:
```javascript
module.exports = {
    list: function(req, res) {
        if (req.isSocket) {
            // Lets emit additional event to our user.
            sails.sockets.emit(req.socket, 'another.action');
            // And return data to our callback
            return res.json({
                success: true
            });
        }
    }
};
```

As you see Sails.js socket programming is a searching for ballance between emiting data using pure socket events (`sails.sockets.emit()`) and returning data using `return res.json()` functions in your actions.

Ohh. and a last thing.
Your response methods will still work with WebSockets too ;)
So you will be able to use `res.serverError();` or `res.notFound()` as well as `res.json()` and `res.ok()` methods.

Hope right now you have a better understanding of Sockets in Sails.js application.
