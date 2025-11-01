# Creating Nodes

This guide walks through creating custom nodes that communicate with the Honeybee Core manager using the protocol defined in [protocol.md](protocol.md).

## Overview

Nodes are independent processes that connect to the [`NodeManager`](../../honeybee_core/src/node_manager/manager.rs) via TCP and exchange JSON messages. The protocol is bidirectional and uses the [`MessageEnvelope`](../../honeybee_core/bee_message/src/common.rs) wrapper with directional message types.

## Basic Node Structure

A node implementation needs to:

1. Establish a TCP connection to the manager
2. Send a [`NodeRegistration`](../../honeybee_core/bee_message/src/node_to_manager.rs) message
3. Wait for and process a [`RegistrationAck`](../../honeybee_core/bee_message/src/manager_to_node.rs)
4. Send periodic [`NodeStatusUpdate`](../../honeybee_core/bee_message/src/node_to_manager.rs) heartbeats
5. Handle incoming [`NodeCommand`](../../honeybee_core/bee_message/src/manager_to_node.rs) messages
6. Send [`NodeDrop`](../../honeybee_core/bee_message/src/node_to_manager.rs) before disconnecting

## Example Implementations

The project includes three complete reference implementations:

### Rust Implementation

See [`node_example/rust/src/main.rs`](../../node_example/rust/src/main.rs) for the full implementation.

**Key components:**

```rust
struct NodeClient {
    node_id: u64,
    node_name: String,
    address: String,
    port: u16,
    node_type: NodeType,
    server_address: String,
}

impl NodeClient {
    async fn connect(&self) -> Result<TcpStream, Box<dyn Error>> {
        // Establish TCP connection
    }

    async fn register(&self, stream: &mut TcpStream) -> Result<(), Box<dyn Error>> {
        // Send NodeRegistration
    }

    async fn send_status_update(&self, stream: &mut TcpStream, status: NodeStatus) 
        -> Result<(), Box<dyn Error>> {
        // Send NodeStatusUpdate
    }

    async fn handle_message(&self, envelope: MessageEnvelope<ManagerToNodeMessage>, 
        stream: &mut TcpStream) -> Result<(), Box<dyn Error>> {
        // Process commands from manager
    }

    async fn run(&self) -> Result<(), Box<dyn Error>> {
        // Main event loop with heartbeats
    }
}
```

**Running the Rust example:**

```bash
cd node_example/rust
cargo run
```

### Python Implementation

See [`node_example/python/main.py`](../../node_example/python/main.py) for the full implementation.

**Key components:**

```python
class NodeClient:
    async def connect(self) -> tuple:
        # Establish TCP connection
        
    async def register(self, writer: asyncio.StreamWriter) -> None:
        # Send NodeRegistration
        
    async def send_status_update(self, writer: asyncio.StreamWriter, 
        status: NodeStatus) -> None:
        # Send NodeStatusUpdate
        
    async def handle_message(self, envelope: Dict[str, Any], 
        writer: asyncio.StreamWriter) -> None:
        # Process commands from manager
        
    async def heartbeat_loop(self, writer: asyncio.StreamWriter) -> None:
        # Periodic status updates
        
    async def run(self) -> None:
        # Main event loop
```

**Running the Python example:**

```bash
cd node_example/python
python main.py
```

### Go Implementation

See [`node_example/go/main.go`](../../node_example/go/main.go) and [`node_example/go/client/client.go`](../../node_example/go/client/client.go) for the full implementation.

**Key components:**

```go
type NodeClient struct {
    nodeID        uint64
    nodeName      string
    address       string
    port          uint16
    nodeType      protocol.NodeType
    serverAddress string
    conn          net.Conn
    writer        *bufio.Writer
    reader        *bufio.Reader
}

func (nc *NodeClient) connect() error {
    // Establish TCP connection
}

func (nc *NodeClient) register() error {
    // Send NodeRegistration
}

func (nc *NodeClient) sendStatusUpdate(status protocol.NodeStatus) error {
    // Send NodeStatusUpdate
}

func (nc *NodeClient) handleMessage(envelope protocol.MessageEnvelope) error {
    // Process commands from manager
}

func (nc *NodeClient) Run() error {
    // Main event loop with heartbeats
}
```

