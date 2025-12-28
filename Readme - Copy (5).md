## Setup

  - [Common features](#common-features) 
  - [Backend model DB connect and GET enpoint](#backend-model-db-connect-and-get-enpoint) 
  - [Frontent config and send GET request](#frontent-config-and-send-get-request) 
  - [Deployment](#deployment)




### Common features

#### Current dev configuration

- Frontend panel Port: 5174
- Frontend sudoku Port: 5175
- Backend Port: 5003 

#### 1. Place clientsettings.json in root frontend/public
#### 2. Update each game project's **vite.config.ts**:

  - Enable access to common and root public
  - Set local dev port explicitly 
  - Set base for production

      ```ts
      import path from 'path';
      export default defineConfig({
        publicDir: path.resolve(__dirname, '../public'),
        resolve: {
          alias: {
            '@common': path.resolve(__dirname, '../common'),     
          },
        },
        server: { port: 5174 },
        base: '/sudoku/'
      });
      ```
#### 3. Add common path to **tsconfig.json**:


  ```json
  "paths": {
    "@common/*": ["../common/*"]
  }
  ```
#### 4. Use alias when importing common component

  ```ts
  import { sendGETRequest } from '@common/restAPI';
  import { loadCommonConfig } from '@common/config';
  ```

#### 5. Adding new frontend project

  - Run from terminal:
    ```powershell
    cd frontend
    npm create vite@latest panel -- --template react-ts
    ```
  - Add workspace to frontend/package.json:
    ```json
    {
      "private": true,
      "workspaces": [
        "common",
        "sudoku",
        "panel"
      ],
      "dependencies": {
        "http-status-codes": "^2.3.0"
      }
    }
    ```
  - Add to App.tsx:
    ```ts
    const [isConfigLoaded, setConfigLoaded] = useState<boolean>(false);
    useEffect( () => { 
      loadCommonConfig(setConfigLoaded);     
    }, []);
    ```
  
  - Update project's **vite.config.ts**
  - Add common path to **tsconfig.json** 
  - Update build and deploy workflow in **deploy-frontend.yml**
  - Update Nginx config file **games**
  - Update .gitignore
  - Add CORS policy entry to backend

  

### Backend model DB connect and GET enpoint 

1. Create Model that matches table and columns, [Key] on PK column
2. Create DbContext-based class
    - Add DbSet member
    - Connect to DB table in OnModelCreating
3. Add ConnectionString to appsettings.json
4. Create Controller with 1 endpoint
5. Add Services (Controllers and DbContext) to container and map Controllers

Sudoku:
  - Controller class: SudokuController
  - GET endpoint "board"
  - URL for browser check: http://localhost:5003/api/sudoku/board

### Frontent config and send GET request

1. Add .env and .env.production to root folder frontend
2. Update backend CORS policy with frontend Port defined in vite.config.ts  
2. Add env.d.ts common folder
2. Add config.ts common folder
1. Add restAPI.ts to common folder
2. Add states, Load config, send GET request and handle response in App.tsx

    ```ts
    function App() {
    const [isConfigLoaded, setConfigLoaded] = useState<boolean>(false);
    const [areBoardsLoaded, setBoardsLoaded] = useState<boolean>(false);
    const [board, setBoard] = useState<string>("");
    const [solution, setSolution] = useState<string>("");
    
    useEffect( () => { 
      loadCommonConfig(setConfigLoaded);     
    }, []);

    useEffect( () => { if( isConfigLoaded){
        sendGETRequest('api/sudoku/board', handleInit );
    }      
    }, [isConfigLoaded]);

    const handleInit = ( jsonResp: any ) => {    
      //console.log("Response to GET : ", jsonResp );
      setBoard(jsonResp.boards[0].board);
      setSolution(jsonResp.boards[0].solution);
      setBoardsLoaded(true);
    }
    ```

### Deployment

Nging Linux on barryonweb.com

#### 1. Access Linux server using SSH from Powershell
  
  ```powershell
  ssh barry75@barryonweb.com
  ```

#### 2. Restart Linux if needed

  ```bash
  sudo reboot
  apt list --upgradable
  sudo apt update
  sudo apt upgrade -y
  ```

#### 3. Install Nginx
    
  ```bash
  nginx -v
  sudo apt install nginx -y
  sudo systemctl status nginx
  ```

Nginx welcome Page available on browse domain

#### 4. Install and/or activate firewall

  ```bash
  sudo ufw status
  ```


#### 5. Install .Net runtime

  - Ubuntu does not include the latest .NET runtime in its repositories. Microsoft maintains its own repository with up-to-date .NET versions. To install .NET, we need to tell Ubuntu about this repository.
    - wget downloads a file from the internet.
    - The URL points to Microsoft’s package configuration file for particular Ubuntu version
      - $(lsb_release -rs) automatically detects Ubuntu version (e.g., 22.04) and downloads the correct file.
    - O packages-microsoft-prod.deb saves the downloaded file with that name.

    ```bash
    wget https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
    ```
  
  - dpkg -i installs the downloaded package, that adds Microsoft’s repository to the system, so **apt can now fetch .NET packages**
    ```bash
    sudo dpkg -i packages-microsoft-prod.deb
    ```
  
  - Delete the downloaded .deb file 
    ```bash
    rm packages-microsoft-prod.deb
    ```

  - After this, the system knows where to get Microsoft packages to install ASP.NET Runtime and verify 
    ```bash
    sudo apt update
    sudo apt install aspnetcore-runtime-8.0
    dotnet --info
    ```

#### 6. Install MySQL

  - Install, verify and login as root
    ```bash
    sudo apt install mysql-server -y
    sudo mysql_secure_installation
    sudo systemctl status mysql
    sudo mysql -u root -p
    ```

#### 8. Install node and npm

  ```bash
  curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
  sudo apt install -y nodejs build-essential
  node -v
  npm -v
  ```

#### 7. Frontend deployment

- Build each frontend

  ```powershell
  npm install
  npm run build
  ```

- Create server destination folders and set permissions

  ```bash
  sudo mkdir -p /var/www/games/frontend/sudoku
  sudo mkdir -p /var/www/games/frontend/panel
  sudo mkdir -p /var/www/games/backend
  sudo chown -R www-data:www-data games
  sudo chmod -R 755 games
  sudo usermod -aG www-data barry75
  groups barry75
  sudo chmod -R 775 /var/www/games
  ```

- Copy dist source folders to destination folders 

  ```powershell
  scp -r .\panel\dist\* barry75@barryonweb.com:/var/www/games/frontend/panel/
  ```

#### 8. Nginx configuration

Create /etc/nginx/sites-available/games
 
1-Enable the site
sudo ln -s /etc/nginx/sites-available/games /etc/nginx/sites-enabled/


2-Test config
sudo nginx -t


3-Reload Nginx
sudo systemctl reload nginx

Disable Nginx deault site
sudo rm /etc/nginx/sites-enabled/default


Check site is properly linked
ls -l /etc/nginx/sites-enabled/

Test Frontend: 

  Browse: http://barryonweb.com/sudoku/


#### 9. Backend deployment

dotnet publish -c Release -r linux-x64 --self-contained false -o ./publish

copy files

MySQL
- current user: SELECT CURRENT_USER(), USER();
- all users: SELECT user, host, plugin FROM mysql.user;
- As root
  - Create user barry75 
  - Create DB
  - Grant access to barry75

  ```sql
  CREATE DATABASE db_games CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
  CREATE USER 'barry75'@'localhost' IDENTIFIED BY 'abc123';
  GRANT ALL PRIVILEGES ON db_games.* TO 'barry75'@'localhost';
  FLUSH PRIVILEGES;
  ```


- Login as barry75
  - see dbs: SHOW DATABASES;
  - Run script

- Test backend

  -Locally

      curl http://localhost:5000/api/sudoku/board
      curl -v http://127.0.0.1:5001/api/users/all


  -Remotely

      Browse http://barryonweb.com/api/sudoku/board
      curl -v https://games.barryonweb.com/api/users/all



## CI/CD pipelines

### Github workflow for frontend yml

  - GitHub creates a fresh, isolated virtual machine (Ubuntu runner). It starts empty — the repo is cloned, but nothing else exists

  - Inside that environment, npm install creates a node_modules folder which lives inside the ephemeral runner’s filesystem

  - After npm run build, the dist folder is generated

  - The next step (scp) transfers only the dist folder to the server

  - After the workflow completes VM is destroyed and he temporary node_modules folder is deleted automatically when the runner finishes

### Move the build+deploy script from local env into a pipeline 

  - runs on GitHub whenever code pushed
  - local env and human launch is no longer required to build or copy files to the server


### 1. Create deployment ps1 script to

  - build
  - prepare/copy artifacts
  - transfer to server


### 2. SSH connection GitHub - Linux server

  - Create and update key pair 

        ssh-keygen -t ed25519 -C "github-ci" -f github_ci
    
    - copy keys to ~/.ssh/
  
  - Add the Public key to Linux server

    - append github_ci.pub content to ~/.ssh/authorized_keys on Linux
  
  - Test local to Linux connection: 
  
        ssh -i ~/.ssh/github_ci barry75@barryonweb.com

  - Add the Private Key to GitHub Secrets

    - GitHub: Settings → Secrets and variables → Actions → New repository secret
      - Paste full conetnt of private key github_ci
    - (Optional TODO) Add Known Hosts Fingerprint

  - Test connection

    - Create .github/workflows/test-ssh.yml
    - GitHub Actions - Run workflow

  - Deploy frontend manually or automatically upon commit

    - Create .github/workflows/deploy-frontend.yml

### 3. Backend GitHub workflow

1. Build backend on GitHub (dotnet publish)

2. Transfer new backend files via scp

3. Transfer nginx config file games to /var/www/games/nginx/

4. Copy nginx config file from /var/www/games/nginx/ to /etc/nginx/sites-available/games

5. (Re)create the symlink in /etc/nginx/sites-enabled/

6. Reload Nginx to apply changes 
  
7. Restart backend 
  

#### Create .github/workflows/deploy-backend.yml
  - Restart by creating service
    - Create service file sudo nano /etc/systemd/system/games-backend.service

      ```bash
      [Unit]
      Description=Games Backend
      After=network.target

      [Service]
      WorkingDirectory=/var/www/games/backend
      ExecStart=/usr/bin/dotnet /var/www/games/backend/backend.dll
      Restart=always
      RestartSec=5
      User=barry75
      Environment=ASPNETCORE_ENVIRONMENT=Production

      [Install]
      WantedBy=multi-user.target
      ```

    - Reload systemd

          sudo systemctl daemon-reload
          sudo systemctl enable games-backend
          sudo systemctl start games-backend
      This ensures the service is registered, starts on boot, and can be restarted via systemctl.
  - Enable no password to restart service

        sudo visudo 
        barry75 ALL=(ALL) NOPASSWD: /bin/systemctl restart games-backend
        barry75 ALL=(ALL) NOPASSWD: /bin/systemctl reload nginx
        barry75 ALL=(ALL) NOPASSWD: /bin/cp, /bin/ln, /usr/sbin/nginx

  - Follow logs in realtime

        sudo journalctl -u games-backend -f

## Localization

### Powershell
    
  ```powershell
  [Console]::OutputEncoding = [System.Text.Encoding]::UTF8
  chcp 65001
  mysql --default-character-set=utf8mb4 -u barry75 -p
  ```

### MySQL

  ```sql
  SHOW VARIABLES LIKE 'char%';  
  SHOW VARIABLES LIKE 'collation%';
  SHOW CREATE TABLE localization;  
  SET NAMES utf8mb4;   
  ```

### ASP.Net

- appsettings.json

  ```json
  CharSet=utf8mb4
  ```

### React

- index.html

  ```html
  <meta charset="UTF-8" />
  ```



  


















