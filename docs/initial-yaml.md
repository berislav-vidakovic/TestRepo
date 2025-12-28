### Initial YAML file for CI/CD pipeline deployment

```yaml
name: Deploy Java Backend ChatAppJn

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  backend-build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      # Checkout repository
      - name: Checkout repository
        uses: actions/checkout@v4
        
      # Setup Java and Maven
      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 21
          cache: maven

      # Build backend
      - name: Build backend
        run: |
          mvn clean package -DskipTests

      # Setup SSH
      - name: Setup SSH
        uses: webfactory/ssh-agent@v0.9.0
        with:
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
      
      # Retrieve the fingerprint and store it in the GitHub Actions VM's known_hosts file.
      - name: Add server to known_hosts
        run: |
          mkdir -p ~/.ssh
          ssh-keyscan barryonweb.com >> ~/.ssh/known_hosts

      # Copy published files to server
      - name: Deploy backend files 
        run: |
          ssh barry75@barryonweb.com "mkdir -p /var/www/chatapp/backend/chatjn/"
          scp target/chatappjn-0.0.1-SNAPSHOT.jar barry75@barryonweb.com:/var/www/chatapp/backend/chatjn/

      # Deploy application.yaml config
      - name: Deploy Spring config file
        run: |
          ssh barry75@barryonweb.com "mkdir -p /var/www/chatapp/backend/chatjn/config/"
          scp src/main/resources/application.yaml barry75@barryonweb.com:/var/www/chatapp/backend/chatjn/config/
      
      # Deploy Nginx config and reload
      - name: Update Nginx config
        run: |
          ssh barry75@barryonweb.com "mkdir -p /var/www/chatapp/nginx/"
          scp chatjn.barryonweb.com barry75@barryonweb.com:/var/www/chatapp/nginx/
          ssh barry75@barryonweb.com "
            sudo cp /var/www/chatapp/nginx/chatjn.barryonweb.com /etc/nginx/sites-available/ &&
            sudo ln -sf /etc/nginx/sites-available/chatjn.barryonweb.com /etc/nginx/sites-enabled/ &&
            sudo nginx -t && sudo systemctl reload nginx"

      # Restart backend via systemd
      - name: Restart backend service
        run: |
          ssh barry75@barryonweb.com "sudo systemctl restart chatappjn.service"

      # See logs after restart
      - name: Show backend logs
        run: |
          ssh barry75@barryonweb.com "journalctl -u chatappjn.service -n 50 --no-pager"
```
  