# ChatApp - Full Stack Messaging Application

A modern real-time chat application built from scratch.  
This repository documents the entire process — from setup to deployment — commit by commit. 



<div style="margin-bottom: 12px;">
<img src = "assets/react.png" style="height:25px; margin-right: 15px;" /> 
<img src = "assets/ts.png" style="height:25px; margin-right: 15px;" /> 
<img src = "assets/node.png" style="height:25px; margin-right: 15px;" /> 
<img src = "assets/aspnet.png" style="height:25px; margin-right: 15px;" /> 
<img src = "assets/cs.png" style="height:25px; margin-right: 15px;" /> 
<img src = "assets/DotNet.png" style="height:25px; margin-right: 15px;" /> 
<img src = "assets/mysql.png" style="height:25px; margin-right: 15px;" /> 
<img src = "assets/nginx.jpg" style="height:25px; margin-right: 15px;" /> 
<img src = "assets/linux.png" style="height:25px; margin-right: 15px;" /> 
</div>


## Commit Log
  - [1-Initial commit](#1-initial-commit) - Project root, git init, link to remote repo
  - [2-Frontend skeleton](#2-frontend-skeleton)
  - [3-Backend skeleton](#3-backend-skeleton)
  - [4-Database init](#4-database-init)
  - [5-Backend connection to DB](#5-backend-connection-to-db)
  - [6-Frontend to backend API connection](#6-frontend-to-backend-api-connection)
  - [7-Web socket connection](#7-web-socket-connection)
  - [8-Maintain client collection](#8-maintain-client-collection)
  - [9-Creating frontend UI components](#9-creating-frontend-ui-components)
  - [10-User registration](#10-user-registration)
  - [11-User login and logout](#11-user-login-and-logout)
  - [12-Send message](#12-send-message)
  - [13-Create new chat](#13-create-new-chat)
  - [14-Application deployment](#14-application-deployment)



Each section that follows describes details for particular commit message

## 1-Initial commit

Created project root folder and linked to GitHub Repo new branch dev:

- Initializing a new Git repository in the current folder. This command creates a hidden folder called .git/ that stores all the commit history, branches, and config. After this command, the project directory becomes a local Git repo, but it’s not connected to any remote yet.

  ```ps1
  git init
  ```

- Staging (marking) all files in the current folder for the next commit and creating a new commit — a permanent snapshot of the project: 

  ```ps1
  git add .
  git commit -m "Initial commit"
  ```

- Link  local repo to a remote repo (using SSH) with "origin" as the default name for the remote, and verify. When push or pull code, Git will use this GitHub repo as the remote source. 

  ```ps1
  git remote add origin git@github.com:berislav-vidakovic/ChatApp.git
  git remote -v
  ```
 
- Rename the current branch, -m stands for “move”, from main to dev

  ```ps1
  git branch -m main dev
  ```

- Push the content and link local branch to the remote branch. This will upload all commits and branch data to remote repo, create a branch named dev there if it doesn’t already exist and link local branch to it 

  ```ps1
  git push --set-upstream origin dev
  ```

## 2-Frontend skeleton

![DB](assets/react.png) 
![DB](assets/ts.png) 

Creating frondend directory and basic skeleton from the project root in Powershell Terminal of VSCode:

  ```ps1
  npm create vite@latest frontend -- --template react-ts
  cd frontend
  npm install
  npx tsc --noEmit
  ```

- Options selected: React and Typescript
- Explicitly set port in vite.config.ts

  ```ts
  export default defineConfig({
    plugins: [react()],
    server: { port: 5174 }
  })
  ```

- Created and updated .gitignore

Run frontend:

  ```ps1
  npm run dev
  ```

At this point, 
<a href="assets/fe001.png" target="_blank">
this is rendered </a> on browsing frontend URL:


    http://localhost:5174/

## 3-Backend skeleton

![DB](assets/aspnet.png) 
![DB](assets/cs.png) 


Creating backend directory and basic skeleton from the project root in Powershell Terminal of VSCode:


  ```ps1
  dotnet --version
  ```

Detected version 8 installed. Creating ASP.Net project

  ```ps1
  dotnet new webapi -n backend
  ```
  
Install EF Core and MySQL provider

  ```ps1
  cd backend
  dotnet add package Microsoft.EntityFrameworkCore
  dotnet add package Microsoft.EntityFrameworkCore.Design
  dotnet add package Pomelo.EntityFrameworkCore.MySql
  ```

Checked TargetFramework is net8.0 in backend.csproj

  ```xml
  <TargetFramework>net8.0</TargetFramework>
  ```

The following files are automatically created by .Net framework 
- launchsettings.json - controls how the app runs locally (development environment only)
- backend.http - a REST client test file, that can be used directly in VS Code (with the REST Client extension available ) to send sample HTTP requests to the app. It’s basically a convenient “mini Postman” built into VS Code


Commented out Https direction in Program.cs:

  ```cs
  // app.UseHttpsRedirection(); 
  ```

Run backend:

  ```ps1
  dotnet run
  ```


Testing backend by browsing mini endpoint provided by .Net framework will
<a href="assets/be001.png" target="_blank">
render this </a>

    http://localhost:5201/weatherforecast

For backend to listen on different port it can be changed in lauchsettings.json. To override port in launchsettings.json it can be set explicitly on backend starting:

  ```ps1
  dotnet run --urls "http://localhost:5202"
  ```



## 4-Database init  
![DB](assets/mysql.png) 


Update backend project structure, add Data folder:

      backend/
    ├─ Data/             
    ├─ Program.cs
    └─ appsettings.json

Add script init.sql that creates and populates DB tables in Data folder. 

<img src = "assets/ERD.png" /> 



Check installation and access MySQL shell:

  ```ps1
  mysql --version
  mysql -u root -p
  ```

Created new DB db_chatapp, new user with permissions granted and switched to new DB:

  ```sql
  SELECT VERSION();
  CREATE DATABASE db_chatapp CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
  CREATE USER 'barry75'@'localhost' IDENTIFIED BY 'abc123';
  GRANT ALL PRIVILEGES ON db_chatapp.* TO 'barry75'@'localhost';
  FLUSH PRIVILEGES;
  USE db_chatapp;
  ```

Relogin as new user and use database
  
  ```ps1
  mysql -u barry75 -p db_chatapp 
  ```

Run init.sql script and verify

  ```sql
  SOURCE C:/MyProjects/ChatApp/backend/Data/init.sql
  SHOW TABLES;
  ```
    
## 5-Backend connection to DB

Added 2 health-check endpoints:
- /api/ping - test backend
- /api/pingdb - test connection backend-DB

Updated appsettings.json with connection string:

  ```json
  "ConnectionStrings": {
    "DefaultConnection": "server=localhost;port=3306;database=db_chatapp;user=barry75;password=abc123"
  }
  ```

Updated project structure, added Controllers and Models:

      backend/
    ├─ Controllers/       # REST API
    ├─ Data/              # DbContext
    ├─ Models/            # EF entities
    ├─ Program.cs
    └─ appsettings.json


Added following backend files: 
- Models/HealthCheck.cs
  - Defined model class that maps DB table
- Data/ChatAppContext.cs
  - Defined DbContext-based class ChatAppContext
- Controllers/HealthController.cs
  - Defined ControllerBase-based class HealthController

Register the DbContext-based class ChatAppContext in Program.cs

  ```cs
  var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");
  builder.Services.AddDbContext<ChatAppContext>(options =>
      options.UseMySql(connectionString, ServerVersion.AutoDetect(connectionString)));
  ```


Register Controllers in Program.cs by adding the following:

  ```cs
  builder.Services.AddControllers(); 
  app.MapControllers(); 
  ```


At this point
- <a href="assets/beping.png" target="_blank">this is rendered 
</a> on browsing endpoint ping:

      http://localhost:5201/api/health/ping


- and<a href="assets/bepingdb.png" target="_blank"> this is rendered 
</a> on browsing endpoint pingdb:

      http://localhost:5201/api/health/pingdb




## 6-Frontend to backend API connection

### Status code package

  - Install

    ```bash
    npm install http-status-codes
    ```

  - Usage

    ```ts
    import { StatusCodes } from "http-status-codes"
    switch (res.status) {
      case StatusCodes.OK: // 200
      case StatusCodes.RESET_CONTENT: // 205
      case StatusCodes.BAD_REQUEST: // 400
      case StatusCodes.CONFLICT: // 409
    ```

      

  

### There are 2 API requests implemented:
- **GET**  
  - Request: 
    - endpoint: /api/users 
  - Response: 
    - format expected: [{userId, login, fullName, isOnline } ]
    - handling: set state areUsersReceived to true, writing response content to console
  - React state dependencies: 
    - isConfigLoaded - once config loaded backend URL is available
- **POST** 
  - Request 
    - endpoint: /api/users 
    - body: { message: "GET users response received" } 
  - Response: 
    - format expected: { acknowledged: true }
    - handling: writing response content to console  
  - React state dependencies:
    - isConfigLoaded - once config loaded backend URL is available
    - areUsersReceived - send confirmation after received GET response

### First step is creating endpoints on backend.

- Following files are added:
  
  - Models/User.cs
    - defined class that maps users table from DB
  - Controllers/UsersController.cs
    - defined ControllerBase-based class UsersController with HttpGet and HttpPost message handlers

- Updated DbContext-based class ChatAppContext, added member variable and mapping table name to model

  ```cs
  public DbSet<User> Users { get; set; }
  modelBuilder.Entity<User>().ToTable("users");
  ```


- At this point, <a href="assets/fb001.png" target="_blank">
  this is rendered 
</a> on browsing endpoint:

      http://localhost:5201/api/users




### Frontend-backend connection configuration on frontend

- Added 2 files: .env and env.production to project root with the following content respectively:

  ```js
  VITE_ENV=Development
  VITE_ENV=Production
  ```

- Added clientsettings.json in public directory:

  ```json
  "urlBackend": {
      "Development": {
          "HTTP": "http://localhost:5201",
          "WS": "ws://localhost:5201/websocket" },
      "Production": {
          "HTTP": "http://barryonweb.com/backend",
          "WS": "ws://barryonweb.com/backend/websocket" } }
  ```

- Reading config in utils.ts

  ```ts
  const currentEnv = import.meta.env.VITE_ENV as string; 
  export let URL_BACKEND_HTTP = ""; 
  export let URL_BACKEND_WS = ""; 
  fetch('/clientsettings.json'); // public dir, level index.html 
  URL_BACKEND_HTTP = config.urlBackend[currentEnv].HTTP; 
  URL_BACKEND_WS = config.urlBackend[currentEnv].WS;
  ```

### Next step is implement sending asynchronous GET and POST requests in frontend and get responses logged in console.



- Added the following files  
  - **src/services/restAPI.ts** - implemented generic send GET and POST functions, resolving response and passing it as argument to provided callback function. When the Promise from fetch() resolves, the result is passed to next chained  .then (res) which is the Response object returned by fetch(). It returns a Promise that resolves with parsed JSON and is passed to next .then (jsonResp).

    ```ts
    export async function sendGETRequest(endpoint: string, 
        handleResponse: (data: any) => void ): Promise<any> {
      fetch(`${URL_BACKEND_HTTP}/${endpoint}`) 
        .then(res => { 
          if (!res.ok) throw new Error(`HTTP error! status: ${res.status}`);
          return res.json();
        })
        .then( jsonResp => handleResponse( jsonResp ) )  
        .catch(err => console.error("GET request failed:", err));
    }

    export async function sendPOSTRequest(endpoint: string, msgBody: string, 
        handleResponse: (data: any) => void ): Promise<any> {
      fetch(`${URL_BACKEND_HTTP}/${endpoint}`, {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: msgBody, 
          }) 
        .then(res => { 
          if (!res.ok) throw new Error(`HTTP error! status: ${res.status}`);
          return res.json();
        })
        .then( jsonResp => handleResponse( jsonResp ) ) 
        .catch(err => console.error("POST request failed:", err));
    }
    ```

  - **src/services/utils.ts** - implemented functions that provide endpoint, response handling function, sets state update function (GET only) and body (POST only) 

    ```ts
    export async function getAllUsers(
      handleGetUsers: (data: any) => void,
      setUsersReceived:  Dispatch<SetStateAction<boolean>> ) {
        setUpdateUsersReceived(setUsersReceived); 
        sendGETRequest('api/users', handleGetUsers);
    }
    export async function confirmUsersReceived(handleConfirmUsers: (data: any) => void) {
        const body = JSON.stringify({ message: "GET users response received" });
        sendPOSTRequest('api/users', body, handleConfirmUsers);
    }
    ```

  - **src/services/eventHandlers.ts** - implemented response handling functions

    ```ts
    let setUsersReceivedHandler:  Dispatch<SetStateAction<boolean>>;
    export function setUpdateUsersReceived(
      setUsersReceived:  Dispatch<SetStateAction<boolean>> ){
        setUsersReceivedHandler = setUsersReceived;
    }
    export function handleGetUsers( jsonResp: any ) {
      setUsersReceivedHandler(true);
      console.log("Response to GET users: ", jsonResp );
    }
    export function handleConfirmUsers( jsonResp: any ) {
      console.log("Response to POST confirm users: ", jsonResp );
    }
    ```

- Updated App.tsx 
  - Added new states for loading configuration and getting users. New states are set as dependencies to trigger particular function execution when the value is changed (which happens upon successful execution of loadConfig and getAllUsers functions)
  - Functions getAllUsers and confirmUsersReceived are called with callback functions arguments. This is executed asyncronously, so once response received the function handleGetUsers will be invoked, same for confirmUsersReceived/handleConfirmUsers.

  ```ts
  const [isConfigLoaded, setConfigLoaded] = useState<boolean>(false);
  const [areUsersReceived, setUsersReceived] = useState<boolean>(false);
  useEffect( () => { loadConfig(setConfigLoaded); }, []);
  useEffect( () => { if( isConfigLoaded) getAllUsers(handleGetUsers, setUsersReceived); 
      else console.log("GET-Config not loaded yet");
    }, [isConfigLoaded]);
  useEffect( () => { if( !areUsersReceived ) console.log("POST-Users not received yet");
      else if( !isConfigLoaded ) console.log("POST-Config not loaded yet");
      else confirmUsersReceived(handleConfirmUsers);
    }, [isConfigLoaded, areUsersReceived]);  
  ```



### Final step is to include frontend in CORS policy on backend
CORS (Cross-Origin Resource Sharing) is a security feature implemented by browsers to restrict cross-origin HTTP requests. It ensures that a web application running on one domain cannot access resources from another domain unless explicitly allowed by the server.

- **Added entry in appsettings.json**

  ```json
  "CORS": {
    "AllowedOriginsFrontend": [
      "http://localhost:5174"
    ]
  },
  ```
- **Apply CORS policy in Program.cs**

  ```csharp
  string[]? allowedOrigins = builder.Configuration.GetSection("CORS:AllowedOriginsFrontend").Get<string[]>();
  if( allowedOrigins != null )
    builder.Services.AddCors(options =>
    {
      options.AddPolicy("AllowFrontends",
          policy =>
          {
            policy.WithOrigins(allowedOrigins)
                    .AllowAnyHeader()
                    .AllowAnyMethod();
          });
    });
  app.UseCors("AllowFrontends"); 
  ```
      
At this point, 
<a href="assets/fe002.png" target="_blank">
  GET and POST request/response are logged in console 
</a> on browsing frontend:

    http://localhost:5174/

Note: React intentionally mounts (and unmounts) components twice in development mode to help detect unsafe side effects.


## 7-Web socket connection

### Added support for Web socket health-check messages
- frontend sends challenge {"type":"healthCheck", "content":"ping"}
- backend responses with {"type":"healthResponse", "content":"pong"}

### Backend support for Web socket 

- Added file Middleware/WebSocketMiddleware.cs with WebSocketMiddleware class definition and its method InvokeAsync implementation. InvokeAsync is called the ASP.NET Core middleware framework for each request that passes through WebSocketMiddleware in the pipeline.

- In Program.cs enabled Web socket and registered WebSocketMiddleware:

  ```cs
  using System.Net.WebSockets; 
  var webSocketOptions = new WebSocketOptions
  {
      KeepAliveInterval = TimeSpan.FromMinutes(2),
  };
  app.UseWebSockets(webSocketOptions);
  app.UseMiddleware<WebSocketMiddleware>(); 
  ```
- Sending and receiving WS messages from backend perspective is 
<a href="assets/Ws.png" target="_blank"> shown here. </a>

### Frontend support for Web socket 

- Added **src/services/webSocket.ts** 
  - establishes WS connection by creating new WebSocket object
  - sets connection state variable to true in handling onopen event
  - handles incoming WS message by handling onmessage event:
  ```ts
  export async function connectWS(
    setWsConnected:  Dispatch<SetStateAction<boolean>> ) {
      ws = new WebSocket(URL_BACKEND_WS);
      ws.onopen = () => { setWsConnected(true); };
      ws.onmessage = (event) => { handleWsMessage(event.data); };
  ```
- Updated src/services/eventHandlers.ts - parsing incoming message as JSON and processing 
- Updated src/services/utils.ts - added function sendWsHealthCheck
- Updated src/App.tsx - new state added for monitoring WS connection. On mount WS connection is established, but depending on configuration loaded. Healt check message is sent, but depending on new state - only if WS connection is already established.

  ```ts
  const [isWsConnected, setWsConnected] = useState<boolean>(false);
  useEffect( () => { if( isConfigLoaded) connectWS(setWsConnected); 
      else console.log("WS-Config not loaded yet");
  }, [isConfigLoaded]);
  useEffect( () => { if( isWsConnected) sendWsHealthCheck(); 
      else console.log("WS not established");
    }, [isConfigLoaded, isWsConnected]);
  ```

At this point, Web socket connect, send and receive 
<a href="assets/wsfe01.png" target="_blank">
   are logged in console 
</a> on browsing frontend:

    http://localhost:5174/


## 8-Maintain client collection

### Current workflow on frontend mount:

1. Load configuration - get backend URL for HTTP and WS
2. GET request - /api/initclient - Init message to get ID and save it to sessionStorage
    - backend adds or updates client list with clientId and WS=null and sends ID in response
3. GET request - /api/users - Get users from DB
    - backend checks provided ID in IsValidClientID and sends response
4. POST request - /api/users - Confirm users received
    - backend checks provided ID in IsValidClientID and sends response
5. WebSocket connect - /websocket - establish WS connection
    - backend checks provided ID in IsValidClientID
    - frontend handles event WebSocket::onopen - protocol upgrade and handshake
6. WebSocket healthCheck - /websocket - send ping message to check WS connection
    - backend checks provided ID in IsValidClientID and sends response 
    - frontend handles event WebSocket::onmessage - receive pong 

Backend will maintain client collection in ClientManager's thread-safe dictionary of Client objects (contains WebSocket and Datetime members):

  ```cs
  private ConcurrentDictionary<Guid, Client> _clients { get; set; }
  ```

Client time stamp will be updated on each incoming message. The right place to add timestamp update is method ClientManager::IsValidClientID:

  ```cs
  public bool IsValidClientID(string? id)
  {
    if (Guid.TryParse(id, out Guid parsedClientId))
      if (_clients.ContainsKey(parsedClientId))
      {
        _clients[parsedClientId].SetTimeStamp();
        return true;
      }
    return false;
  }
  ```


### Client collection Happy path:

- frontend retrieves ID from sessionStorage and sends it in Init message
- backend checks ID received
  - if existing in collection - set its WebSocket object to null
  - if not existing - add new client in collection with WebSocket = null
  - returns ID in response { id: &lt;newID&gt; }
- frontent stores &lt;newID&gt; from response to sessionStorage
- frontend retrieves ID from sessionStorage and sends it in any API request
  - backend checks ID, updates timestamp and sends response
- frontend retrieves ID from sessionStorage and sends it in WS connect
  - backend checks ID, updates timestamp and provides protocol upgrade and handshake
- frontend retrieves ID from sessionStorage and sends it in WS message
  - backend checks ID, updates timestamp and processes WS message

### Client collection Error handling:
- frontend provides no ID, invalid ID or valid ID but no existing client in collection
  - in WS message or any API message other than Init message
    - backend generates new ID and sends  { reset: &lt;newID&gt; }  
  - in WS connect message
    - backend 
      - temporarily accepts WS connection 
      - generates new ID and sends  { reset: &lt;newID&gt; }
      - closes WS connection
- frontend 
  - stores &lt;newID&gt; in sessionStorage
  - refreshes browser





### Remove inactive clients from the collection
  - Client closes browser - backend removes client from the list
    ```cs
    if (msgMetaData.MessageType == WebSocketMessageType.Close) // Closing WS connection
    {
      Console.WriteLine($"WebSocket connection CLOSED ");
      _clientManager.RemoveClientByWS(webSocket);
      await webSocket.CloseAsync(WebSocketCloseStatus.NormalClosure, "Connection closed due to idle timeout", CancellationToken.None);
      break;
    }
    ```

  - Idle timeout on backend
    - ClientManager creates Timer on first client added and deletes timer on last client removed from the list    
    - Client object resets current timestamp on each client activity (API request or WS message received)
    - Periodically check all clients 
      - On idle timeout expired (period since last Activity interval exceeds timeout) 
        - close WS connection 
        - remove client from the list

    - Timer implementation
      1. Load settings
          ```cs
          string keyIdleTimeout = "ClientMonitor:IdleTimeoutSec";
          string keyCheckInterval = "ClientMonitor:CheckIntervalMin";
          if (config.GetSection(keyIdleTimeout).Exists())
            _idleTimeoutSec = int.Parse(config[keyIdleTimeout]!);
          if (config.GetSection(keyCheckInterval).Exists())
            _checkIntervalMin = int.Parse(config[keyCheckInterval]!);
          ``` 
      2. Time-elapsed function
          ```cs
          private async void CleanupClients(object? sender, ElapsedEventArgs e)
          {
            foreach (var kvp in _clients)
            {
              if( _clients[kvp.Key].GetIdleTime() > _idleTimeoutSec )
                RemoveClient(kvp.Key);
            }
          }
          ```
      3. TimerStart function - called on AddClient for adding the first client
          ```cs
          public void TimerStart()
          {
            _activityMonitor = new System.Timers.Timer(_checkIntervalMin * 60 * 1000);
            _activityMonitor.AutoReset = true;
            _activityMonitor.Elapsed += CleanupClients;
            _activityMonitor.Start();
          }
          ```
      4. TimerStop function - called on RemoveClient for removing the last client
          ```cs
          public void TimerStop()
          {
            _activityMonitor.Stop();
            _activityMonitor.Dispose();
          }
          ```
      
  - Idle timeout on frontend
    - updated generic POST and GET sending functions to check reset message

## 9-Creating frontend UI components

- Adding style sheet style.css
- Added components/ChatList.tsx
- Added components/ChatWindow.tsx 
- Updated App.tsx

UI contains now all main components:

<img src="assets/UIdummy.png" />

Next step is add Login user dialog, User register dialog and New chat dialog  

-


## 10-User registration 

### Client state management

There are folowing client states: 
1. Connected - initial state
2. LoggedIn - after login procedure
3. Disconnected - removed from client list on backend due to idle timeout 

- Client model for No user logged in
    - users table (user_id, login, full_name, isonline)

- Client model for User logged in
    - users table  
    - user_id logged in
    - chats that the user participates in
    - all messages that belong to the users's chats
    - chat_id that is currently selected

- Client model for disconnected state
    - no data

- Model state update events
  - Event: WS disconnect
    - LoggedIn -> Disconnected
    - Connected -> Disconnected
  - Event: Connect button clicked or reset message from server received
    - Disconnect -> Connected
    - LoggedIn -> Connected
  - Event: any change in users table from any client (user record updated or new user registered) 
    - client receives realtime WS message and updates users model 
    - Connected
    - LoggedIn
  - Event: new message sent in chat the user participates 
    - client updates messages model
    - LoggedIn
  - Event: creating new chat where the user is added 
    - client updates chats model 
    - LoggedIn

State transitons are defined in the following FSM table:
<table border="1" cellspacing="0" cellpadding="6">
  <thead>
    <tr>
      <th>Current State</th>
      <th>Event</th>
      <th>Next State</th>
      <th>Action</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>LoggedIn</td>
      <td>WS_DISCONNECT</td>
      <td>Disconnected</td>
      <td>Notify user, stop realtime updates, attempt reconnect</td>
    </tr>
    <tr>
      <td>Connected</td>
      <td>WS_DISCONNECT</td>
      <td>Disconnected</td>
      <td>Show offline indicator, schedule reconnect</td>
    </tr>
    <tr>
      <td>Disconnected</td>
      <td>CONNECT_CLICKED / RESET_FROM_SERVER</td>
      <td>Connected</td>
      <td>Open WS, reinitialize models</td>
    </tr>
    <tr>
      <td>LoggedIn</td>
      <td>CONNECT_CLICKED / RESET_FROM_SERVER</td>
      <td>Connected</td>
      <td>Refresh session, reconnect cleanly</td>
    </tr>
    <tr>
      <td>Connected</td>
      <td>USERS_UPDATED (WS message)</td>
      <td>Connected</td>
      <td>Update users model (no state change)</td>
    </tr>
    <tr>
      <td>LoggedIn</td>
      <td>USERS_UPDATED (WS message)</td>
      <td>LoggedIn</td>
      <td>Update users model (no state change)</td>
    </tr>
    <tr>
      <td>LoggedIn</td>
      <td>MESSAGE_RECEIVED (WS message)</td>
      <td>LoggedIn</td>
      <td>Append message to messages model</td>
    </tr>
    <tr>
      <td>LoggedIn</td>
      <td>NEW_CHAT_CREATED (WS message)</td>
      <td>LoggedIn</td>
      <td>Add new chat to chats model</td>
    </tr>
  </tbody>
</table>



Both user registration and login function are available in the state 1 only.

### Initial state

- Added interfaces.ts and created interface User:
  ```ts 
  export interface User {
    userId: number;
    login: string;
    fullname: string;
    isonline: boolean;
  } 
  ```

- Updated App.tsx with new states usersRegistered and isUserLoggedIn
- Updated ChatList with parameter usersRegistered
- Updated eventHandlers.ts GET response handler to map 
  - received users JSON [{userId, login, fullName, isOnline}] 
  - to usersRegistered array of User objects (React state variable)
    ```ts
    export function handleGetUsers( jsonResp: any ) {
      setUsersReceivedHandler(true);
      const mappedUsers: User[] = jsonResp.map((u: any) => ({
        userId: u.userId,
        login: u.login,
        fullname: u.fullName,  
        isonline: u.isOnline   
      }));
      setUsersRegisteredHandler(mappedUsers);
    }
    ```

### User registration

On enter full name and user login
- Check if login and full name already exists
- Send POST request to register new user
- Backend updates DB and send response confirmation
- Backend broadcasts WS message to all clients with new user registered


#### Frontend 
  - App.tsx: provide arguments to RegisterDialog 
    - connection state isWsConnected
    - state management function setUsersRegistered 
    - state usersRegistered       
      ```ts
      {showRegisterDialog && (
        <RegisterDialog
          isWsConnected={isWsConnected}  
          setShowRegisterDialog={setShowRegisterDialog}
          setUsersRegistered={setUsersRegistered}
        />
      )}
      ```
  - RegisterDialog.tsx: 
    - adjusted function signature to accept parameters 
      ```ts
      function RegisterDialog({
        isWsConnected, setShowRegisterDialog, setUsersRegistered,
        }: {
          isWsConnected: boolean;
          setShowRegisterDialog: Dispatch<SetStateAction<boolean>>;
          setUsersRegistered: Dispatch<SetStateAction<User[]>>;
        }) 
      ```
    - enabled accept Enter as Confirm button click
      ```ts
      useEffect(() => {
        document.querySelector("#inputLogin")?.focus();
      }, []);
      ```
      ```ts
      onKeyDown={(e) => {
        if (e.key === "Enter") {
          e.preventDefault();
          handleConfirmClick();
        }
      }}
      ```

    - trigger event: user register confirmation button clicked 
      - get values from UI elements 
        - add ref attribute to input elements
          ```ts
          ref={loginRef}
          ref={fullnameRef}
          ```
        - add reference to RegisterDialog function
          ```ts
          const loginRef = useRef<HTMLInputElement>(null);
          const fullnameRef = useRef<HTMLInputElement>(null); 
          ```
      - confirm handler
        - check connection, use references, check non-empty  
          ```ts
          const handleConfirmClick = () => {
            if (!isWsConnected) {
              alert("You are disconnected.");
              setShowRegisterDialog(false);    
              return;
            }  
            const login = loginRef.current?.value.trim() ?? "";
            const fullname = fullnameRef.current?.value.trim() ?? "";
            if (!login || !fullname) {
              alert("Please fill in both fields.");
              return;
            }
          ```
        - assign setUsersRegistered callback to refererence in eventHandlers.ts and asynchronously call registerUser in utils   providing arguments: login, fullName
          ```ts          
          setUpdateUsersRegistered(setUsersRegistered); 
          registerUser(login, fullname);
          ```
  - utils.ts: define registerUser which 
    - creates body as JSON message and, calls POST providing endpoint, body and response handler (defined in eventHandlers.ts)
      ```ts
      export async function registerUser(login: string, fullname: string) {
        const body = JSON.stringify({ register: { login, fullname } } );
        sendPOSTRequest('api/users', body, handleUserRegister);
      }
      ```
  - eventHandlers.ts:
    - define state management reference (callback) and assignment function 
      ```ts
      let setUsersRegisteredHandler:  Dispatch<SetStateAction<User[]>>;
      export function setUpdateUsersRegistered(
        setUsersRegistered:  Dispatch<SetStateAction<User[]>> ){
          setUsersRegisteredHandler = setUsersRegistered;
      }
      ```
    - define response handler which parses JSON response and updates user model appending new user using callback state function
      ```ts
      export function handleUserRegister( jsonResp: any ){
        if( jsonResp.acknowledged ) { 
          const newUser: User = {
            userId: jsonResp.user.userId,
            login: jsonResp.user.login,
            fullname: jsonResp.user.fullName,
            isonline: jsonResp.user.isOnline,
          };
          setUsersRegisteredHandler(prev => {
            const dupe = prev.some(u => u.userId === newUser.userId);
            return dupe ? prev : [...prev, newUser];
          });
        }
        else {
          alert("NOT registered: User already exists");
        }
      }
      ```
    - update WS message handler
      ```ts
      export async function handleWsMessage( jsonMsg: any ) {
        if( jsonMsg.acknowledged )
          handleUserRegister(jsonMsg);
      }
      ```
  - restAPI.ts: adjusted generic POST to provide handler with JSON response also for Conflict(409) or BadRequest(400)
      ```ts
      export async function sendPOSTRequest(endpoint: string, msgBody: string, handleResponse: (data: any) => void ): Promise<any> {
        const postUrl = `${URL_BACKEND_HTTP}/${endpoint}` + `?id=${sessionStorage.getItem("myID")}`;
        console.log("Sending POST: ", `${postUrl} Body:${msgBody}` );
        fetch( postUrl, {
                method: "POST",
                headers: { "Content-Type": "application/json" },
                body: msgBody, 
        }) 
          .then(async (res) => { 
            console.log("...received POST response!");
            const jsonResp = await res.json();
            if( isResetMessageReceived(jsonResp) )
              reconnectApp();
            else
              handleResponse( jsonResp );
            if (!res.ok) throw new Error(`HTTP error! status: ${res.status}`);
          })
          .catch(err => console.log("POST request failed:", err));
      }
      ```

#### Backend
  - handles POST request for new user registration
    - UsersController.cs: update DB and send response to POST that registration was successful
      ```cs
      string? login = credentials.GetProperty("login").GetString();
      string? fullname = credentials.GetProperty("fullname").GetString();
      if (string.IsNullOrWhiteSpace(login) || string.IsNullOrWhiteSpace(fullname))
          return BadRequest(new { acknowledged = false, error = "Missing login or fullname" });
      Console.WriteLine($"Register request: login={login}, fullname={fullname}");
      bool exists = await _context.Users // Check if user already exists
          .AnyAsync(u => u.Login == login || u.FullName == fullname);
      if (exists)
        return Conflict(new { acknowledged = false, error = "User already exists" });
      var newUser = new User // Create and insert new user
      {
          Login = login,
          FullName = fullname
      };
      _context.Users.Add(newUser);  // marks entity as "Added"
      await _context.SaveChangesAsync(); // INSERT new user INTO db table
      Console.WriteLine($"New user inserted: {login}");
      var response = new { acknowledged = true, user = newUser };
      _clientManager.BroadcastWsMessage(response, clientId);
      return Ok(response); 
      ```
    - ClientManager.cs: broadcast new user registration details to all other clients by WebSocket
      ```cs
      public async void BroadcastWsMessage(object message, string clientId)
      {
        if (!Guid.TryParse(clientId, out Guid clientGuid))
          return;
        string json = JsonSerializer.Serialize(message, new JsonSerializerOptions
        {
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase
        });
        var buffer = Encoding.UTF8.GetBytes(json);
        var segment = new ArraySegment<byte>(buffer);
        foreach (var client1 in _clients)
        {
          if (client1.Key == clientGuid)
            continue;
          WebSocket? ws = client1.Value.GetWS();
          if (ws != null)
            if (ws.State == WebSocketState.Open)
              _ = ws.SendAsync(segment, WebSocketMessageType.Text, true, CancellationToken.None);
        }
      }
      ```



## 11-User login and logout

### Login and logout scenarios

- The following scenarios trigger login and logout actions
  #### 1. Login  

    1.1 User clicks on  Login button

    - POST request sent: { userId } to api/session/login
      - backend updates DB status for userId to isOnline=true
      - POST response: { userOnline = true, userId, messages }
        - frontend model update: setCurrentUserId, setCurrentChatId, setMessages

    - WS Broadcast message sent to update user status
      - frontend update setUsersRegistered with new status for particular user

    1.2 Backend restores login state after browser refresh
  
    - WS connection established and backend gets userId
      - SSoT: if userId isOnline in DB
        - retrieve messages for userId 
        - sends WS message refresh with userId, chatId received and messages
          - frontend updates model, if chatId == null sets default

  #### 2. Logout
      
    2.1 User clicks on  Logout button

    - POST request sent: { userId } to api/session/logout 
      - backend updates DB status for userId to isOnline=false
      - POST response: { userOnline = false, userId }
        - frontend model update (null): setCurrentUserId, setCurrentChatId, setMessages
    - WS Broadcast message sent to update user status
      - frontend update setUsersRegistered with new status for particular user

    2.2 Backend detects idle timeout 
    - if client is online  
      - backend updates DB status for userId to isOnline=false
      - WS Broadcast message sent to update user status

            { type = "userSessionUpdate", status = "WsStatus.OK", data = new { userId, isOnline = false } }

        - Frontend handling: update model to offline status


  #### 3. Refresh browser  
    - on WS established client sends 3 parameters obtained from sessionStorage
      - id (mandatory)
      - userId (optional)
      - chatId (optional)
    - on protocol upgrade and handshake backend checks
      - id - if invalid sends reset message
      - userId 
        - if provided and status is online sends userRestoreLogin message
          - frontend restores logged in status

  #### 4. Close browser  
    - not handled: should send logout if logged in before browser closing
    - backend should update offline status in Db and broadcast

  
  
    

### Login

- Workflow
  - Frontend send POST Request with userId in body to the endpoint /api/login
  - Backend
    - update DB status for userId to online=true
    - send Response - message model to user logged in, providing participating chats and messages
    - broadcasts to all clients (including user just logged in) new user status 
  - Frontend
    - handle Response - updates state variable Message[]
    - handle incoming WS message - updates status variable currentUserId

- Backend implemetation
  - updates DB status for userId to online=true. Having userId, backend executes the following SQL query
    ```sql
    UPDATE users SET isonline=true WHERE user_id=<userId>
    ```
  - sends Response - model to user logged in - providing participating chats and messages. Having userId, backend executes one of the following SQL query to obtain relevant records from message table:
    - using JOIN:
      ```sql
      SELECT m.* FROM messages m JOIN chat_users c ON m.chat_id=c.chat_id WHERE c.user_id=<userId>
      ```
    - using sub-query:
      ```sql
      SELECT * FROM messages WHERE chat_id IN (SELECT chat_id FROM chat_users WHERE user_id=<userId>)
      ``` 

  - broadcasts to all clients (including user just logged in) new user status using WS

  - new model Messages that matches result set returned from SQL query
    ```cs
    public class Message
    {
      [Column("msg_id")]
      public int MessageId { get; set; }

      [Column("chat_id")]
      public int ChatId { get; set; } 

      [Column("user_id")]
      public int UserId { get; set; }

      [Column("datetime")]
      public DateTime Timestamp { get; set; }

      [Column("text")]
      public required string Text { get; set; }
    }
    ```
  - added member to ChatAppContext:
    ```cs
    public DbSet<Message> Messages { get; set; }
    ```
  - added LoginController with POST handler in endpoint /api/login
    - update user status to online=true
    - retrieve messages and send filtered for user in POST response 
    - broadcast WS message with new user online status

- Frontend implementation 
  - App.tsx passes registered users as argument to LoginDialog
    ```ts
    function LoginDialog({ 
        setShowLoginDialog, usersRegistered, isWsConnected 
      }: { 
        setShowLoginDialog: Dispatch<SetStateAction<boolean>>;
        usersRegistered: User[];
        isWsConnected: boolean;
    }
    ) {
    ```
  - LoginDialog 
    - adjusts parameter list
      ```ts
      function LoginDialog({ 
          setShowLoginDialog, usersRegistered, isWsConnected 
        }: { 
          setShowLoginDialog: Dispatch<SetStateAction<boolean>>;
          usersRegistered: User[];
          isWsConnected: boolean;
      }
      ) {
      ```
    - creates reference to selected userId:
      ```ts
      const selectedUserRef = useRef<HTMLSelectElement>(null);
      ```
    - feeds list with offline users and attaches the ref:
      ```ts
      {(
        <select ref={selectedUserRef}>
        {usersRegistered
          .filter(u => !u.isonline )
          .map(u => (
            <option key={u.userId} value={u.userId}>
              {u.fullname}
            </option>
          ))
        }
        </select>
      )}
      ```
    - handles Confirm button click
      ```ts

      ```
  - util.ts implements loginUser function
  
    - update messages state 
  - App.tsx - added state variabless
    - currentUserId
    - selectedChatId
    - messages Message[] { }
  - handles WS message
    - update currentUserId
    - update selectedChatId (if any)
  

### Logout

- Frontend send POST Request with logout message { logout: userId }
- Backend
  - updates DB for user to online=false. Having userId, backend executes the following SQL query
    ```sql
    UPDATE users SET isonline=false WHERE user_id=<userId>
    ```
  - send Response - confirm logout 
  - broadcasts to all clients (including the one logged off) new user status
- Frontend 
  - handle Response - clear state var Messages[]  
  - handle WS new status message - clear state var currentUserId 


## 12-Send message

This is the core part of application. Sending message feature is what ChatApp makes it what it is.

- Frontend sends WS message containing
  - message content
  - timestamp
  - chat id
  - sender userId
- Backend
  - updates DB with new message. Having text, timestamp, chatid and senderid backend executes the following SQL query
    ```sql
    INSERT INTO messages(chat_id, user_id, datetime, text) VALUES(<chat_id>, <userId>, <timestamp>, "New message");
    ```
  - broadcast message with metadata to all related clients

## 13-Create new chat

- Frontend sends POST Request containing
  - chat participants userIds - userId1 and userId2
- Backend
  - check if group already exists
      - group all users in array in column:
        ```sql
        SELECT chat_id, JSON_ARRAYAGG(user_id) AS users FROM chat_users GROUP BY chat_id;  
        ```
  - if group does not exist
    - adds new chat 
      ```sql
      INSERT INTO chats VALUES(); 
      INSERT INTO chat_users (chat_id, user_id) VALUES((SELECT LAST_INSERT_ID()),userId1); 
      INSERT INTO chat_users (chat_id, user_id) VALUES((SELECT LAST_INSERT_ID()),userId2); 
      ```
    - broadcast message with new chatid to all related clients
  - sends Response with chatid (new or existing) for client to select it



## 14-Application deployment


### PowerShell

#### Test domain 

    Resolve-DnsName barryonweb.com

### Linux terminal

#### Connect

    ssh root@barryonweb.com

#### Set root password

    passwd

#### Check Linux version

    cat /etc/os-release

#### Check all users on system, create new su user and enable SSH

    getent passwd | awk -F: '$7 ~ /bash$/ {print $1, $3}'
    adduser barry75
    usermod -aG sudo barry75
    mkdir -p /home/barry75/.ssh
    nano /home/barry75/.ssh/authorized_keys
    chown -R barry75:barry75 /home/barry75/.ssh
    chmod 700 /home/barry75/.ssh
    chmod 600 /home/barry75/.ssh/authorized_keys















