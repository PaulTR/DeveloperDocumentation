---
title: Connections
order: 2
section: SDK Documentation
---

Also see [Connecting A Robot](/sphero-robot-basics/connecting-a-robot) for the consumer connection experience.

### Overview
The Sphero Robot SDKs all connect to robots using the same **DiscoveryAgent** concept.  In order to connect, simply register for *robot connection state change* notifications and start Discovery.  If your App is specific to one type of robot (Sphero or Ollie), use the DiscoveryAgent specific to the protocol.  

### Import Sphero Robot SDK

```objective-c
#import <RobotKit/RobotKit.h>
#import <RobotUIKit/RobotUIKit.h> // optional UI classes
```

<div class="swift language-only">
{{#markdown}}
This should be in your bridging header. If you don't have a bridging header see [Getting Started](/sdk-documentation/getting-started)
{{/markdown}}
</div>

```swift
#import <RobotKit/RobotKit.h>
```


```java
import com.orbotix.le.DiscoveryAgentLE;
import com.orbotix.le.RobotLE;
import com.orbotix.common.RobotChangedStateListener;
```

### Register for Connection State Changes

```objective-c
[[RKRobotDiscoveryAgent sharedAgent] addNotificationObserver:self 
                                        selector:@selector(handleRobotStateChangeNotification:)];
```


```swift
RKRobotDiscoveryAgent.sharedAgent().addNotificationObserver(self, selector: "handleRobotStateChangeNotification:")
```

```java
// or use 'this' and implement RobotChangedStateListener
// DualStackDiscoveryAgent can be replaced with DiscoveryAgentClassic if you are only supporting Sphero
// or DiscoveryAgentLE if you are only supporting Ollie
DualStackDiscoveryAgent.getInstance().addDiscoveryListener(new RobotChangedStateListener() {
    @Override
    public void handleRobotChangedState(Robot robot, RobotChangedStateNotificationType type) {

    }
}); 
```

##### Handle the Connection State Change

<!-- <p class="objective-c language-only">
  Implement the method `- (void)handleRobotStateChangeNotification:(RKRobotChangedStateNotification *)n` to be able to handle robot state change events.
</p> -->

```objective-c
- (void)handleRobotStateChangeNotification:(RKRobotChangedStateNotification *)n {
    switch(n.type) {
        case RKRobotOnline: // robot is online and ready for commands
            break;
        case RKRobotDisconnected:
            break;
        case RKRobotFailedConnect:
            break;
    }
}
```

```swift
func handleRobotStateChangeNotification(notification: RKRobotChangedStateNotification) {
    switch (notification.type) {
        case .Online:
            break
        case .Disconnected:
            break
        case .FailedConnect:
            break
        default:
    }
}
```

```java
@Override
public void handleRobotChangedState(Robot robot, RobotChangedStateNotificationType type) {
    switch (type) {
        case Online:
            break;
        case Disconnected:
            break;
    }
}
```

```javascript
// changedState
```

### Start Discovery
*Warning: Discovering devices uses substantial resources. Do not leave discovery running when you are not about to connect to a robot.*

```objective-c
- (void)appDidBecomeActive:(NSNotification *)n {
  [RKRobotDiscoveryAgent startDiscovery];
}
```

```swift
func appDidBecomeActive(n: NSNotification) {
    RKRobotDiscoveryAgent.startDiscovery()
}
```

```java
@Override
protected void onStart() {
    super.onStart();
    // This line assumes that this object is a Context
    RobotDiscoveryAgent.getInstance().startDiscovery(this);
}
```

```unity
// todo
```
 
### Caching a Robot

<div class="objective-c language-only">
{{#markdown}}
When robot connects, you will get an object with the type `id<RKRobotBase>`. This protocol encompasses the basics of a Bluetooth robot, but does not do much robot-specific functionality. To get some neat built-in functionality, we will create a `RKConvenienceRobot` object when we receive the connected notification. The classes `RKOllie` and `RKSphero` provide even more functionality specific to each of the robots and are subclasses of `RKConvenienceRobot`.
{{/markdown}}
</div>

```objective-c
// AppDelegate.h
#import <RobotKit/RobotKit.h>

@interface AppDelegate : UIResponder <UIApplicationDelegate>

@property (strong, nonatomic) UIWindow* window;
@property (strong, nonatomic) id<RKRobotBase> robot;

@end

```

```objective-c
// AppDelegate.mm
-(void) handleRobotStateChange:(RKRobotChangedStateNotification *) n{
    switch(n.type){
        case RKRobotOnline:
            _robot = n.robot;
            break;
        case RKRobotDisconnected:
            break;
    }
}
```

<span class="swift language-only">
{{#markdown}}
When robot connects, you will get an object with the type `RKRobotBase`. This protocol encompasses the basics of a Bluetooth robot, but does not do much robot-specific functionality. To get some neat built-in functionality, we will create a `RKConvenienceRobot` object when we receive the connected notification. The classes `RKOllie` and `RKSphero` provide even more functionality specific to each of the robots and are subclasses of `RKConvenienceRobot`.
{{/markdown}}
</span>

```swift
var robot: RKConvenienceRobot!

{...}

func handleRobotStateChangeNotification(notification: RKRobotChangedStateNotification) {
    switch (notification.type) {
        case .Online:
            robot = RKConvenienceRobot(robot: notification.robot)
            break
        case .Disconnected:
            break
        default:
    }
}
```

```java
private ConvenienceRobot _robot;

{...}

@Override
public void handleRobotChangedState(Robot robot, RobotChangedStateNotificationType type) {
    switch (type) {
        case Online:
        // Bluetooth Classic (Sphero)
        if (robot instanceof RobotClassic) {
            _robot = new Sphero(robot);
        }
        // Bluetooth LE (Ollie)
        if (robot instanceof RobotLE) {
            _robot = new Ollie(robot);
        }
        break;
    }
}
```

```javascript
// changedState
```

### Disconnecting a Robot

When you are done with the robot, it is important to disconnect it so that the next application can use it. There are two methods to accomplish this:

The easiest way to disconnect any of the Sphero Robots is to 'sleep' the robot.  There is a ```sleep``` function on both the convenience robot object and on the robot base object.

```objective-c
[_robot sleep];  // sleeps and disconnects a robot.
```

```swift
robot.sleep() // sleeps and disconnects a robot.
```

#### Convenience Robot Method

<div class="objective-c language-only">
{{#markdown}}
If a `RKConvenienceRobot` is available, disconnection is accomplished by calling the method `- [RKConvenienceRobot disconnect]` and the robot will take care of the rest for you.
{{/markdown}}
</div>

```objective-c
@property (strong, nonatomic) RKConvenienceRobot *robot; // Assume that this is set when the robot connects

{...}

- (void)disconnectRobot {
    [_robot disconnect];
}
```



```swift
var robot: RKConvenienceRobot!  // Assume that this is set when the robot connects

{...}

func disconnectRobot() {
    robot.disconnect()
}
```

```java
private ConvenienceRobot _robot; // Assume that this is set when the robot connects

{...}

@Override
public void onStop() {
    super.onStop();
    _robot.disconnect();
}
```

```javascript
// onStop
```

#### Robot Method

<div class="objective-c language-only">
{{#markdown}}
If you have a `id<RKRobotBase>`, disconnection is a bit more manual. `RKRobotLE` objects need to have `- [RKRobotLE sleep]` called on them as to avoid leaving the processor awake while the robot is not connected. Disconnection will be automatic from the sleep. `RKRobotClassic` objects can just have `- [RKRobotClassic disconnect]` called on them.
{{/markdown}}
</div>

```objective-c
@property (strong, nonatomic) id<RKRobotBase> robot; // Assume this is set when the robot connects

{...}

- (void)disconnectRobot {
    if ([_robot isKindOfClass:[RKRobotLE class]]) {
        [_robot sleep];
    }
    else if ([_robot isKindOfClass:[RKRobotClassic class]]) {
        [_robot disconnect];
    }
}
```

<div class="swift language-only">
{{#markdown}}
If you have a `RKRobotBase`, disconnection is a bit more manual. `RKRobotLE` objects need to have `sleep()` called on them as to avoid leaving the processor awake while the robot is not connected. Disconnection will be automatic from the sleep. `RKRobotClassic` objects can just have `disconnect()` called on them.
{{/markdown}}
</div>

```swift
var robot: RKRobotBase! // Assume this is set when the robot connects

{...}

func disconnectRobot() {
    if (robot.isKindOfClass(RKRobotLE)) {
        robot.sleep()
    } else if (robot.isKindOfClass(RKRobotClass)) {
        robot.disconnect()
    }
}
```

```java
private Robot _robot;

{...}

@Override
public void onStop() {
    super.onStop();
    if (_robot instanceof RobotLE) {
        _robot.sleep();
    }
    else if (_robot instanceof RobotClassic) {
        _robot.disconnect();
    }
}
```

```unity
// Coming soon
```

