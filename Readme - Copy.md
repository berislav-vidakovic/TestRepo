# ChatApp – Full Stack Messaging Application

A modern **real-time chat application** built from scratch using React, TypeScript, ASP.Net (C#), MySQL, and WebSockets.  
Demonstrates full-stack architecture, real-time communication, and client management.

<div style="margin-bottom: 12px;">
<img src="assets/react.png" style="height:25px; margin-right: 15px;" />
<img src="assets/ts.png" style="height:25px; margin-right: 15px;" />
<img src="assets/node.png" style="height:25px; margin-right: 15px;" />
<img src="assets/aspnet.png" style="height:25px; margin-right: 15px;" />
<img src="assets/cs.png" style="height:25px; margin-right: 15px;" />
<img src="assets/DotNet.png" style="height:25px; margin-right: 15px;" />
<img src="assets/mysql.png" style="height:25px; margin-right: 15px;" />
<img src="assets/nginx.jpg" style="height:25px; margin-right: 15px;" />
<img src="assets/linux.png" style="height:25px; margin-right: 15px;" />
</div>

---

## Project Overview

- **Frontend:** React (TypeScript)  
- **Backend:** ASP.Net Core Web API (C#)  
- **Database:** MySQL  
- **Real-time communication:** WebSockets  
- **Deployment:** Linux (Nginx)  

**Key Features:**

- User registration and retrieval via REST API  
- Real-time messaging using WebSockets  
- Health-check and session management for clients  
- Thread-safe client collection with idle-timeout removal  

---

## Live Demo & Screenshots

<a href="https://barrytheanalyst.eu" target="_blank">Run App</a>

<a href="Details.md" target="_blank">View Commit Log</a>


## Endpoints and message structures

### REST API

All GET and POST requests with endpoints are available 
<a href="http://localhost:5201/swagger/index.html">on this link</a> automatically created by Swagger.

### Websocket

Endpoint

    /websocket

Parameters

    ?id=08e42c86-b8b6-495d-b202-98f039df90c1

WsbSocket message format is unified, each message following this structure:

    { type, status, data: { } }

Frontend WS messages

      { type: "healthCheck", status: "WsStatus.Request", data: { id, content: "ping" } }
      

  - Handling on backend - send response

        { type = "health", status = "WsStatus.OK", data = new { response = "pong" } }



Backend WS messages

  - Single client

        
        { type = "reset", 
          status = "WsStatus.ResetContent", 
          data = {
            reset:"08e42c86-b8b6-495d-b202-98f039df90c1", 
          }
        }

  - Broadcast message

        { type = "userSessionUpdate", status = "WsStatus.OK", data = new{ userId, isOnline } }

        { type = "userRegister", status = "WsStatus.OK", data = { acknowledged = true, user = newUser } }






## Setup Instructions

### Prerequisites
- Node.js  
- .Net 8  
- MySQL  
- VSCode recommended  

### Clone & Install
  ```bash
  git clone git@github.com:berislav-vidakovic/ChatApp.git
  cd ChatApp
  ```

## Contributions 

The following features can be added to improve this app

### Delete user
Enable user deletion, similar like user registration enabled added. The open question is how to handle existing chats, especially 2-user chats - whether to delete them

### Rename chat
Similar to Teams, enable defining custom chat name. This would require model change, adding chat name as new attributte

### Edit or delete message sent 
Within limited interval sent message is available to edit or delete

### Search
Search message content, users, chats by name

### Favorites
As Personal preferences group some chats to Favorites group. Require model change, store or mark particular chats as favorites for particular user

### Join chat
Join to existing chat

### Message notification
Show circle with number of new messages in inactive chat