**Running the Go example:**

```bash
cd node_example/go
go run main.go
```

## Step-by-Step Implementation Guide

### 1. Configuration

Define your node's identity and connection details:

```rust
let node_id = rand::random::<u64>();
let node_name = format!("my-node-{}", node_id % 1000);
let address = "0.0.0.0".to_string();
let port = 8080;
let node_type = NodeType::Agent;
let server_address = "127.0.0.1:9001".to_string();
```

See the main functions in [Rust](../../node_example/rust/src/main.rs), [Python](../../node_example/python/main.py), and [Go](../../node_example/go/main.go) examples.

### 2. TCP Connection

Establish a connection to the manager:

**Rust:**
```rust
async fn connect(&self) -> Result<TcpStream, Box<dyn Error>> {
    println!("Connecting to server at {}...", self.server_address);
    let stream = TcpStream::connect(&self.server_address).await?;
    println!("Connected to server!");
    Ok(stream)
}
```

**Python:**
```python
async def connect(self) -> tuple:
    print(f"Connecting to server at {self.server_host}:{self.server_port}...")
    reader, writer = await asyncio.open_connection(
        self.server_host, self.server_port
    )
    print("Connected to server!")
    return reader, writer
```

**Go:**
```go
func (nc *NodeClient) connect() error {
    fmt.Printf("[INFO] Connecting to server at %s...\n", nc.serverAddress)
    conn, err := net.Dial("tcp", nc.serverAddress)
    if err != nil {
        return fmt.Errorf("failed to connect: %w", err)
    }
    nc.conn = conn
    nc.writer = bufio.NewWriter(conn)
    nc.reader = bufio.NewReader(conn)
    fmt.Println("[INFO] Connected to server successfully")
    return nil
}
```

### 3. Registration

Send a [`NodeRegistration`](../../honeybee_core/bee_message/src/node_to_manager.rs) message wrapped in a [`MessageEnvelope`](../../honeybee_core/bee_message/src/common.rs):

**Rust:**
```rust
async fn register(&self, stream: &mut TcpStream) -> Result<(), Box<dyn Error>> {
    println!("Sending registration...");

    let registration = NodeRegistration {
        node_id: self.node_id,
        node_name: self.node_name.clone(),
        address: self.address.clone(),
        port: self.port,
        node_type: self.node_type.clone(),
    };

    let envelope = MessageEnvelope::new(
        PROTOCOL_VERSION,
        NodeToManagerMessage::NodeRegistration(registration),
    );

    let json = serde_json::to_string(&envelope)?;
    stream.write_all(json.as_bytes()).await?;
    stream.flush().await?;

    println!("Registration sent successfully!");
    Ok(())
}
```

See complete implementations in [Rust](../../node_example/rust/src/main.rs), [Python](../../node_example/python/main.py), and [Go](../../node_example/go/client/client.go).

### 4. Processing Registration Acknowledgment

Wait for and handle the [`RegistrationAck`](../../honeybee_core/bee_message/src/manager_to_node.rs):

**Rust:**
```rust
async fn handle_message(
    &self,
    envelope: MessageEnvelope<ManagerToNodeMessage>,
    stream: &mut TcpStream,
) -> Result<(), Box<dyn Error>> {
    if envelope.version != PROTOCOL_VERSION {
        eprintln!(
            "Protocol version mismatch: got {}, expected {}",
            envelope.version, PROTOCOL_VERSION
        );
    }

    match envelope.message {
        ManagerToNodeMessage::RegistrationAck(ack) => {
            if ack.accepted {
                println!("Registration acknowledged by manager");
                if let Some(msg) = ack.message {
                    println!("Manager message: {}", msg);
                }
            } else {
                eprintln!("Registration rejected by manager");
                if let Some(msg) = ack.message {
                    eprintln!("Rejection reason: {}", msg);
                }
            }
        }
        // ... handle other message types
    }
    Ok(())
}
```

