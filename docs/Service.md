### Backend chatappjn.service file

  ```bash
  [Unit]
  Description=ChatAppJn Spring Boot Backend
  After=network.target

  [Service]
  WorkingDirectory=/var/www/chatappjn
  ExecStart=/usr/bin/java -jar /var/www/chatappjn/chatappjn-0.0.1-SNAPSHOT.jar
  SuccessExitStatus=143
  Restart=always
  RestartSec=10
  User=barry75
  Environment=JAVA_OPTS="-Xms256m -Xmx512m"
  Environment=SPRING_PROFILES_ACTIVE=prod

  [Install]
  WantedBy=multi-user.target
  ```