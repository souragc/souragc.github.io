---
layout: post
title: MultiUserChat
date: 2021-05-20 18:42:59
summary: Client server group chat application.
tags:
 - project
---

An application created for networks course project.

### Server

Written in C. Server first allocates memory as shared for each client to use. Then it waits for connection from client.
When a client connects, a seperate process is created to handle the client. User is first asked to login or register.
After this another process is created and this process waits for message from the user and forwards it to all other clients.
Parent waits for message from other clients and send it to the user.

Message is passed from user to user using a shareed memory. 200 bytes is given to each client. First byte is used to know
if the client is still active. This is used so that once a client disconnects, another client can reuse the same memory.

There are special messages. These are used for the following:
- When a client connects, other clients get a message saying new client joined.
- When others receive the above message, each client then sends a message to tell they are already active. This is used because
incoming clients won't know who all are active already.
- Another message is used when a client quits to notify others that client left the chat.

### Client

Uses tkinter module for the UI. First asks the user to login/register. After this, a chat window is opened. User can send 
messages from here. There is an option to view currently active users.

### Screenshots

Welcome window  
![welcome](/imgs/multiuserchat/welcome.png)

Register window  
![register](/imgs/multiuserchat/register.img)

Chat window  
![chat](/imgs/multiuserchat/chat.png)

Active users window  
![active](/imgs/multiuserchat/active.png)


### Code

You can find the full code of the project [here](https://github.com/souragc/Multiuserchat)
