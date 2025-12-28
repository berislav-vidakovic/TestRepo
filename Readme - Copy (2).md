# Java backend with MongoDb, CI/CD and Nginx 



<div style="margin-bottom: 12px;">
<img src = "src/main/resources/static/images/java.png" style="height:35px; margin-right: 15px;" /> 
<img src = "src/main/resources/static/images/spring.png" style="height:35px; margin-right: 15px;" /> 
<img src = "src/main/resources/static/images/mongodb.png" style="height:35px; margin-right: 15px;" /> 
<img src = "src/main/resources/static/images/cicd.png" style="height:35px; margin-right: 15px;" /> 
<img src = "src/main/resources/static/images/yaml.png" style="height:35px; margin-right: 15px;" /> 
<img src = "src/main/resources/static/images/nginx.jpg" style="height:35px; margin-right: 15px;" /> 
</div>


## Table of Contents

0. [Prerequisites](docs/Prerequisites.md)
1. [Java Spring backend](#1-java-spring-backend)
2. [MongoDB](#2-mongodb)
3. [Nginx configuration](#3-nginx-configuration)
4. [Register backend as service](#4-register-backend-as-service)
5. [CI/CD pipeline](#5-cicd-pipeline)
6. [WebSocket](#6-websocket)
7. [Get all users from MongoDB](#7-get-all-users-from-mongodb)
8. [Register new User and hashing password](#8-register-new-user-and-hashing-password)
9. [JWT Authentication incremental build](#9-jwt-authentication-incremental-build)
10. [Login with password and logout](#10-login-with-password-and-logout)
11. [MongoDB collections messages and chats](#11-mongodb-collections-messages-and-chats)
12. [Sending Ws message](#12-sending-ws-message)
13. [Introducing Role-Based Access Control (RBAC)](#13-introducing-role-based-access-control-rbac)

   

## 1. Java Spring backend 
<img src = "src/main/resources/static/images/java.png" style="height:35px; margin-right: 15px;" /> <img src = "src/main/resources/static/images/spring.png" style="height:35px; margin-right: 15px;" /> 


- Generate Spring Boot Project on  https://start.spring.io

    - Select latest stable Spring Boot (no RC2, SNAPSHOT), Java 21
    - Add Dependencies:

      - Spring Web (for REST API)
      - WebSocket (for WebSocket support)
      - Spring Boot DevTools
    
    - Download and Extract

- Define Port (default is 8080) in application.yaml

    ```yaml
    server:
      port: 8081
    ```

- Build, Run

    ```bash
    mvn clean package -DskipTests
    mvn spring-boot:run
    ```
  **- Built jar file** target\chatappjn-0.0.1-SNAPSHOT.jar

- Git  <a href="docs/Git.md">
create remote repo, init, commit and  push
</a>

- Add ping endpoint

  - Create Controllers/PingController.java

- Add CORS policy for frontend access

  - Create <a href="docs/CORS.md">Config/CorsConfig.java</a>


## 2. MongoDB
<img src = "src/main/resources/static/images/mongodb.png" style="height:35px; margin-right: 15px;" /> 

- Access MongoDB remotely or locally

  ```bash
  mongosh "mongodb://barry75@barryonweb.com:27017/chatappdb"
  mongosh -u barry75 -p --authenticationDatabase chatappdb
  ```

- Create <a href="docs/MongoDb.md"> user, database, collection and document
</a>



- Connect backend to DB
  - Update application.yaml with DB connection details
  - Add MongoDB dependency to pom.xml
  - Add Model
  - Add Repository - subclass of MongoRepository
  - Add Controller Controllers/PingDbController.java

## 3. Nginx configuration
<img src = "src/main/resources/static/images/nginx.jpg" style="height:35px; margin-right: 15px;" /> 


- Create <a href="docs/basic-nginx-cfg.md">
basic Nginx config file chatjn.barryonweb.com
</a>
 
- Copy, Enable Nginx site and check sites enabled

    ```bash
    scp chatjn.barryonweb.com barry75@barryonweb.com:/var/www/chatapp/nginx/
    sudo cp /var/www/chatapp/nginx/chatjn.barryonweb.com /etc/nginx/sites-available/
    sudo ln -s /etc/nginx/sites-available/chatjn.barryonweb.com /etc/nginx/sites-enabled/
    ls -l /etc/nginx/sites-enabled/
    ```

- Issue SSL certificate for the subdomain

  ```bash
  sudo certbot --nginx -d chatjn.barryonweb.com
  ```

  - SSL manager will <a href="docs/ssl-nginx-cfg.md">
update Nginx config file </a>

- Check Nginx syntax and reload

  ```bash
  sudo nginx -t
  sudo systemctl reload nginx
  ```


- Build backend, copy and run manually 

  ```bash
  mvn clean package
  scp target/chatappjn-0.0.1-SNAPSHOT.jar barry75@barryonweb.com:/var/www/chatapp/backend/chatjn/
  java -jar chatappjn-0.0.1-SNAPSHOT.jar
  ```

- Test 

  - Test locally
    ```bash
    curl http://localhost:8081/api/ping
    curl http://localhost:8081/api/pingdb
    ```

  - Test via Nginx + SSL
    ```bash
    curl -k https://chatjn.barryonweb.com/api/ping
    curl -k https://chatjn.barryonweb.com/api/pingdb
    ```


## 4. Register backend as service

- Create <a href = "docs/Service.md"> chatappjn.service file
</a> and copy to /etc/systemd/system

  ```bash
  scp chatappjn.service barry75@barryonweb.com:/etc/systemd/system/ 
  ```

- Reload systemd to register the service

      sudo systemctl daemon-reload

- Start/stop the service, check status, enable auto start on boot 

      sudo systemctl start chatappjn
      sudo systemctl stop chatappjn
      sudo systemctl status chatappjn
      sudo systemctl enable chatappjn

- Follow logs in realtime

      sudo journalctl -u chatappjn -f

- Enable no password to restart service in /etc/sudoers and verify

      sudo visudo 
      barry75 ALL=(ALL) NOPASSWD: /bin/systemctl restart chatappjn.service
      sudo -l -U barry75



## 5. CI/CD pipeline
<img src = "src/main/resources/static/images/yaml.png" style="height:35px; margin-right: 15px;" /> <img src = "src/main/resources/static/images/cicd.png" style="height:35px; margin-right: 15px;" /> 


  - Create folder .github\workflows 
  - Add <a href="docs/initial-yaml.md">yaml file</a> for deployment, reload Nginx and restart backend service
  - Github
    - Repo → Settings → Secrets and variables → Actions
      - Create SSH_PRIVATE_KEY in Repository Secrets
      - Copy content of ~\\.ssh\github_ci private key


## 6. Websocket 

- Add 4 Java files
  - Services/<a href="docs/Client.md">Client.java</a>
  - Services/<a href="docs/IdleMonitor.md">IdleMonitor.java</a>
  - WebSockets/<a href="docs/SessionMonitor.md">SessionMonitor.java</a>
  - WebSockets/<a href="docs/WebSocketHandler.md">WebSocketHandler.java</a>

- Add <a href="docs/WebSocketCfg.md">WebSocketConfig file</a>
- Add timeout parameters in application.yaml

    ```yaml
    websocket:
      timeout-mins: 1
      check-interval-sec: 45
    ```

## 7. Get all users from MongoDB

- Create and populate users collection

  ```js
  db.users.insertMany([
    { login: "shelly", full_name: "Sheldon", isonline: false },
    { login: "lenny",  full_name: "Leonard", isonline: false },
    { login: "raj",    full_name: "Rajesh",  isonline: false },
    { login: "howie",  full_name: "Howard",  isonline: false }
  ])
  db.users.find()
  ```
- Add Model, Repository, Controller
- MongoDB automatically generates string id field

## 8. Register new User and hashing password 

- Extend collection documents with new field with password (update all or only documents where password is missing)

  ```js
  db.users.updateMany(
    {},
    { $set: { password: "" } }
  )
  db.users.updateMany(
    { password: { $exists: false } },
    { $set: { password: "" } }
  )
  ```

- Add passwordEncoder dependency to pom.xml
- Add <a href="docs/SecurityConfig.md">SecurityConfig.java</a> with @Bean PasswordEncoder and SecurityFilterChain   
- Add @Autowired PasswordEncoder, ObjectMapper, WebSocketHandler to UsersController

## 9. JWT Authentication incremental build

### 1. Refresh endpoint returning dummy tokens

- AuthorizationController.java 
  - Received Request: { refreshToken } 
  - Sending Response  { dummyAccessToken, dummyRefreshToken }
  - Sending WS broadcast { type: userSessionUpdate, data } 

- Enable endpoint /api/auth/refresh in SecurityConfig.java

    ```java
    .requestMatchers("/api/auth/refresh").permitAll() 
    ```

### 2. Create Model and Repository with dummy Controller check

- Add RefreshToken class (Model) that matches MongoDB collection
  ```js
  { "_id", "userid", "token", "expires" } 
  ```
- Add RefreshTokenRepository class   
- Check in Controller if the received token exists in MongoDB collection
  - if exists
    - if expired send code 201 (Created)
    - if valid send code 200 (OK)
  - if it does not exist send code 201 (Created)
  - Response sending 
    - dummyAccessToken
    - refreshToken
      - dummy for status 201
      - received one for status 200

### 3. Controller check upgrade

- For invalid or expired refreshToken
  - Return HttpStatus.UNAUTHORIZED (401)  
- For valid refreshToken
  - Find user in users collection by userId from refreshToken collection
  - Update user status to online
  - Generate new refreshToken
  - Return HttpStatus.OK (200) with { dummyAccessToken, refreshToken, userId, isOnline: true }   
- MongoDB CRUD actions
  - **Delete** dummy users from user collection
    ```js
    db.users.deleteOne({login:'p'})
    db.users.deleteMany({login: { $in:['b','c']}})
    db.users.deleteMany({login: { $nin:['lenny','shelly','raj','howie']}})

    ```

  - **Read** userid by full name, user count, particular fields:
    ```js
    db.users.find({full_name:'Sheldon'},{_id:1})
    db.users.countDocuments()
    db.users.find({},{login:1,_id:0,isonline:1})
    ```
    OUTPUT: [ { _id: ObjectId('692326918a68875daa63b113') } ]  
  - **Create** dummy token with valid userid in MongoDB:
    ```js
    db.refreshTokens.insertOne({ 
      userid: ObjectId('692326918a68875daa63b113'), 
      token:"initialRefreshToken", 
      expires: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000) })
    ```  
  - **Update** token by userId / set all users to offline
    ```js
    db.refreshTokens.updateOne(
      { userid: ObjectId("692326918a68875daa63b113") },  // filter
      {
        $set: {
          token: "validRefreshToken",
          expires: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)
        }
      }
    );
    db.users.updateMany({},{$set: {isonline: false}})
    ```
- Added UserRepository reference (Autowired) to AuthController
  - Update user status to online for user in users collection found by userId from refreshToken collection

- Renewed refreshToken dummy: newRefreshToken

- Testing on Frontend  
  1. Request { refreshToken: dummyRefreshToken }  
      - Expected Response: HttpStatus.UNAUTHORIZED (401)
  2. Request { refreshToken: validRefreshToken }  
      - Token valid - renew refreshToken and Expiry date and save to MongoDB collection (update)
      - Expected Response: HttpStatus.OK (200) { dummyAccessToken, newRefreshToken, userId, isOnline: true }

### 4. Valid refresh token path - token renewal

- Added UserMonitor
- Added useridle timeout and interval into application.yaml
- Updated SessionMonitor for Autologout if user is logged in
- Added JWT dependencies into pom.xml
- Added Config/JwtBuilder.java 
  ```java
  public class JwtBuilder {
    private static final Key SECRET_KEY = 
      Keys.hmacShaKeyFor("KeyForJWTauthenticationInChatApp".getBytes());
    private static final long EXPIRATION_TIME_MS = 60*60*1000; // 1 hour
    public static String generateToken(String userId, String username) {
      return Jwts.builder()
              .setSubject(username)
              .claim("userId", userId)
              .setIssuedAt(new Date())
              .setExpiration(new Date(System.currentTimeMillis() + EXPIRATION_TIME_MS))
              .signWith(SECRET_KEY, SignatureAlgorithm.HS256)
              .compact();
    }
  }
  ```
- AuthController
  - Call UserMonitor::updateUserActivity on user login (valid refreshToken)
  - Generate new refreshToken and save to DB 
    ```java
    String refreshToken = UUID.randomUUID().toString();
    ```
  - Generate new accessToken 
    ```java
    String newAccessToken = JwtBuilder.generateToken(user.getId(), user.getLogin());
    ```

### 5. Handling Request with accessToken in Authorization Bearer header

- Added static method getSecretKey in Config/JwtBuilder.java
- Added file Config/JwtValidator.java 
  ```java
  try { 
    Claims claims = Jwts.parserBuilder()
      .setSigningKey(JwtBuilder.getSecretKey())  
      .build()
      .parseClaimsJws(accessJWT)
      .getBody();    
    request.setAttribute("userId", claims.getSubject());
  } 
  catch (Exception e) {
    response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
    response.getWriter().write("Invalid or expired token");
    return;
  }
  ```
  - Request enters JwtValidator filter first. The filter runs before any controller

- Response Status code For missing Authorization header: 400 (Bad Request) 
- Response Status code For invalid/expired accessToken: 401 (Unauthorized) 

### 6. Adding support for Frontend 2-retry pattern   

- Adjust filter to return JSON for invalid/expired token and missing authorization header - updated Config/JwtValidator.java 
  ```java
  response.setContentType("application/json");
  Map<String, Object> error = Map.of(
    "acknowledged", false,
    "error", "Invalid or expired token"
  );
  response.getWriter().write(new ObjectMapper().writeValueAsString(error));  
  ```

## 10. Login with password and logout


- SecurityConfig → defines which endpoints are public/protected
- JwtValidator → validates the token and sets Spring Security authentication
- PublicEndpoints - static class as SSoT for public endpoints, used by both SecurityConfig and JwtValidator
- Set authentication for protected edndpoints in JwtValidator:
  ```java
  // Set Spring Security Authentication - tells Spring Security that the request is authenticated
  UsernamePasswordAuthenticationToken auth =
          new UsernamePasswordAuthenticationToken(username, null, List.of());
  SecurityContextHolder.getContext().setAuthentication(auth);
  ```
- register JwtValidator to be executed first in chain, in SecurityConfig:
  - Add member variable and constructor
    ```java
    public class SecurityConfig {
      private final JwtValidator jwtValidator;
      public SecurityConfig(JwtValidator jwtValidator) {
          this.jwtValidator = jwtValidator;
      }
    ```
  - Call addFilterBefore() directly on http, outside the authorizeHttpRequests lambda:
    ```java
    .addFilterBefore(jwtValidator, UsernamePasswordAuthenticationFilter.class);
    ```
- Workflow
  1. Request comes in.
  2. JwtValidator runs before Spring Security (addFilterBefore)
      - If endpoint is public → skips JWT validation
      - If endpoint is protected → validates JWT, sets authentication
  3. Spring Security checks .anyRequest().authenticated()
      - If authentication is set → passes
      - If authentication is missing/invalid → 403 Forbidden

- Added Autowired passwordEncoder member to AuthController
- Added endpoint /login to AuthController
  - White list in SecurityConfig::filterChain 
  - White list in Filter JwtValidator (not protected i.e. no accessToken required)
- Added /logout protected endpoint
  - White list in SecurityConfig::filterChain 
  
## 11. MongoDB collections messages and chats

- Rename field in MongoDB collection, drop collection, show collections
  ```js
  db.users.updateMany({}, { $rename: {"full_name":"fullName"}});
  db.chats.drop()
  show collections
  db.getCollectionNames()
  ```

- Create chats collection initially populated
  ```js
  const sheldon = db.users.findOne({ fullName: "Sheldon" })._id;
  const leonard = db.users.findOne({ fullName: "Leonard" })._id;
  const rajesh  = db.users.findOne({ fullName: "Rajesh" })._id;
  db.chats.insertMany([ { userIds: [sheldon, leonard], chatName:"Shelly,Lenny" },
                        { userIds: [sheldon, rajesh], chatName:"Shelly,Raj" },
                        { userIds: [leonard, rajesh], chatName:"Lenny,Raj" } ]);
  ```

- Find chat by pattern (ending ID) and add name field
  ```js
  const chat118id = db.chats.findOne({ $expr: { $regexMatch: {
    input: { $toString: "$_id" },
    regex: "123$"      
  } } })._id;
  db.chats.updateOne({_id:chat117id},{$set:{chatName:"Shelly,Lenny"}});
  ```

- Remove field from document
  ```js
  db.chats.updateOne({_id:chat117id},{$unset:{chatname:""}});
  db.chats.updateOne({_id:chat117id},{$unset:{chatname:1}});
  ```

- Populate message collection
  ```js
  db.messages.insertMany([
    {"chatId":chat117id,"userId":user113id,"datetime": ISODate("2025-11-29T10:00:00Z"),"text":"Hi there 1"},
    {"chatId":chat117id,"userId":user114id,"datetime": ISODate("2025-11-29T10:05:00Z"),"text":"Hi there 1 reply"},{"chatId":chat118id,"userId":user113id,"datetime": ISODate("2025-11-29T10:10:00Z"),"text":"Hi there 2"},
    {"chatId":chat118id,"userId":user115id,"datetime": ISODate("2025-11-29T10:15:00Z"),"text":"Hi there 2 reply"}]);
  ```

- Remove documents in which there chatId field does not exist / _class field exists / based by field value
  ```js
  db.messages.deleteMany({ chatId: { $exists: false } });
  db.chats.deleteMany({ _class: { $exists: true} });
  db.chats.deleteMany({ chatName: { $nin: ['Sheldon,Howard']} });
  ```

## 12. Sending Ws message

- Selecting MongoDB messages based on Date
  ```js
  db.messages.find( { datetime: { $gt: ISODate("2025-11-29T00:00:00Z") } }, { _id: 0, datetime: 1, text: 1 } )
  ```

- Delete messages based on Date
  ```js
  db.messages.deleteMany({datetime: { $gt: ISODate("2025-11-30T00:00:00Z") } })
  ```

## 13. Introducing Role-Based Access Control (RBAC)

### Before RBAC:

- User logs in → backend returns accessToken + refreshToken
- Frontend uses accessToken for protected API calls
- Backend just checks "is this token valid?"
- Current JWT Structure - JwtBuilder contains 2 fields
  ```java
  public static String generateToken(String userId, String username) {
    return Jwts.builder()
      .setSubject(username)  // Field username
      .claim("userId", userId)  // Field userId
      .setIssuedAt(new Date())
      .setExpiration(new Date(System.currentTimeMillis() + EXPIRATION_TIME_MS))
      .signWith(SECRET_KEY, SignatureAlgorithm.HS256)
      .compact();
  }
  ```
- JwtValidator ( OncePerRequestFilter subclass ) in parsing extracted Autorization: Bearer content needs to read both fields and set them as request attributes
  ```java
  Claims claims = Jwts.parserBuilder()
    .setSigningKey(JwtBuilder.getSecretKey())  
    .build()
    .parseClaimsJws(accessJWT)
    .getBody();
  // extract fields added in JwtBuilder::generateToken
  String username = claims.getSubject();
  String userId = claims.get("userId", String.class);
  // Store them into request attributes (Controller gets them)
  request.setAttribute("username", username);
  request.setAttribute("userId", userId);
  ```
- Controller usage of field passed from OncePerRequestFilter subclass (in protected endpoint only)
  ```java   
  @PostMapping("/chat/new")
  public ResponseEntity<?> createNewChat(@RequestParam("id") String clientId, @RequestBody Map<String, Object> body,
            @RequestAttribute("userId") String userId, @RequestAttribute("username") String username
  ) {
    try {        
      System.out.println("RequestAttribute(userId): " + userId);
      System.out.println("RequestAttribute(username): " + username);
  ```

### After RBAC:

- Everything is the same mechanically
  - still one accessToken + one refreshToken per session
  - refresh token workflow is unchanged
- Only difference: 
  - accessToken now carries extra info (role & claims)  
- Backend checks both:
    - Token is valid
    - Token’s claims allow the action
- Building JWT 
  - read the role & claims from DB when building the token
  - add role and claims to the JWT payload

### Roles and Claims:

- Role: Admin 
  - Claims: sendMessage, createChat, manageUsers
- Role: Prime 
  - Claims: sendMessage, createChat
- Role: Basic 
  - Claims: sendMessage


### MongoDB collections

- Create new collection
  ```js
  db.roles.insertMany( [
    { "role": "Admin", "claims": ["createChat", "sendMessage", "manageUsers"] },
    { "role": "Prime", "claims": ["createChat", "sendMessage"] },
    { "role": "Basic", "claims": ["sendMessage"] } ] )
  ```

- Extend users document with roles field
  ```js
  db.users.updateMany({login:'shelly'},{$set: {role: 'Admin'}})
  db.users.updateMany({login:'lenny'},{$set: {role: 'Prime'}})
  db.users.updateMany({login:'raj'},{$set: {role: 'Basic'}})
  db.users.updateMany({login:'howie'},{$set: {role: 'Basic'}})
  ```

- MongoDB commands
  1. Add roles field as array value
  2. Append new single role
  3. Append new single role if it does not exist (prevent duplicates)
  4. Append many roles (with preventing duplicates)
  5. Remove role from roles array
  6. Remove many array elements
  7. Delete old role field
      ```js
      1. db.users.updateOne({login: 'howie'},{ $set: {roles:['Basic']}})
      2. db.users.updateOne({login: 'howie'},{ $push: {roles: 'Prime'}})
      3. db.users.updateOne({login: 'howie'},{ $addToSet: {roles: 'Admin'}})
      4. db.users.updateOne({login: 'howie'},
        { $addToSet: {roles: { $each: ['Prime','Admin1']}}})
      5. db.users.updateOne({login: 'howie'},{ $pull: {roles: 'Admin1'}})
      6. db.users.updateOne({login: 'howie'},{ $pull: {roles: { $in: ['Prime','Admin']}} })
      7. db.users.updateOne({login: 'howie'},{ $unset: {role:''}})
      ```

### Update User model with roles array field

  ```java
  @Field("roles")
  private List<String> roles;     
  ```

### Java implementation 

- Create Role model and Repository
- **JwtBuilder** - Add Roles and Claims as parameters - called by Authentication service in buildAuthUser()
  ```java
  public static String generateToken(String userId, String username,
    List<String> roles, List<String> claims) { // Added 2 new parameters
    return Jwts.builder()
      .setSubject(username)
      .claim("userId", userId)
      .claim("roles", roles) // Added
      .claim("claims", claims) // Added
      .setIssuedAt(new Date())
      .setExpiration(new Date(System.currentTimeMillis() + EXPIRATION_TIME_MS))
      .signWith(SECRET_KEY, SignatureAlgorithm.HS256)
      .compact();
  }
  ```
- **Authentication service**
  - Inject RoleRepository (@Autowired)
  - Add method Authentication::collectClaims for collecting claims by provided user object
    ```java
    private List<String> collectClaims(User user) {
      // No roles → no claims
      if (user.getRoles() == null || user.getRoles().isEmpty())
        return List.of();
      // Fetch role documents from DB
      var roleDocs = roleRepository.findByRoleIn(user.getRoles());
      // Merge claims from all role documents
      return roleDocs.stream()
        .flatMap(r -> r.getClaims().stream())
        .distinct()
        .toList();
    } 
    ```
  - Update buildAuthUser() to pass roles and claims as arguments to JwtBuilder
    ```java
    List<String> roles = user.getRoles();
    List<String> claims = collectClaims(user);
    String accessToken = JwtBuilder.generateToken(
            user.getId(),
            user.getLogin(),
            roles,
            claims
    );
    ```

- **JwtValidator** -  Update OncePerRequestFilter subclass with roles and claims extract and set
  ```java
  // extract fields added in JwtBuilder::generateToken
  List<String> roles = claims.get("roles", List.class);
  List<String> userClaims = claims.get("claims", List.class);
  // Store them into request attributes
  request.setAttribute("roles", roles);
  request.setAttribute("claims", userClaims);
  ```

- **Controller** - Check in protected endpoint for required claim
  ```java
  @PostMapping("/chat/new")
  public ResponseEntity<?> createChat(
        @RequestAttribute("claims") List<String> claims) {

    if (!claims.contains("createChat")) {
        return ResponseEntity.status(403).body("Forbidden");
    }
  ```

## 14. Adding support for connect frontend in Production

- Enable frontend in CORS policy - update WebMvcConfigurer

  ```java
  @Configuration
  public class CorsConfig {
    @Bean
    public WebMvcConfigurer corsConfigurer() {
      return new WebMvcConfigurer() {
        @Override
        public void addCorsMappings(CorsRegistry registry) {
          registry.addMapping("/**")
                  .allowedOrigins(
                    "http://localhost:5177", // Dev
                    "https://chatjnclient.barryonweb.com" //Prod
                  )
                  .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                  .allowedHeaders("*")
                  .allowCredentials(true);
        }
      };
    }
  } 
  ```

- Add Websocket support in Nginx config
  - For Nginx, the default WebSocket timeout is 60 seconds

  ```nginx
  location /websocket {
    proxy_pass http://127.0.0.1:8081;

    # REQUIRED for WebSockets
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";

    # Pass client info headers
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;  

    # Prevent default 60-second idle disconnect
    proxy_read_timeout 3600;
    proxy_send_timeout 3600;
    proxy_connect_timeout 3600;
  }
  ```

- Allow frontend access to Registry in Websocket Config

  ```java
  @Configuration
  @EnableWebSocket
  public class WebSocketConfig implements WebSocketConfigurer {

    private final WebSocketHandler webSocketHandler;

    public WebSocketConfig(WebSocketHandler webSocketHandler) {
        this.webSocketHandler = webSocketHandler;
    }

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(webSocketHandler, "/websocket")
                .setAllowedOrigins("http://localhost:5177", // Dev
                                  "https://chatjnclient.barryonweb.com" // Prod
                                  ); 
    }
  }
  ```
