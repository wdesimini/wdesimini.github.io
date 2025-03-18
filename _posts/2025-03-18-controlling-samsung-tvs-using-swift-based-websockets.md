---
title: "Controlling Samsung TVs Using Swift-Based Websockets"
date: 2025-03-18
tags: [swift, iOS, websockets, samsung, api, smartview, remote, tvcommanderkit, starscream]
excerpt: "Learn how to establish a Samsung TV WebSocket connection, handle authorization, and send commands using Swift."
---

Samsung Smart TVs can be controlled remotely using WebSockets, allowing developers to send commands for navigation, volume control, power, and more. In this post, we’ll explore a basic approach to handling WebSocket connections, authorization, and command-writing with a Samsung TV using Swift. 

For a more complete version of this that includes more commands, features, unit tests and error-handling, feel free to visit my [TVCommanderKit](https://github.com/wdesimini/TVCommanderKit) repo on GitHub.

## 1. Establishing a WebSocket Connection

To begin, we need to build a URL using a given TV IP address, client app name, and optional token value. Afterwards, we'll import [Starscream](https://github.com/daltoniam/Starscream) and use this URL to create a WebSocket and establish a secure connection to the TV’s WebSocket server.

### Connecting to the TV

First, we'll declare a simple `TVWebSocket` class along with a delegate protocol and simple certificate pinner:

*Note - For the purpose of this example, I am using a simple certificate pinner `TVCertificatePinner`. If you're worried about MITM attacks, make sure to use a more refined approach for certificate-pinning.*

```swift
import Foundation
import Starscream

protocol TVWebSocketDelegate: AnyObject { 
}

class TVWebSocket {
    private var webSocket: WebSocket?
    private weak var delegate: (any TVWebSocketDelegate)?

    func configure(delegate: (any TVWebSocketDelegate)?) {
        self.delegate = delegate
    }

    func connect(to ipAddress: String, app: String, token: String?) {
        var components = URLComponents()
        components.path = "/api/v2/channels/samsung.remote.control"
        components.port = 8002
        components.scheme = "wss"
        components.host = ipAddress
        var queryItems = [URLQueryItem]()
        if let encodedApp = app.data(using: .utf8)?.base64EncodedString() {
            queryItems.append(.init(name: "name", value: encodedApp))
        }
        if let token = token {
            queryItems.append(.init(name: "token", value: token))
        }
        components.queryItems = queryItems
        guard let url = components.url?.absoluteString
            .removingPercentEncoding.flatMap(URL.init) else {
            return
        }
        webSocket = WebSocket(request: .init(url: url), certPinner: TVCertificatePinner())
        webSocket?.connect()
    }

    func disconnect() {
        webSocket?.disconnect()
    }
}

class TVCertificatePinner: CertificatePinning {
    func evaluateTrust(trust: SecTrust, domain: String?, completion: ((PinningState) -> ())) {
        completion(.success)
    }
}
```

## 2. Handling Authorization

Samsung TVs require authorization when sending commands. Upon the first connection, the TV will prompt the user to allow or deny control. After choosing "allow", the TV will send a packet containing our new token, indicating the connection was authorized.

### Receiving Authorization Token

Add the following data types to encapsulate the auth response we need to parse:

```swift
struct TVResponse<Body: Codable>: Codable {
    let data: Body?
    let event: String
}

struct TVAuthResponseBody: Codable {
    let clients: [TVClient]
    let token: String?
}

struct TVClient: Codable {
    struct Attributes: Codable {
        let name: String?
        let token: String?
    }

    let attributes: Attributes
    let deviceName: String
}
```

Update `TVWebSocketDelegate`:

```swift
protocol TVWebSocketDelegate: AnyObject { 
    func tvDidConnect()
    func tvDidDisconnect()
    func tvDidReceiveError(_ error: Error?)
    func tvDidReceiveToken(_ token: String?)
}
```

Conform `TVWebSocket` to `WebSocketDelegate`, set `webSocket?.delegate` within the initial connection method, and add the following in order to receive connection updates, errors, and packets containing authorization info for new and refreshed tokens:

```swift
func didReceive(event: WebSocketEvent, client: WebSocketClient) {
    switch event {
    case .connected:
        delegate?.tvDidConnect()
    case .cancelled, .disconnected:
        delegate?.tvDidDisconnect()
    case .text(let text):
        text.data(using: .utf8).flatMap(handlePacket(_:))
    case .binary(let data):
        handlePacket(data)
    case .error(let error):
        delegate?.tvDidReceiveError(error)
    default:
        break
    }
}

private func handlePacket(_ data: Data) {
    guard let authResponse = try? JSONDecoder().decode(
        TVResponse<TVAuthResponseBody>.self, from: data),
            authResponse.event == "ms.channel.connect" else {
        return
    }
    if let newToken = authResponse.data?.token {
        delegate?.tvDidReceiveToken(newToken)
    } else if let existingToken = authResponse.data?.clients.first?.attributes.token {
        delegate?.tvDidReceiveToken(existingToken)
    }
}
```

## 3. Sending Commands to the TV

With authorization complete, commands can be sent over the WebSocket connection.

### Writing a Command

Add the following data type for command packets:

```swift
struct TVCommand: Codable {
    struct Params: Codable {
        let cmd: String
        let dataOfCmd: String
        let option: Bool
        let typeOfRemote: String

        enum CodingKeys: String, CodingKey {
            case cmd = "Cmd"
            case dataOfCmd = "DataOfCmd"
            case option = "Option"
            case typeOfRemote = "TypeOfRemote"
        }
    }

    let method: String
    let params: Params
}
```

Add the following to `TVWebSocket` to send commands:

```swift
func sendCommand(_ key: String, completion: @escaping () -> Void) {
    let command = TVCommand(
        method: "ms.remote.control",
        params: .init(
            cmd: "Click",
            dataOfCmd: key,
            option: false,
            typeOfRemote: "SendRemoteKey"
        )
    )
    if let data = try? JSONEncoder().encode(command),
       let packet = String(data: data, encoding: .utf8) {
        webSocket?.write(string: packet, completion: completion)
    }
}
```

Here are a few common codes to start with:

| Action      | Key Code        |
|-------------|-----------------|
| Volume Up   | KEY_VOLUP       |
| Volume Down | KEY_VOLDOWN     |
| Mute        | KEY_MUTE        |
| Channel Up  | KEY_CHUP        |
| Channel Down| KEY_CHDOWN      |
| Power Off   | KEY_POWEROFF    |
| Home        | KEY_HOME        |