### 5. Heartbeat Loop

Send periodic [`NodeStatusUpdate`](../../honeybee_core/bee_message/src/node_to_manager.rs) messages:

**Rust:**
```rust
const HEARTBEAT_INTERVAL: Duration = Duration::from_secs(30);

async fn run(&self) -> Result<(), Box<dyn Error>> {
    loop {
        match self.connect().await {
            Ok(mut stream) => {
                self.register(&mut stream).await?;
                
                let mut interval = time::interval(HEARTBEAT_INTERVAL);
                
                loop {
                    tokio::select! {
                        _ = interval.tick() => {
                            if let Err(e) = self.send_status_update(
                                &mut stream, NodeStatus::Running
                            ).await {
                                eprintln!("Heartbeat failed: {}", e);
                                break;
                            }
                            println!("Heartbeat sent");
                        }
                        // ... handle incoming messages
                    }
                }
            }
            Err(e) => {
                eprintln!("Connection failed: {}", e);
            }
        }
        
        println!("Reconnecting in 5 seconds...");
        time::sleep(Duration::from_secs(5)).await;
    }
}
```

See heartbeat implementations in [Rust](../../node_example/rust/src/main.rs), [Python](../../node_example/python/main.py), and [Go](../../node_example/go/client/client.go).

### 6. Handling Commands

Process [`NodeCommand`](../../honeybee_core/bee_message/src/manager_to_node.rs) messages from the manager:

**Rust:**
```rust
match envelope.message {
    ManagerToNodeMessage::NodeCommand(cmd) => {
        println!("Received command: {}", cmd.command);

        match cmd.command.as_str() {
            "stop" => {
                self.send_status_update(stream, NodeStatus::Stopped).await?;
                self.send_event(stream, NodeEvent::Stopped).await?;
                println!("Node stopped by command");
            }
            "status" => {
                self.send_status_update(stream, NodeStatus::Running).await?;
            }
            "restart" => {
                println!("Restart command received");
                self.send_status_update(stream, NodeStatus::Stopped).await?;
                time::sleep(Duration::from_millis(500)).await;
                self.send_status_update(stream, NodeStatus::Running).await?;
                self.send_event(stream, NodeEvent::Started).await?;
            }
            _ => {
                println!("Unknown command: {}", cmd.command);
                self.send_event(
                    stream,
                    NodeEvent::Error {
                        message: format!("Unknown command: {}", cmd.command),
                    },
                ).await?;
            }
        }
    }
    // ... handle other message types
}
```

See command handling in [Rust](../../node_example/rust/src/main.rs), [Python](../../node_example/python/main.py), and [Go](../../node_example/go/client/client.go).

### 7. Graceful Shutdown

Send [`NodeDrop`](../../honeybee_core/bee_message/src/node_to_manager.rs) before closing the connection:

**Rust:**
```rust
async fn send_node_drop(&self, stream: &mut TcpStream) -> Result<(), Box<dyn Error>> {
    println!("Sending NodeDrop...");

    let envelope = MessageEnvelope::new(
        PROTOCOL_VERSION,
        NodeToManagerMessage::NodeDrop
    );

    let json = serde_json::to_string(&envelope)?;
    stream.write_all(json.as_bytes()).await?;
    stream.flush().await?;

    Ok(())
}

// In the main loop before disconnecting:
let _ = self.send_node_drop(&mut stream).await;
```

## Message Type Reference

### Node Types

[`NodeType`](../../honeybee_core/bee_message/src/common.rs) variants:

```rust
pub enum NodeType {
    Full,   // Full-featured honeypot
    Agent,  // Lightweight monitoring agent
}
```

### Node Status

[`NodeStatus`](../../honeybee_core/bee_message/src/common.rs) variants:

```rust
pub enum NodeStatus {
    Deploying,  // Node is bootstrapping
    Running,    // Node is healthy
    Stopped,    // Node intentionally halted
    Failed,     // Node encountered an error
    Unknown,    // Status cannot be determined
}
```

