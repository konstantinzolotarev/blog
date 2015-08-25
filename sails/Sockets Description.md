# An Article about basic [Sails.js](http://sailsjs.org) Socket system

## Introduction
Sails.js is a good framework and it has good Socket.io implementation out of the box. Awesome !

But when you read several articles about Socket.io + Express.js, tested something and install new Sails.js application. You realize that you don't understand anything. 

- How and where you need to create event listener in Sails.js ?
- When should you fire events ?
- WTF is Resourceful PubSub and why they are proud of it ?

Let I try to explain.

## Socket.io in Sails.js
Actually Sails.js has a bit different Socket.io system that you imagine from 
