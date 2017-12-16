# [Scaledrone](https://www.scaledrone.com/) Swift

> Use the Scaledrone Swift client to connect to the Scaledrone realtime messaging service

This project is still a work in progress, pull requests and issues are very welcome.


## Installation

### CocoaPods

Check out [Get Started](http://cocoapods.org/) tab on [cocoapods.org](http://cocoapods.org/).

To use Scaledrone in your project add the following 'Podfile' to your project
```ruby
pod 'Scaledrone', '~> 0.2.0'
```

Then run:
```
pod install
```

## Usage

First thing is to import the framework. See the Installation instructions on how to add the framework to your project.

```swift
import Scaledrone
```

Once imported, you can connect to Scaledrone.

```swift
scaledrone = Scaledrone(channelID: "your-channel-id")
scaledrone.delegate = self
scaledrone.connect()
```

After you are connected, there are some delegate methods that we need to implement.

#### scaledroneDidConnect

```swift
func scaledroneDidConnect(scaledrone: Scaledrone, error: NSError?) {
    print("Connected to Scaledrone")
}
```

#### scaledroneDidReceiveError

```swift
func scaledroneDidReceiveError(scaledrone: Scaledrone, error: NSError?) {
    print("Scaledrone error", error ?? "")
}
```

#### scaledroneDidDisconnect

```swift
func scaledroneDidDisconnect(scaledrone: Scaledrone, error: NSError?) {
    print("Scaledrone disconnected", error ?? "")
}
```

## Authentication

Implement the **`ScaledroneAuthenticateDelegate`** protocol and set an additional delegate
```swift
scaledrone.authenticateDelegate = self
```

Then use the authenticate method to authenticate using a JWT

```swift
scaledrone.authenticate(jwt: "jwt_string")
```

#### scaledroneDidAuthenticate

```swift
func scaledroneDidAuthenticate(scaledrone: Scaledrone, error: NSError?) {
    print("Scaledrone authenticated", error ?? "")
}
```

## Sending messages

```swift
scaledrone.publish(message: "Hello from Swift", room: "myroom")
// Or
room.publish(message: ["foo": "bar", "1": 2])
```

## Subscribing to messages

Subscribe to a room and implement the **`ScaledroneRoomDelegate`** protocol, then set additional delegation

```swift
let room = scaledrone.subscribe(roomName: "myroom")
room.delegate = self
```

#### scaledroneRoomDidConnect

```swift
func scaledroneRoomDidConnect(room: ScaledroneRoom, error: NSError?) {
    print("Scaledrone connected to room", room.name, error ?? "")
}
```

#### scaledroneRoomDidReceiveMessage

```swift
func scaledroneRoomDidReceiveMessage(room: ScaledroneRoom, message: Any) {
    if let message = message as? [String : Any] {
        print("Received a dictionary:", message)
    }
    if let message = message as? [Any] {
        print("Received an array:", message)
    }
    if let message = message as? String {
        print("Received a string:", message)
    }
}
```

## Observable rooms

Implement the **`ScaledroneObservableRoomDelegate`** protocol, then set additional delegation.

> Observable room names need to be prefixed with *observable-*

```swift
let room = scaledrone.subscribe(roomName: "observable-room")
room.delegate = self
room.observableDelegate = self
```

#### scaledroneObservableRoomDidConnect
```swift
func scaledroneObservableRoomDidConnect(room: ScaledroneRoom, members: [ScaledroneMember]) {
    // The list will contain yourself
    print(members.map { (m: ScaledroneMember) -> String in
        return m.id
    })
}
```

#### scaledroneObservableRoomMemberDidJoin
```swift
func scaledroneObservableRoomMemberDidJoin(room: ScaledroneRoom, member: ScaledroneMember) {
    print("member joined", member, member.id)
}
```

#### scaledroneObservableRoomMemberDidLeave
```swift
func scaledroneObservableRoomMemberDidLeave(room: ScaledroneRoom, member: ScaledroneMember) {
    print("member left", member, member.id)
}
```

## Basic Example
```swift
import UIKit

class ViewController: UIViewController, ScaledroneDelegate, ScaledroneRoomDelegate {

    let scaledrone = Scaledrone(channelID: "your-channel-id")

    override func viewDidLoad() {
        super.viewDidLoad()
        scaledrone.delegate = self
        scaledrone.connect()
    }

    func scaledroneDidConnect(scaledrone: Scaledrone, error: NSError?) {
        print("Connected to Scaledrone channel", scaledrone.clientID)
        let room = scaledrone.subscribe(roomName: "notifications")
        room.delegate = self
    }

    func scaledroneDidReceiveError(scaledrone: Scaledrone, error: NSError?) {
        print("Scaledrone error")
    }

    func scaledroneDidDisconnect(scaledrone: Scaledrone, error: NSError?) {
        print("Scaledrone disconnected")
    }

    func scaledroneRoomDidConnect(room: ScaledroneRoom, error: NSError?) {
        print("Scaledrone connected to room", room.name)
    }

    func scaledroneRoomDidReceiveMessage(room: ScaledroneRoom, message: String) {
        print("Room received message:", message)
    }
}
```

For a longer example see the `ViewController.swift` file.

## Todo:

* Automatic reconnection
