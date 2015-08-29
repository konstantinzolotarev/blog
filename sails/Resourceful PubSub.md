# Resourceful PubSub

Most applications that are created use same pattern.

- User fetch data from server
- Update/Create/Delete data

For this operations Sails.js team created `blueprints` that generates REST API for you models and allow you to work with records without need of creating Controllers, actions. But it's another article.

Assuming that we are creating Chat application. 
Our actions mostly will be this:

- Browser request data from server
- Server respond with data to browser (using Socket or AJAX)
- User fill form in browser and browser send data to server with new message
- Server validates data
- Create a new record in DB
- Reply to browser with success/error
- Broadcast Socket event to others that message created

Assuming that we don't use `blueprints` and `Resourceful PubSub` we need to implement all this steps in our application.
Lets start from server side:

Creating a model `Message.js`:
```javascript
module.exports = {
    attributes: {

        from: 'String',

        message: 'String'
    }
};
```

We wouldn't create anything complicated. So now we need a controller `MessageController.js`

```javascript
module.exports = {
    // Fetch list of messages
    index: function(req, res) {
        Message.find().exec(function(err, messages) {
            // ... error check and so on
            res.json(messages);
        });
    },

    // Create a new message
    create: function(req, res) {
        // ... Validation here
        Message.create({
            from: req.param('from'),
            message: req.param('message')
        }).exec(function(err, message) {
            // ... Validation
            // Send new message to all chat members using socket
            sails.sockets.blast('message', message);
            // Reply to browser who sent data
            return res.json(message);
        });
    }
};
```

Simple enough isn't it ?  If we will add validation to the model it will be even simplier. *Note that we didn't create edit and delete actions*
**But** sails.js provide you a way with creating more simple applications.
Just after you will create a model and controller `MessageController.js`:
```javascript
module.exports = {};
```

Holy guacamole ! Thats all ?

Sails.js `blueprints` will create a REST API for you.

- `GET /message` - will return all messages already created
- `POST /message` - will create a new message
- `PUT /message/:id` - will update message
- `DELETE /message/:id` - will remove message

Great ! But what about sockets ?
Here is whole magic. Sails.js also provide you with `Resourceful PubSub`.
It means that framework will be able to notify you on every model action that was made on backend side. You just need to subscribe to needed model using `sails.sockets.watch()` method.
After this you will be able to catch every model action in frontend using `io.socket.on()` method and name of the model you want to watch.

We have a chat application and we created `Message` model we will subscribe to updates using:
```javascript
// We 
io.socket.on('message', function(data) {
    // ... your code here
});
```

Now if somebody will create new message, udate existing one or remove message Sails.js will notify you about this.

Here is sample of newly created message:
```javascript
{
    data: {
        createdAt: "2014-08-01T05:50:19.855Z"
        id: 1
        from: "joe",
        message: "Hey !",
        updatedAt: "2014-08-01T05:50:19.855Z"
    },
    id: 1,
    verb: "created"
}
```

Looks very cool. But you told something about `sails.sockets.watch()` ?

Yeah. This method should be called on the backend and only after that you will receive notifications about model changes.

**But !** If you will fetch data from server using `io.socket.get('/message')` sails.js will automatically subscribe you to updates of `Message` model.

You can read a bit more details about this [here](http://sailsjs.org/documentation/reference/web-sockets/resourceful-pub-sub)

### Ok it's very useful for simple applications and what about somethign more complicated ?

Unfortunately Sails.js couldn't all work for you. And you will have to add notifications in your backend by yourself. 
Sails.js provides you with list of methods that could be used for PubSub events in Sails.js way.

**I'm using `Message` model for all examples, just because we are creating chat application** 
But all this methods are available at any model in your app.

- `Message.publishCreate()` - will send event `message` with `verb: 'created'` in data.
- `Message.publishUpdate()` - will send event `message` with `verb: 'updated'` in data. And `previous` will contain prev object before changes.
- `Message.publishDestroy()` - will send event `message` with `verb: 'destroyed'` in data.

There are more actions you will find [here](http://sailsjs.org/documentation/reference/web-sockets/resourceful-pub-sub).


`It's a GitHub repo so if you want to add something you could create a PR. Or if you want to know more about something you are welcome to create an issue`
