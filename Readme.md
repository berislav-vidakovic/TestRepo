## Java backend with GraphQL and PostgreSQL


<div style="margin-bottom: 12px;">
<img src = "docs/java.png" style="height:35px; margin-right: 15px;" /> 
<img src = "docs/spring.png" style="height:35px; margin-right: 15px;" /> 
<img src = "docs/jpa.svg" style="height:35px; margin-right: 15px;" /> 
<img src = "docs/postgresqlb.png" style="height:35px; margin-right: 15px;" /> 
<img src = "docs/graphql.png" style="height:35px; margin-right: 15px;" /> 
</div>


This project uses Spring Boot GraphQL.
A Hasura-based implementation is planned to compare backend-managed vs database-driven GraphQL architectures.



### Table of Content

1. [Build project skeleton](#1-build-project-skeleton)
2. [Install PostgreSQL, create DB and connect backend](#2-install-postgresql-create-db-and-connect-backend)
3. [Run PostgreSQL scripts to create and populate table](#3-run-postgresql-scripts-to-create-and-populate-table)
4. [GraphQL healthcheck queries](#4-graphql-healthckeck-queries)
5. [GraphQL fetch all users query](#5-graphql-fetch-all-users-query)



### 1. Build project skeleton

1. Generate Spring Boot Project on  https://start.spring.io

    - Fill

      - Project	Maven
      - Language	Java
      - Spring Boot - latest stable (no RC2, no SNAPSHOT)
      - Group	gamesjg
      - Artifact	gamesjg
      - Name	gamesjg
      - Packaging	Jar
      - Java	21

    - Add Dependencies:

      - Spring Web
      - Spring for GraphQL
      - Spring Data JPA
      - PostgreSQL Driver
      - Validation
      - Spring Security
    
    - Download and Extract
  
2. Define Port (default is 8080) in application.yaml (root key)

      ```yaml
      server:
        port: 8083
      ```

3. Build

        mvn clean -Dskiptest    

4. Git init, commit, push

    - Create Repo on GitHub
    - Run
      ```bash
      git init
      git add .
      git commit -m "Initial commit"
      ```
    - Get Remote Repo SSH link, run and verify
      ```bash
      git remote add origin git@github.com:berislav-vidakovic/ChatAppJn.git
      git remote -v
      ```
    - Push code (--set-upstream the same -u flag)
      ```bash
      git push -u origin main
      ```


### 2. Install PostgreSQL, create DB and connect backend

- Install and check status
  ```bash
  sudo apt update
  sudo apt install postgresql postgresql-contrib
  sudo systemctl status postgresql
  ```

- Enable external access, restart, verifiy is running
  ```bash
  sudo nano /etc/postgresql/16/main/postgresql.conf
  listen_addresses = '*' # replace commented line #listen_addresses = 'localhost'   
  sudo nano /etc/postgresql/16/main/pg_hba.conf  # add to end
  host    gamesdb         barry75         0.0.0.0/0               scram-sha-256
  sudo systemctl restart postgresql
  sudo systemctl status postgresql
  ```

- Check all PostgreSQL services and status:
  ```bash
  systemctl list-units | grep postgresql
  systemctl status postgresql@16-main # main - has to be running
  systemctl status postgresql
  ```

- Verify listening ports:
  ```bash
  sudo ss -tulnp | grep 5432
  ```

- Check logs for failed service
  ```bash
  sudo journalctl -u postgresql@16-main.service -b
  ```

- Check Postgres binaries postgres and psql location, add to PATH and apply
  ```bash
  ls /usr/lib/postgresql/16/bin
  nano ~/.bashrc  # Add line to the end
  export PATH=/usr/lib/postgresql/16/bin:$PATH
  source ~/.bashrc # apply immediately
  ```

- Add to secure path
  ```bash
  sudo visudo
  Defaults        secure_path="/usr/lib/postgresql/16/bin:
  ```

- Check config file syntax:
  ```bash
  sudo -u postgres pg_ctl -D /etc/postgresql/16/main -t 1 start
  ```

- Connect to PostgreSQL as super user, create database, create user and grant access 
  ```postgres
  sudo -u postgres psql
  CREATE DATABASE gamesdb;
  CREATE USER barry75 WITH PASSWORD 'strongPwd';
  GRANT ALL PRIVILEGES ON DATABASE gamesdb TO barry75;
  ```

- Test external access from local bash
  ```bash
  psql -h barryonweb.com -p 5432 -U barry75 -d gamesdb
  ```

- Add connection to backend application.yaml (under spring key)
  ```yaml
  datasource:
    url: jdbc:postgresql://barryonweb.com:5432/gamesdb
    username: barry75
    password: strongPwd!
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
  ```

- Run backend

        mvn spring-boot:run

- For Connection OK there is JPA logs
  ```bash
  Initialized JPA EntityManagerFactory for persistence unit 'default'
  ```

### 3. Run PostgreSQL scripts to create and populate table

- Version, verify current user and DB
  ```bash
  psql --version
  ls /etc/postgresql/
  psql -U barry75 -d gamesdb
  SELECT current_user;
  SELECT current_database();
  ```

- PostgreSQL Internal and external connection (postgres is the default maintenance DB that always exists)
  ```bash
  psql -U barry75 -d gamesdb
  psql -h barryonweb.com -p 5432 -U barry75 -d gamesdb
  psql -h barryonweb.com -p 5432 -U barry75 -d postgres
  ```

- Show the owner and privileges of the public schema
  ```postgres
  \dn+ public
  ```

- Connect as superuser to Postgres on VPS and grant priviledges to create table
  ```bash
  sudo -u postgres psql -d gamesdb
  GRANT ALL PRIVILEGES ON SCHEMA public TO barry75;
  ```

- Run scripts (relative path) 
  ```postgres
  \i schema.sql
  \i data.sql
  ```

- Use DB, list DBs, list users, list tables, table structure
  ```postgres
  \c gamesdb
  \l
  \du
  \dt
  \d healthcheck
  ```

- PostgreSQL Prompt for new command and to continue previous command: 
  ```postgres
  gamesdb=> 
  gamesdb->
  ```

### 4. GraphQL healthckeck queries 

- Create GraphQL schema in resources/graphql directory:
  ```graphql
  type Query {
    ping: String
  }
  ```

- Create Controller in Controller directory
  ```java
  package gamesjg.Controllers;
  import org.springframework.graphql.data.method.annotation.QueryMapping;
  import org.springframework.stereotype.Controller;
  @Controller
  public class PingControllerGraphQL {
    // Method name ping() corresponds to the Query field in schema.graphqls
    @QueryMapping
    public String ping() {
      return "pong";
    }
  }
  ```

- Add minimal Config to Config directory
  ```java
  package gamesjg.Config;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.security.config.annotation.web.builders.HttpSecurity;
  import org.springframework.security.web.SecurityFilterChain;
  @Configuration
  public class SecurityConfig {
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
      http
        .csrf(csrf -> csrf.disable())
        .authorizeHttpRequests(auth -> auth.anyRequest().permitAll() )  // allow all requests
        .httpBasic(httpBasic -> httpBasic.disable())   // disable HTTP Basic
        .formLogin(formLogin -> formLogin.disable()); // disable login form
      return http.build();
    }
  }
  ```

- Test from Postman as POST with JSON body:
  ```json
  {
    "query": "{ ping }"
  }
  ```
  - Response expected:
    ```json
    {
      "data": {
        "ping": "pong"
      }
    }
    ```

- Add pingDb query
  - Extend GraphQL schema definition
  - Add Model and Repository
  - Add Controller
  - Test from Postman as POST with JSON body:
    ```json
    {
      "query": "{ pingDb }"
    }
    ```
    - Response expected 
      - Row exists:
        ```json
        {
          "data": {
            "pingDb": "Hello world from PostgreSQL!"
          }
        }
        ```
      - No row exists:
        ```json
        {
          "data": {
            "pingDb": null
          }
        }
        ```
      - Database Error:
        ```json
        {
          "errors": [
            {
              "message": "Database connection failed",
              "locations": [...],
              "path": ["pingDb"]
            }
          ]
        }
        ```


### 5. GraphQL fetch all users query

- Extend schema.graphqls
  ```graphql
  type Query {
    ...
    usersOverview: UsersOverview
  }

  type UsersOverview {
    id: ID!
    users: [User!]!
    techstack: [String!]!
  }

  type User {
    userId: ID!
    login: String!
    fullName: String
    isOnline: Boolean!
  }
  ```

- Create UsersOverview DTO

- Create Controller 