### Events

[`NodeEvent`](../../honeybee_core/bee_message/src/node_to_manager.rs) variants:

```rust
pub enum NodeEvent {
    Started,
    Stopped,
    Alarm,
    Error { message: String },
}
```

## Error Handling

Best practices for handling errors:

1. **Connection Failures**: Implement reconnection logic with exponential backoff
2. **Parse Errors**: Log malformed messages and continue reading
3. **Version Mismatches**: Log warnings but attempt to continue
4. **Command Errors**: Send [`NodeEvent::Error`](../../honeybee_core/bee_message/src/node_to_manager.rs) with details
5. **Network Errors**: Send [`NodeDrop`](../../honeybee_core/bee_message/src/node_to_manager.rs) if possible, then reconnect

See error handling examples in all three reference implementations.

## Testing Your Node

1. **Start the Manager**:
   ```bash
   cd honeybee_core
   cargo run
   ```

2. **Run Your Node**:
   ```bash
   # Your node implementation
   cargo run  # or python main.py, or go run main.go
   ```

3. **Check Manager Logs**:
   Look for registration confirmation and heartbeat logs in [`logs/debug-*.log`](../../honeybee_core/logs/) (if logging is configured in [`bee_config.toml`](../../honeybee_core/bee_config.toml)).

4. **Test Commands**:
   - The manager will accept commands via its command interface (future feature)
   - Manually test by modifying the manager to send commands

## Configuration

Node configuration typically includes:

```toml
[node]
id = 123456789
name = "my-honeypot"
type = "Agent"  # or "Full"
address = "0.0.0.0"
port = 8080

[manager]
address = "127.0.0.1:9001"

[heartbeat]
interval_seconds = 30
```

See [`bee_config.toml`](../../honeybee_core/bee_config.toml) for manager configuration example.

## Advanced Topics

### Custom Node Types

Extend [`NodeType`](../../honeybee_core/bee_message/src/common.rs) to support custom node categories.

### Custom Events

Add new variants to [`NodeEvent`](../../honeybee_core/bee_message/src/node_to_manager.rs) for domain-specific notifications.

### TLS Support

Wrap TCP streams with TLS for encrypted communication:

**Rust:**
```rust
use tokio_rustls::{TlsConnector, rustls::ClientConfig};

// Configure TLS
let config = ClientConfig::builder()
    .with_safe_defaults()
    .with_root_certificates(root_store)
    .with_no_client_auth();

let connector = TlsConnector::from(Arc::new(config));
let domain = rustls::ServerName::try_from("manager.example.com")?;

// Connect with TLS
let stream = TcpStream::connect(&self.server_address).await?;
let tls_stream = connector.connect(domain, stream).await?;
```

### Metrics Collection

Implement custom metrics by extending the event system:

```rust
pub enum NodeEvent {
    Started,
    Stopped,
    Alarm,
    Error { message: String },
    Metric { name: String, value: f64, timestamp: u64 },
}
```

## Troubleshooting

Common issues and solutions:

| Issue | Cause | Solution |
|-------|-------|----------|
| Connection refused | Manager not running | Start the manager with `cargo run` in [`honeybee_core`](../../honeybee_core/) |
| Registration fails | Protocol version mismatch | Ensure [`PROTOCOL_VERSION`](../../honeybee_core/bee_message/src/common.rs) matches |
| No heartbeats received | Interval too short/long | Adjust to 30 seconds (recommended) |
| Parse errors | JSON format mismatch | Check message structure against [protocol.md](protocol.md) |
| Connection drops | Network issues | Implement reconnection logic as in examples |

## Next Steps

- Read the complete [Protocol Definition](protocol.md)
- Explore the [`NodeManager`](../../honeybee_core/src/node_manager/manager.rs) implementation
- Study the [`Node`](../../honeybee_core/src/node_manager/node.rs) handler implementation
- Review message types in [`bee_message`](../../honeybee_core/bee_message/src/)
- Check the example implementations for your preferred language