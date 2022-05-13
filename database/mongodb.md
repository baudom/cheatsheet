# mongodb

**Foreword**: [MongoDB][mongodb] is an NoSQL (Not-Only-SQL) Database, which stores its content in Collections in a JSON-like format. The following Documentation is performed with MongoDB Community Server.

# Table of Content

-   [Installation (Local Server)](#installation-local-server)
-   [Installation (Docker)](#installation-docker)
-   [MongoDB Shell (mongosh)](#mongodb-shell-mongosh)
-   [Authorisation (SCRAM)](#authorisation-scram)

# Installation (Local Server)

## Windows

-   Head to [Community Downloads][mongodbcommunity], choose the latest version for windows and your preferred package (.msi recommended)
-   Follow the installation instructions and you're ready to use mongoDB on your local machine

## Linux

**WIP**

# Installation (Docker)

**Disclaimer**: A running instance of docker is required.

## 1 Pull latest stable docker image

```shell
docker pull mongo
```

## 2 Create volume

**Note**: This step is required for Windows, optional for Linux and macOS

```shell
docker volume create <volume>
```

-   `<volume>`: your volume identifier

## 3 Create and run container

```shell
docker run -it -v <volume>:/etc/mongodb -p 27017:27017 --name <name> -d mongo
```

-   `-it`: run the installation interactive
-   `-v`: used volume and mounting path
-   `-p`: inner port : outer port (outer port can be chosen as you want)
-   `--name`: alias of your container, e.g. "mongodb"
-   `-d`: used image

# MongoDB Shell (mongosh)

## Windows

-   Download [MongoDB Shell][mongosh] (known as mongosh)
-   Unzip or install the downloaded .zip or .msi (.msi recommended)

## Linux

**WIP**

# Authorisation (SCRAM)

**Disclaimer**: Authorisation was set up with mongosh@1.4.1 and [this Tutorial][tutorial] published from MongoDB. A running instance of mongodb server without any authorisation is also required.

## 1 Setup authorisation and create admin user

### 1.1 Connect to MongoDB

```shell
mongosh --port <port>
```

-   `<port>`: Port of MongoDB instance (default: 27017)

### 1.2 Switch to authentication database

```shell
use <database>
```

-   `<database>`: Name of database (recommended: admin)

### 1.3 Create administrator within admin database

```shell
db.createUser({
  user: "<username>",
  pwd: passwordPrompt(),
  roles: [
    { role: "userAdminAnyDatabase", db: <database> },
    { role: "readWriteAnyDatabase", db: <database> }
  ],
  mechanisms: [ "SCRAM-SHA-256" ]
})
```

-   `<username>`: Placeholder for your username, default can be "admin"
-   `passwordPrompt()`: Opens a prompt to input the password.
-   `userAdminAnyDatabase`: The created User can create users, grant or revoke roles from users and create or modify
    custom roles
-   `<database>`: Placeholder for you database, use the same as used in 2.2
-   `readWriteAnyDatabase`: The created user can read and write any database on this server
-   An example of all available build-in roles can be found [here][buildinroles]

### 1.4 Shutdown MongoDB and exit shell

```shell
db.adminCommand({ shutdown: true })
```

### 1.5 Open MongoDB Configuration File `mongod.cfg`

-   Windows: `cd C:/<install location>/MongoDB/Server/<version>/bin`
-   Linux: `cd /var/lib/mongodb`

### 1.6 Enable Authorisation

-   set or add `security.authorization` to `enabled` and save (you may need admin rights)

### 1.7 Restart MongoDB

-   Windows (navigate in directory mentioned in 5: Open MongoDB Configuration File):

```shell
./mongod.exe --auth --port <port> --dbpath "C:/<install location>/MongoDB/Server/\<version>/data"
```

-   Linux:

```shell
mongod --auth --port <port> --dbpath "/var/lib/mongodb"
```

### 1.8 Connect to Database with previous created use

```shell
mongosh --port <port> --authenticationDatabase <database> -u <username> -p
```

### 1.9 Switch to authentication Database and check user

```shell
use <database>
```

```shell
show users
```

## 2 Create and update additional user

### 2.1 Create user with multiple roles on multiple databases

```shell
db.createUser({
  user: <username>,
  pwd: passwordPrompt(),
  roles: [
    { role: "readWrite", db: <database1> },
    { role: "read", db: <database2> }
  ],
  mechanisms: [ "SCRAM-SHA-256" ]
})
```

### 2.2 Revoke Roles from user

```shell
db.revokeRolesFromUser(<username>, [{ role: "read", db: <database2> }], {})
```

### 2.3 Grant Roles to user

```shell
db.grantRolesToUser(<username>, [{ role: "readWrite", db: <database2> }] , {})
```

### 2.4 Update whole User

-   **Important**: The whole array of assigned roles will be replaced with the new roles. [See more Information][updateuser]

```shell
db.updateUser(<username>,{
  customData: {
    hello: "world"
  },
  roles: [ { role: "read", db: <database1> } ],
  pwd: passwordPrompt(),
  mechanisms: [ "SCRAM-SHA-256" ]
})
```

### 2.5 Check updated user

```shell
show users
```

[mongodbcommunity]: https://www.mongodb.com/try/download/community
[mongodb]: https://www.mongodb.com
[tutorial]: https://www.mongodb.com/docs/manual/core/security-scram/
[mongosh]: https://www.mongodb.com/try/download/shell
[buildinroles]: https://www.mongodb.com/docs/manual/reference/built-in-roles/
[updateuser]: https://www.mongodb.com/docs/manual/reference/method/db.updateUser/
