# An Article about basic [Sails.js](http://sailsjs.org) Socket system

## Introduction
Sails.js is a good framework and it has good Socket.io implementation out of the box. Awesome !

But when you read several articles about Socket.io + Express.js, tested something and install new Sails.js application. You realize that you don't understand anything. 

- How and where you need to create event listener in Sails.js ?
- When should you fire events ?
- WTF is Resourceful PubSub and why they are proud of it ?

Let I try to explain.

## Socket.io in Sails.js
Actually Sails.js has a bit different Socket system that you expects. 
No... Of course it has all Socket.io functions and this functions works good.
**But!**
Normally you creates events listeners on server and on client code. And send events between them. In Sails.js your event listeners are your Controller actions.

So if you will create a new controller `MessageController.js` with action `list` - it will became accessible via Socket (and HTTP also).

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
Client side code: 
```javascript
io.socket.get('/message/list', function(data, req) {
   // ... data will contain exact same JSON
});
```


