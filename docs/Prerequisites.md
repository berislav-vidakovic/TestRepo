## Prerequisites 

### Table of Content 

1. [Create subdomain on VPS (production server)](#1-create-subdomain-on-vps-production-server)
2. [Install Nginx and enable on boot](#2-install-nginx-and-enable-on-boot)
3. [Install Java runtime](#3-install-java-runtime)
4. [Install MongoDB and enable on boot](#4-install-mongodb-and-enable-on-boot)
5. [Establish SSH connections Dev-Github-Prod](#5-establish-ssh-connections-dev-github-prod)
   - [5.1 SSH connection Dev-Github](#51-ssh-connection-dev-github)
   - [5.2 SSH connection Github-Prod](#52-ssh-connection-github-prod)
6. [Install certbot for issuing SSL certificate](#6-install-certbot-for-issuing-ssl-certificate)

### 1. Create subdomain on VPS (production server)

  - Add A record in DNS records on server admin

### 2. Install Nginx and enable on boot 

  ```bash
  nginx -v
  sudo apt install nginx -y
  sudo systemctl status nginx
  sudo systemctl enable nginx
  sudo systemctl is-enabled nginx
  ```


### 3. Install Java runtime

  ```bash
  wget https://github.com/adoptium/temurin21-binaries/releases/download/jdk-21.0.9+10/OpenJDK21U-jdk_x64_linux_hotspot_21.0.9_10.tar.gz
  sudo mkdir -p /usr/lib/jvm
  sudo tar -xzf OpenJDK21U-jdk_x64_linux_hotspot_21.0.9_10.tar.gz -C /usr/lib/jvm
  sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/jdk-21.0.9+10/bin/java 1
  sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/jdk-21.0.9+10/bin/javac 1
  sudo update-alternatives --config java
  sudo update-alternatives --config javac
  java --version
  javac --version
  ```


### 4. Install MongoDB and enable on boot

1. Import MongoDB public GPG key

    ```bash
    curl -fsSL https://pgp.mongodb.com/server-7.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor
    ```

2. Add MongoDB repository

    ```bash
    echo "deb [ signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] \
    https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | \
    sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
    ```

3. Install MongoDB

    ```bash
    sudo apt update
    sudo apt install -y mongodb-org
    ```

4. Add security block to /etc/mongod.conf

    ```yaml
    security:
      authorization: enabled
    ```

5. Set permissions recursively

    ```bash
    sudo chown -R mongodb:mongodb /var/lib/mongodb
    sudo chown -R mongodb:mongodb /var/log/mongodb
    ```

6. Start with no access control 

    ```bash
    sudo mongod --dbpath /var/lib/mongodb --noauth --logpath /var/log/mongodb/noauth.log
    ```

7. Create user dbadmin in DB admin and verify

    ```bash
    mongosh
    use admin
    db.createUser({ user: "dbadmin", pwd: "abc123", roles: [ { role: "root", db: "admin" } ] })
    db.getUsers()
    exit
    ```

8. Stop no-auth mode, restart normally, login to DB admin with user dbadmin

    ```bash
    sudo pkill mongod
    sudo systemctl start mongod
    mongosh -u dbadmin -p --authenticationDatabase admin
    ```

9. Change password

    ```bash
    db.updateUser( "dbadmin", { pwd: "abc123" } )
    ```

10. Make MongoDB available externally /etc/mongod.conf

    ```yaml
    net:
      port: 27017
      bindIp: 0.0.0.0
    ```

    - Download MongoDB Shell from https://www.mongodb.com/try/download/shell
    - Access from PowerShell

      ```powershell
      .\mongosh "mongodb://barry75@barryonweb.com:27017/chatappdb"
      ``` 

11. Run MongoDB manually, check status, enable on boot  

    ```bash
    sudo systemctl start mongod
    sudo systemctl status mongod
    sudo systemctl enable mongod
    ```


### 5. Establish SSH connections Dev-Github-Prod

#### 5.1. SSH connection Dev-Github

- Windows (Dev)
  ```bash
  ssh-keygen -t ed25519 -C "mail@mail.com"
  Get-Service ssh-agent – Check status
  Set-Service -Name ssh-agent -StartupType Automatic
  Start-Service ssh-agent
  ssh-add C:\Users\User\.ssh\id_ed25519
  ```

- Content of pub key copy to GitHub Settings – New SSH key

#### 5.2. SSH connection Github-Prod


  - (Dev)Create and update key pair 

        ssh-keygen -t ed25519 -C "github-ci" -f github_ci
    
    - copy keys to ~/.ssh/
  
  - (Prod)Add the Public key to Linux server

    - append github_ci.pub content to ~/.ssh/authorized_keys on Linux
  
  - Test local to Linux connection: 
  
        ssh -i ~/.ssh/github_ci barry75@barryonweb.com

  - Add the Private Key to GitHub Secrets

    - GitHub: Settings → Secrets and variables → Actions → New repository secret
      - Paste full content of private key github_ci
    - (Optional TODO) Add Known Hosts Fingerprint

  - Test connection

    - Create .github/workflows/test-ssh.yml

      ```yaml
      name: Test SSH Connection

      on:
        workflow_dispatch:  # Allows trigger manually in GitHub

      jobs:
        test-ssh:
          runs-on: ubuntu-latest

          steps:
            - name: Checkout repository
              uses: actions/checkout@v4

            - name: Start SSH agent and load key
              uses: webfactory/ssh-agent@v0.9.0
              with:
                ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

            - name: Test SSH connection
              run: ssh -o StrictHostKeyChecking=no barry75@barryonweb.com "echo Connected successfully from GitHub!"
      ```

    - GitHub Actions - Run workflow




### 6. Install certbot for issuing SSL certificate

  ```bash
  sudo apt update
  sudo apt install certbot python3-certbot-nginx
  which certbot
  certbot --version
  ```
