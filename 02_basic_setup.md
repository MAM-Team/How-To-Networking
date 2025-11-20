# 02 – Creating a Basic Client/Server Setup with Unity Transport

This tutorial continues from the previous section and shows how to build the *smallest possible* online setup in Unity using the **Unity Transport Package (UTP)**.

You’ll learn how to initialize a server, start a client, send a small message, and see the data flow.

---

## 1) Installing Unity Transport

Open Unity and install:

**Window → Package Manager → Unity Registry → “Unity Transport”**

This gives you access to:
- `NetworkDriver`
- `NetworkConnection`
- UDP-based communication
- Pipelines & reliability utilities

---

## 2) Creating the Network Script

Create a C# script named `BasicTransportTest.cs` and add it to an empty GameObject.

```csharp
using Unity.Collections;
using Unity.Networking.Transport;
using UnityEngine;

public class BasicTransportTest : MonoBehaviour
{
    public enum Mode { Server, Client }
    public Mode mode = Mode.Server;

    private NetworkDriver driver;
    private NativeList<NetworkConnection> connections;

    void Start()
    {
        driver = NetworkDriver.Create();

        if (mode == Mode.Server)
        {
            var endpoint = NetworkEndPoint.AnyIpv4;
            endpoint.Port = 7777;

            if (driver.Bind(endpoint) != 0)
                Debug.Log("Failed to bind to port 7777");
            else
                driver.Listen();

            connections = new NativeList<NetworkConnection>(16, Allocator.Persistent);
            Debug.Log("Server started on port 7777");
        }
        else
        {
            connections = new NativeList<NetworkConnection>(1, Allocator.Persistent);
            var endpoint = NetworkEndPoint.Parse("127.0.0.1", 7777);
            var conn = driver.Connect(endpoint);
            connections.Add(conn);
            Debug.Log("Client connecting...");
        }
    }

    void Update()
    {
        driver.ScheduleUpdate().Complete();

        if (mode == Mode.Server)
            ServerUpdate();
        else
            ClientUpdate();
    }

    void ServerUpdate()
    {
        // Clean up lost connections
        for (int i = 0; i < connections.Length; i++)
        {
            if (!connections[i].IsCreated)
            {
                connections.RemoveAtSwapBack(i);
                i--;
                break;
            }
        }

        // Accept new clients
        NetworkConnection c;
        while ((c = driver.Accept()) != default)
        {
            connections.Add(c);
            Debug.Log("Client connected!");
        }

        // Process messages
        DataStreamReader stream;
        for (int i = 0; i < connections.Length; i++)
        {
            NetworkEvent.Type cmd;
            while ((cmd = driver.PopEventForConnection(connections[i], out stream)) != NetworkEvent.Type.Empty)
            {
                if (cmd == NetworkEvent.Type.Data)
                {
                    var msg = stream.ReadByte();
                    Debug.Log("Server received: " + msg);

                    // Echo message back to client
                    var writer = driver.BeginSend(connections[i]);
                    writer.WriteByte((byte)(msg + 1));
                    driver.EndSend(writer);
                }
                else if (cmd == NetworkEvent.Type.Disconnect)
                {
                    Debug.Log("Client disconnected");
                    connections[i] = default;
                }
            }
        }
    }

    void ClientUpdate()
    {
        if (!connections[0].IsCreated)
            return;

        // Receive messages
        DataStreamReader stream;
        NetworkEvent.Type cmd;
        while ((cmd = driver.PopEventForConnection(connections[0], out stream)) != NetworkEvent.Type.Empty)
        {
            if (cmd == NetworkEvent.Type.Connect)
            {
                Debug.Log("Client connected, sending test message...");

                var writer = driver.BeginSend(connections[0]);
                writer.WriteByte(42);
                driver.EndSend(writer);
            }
            else if (cmd == NetworkEvent.Type.Data)
            {
                var msg = stream.ReadByte();
                Debug.Log("Client received: " + msg);
            }
            else if (cmd == NetworkEvent.Type.Disconnect)
            {
                Debug.Log("Disconnected from server");
                connections[0] = default;
            }
        }
    }

    void OnDestroy()
    {
        driver.Dispose();
        connections.Dispose();
    }
}
```

---

## 3) How to Use It

1. Create an empty GameObject → Add `BasicTransportTest`.
2. Duplicate it.
3. One object = Server (`mode = Server`)
4. One object = Client (`mode = Client`)
5. Press Play.
6. The client will connect automatically and exchange a byte with the server.

This simple byte-roundtrip is the core of all future networking logic.

---

## 4) What You Learned Here

- How to create a UDP server with Unity Transport  
- How a client connects  
- How to send a byte  
- How to receive a byte  
- How to accept multiple clients on the server  
- How to handle disconnects  

This structure scales naturally into full MMO-style logic once replicated states and messages are layered on top.

---

Next steps can explore message structs, reliable pipelines, and movement synchronization.