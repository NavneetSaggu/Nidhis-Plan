# gRPC

Steps on making gRPC work

1. Create a module which you’ll share with both the services that will need your grpc contracts (server and client), make this a maven dependency in both the server and client
2. In this module, write down request / response in proto formats in Name.proto
3. In this module, write down service in proto format NameService in Name.proto
4. Compile this (on terminal or using a maven plugin) to produce a NameServiceGrpc Java file
5. Within this file, you’ll produce a NameServiceGrpc.NameServiceImplBase
6. In the server, extend this NameServiceGrpc.NameServiceImplBase file to write the actual business logic that’ll be used to serve gRPC requests. You can call this NameGrpcService.
7. In the client, inject NameServiceGrpc into your client, use it’s blocking stub to make gRPC calls.

# Proto

Anything that’s optional in proto can be null

Anything that’s not optional in proto has to have some default value, either empty string or 0, otherwise it’ll throw a parse error