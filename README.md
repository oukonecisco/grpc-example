# gRPC python example

gRPC is a modern, open-source, high-performance Remote Procedure Call (RPC) framework that can run in any environment. It's used to connect services in and across data centers with pluggable support for load balancing, tracing, health checking, and authentication. It's also applicable in the last mile of distributed computing to connect devices, mobile applications, and browsers to backend services.

## To implement gRPC in Python, you need to:

1. **Define a service in a `.proto` file**: This includes specifying the RPC methods and their request and response types.
2. **Generate the gRPC code**: Use the protocol buffer compiler (`protoc`) to generate client and server code in Python.
3. **Implement the server**: Write the Python server that implements the service.
4. **Create the client**: Write a client that calls the service.

Here's a basic example:

### Step 1: Define the Service in a .proto file

Let's say you have a simple service for sending messages. Your `message.proto` might look like this:

```protobuf
syntax = "proto3";

package message;

// The message service definition.
service MessageService {
  // Sends a message
  rpc SendMessage (MessageRequest) returns (MessageResponse) {}
}

// The request message containing the user's name.
message MessageRequest {
  string message = 1;
}

// The response message containing the greetings
message MessageResponse {
  string result = 1;
}
```

### Step 2: Generate the gRPC Code

You would use the `protoc` command-line tool to generate Python gRPC code from your `.proto` file:

```bash
python -m grpc_tools.protoc -I. --python_out=. --grpc_python_out=. message.proto
```

This will generate `message_pb2.py` and `message_pb2_grpc.py` which contain the classes for your messages and service.

### Step 3: Implement the Server

Here's an example of what your server implementation might look like:

```python
from concurrent import futures
import grpc
import message_pb2
import message_pb2_grpc

class MessageService(message_pb2_grpc.MessageServiceServicer):

    def SendMessage(self, request, context):
        return message_pb2.MessageResponse(result=f"Received: {request.message}")

def serve():
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    message_pb2_grpc.add_MessageServiceServicer_to_server(MessageService(), server)
    server.add_insecure_port('[::]:50051')
    server.start()
    server.wait_for_termination()

if __name__ == '__main__':
    serve()
```

### Step 4: Create the Client

The client code might look like this:

```python
import grpc
import message_pb2
import message_pb2_grpc

def run():
    with grpc.insecure_channel('localhost:50051') as channel:
        stub = message_pb2_grpc.MessageServiceStub(channel)
        response = stub.SendMessage(message_pb2.MessageRequest(message='Hello!'))
        print("MessageService client received: " + response.result)

if __name__ == '__main__':
    run()
```

### Additional Steps:

- Install gRPC and gRPC tools: You need to install the gRPC package and the gRPC tools for Python. This can be done via pip:

  ```bash
  pip install grpcio
  pip install grpcio-tools
  ```

- Running the Server and Client: Run the server script in one terminal and the client script in another. The client will send a message to the server, which will respond.

This is a very basic example. In a real-world scenario, you might have more complex data types, error handling, and might need to secure the communication with SSL/TLS.
