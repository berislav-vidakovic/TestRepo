## MongoDB

<img src = "../src/main/resources/static/images/mongodb.png" style="height:35px; margin-right: 15px;" /> 


### 1. Create DB,  create user within, Show DBs

  ```bash
  use chatappdb
  db.createUser({ user: "barry75", pwd: "abc123", roles: [ { role: "readWrite", db: "chatappdb" } ] })
  db.updateUser( "barry75", { roles: [ { role: "dbOwner", db: "chatappdb" } ] }  )
  show dbs
  ```

### 2. Login to DB with user defined within

  ```bash
  mongosh -u barry75 -p --authenticationDatabase chatappdb
  ```

### 3. Insert document into test collection

  ```bash
  db.test.insertOne({ pingdb: 'Hello world from MongoDB' })
  ```


### 4. Show all collections 

  ```bash
  show collections
  db.getCollectionNames()
  ```

### 5. Fetch content of Collection

  ```bash
  db.test.find()
  ```

### 6. Fetch document with particular field

  ```bash
  db.test.find({ pingdb: { $exists: true } })
  ```

### 7. Fetch only particular document, no id and print value

  ```bash
  db.test.find({ pingdb: { $exists: true } }, { pingdb: 1, _id: 0 } )
  let doc =db.test.findOne( { pingdb: { $exists: true } }, { pingdb: 1, _id: 0 } ); 
  print(doc.pingdb)
  ```

