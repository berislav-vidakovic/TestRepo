## Frontend Deployment

1. [Create minimal Nginx config file](#1-create-minimal-nginx-config-file)  
2. [Issue SSL certificate](#2-issue-ssl-certificate)  
3. [Initialize Git and make first commit](#3-initialize-git-and-make-first-commit)  
4. [Add CI/CD yaml](#4-add-cicd-yaml)
5. [Troubleshooting](#5-troubleshooting)  
6. [Deployment topology](#6-deployment-topology)

### 1. Create minimal Nginx config file
  ```nginx
  server {
    listen 80;
    server_name gamesjclient.barryonweb.com;

    root /var/www/games/frontend/panel;
    index index.html;

    location / {
      try_files $uri /index.html;
    }
  }
  ```

### 2. Issue SSL certificate

- Output: Certbot will update Nginx config file
  ```nginx
  server {
    server_name gamesjclient.barryonweb.com;

    root /var/www/games/frontend/panel;
    index index.html;

    location / {
      try_files $uri /index.html;
    }

    listen 443 ssl; # managed by Certbot
      ssl_certificate /etc/letsencrypt/live/gamesjclient.barryonweb.com/fullchain.pem; # managed by Certbot
      ssl_certificate_key /etc/letsencrypt/live/gamesjclient.barryonweb.com/privkey.pem; # managed by Certbot
      include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
      ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
  }

  server {
      if ($host = gamesjclient.barryonweb.com) {
          return 301 https://$host$request_uri;
      } # managed by Certbot

    listen 80;
    server_name gamesjclient.barryonweb.com;
      return 404; # managed by Certbot
  }
  ```

- Copy nginx file to server manually
  ```bash
  scp gamesjclient.barryonweb.com barry75@barryonweb.com:/var/www/games/nginx/
  sudo cp /var/www/games/nginx/gamesjclient.barryonweb.com /etc/nginx/sites-available/ 
  ```

- Enable the site, check sites enabled, check syntax and restart
  ```bash
  sudo ln -sf /etc/nginx/sites-available/gamesjclient.barryonweb.com /etc/nginx/sites-enabled/
  ls -l /etc/nginx/sites-enabled/
  sudo nginx -t 
  sudo systemctl reload nginx
  ```

- Issue SSL certificate for the subdomain
  ```bash
  sudo certbot --nginx -d gamesjclient.barryonweb.com
  ```

- Check SSL certificate installed
  ```bash
  sudo ls -l /etc/letsencrypt/live/gamesjclient.barryonweb.com
  ```

- Copy updated file back to local Repo
  ```bash
  scp barry75@barryonweb.com:/etc/nginx/sites-available/gamesjclient.barryonweb.com ./
  ```


### 3. Initialize Git and make first commit 

1. Create Github Repo

2. <a href="docs/Git.md">Create remote repo, init, commit and  push
</a>

3. SSH connection Dev to Remote Repo

  - Test connection Dev-VPS
    ```bash
    ssh -i ~/.ssh/github_ci barry75@barryonweb.com
    ```

  - Establish connection Github-VPS (Repository-specific)
    - Add the Private Key ~/.ssh/github_ci to GitHub Secrets
      - GitHub: Settings → Secrets and variables → Actions → New repository secret
      - Create secret key: 
        - Name: SSH_PRIVATE_KEY
        - Content: Paste full content of private key github_ci


4. Compile Typescript

  ```ts
  npm run build
  ```



### 4. Add CI/CD yaml
  ```yaml
  name: Deploy TypeScript Frontend

  on:
    push:
      branches:
        - main
    workflow_dispatch:

  jobs:
    frontend-build-and-deploy:
      runs-on: ubuntu-latest

      steps:
        # Checkout code
        - name: Checkout repository
          uses: actions/checkout@v4

        # Setup Node.js environment
        - name: Setup Node.js
          uses: actions/setup-node@v3
          with:
            node-version: 20

        # GitHub - Install frontend dependencies and build
        - name: Build frontend
          run: |
            npm install  
            cd panel
            npm install
            npm run build
            cd ../sudoku
            npm install
            npm run build
            cd ../connect4
            npm install
            npm run build

        # Start SSH agent with GitHub secret key
        - name: Setup SSH
          uses: webfactory/ssh-agent@v0.9.0
          with:
            ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

        # Add server to known_hosts to avoid verification errors
        - name: Add server to known_hosts
          run: |
            mkdir -p ~/.ssh
            ssh-keyscan barryonweb.com >> ~/.ssh/known_hosts
        
        # Nginx config - transfer, enable, check syntax and restart Nginx
        - name: Update Nginx config
          run: |
            ssh barry75@barryonweb.com "mkdir -p /var/www/games/nginx"
            scp gamesjclient.barryonweb.com barry75@barryonweb.com:/var/www/games/nginx/
            ssh barry75@barryonweb.com "
              sudo cp /var/www/games/nginx/gamesjclient.barryonweb.com /etc/nginx/sites-available/ &&
              sudo ln -sf /etc/nginx/sites-available/gamesjclient.barryonweb.com /etc/nginx/sites-enabled/ &&
              sudo nginx -t &&
              sudo systemctl reload nginx"
        

        # Transfer frontend from GitHub Repo to Ubuntu server via SCP
        - name: Deploy frontend via SSH to server
          run: |
            ssh barry75@barryonweb.com "mkdir -p /var/www/games/frontend/panel"
            scp -r panel/dist/* barry75@barryonweb.com:/var/www/games/frontend/panel/
            ssh barry75@barryonweb.com "mkdir -p /var/www/games/frontend/sudoku"
            scp -r sudoku/dist/* barry75@barryonweb.com:/var/www/games/frontend/sudoku/
            ssh barry75@barryonweb.com "mkdir -p /var/www/games/frontend/connect4"
            scp -r connect4/dist/* barry75@barryonweb.com:/var/www/games/frontend/connect4/
  ```

- Change  SSH port from default 22 to 2222
  - Check configured listening Port
    ```bash
    sudo grep -R "^Port" /etc/ssh
    [sudo] password for barry75:
    /etc/ssh/sshd_config:Port 2222
    ```

  - Check what port is actually listening
    ```bash
    sudo ss -tlnp | grep ssh
    LISTEN 0      4096         0.0.0.0:22         0.0.0.0:*    users:(("sshd",pid=777577,fd=3),("systemd",pid=1,fd=125))                                                         
    LISTEN 0      4096            [::]:22            [::]:*    users:(("sshd",pid=777577,fd=4),("systemd",pid=1,fd=129))   
    ```

  - Disable ssh.socket (overrides Port configured) and verify status
    ```bash
    sudo systemctl disable --now ssh.socket
    Removed "/etc/systemd/system/sockets.target.wants/ssh.socket".
    Removed "/etc/systemd/system/ssh.service.requires/ssh.socket".
    sudo systemctl status ssh.socket
    ```

  - Restart SSH service and check actual port listening
    ```bash
    sudo systemctl restart ssh
    barry75@vps1:~$ sudo ss -tlnp | grep ssh
    LISTEN 0      128          0.0.0.0:2222       0.0.0.0:*    users:(("sshd",pid=777974,fd=3))                                                                                  
    LISTEN 0      128             [::]:2222          [::]:*    users:(("sshd",pid=777974,fd=4))     
    ```
  
  - Connect to server with new Port
    ```bash
    ssh -p 2222 barry75@barryonweb.com
    ```
  
  - Update yaml with new port
    ```yaml
    ssh-keyscan -p 2222
    ssh -p 2222
    scp -P 2222
    rsync -az -e "ssh -p 2222"
    ```
  
  - Disable password authentication and root login
    ```bash
    sudo nano /etc/ssh/sshd_config
    PasswordAuthentication no
    PubkeyAuthentication yes
    PermitRootLogin no
    sudo systemctl restart ssh
    ```

### 5. Troubleshooting 

1. Check Nginx health & config
    ```bash
    sudo systemctl status nginx --no-pager
    sudo nginx -t
    ```

2. Inspect Nginx error log
    ```bash
    sudo tail -n 200 /var/log/nginx/error.log
    sudo tail -f /var/log/nginx/error.log # follow live
    ```

3. Root folder in Nginx config must contain index.html

4. Error
    ```bash
    Failed to load module script: Expected a JavaScript-or-Wasm module script but the server responded with a MIME type of "text/html". Strict MIME type checking is enforced for module scripts per HTML spec.
    ```
    - Add trailing slash in Nginx config

5. Error
    ```bash
    gamesjclient.barryonweb.com/:1  GET https://gamesjclient.barryonweb.com/ 403 (Forbidden)
    ```
    - Redirect root to /panel in Nginx config
      ```nginx
      location = / {
        return 302 /panel/;
      }
      ```

6. Websocket Error
    ```bash
    WebSocket closed by server: 
    ```

    - Backend WebSocket Config - add new origin 
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
                      .setAllowedOrigins("http://localhost:5174",
                        "http://localhost:5176",
                        "https://gamesjclient.barryonweb.com" ); //frontend
          }
      }
      ```

### 6. Deployment topology

On the frontend, ES modules already give singleton semantics, so a function-based facade is used instead of a class to keep it idiomatic and lightweight.

This is common codebase, with runtime configuration-based switch to one of the following backends
- REST API with MySQL
- GraphQL API with Hasura and PostgreSQL


### 7. GraphQL support

 - Test endpoint ping
    ```ts
    async function testGraphQLendpoint(){
      const url = "http://localhost:8083/graphql";
      const body = JSON.stringify({ query: "{ pingDb }"});
      const res = await fetch( url, {
        method: "POST",
        headers: {
          "Content-Type": "application/json"
        },
        body
      });  
      const json = await res.json();
      console.log( "GraphQL response: ", json);
    }
    ```

 - Get all users
    ```ts
    const body = JSON.stringify({ 
        query: `{ UsersAll { id, techstack, users { userId, login, fullName, isOnline } } }` });

    const res = await fetch( url, {
      method: "POST",
      headers: {
        "Content-Type": "application/json"
      },
      body
    });  
    const json = await res.json();
    console.log( "GraphQL getAllUsers response: ", json);  
    ```

- Send mutation 
  ```ts
  export async function mutationRegisterUser(input: RegisterUserInput): Promise<RegisterUserResponse> {
  const body = JSON.stringify({
    query: `
      mutation ...
  ```
