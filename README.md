![1](https://user-images.githubusercontent.com/93249038/219044259-7bd51802-6418-4e9c-92f4-2946573438f3.jpg)


Prerequisites
This article is not training material for Go language. I assume you have some experience already.
You have to have installed and configured Go v1.11 before start. We are going to use Go modules capabilities.
You have to have experience how to install/configure any SQL database to use it as persistent storage for this tutorial
API first
What does it mean for me?

API definition MUST be language-, protocol-, transport- neutral
API definition and API implementation MUST be loosely coupled
API versioning
I need to exclude manual work to sync API definition, API implementation and API documentation. I need API implementation stubs/skeleton and API documentation are generated from API definition automatically.
I highlight are these point during the tutorial.

“To Do list” microservice
“To Do list” microservice is allowing to manage “To Do” items. ToDo item contains following fields:

ID (unique integer identifier)
Title (text)
Description (text)
Reminder (timestamp)
ToDo service contains classical CRUD methods Create, Read, Update, Delete and ReadAll method also.

Part 1: create gRPC CRUD service
Step 1: Create API definition

Part 1 is about how to develop gRPC CRUD service and client to test it.

Source code for Part 1 is available here:

amsokol/go-grpc-http-rest-microservice-tutorial
Source code for tutorial "How to develop Go gRPC microservice with HTTP/REST endpoint, middleware, Kubernetes…
github.com

Before we start we need to create Go project structure.

There is great template for Go project:

golang-standards/project-layout
Standard Go Project Layout. Contribute to golang-standards/project-layout development by creating an account on GitHub.
github.com

Please use it like I do!

I use Windows 10 x64 environment. Nevertheless I believe it is not problem for you to translate CMD commands to MacOS/Linux BASH.

First create and enter root project folder go-grpc-http-rest-microservice-tutorial (locate it outside GOPATH to use Go modules). Than initialize Go project:

mkdir go-grpc-http-rest-microservice-tutorial
cd go-grpc-http-rest-microservice-tutorial
go mod init github.com/<you>/go-grpc-http-rest-microservice-tutorial
Create folder structure for API definition:

mkdir -p api\proto\v1
where v1 is API version.

API versioning: it is my best practice to locate major versions of APIs in different folders.

Next create todo-service.proto file inside api\proto\v1 folder and add ToDoService definition with only one method Create for the beginning:


You can get proto language specification here:

Language Guide (proto3) | Protocol Buffers | Google Developers
This guide describes how to use the protocol buffer language to structure your protocol buffer data, including .proto…
developers.google.com

As you can see our API definition is absolutely language-, protocol-, transport- neutral. This is one of the key feature of protobuf.

To compile Proto file we need to install necessary tools and add packages.

Download Proto compiler binaries here:
protocolbuffers/protobuf
Protocol Buffers - Google's data interchange format - protocolbuffers/protobuf
github.com

Extract package to any folder on your PC and add “bin” to PATH environment variable
Create “third_party” folder in the “go-grpc-http-rest-microservice-tutorial”
Copy everything from Proto compiler “include” folder to “third_party” folder:

Result project structure should look like this
Install Go language code generator plugin for Proto compiler:
go get -u github.com/golang/protobuf/protoc-gen-go
Create protoc-gen.cmd (protoc-gen.sh for MacOS/Linux) file in the “third_party” folder:

Create output folder for generated Go files:
mkdir -p pkg/api/v1
Ensure we are in go-grpc-http-rest-microservice-tutorial folder and run compilation:
.\third_party\protoc-gen.cmd
for MacOS/Linux:

./third_party/protoc-gen.sh
It creates todo-service.pb.go file inside “pkg/model/v1” folder:


Result project structure should look like this
Please ignore .vscode folder. It is managed by Visual Studio Code that I use as Go IDE.

Cool. Lets add remaining methods ToDo service methods and compile:


And run proto compiler again to update Go code:

.\third_party\protoc-gen.cmd
for MacOS/Linux:

./third_party/protoc-gen.sh
You have to add Go files generation as CI/CD step in real life to avoid to do it manually

Done. API definition is ready.

Step 2: Develop API implementation using Go language

I use MySQL database from Google Cloud as persistent store for this tutorial. You can use another SQL database you like.

Script for MySQL to create ToDo table is the following:


I avoid steps how to install, configure SQL database and create table in this tutorial

Create file “pkg/service/v1/todo-service.go” with the following content:


Created “pkg/service/v1/todo-service.go” file is API implementation. Result project structure should look like this:


Done.

Step 3: Write API implementation tests

It does not matter what we are developing we must write tests. It is LAW.

There is fantastic mock library for to test SQL database interactions:

DATA-DOG/go-sqlmock
Sql mock driver for golang to test database interactions - DATA-DOG/go-sqlmock
github.com

I use it to create tests for ToDo service.

Put this file to the “pkg/service/v1” folder. Result project structure should look like this:


Done.

Step 4: Create gRPC server startup

Create file“pkg/protocol/grpc/server.go” with the following content:


RunServer function registers ToDo service and starts gRPC server.

You have to configure TLS for gRPC server in real life. See example how to do it.

Next create “pkg/cmd/server/server.go” file with the following content:


This RunServer function reads start parameters from command line, creates SQL database connection pool, creates ToDo service instance and call previous RunServer function of gRPC server.

Last is to create “cmd/server/main.go” file with the following content:


That’s all on server side. Result project structure should look like this:


Step 5: Create gRPC client

Create “cmd/client-grpc/main.go” file with the following content:


That’s all on client side. Result project structure should look like this:


Step 6: Run gRPC server and client

The last step is to ensure that gRPC server works.

Start terminal to build and run gRPC server (replace parameters according to your SQL database server):

cd cmd/server
go build .
server.exe -grpc-port=9090 -db-host=<HOST>:3306 -db-user=<USER> -db-password=<PASSWORD> -db-schema=<SCHEMA>
If we see:

2018/09/09 08:02:16 starting gRPC server...
It means server is started.

Open another terminal to build and run gRPC client:

cd cmd/client-grpc
go build .
client-grpc.exe -server=localhost:9090
If we see something like this:

![vscode](https://user-images.githubusercontent.com/93249038/219044347-1cfdca16-4b54-4d32-85f4-fe2cf35cc751.jpg)


2018/09/09 09:16:01 Create result: <api:"v1" id:13 >
2018/09/09 09:16:01 Read result: <api:"v1" toDo:<id:13 title:"title (2018-09-09T06:16:01.5755011Z)" description:"description (2018-09-09T06:16:01.5755011Z)" reminder:<seconds:1536473762 > > >
2018/09/09 09:16:01 Update result: <api:"v1" updated:1 >
2018/09/09 09:16:01 ReadAll result: <api:"v1" toDos:<id:9 title:"title (2018-09-09T04:45:16.3693282Z)" description:"description (2018-09-09T04:45:16.3693282Z)" reminder:<seconds:1536468316 > > toDos:<id:10 title:"title (2018-09-09T04:46:00.7490565Z)" description:"description (2018-09-09T04:46:00.7490565Z)" reminder:<seconds:1536468362 > > toDos:<id:13 title:"title (2018-09-09T06:16:01.5755011Z)" description:"description (2018-09-09T06:16:01.5755011Z) + updated" reminder:<seconds:1536473762 > > >
2018/09/09 09:16:01 Delete result: <api:"v1" deleted:1 >
Everything works fine.
