# gRPC Proto Definitions

Shared protobuf definitions for gRPC services. Use this repository as a **git submodule** in your client and server projects.

## Proto Files

| File | Package | Description |
|------|---------|-------------|
| `greeting/greeting.proto` | `greeting` | Greeting service with all 4 gRPC patterns |

### Greeting Service Methods

- **SayHello** - Unary RPC (single request/response)
- **SayHelloServerStream** - Server streaming (single request, stream of responses)
- **SayHelloClientStream** - Client streaming (stream of requests, single response)
- **SayHelloBidirectional** - Bidirectional streaming (stream both ways)

---

## Adding as Submodule

### In Your Server Project

```bash
cd your-server-project
git submodule add git@github.com:ranadeeptujala/grpcproto.git src/main/proto/grpcproto
git commit -m "Add grpcproto submodule"
```

### In Your Client Project

```bash
cd your-client-project
git submodule add git@github.com:ranadeeptujala/grpcproto.git src/main/proto/grpcproto
git commit -m "Add grpcproto submodule"
```

---

## Cloning Projects with This Submodule

### Option 1: Clone with Submodules

```bash
git clone --recursive <your-project-url>
```

### Option 2: Initialize After Cloning

```bash
git clone <your-project-url>
cd your-project
git submodule update --init --recursive
```

---

## Updating the Submodule

When proto definitions are updated, run this in your client/server projects:

```bash
# Pull latest changes in submodule
git submodule update --remote grpcproto

# Commit the updated reference
git add grpcproto
git commit -m "Update grpcproto submodule to latest"
```

---

## Project Configuration

### Java/Spring Boot (Gradle)

**build.gradle:**

```groovy
plugins {
    id 'com.google.protobuf' version '0.9.4'
}

dependencies {
    implementation 'io.grpc:grpc-netty-shaded:1.59.0'
    implementation 'io.grpc:grpc-protobuf:1.59.0'
    implementation 'io.grpc:grpc-stub:1.59.0'
    compileOnly 'org.apache.tomcat:annotations-api:6.0.53'
}

protobuf {
    protoc {
        artifact = 'com.google.protobuf:protoc:3.25.1'
    }
    plugins {
        grpc {
            artifact = 'io.grpc:protoc-gen-grpc-java:1.59.0'
        }
    }
    generateProtoTasks {
        all()*.plugins {
            grpc {}
        }
    }
}

sourceSets {
    main {
        proto {
            // Include the submodule proto files
            srcDir 'src/main/proto/grpcproto'
        }
    }
}
```

### Java/Spring Boot (Maven)

**pom.xml:**

```xml
<dependencies>
    <dependency>
        <groupId>io.grpc</groupId>
        <artifactId>grpc-netty-shaded</artifactId>
        <version>1.59.0</version>
    </dependency>
    <dependency>
        <groupId>io.grpc</groupId>
        <artifactId>grpc-protobuf</artifactId>
        <version>1.59.0</version>
    </dependency>
    <dependency>
        <groupId>io.grpc</groupId>
        <artifactId>grpc-stub</artifactId>
        <version>1.59.0</version>
    </dependency>
</dependencies>

<build>
    <extensions>
        <extension>
            <groupId>kr.motd.maven</groupId>
            <artifactId>os-maven-plugin</artifactId>
            <version>1.7.1</version>
        </extension>
    </extensions>
    <plugins>
        <plugin>
            <groupId>org.xolstice.maven.plugins</groupId>
            <artifactId>protobuf-maven-plugin</artifactId>
            <version>0.6.1</version>
            <configuration>
                <protocArtifact>com.google.protobuf:protoc:3.25.1:exe:${os.detected.classifier}</protocArtifact>
                <pluginId>grpc-java</pluginId>
                <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.59.0:exe:${os.detected.classifier}</pluginArtifact>
                <protoSourceRoot>${project.basedir}/src/main/proto/grpcproto</protoSourceRoot>
            </configuration>
            <executions>
                <execution>
                    <goals>
                        <goal>compile</goal>
                        <goal>compile-custom</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

### Python

**requirements.txt:**

```
grpcio==1.59.0
grpcio-tools==1.59.0
```

**Generate Python stubs:**

```bash
python -m grpc_tools.protoc \
    -I./grpcproto \
    --python_out=./generated \
    --grpc_python_out=./generated \
    grpcproto/greeting/greeting.proto
```

### Go

**Generate Go stubs:**

```bash
protoc \
    --go_out=. \
    --go-grpc_out=. \
    --proto_path=grpcproto \
    grpcproto/greeting/greeting.proto
```

---

## Usage Examples

### Server Implementation (Java)

```java
import com.example.grpc.proto.GreetingServiceGrpc;
import com.example.grpc.proto.HelloRequest;
import com.example.grpc.proto.HelloResponse;
import io.grpc.stub.StreamObserver;

public class GreetingServiceImpl extends GreetingServiceGrpc.GreetingServiceImplBase {

    @Override
    public void sayHello(HelloRequest request, StreamObserver<HelloResponse> responseObserver) {
        HelloResponse response = HelloResponse.newBuilder()
            .setMessage("Hello, " + request.getName() + "!")
            .setTimestamp(java.time.Instant.now().toString())
            .setThreadInfo(Thread.currentThread().getName())
            .build();
        
        responseObserver.onNext(response);
        responseObserver.onCompleted();
    }
}
```

### Client Implementation (Java)

```java
import com.example.grpc.proto.GreetingServiceGrpc;
import com.example.grpc.proto.HelloRequest;
import com.example.grpc.proto.HelloResponse;
import io.grpc.ManagedChannel;
import io.grpc.ManagedChannelBuilder;

public class GreetingClient {
    
    public static void main(String[] args) {
        ManagedChannel channel = ManagedChannelBuilder
            .forAddress("localhost", 9090)
            .usePlaintext()
            .build();
        
        GreetingServiceGrpc.GreetingServiceBlockingStub stub = 
            GreetingServiceGrpc.newBlockingStub(channel);
        
        HelloRequest request = HelloRequest.newBuilder()
            .setName("World")
            .setLanguage("en")
            .build();
        
        HelloResponse response = stub.sayHello(request);
        System.out.println("Response: " + response.getMessage());
        
        channel.shutdown();
    }
}
```

### Client Implementation (Python)

```python
import grpc
from generated import greeting_pb2, greeting_pb2_grpc

def run():
    with grpc.insecure_channel('localhost:9090') as channel:
        stub = greeting_pb2_grpc.GreetingServiceStub(channel)
        
        request = greeting_pb2.HelloRequest(name="World", language="en")
        response = stub.SayHello(request)
        
        print(f"Response: {response.message}")

if __name__ == '__main__':
    run()
```

---

## Directory Structure Example

```
your-project/
├── src/
│   └── main/
│       ├── java/
│       │   └── com/example/...
│       └── proto/
│           └── grpcproto/          ← submodule
│               └── greeting/
│                   └── greeting.proto
├── build.gradle (or pom.xml)
└── .gitmodules
```

---

## Troubleshooting

### Submodule shows empty folder

```bash
git submodule update --init --recursive
```

### Detached HEAD in submodule

```bash
cd grpcproto
git checkout main
git pull origin main
```

### Remove submodule

```bash
git submodule deinit -f grpcproto
rm -rf .git/modules/grpcproto
git rm -f grpcproto
```

