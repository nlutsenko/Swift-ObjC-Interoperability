build-lists: true
autoscale: true
theme: fira

<br>
# Swift <-> ObjC 
# Interoperability
<br>
###  `Nikita Lutsenko`
####  @nlutsenko
#### Parse, Facebook, Banana!

---

## Swift <-> ObjC 
## Interoperability

---

## Swift <-> ObjC 
## I14y

---

![fit](Resources/bridge.jpg)

---

# Agenda

- ObjC -> Swift
- Swift <- ObjC
- Swift <-> ObjC

---

## ObjC -> Swift

---

![fit](Resources/itjustworks.png)

---

## ObjC -> Swift

- #ItJustWorks
- 99% of API is `@available`
- Swifty Names
- Type Safety
- Much better with Swift 3.0

---

## NSInvocation and Friends

```swift
NSInvocation()
NSMethodSignature()

NSOperation(invocation: )
NSProxy.methodSignatureForSelector()

...
```

## `NS_SWIFT_UNAVAILABLE("NSInvocation and related APIs not available")`

---

## ObjC -> Swift 2.*

- Nullability Annotations
- Error Handling
- `NS_SWIFT_NAME`
- `NS_SWIFT_UNAVAILABLE`
- `NS_REFINED_FOR_SWIFT`
- `NS_SWIFT_NOTHROW`

---

ObjC
<br/>

```objectivec
- (BOOL)performRequest:(NSURLRequest *)request;
- (BOOL)performRequest:(NSURLRequest *)request error:(NSError **)error;
```

<br/>

Swift 2.*
<br/>

```swift
public func performRequest(request: NSURLRequest!) -> Bool
public func performRequest(request: NSURLRequest!, error: ()) throws
```

^ Introduction to example interface

---

## Nullability Annotations


<br/>

```objectivec

- (BOOL)performRequest:(nullable NSURLRequest *)request;
- (BOOL)performRequest:(nullable NSURLRequest *)request error:(NSError **)error;

```

<br/>

```swift
public func performRequest(request: NSURLRequest?) -> Bool
public func performRequest(request: NSURLRequest?, error: ()) throws
```

^ Nullability Annototations with `nullable`

---

## Nullability Annotations via Regions

```objectivec
NS_ASSUME_NONNULL_BEGIN

- (BOOL)performRequest:(NSURLRequest *)request;
- (BOOL)performRequest:(NSURLRequest *)request error:(NSError **)error;

NS_ASSUME_NONNULL_END
```

<br>

```swift
public func performRequest(request: NSURLRequest) -> Bool
public func performRequest(request: NSURLRequest, error: ()) throws
```

^ Nullability Annototations with NS_ASSUME_NONNULL_*

---

# `NS_SWIFT_NAME`

```objectivec

- (BOOL)performRequest:(nullable NSURLRequest *)request 
                       NS_SWIFT_NAME(perform(request:));
                       
- (BOOL)performRequest:(nullable NSURLRequest *)request
                 error:(NSError **)error NS_SWIFT_NAME(perform(request:));

```

<br/>

```swift
public func perform(request request: NSURLRequest?) -> Bool
public func perform(request request: NSURLRequest?) throws
```

^ NS_SWIFT_NAME

---

# `NS_SWIFT_UNAVAILABLE`

```objectivec

- (BOOL)performRequest:(nullable NSURLRequest *)request 
                       NS_SWIFT_UNAVAILABLE("");
                       
- (BOOL)performRequest:(nullable NSURLRequest *)request
                 error:(NSError **)error NS_SWIFT_NAME(perform(request:));

```

<br/>

```swift
public func perform(request request: NSURLRequest?) throws
```

^ NS_SWIFT_UNAVAILABLE

---

# `NS_SWIFT_UNAVAILABLE`

```objectivec

- (BOOL)performRequest:(nullable NSURLRequest *)request;
- (BOOL)performRequest:(nullable NSURLRequest *)request
                 error:(NSError **)error;

```

<br/>

```swift
public func performRequest(request: NSURLRequest) -> Bool
public func performRequest(request: NSURLRequest, error: ()) throws
```

^ NS_SWIFT_UNAVAILABLE Doesn't need NS_SWIFT_NAME() anymore

---

# `NS_SWIFT_UNAVAILABLE`

```objectivec

- (BOOL)performRequest:(nullable NSURLRequest *)request
                       NS_SWIFT_UNAVAILABLE("");
- (BOOL)performRequest:(nullable NSURLRequest *)request
                 error:(NSError **)error;

```

<br/>

```swift
public func performRequest(request: NSURLRequest?) throws
```

^ NS_SWIFT_UNAVAILABLE Doesn't need NS_SWIFT_NAME() anymore

---

# Combine Everything

```objectivec

- (BOOL)performRequest:(nullable NSURLRequest *)request 
                       NS_SWIFT_UNAVAILABLE("");
                       
- (BOOL)performRequest:(nullable NSURLRequest *)request
                 error:(NSError **)error NS_SWIFT_NAME(perform(request:));

```

<br/>

```swift
public func perform(request request: NSURLRequest?) throws
```

^ NS_SWIFT_NAME is still useful, since it provides Swiftier API

---

ObjC

```objectivec
- (void)getFirstName:(NSString **)firstName
            lastName:(NSString **)lastName;

```

<br/>
Swift

```swift
func getFirstName(firstName: AutoreleasingUnsafeMutablePointer<NSString?>,
                  lastName: AutoreleasingUnsafeMutablePointer<NSString?>)
```

^ Bad API to import into Swift

---

ObjC

```objectivec
- (void)getFirstName:(NSString **)firstName
            lastName:(NSString **)lastName 
                     NS_SWIFT_NAME(get(firstName:lastName:));

```

<br/>
Swift

```swift
func get(firstName firstName: AutoreleasingUnsafeMutablePointer<NSString?>,
         lastName: AutoreleasingUnsafeMutablePointer<NSString?>)
```

^ We might have gone this way and still used this API.

---

ObjC

```objectivec
- (void)getFirstName:(NSString **)firstName
            lastName:(NSString **)lastName NS_REFINED_FOR_SWIFT;

```

<br/>
Swift

```swift
extension Person {
  var names: (firstName: String?, lastName: String?) {
      var firstName: NSString? = nil
      var lastName: NSString? = nil
      __getFirstName(&firstName, lastName: &lastName)
      return (firstName: firstName as String?, lastName: lastName as String?)
  }
}
```

^ Use `NS_REFINED_FOR_SWIFT` and create nice Swift wrapper

---

### ObjC -> Swift 3.0

- "The Grand Rename"
- Objective-C Lightweight Generics
- `NS_NOESCAPE`/`CF_NOESCAPE`
- `NS_EXTENSIBLE_STRING_ENUM`

---

# ObjC Generics

```objectivec
@interface MyNetworkController<Request: NSURLRequest *> : NSObject

- (void)performRequest:(nullable Request)request;

@end

```

<br/>

```swift
public class MyNetworkController : NSObject {
    public func performRequest(request: NSURLRequest?)
}
```

^ Swift 2.0 does not import ObjC Generics

---

# ObjC Generics

```objectivec
@interface MyNetworkController<Request: NSURLRequest *> : NSObject

- (void)performRequest:(nullable Request)request;

@end

```

<br/>

```swift
public class MyNetworkController<Request : NSURLRequest> : NSObject {
    public func perform(_ request: Request?)
}

```

^ Swift 3.0 imports ObjC Generics

---

# `NS_NOESCAPE/CF_NOESCAPE`

<br>

```objectivec
- (void)executeActions:(void (NS_NOESCAPE ^)(void))actions;
```

<br/>

```swift
func executeActions(_ actions: @noescape () -> Void)
```

^ NS_NOESCAPE

---

# `NS_NOESCAPE/CF_NOESCAPE`

<br>

```objectivec
- (void)executeActions:(void (NS_NOESCAPE ^)(void))actions
                           NS_SWIFT_NAME(execute(actions:));
```

<br/>


```swift
func execute(actions: (@noescape () -> Void))
```

^ NS_NOESCAPE

---

# NS_EXTENSIBLE_STRING_ENUM

ObjC

```objectivec
typedef NSString *NSRunLoopMode;
extern NSRunLoopMode const NSDefaultRunLoopMode;
```

<br/>
Swift 2

```swift
public typealias NSRunLoopMode = NSString
public let NSDefaultRunLoopMode: String
```

^ NS_EXTENSIBLE_STRING_ENUM

---

# NS_EXTENSIBLE_STRING_ENUM

ObjC

```objectivec
typedef NSString *NSRunLoopMode NS_EXTENSIBLE_STRING_ENUM;
extern NSRunLoopMode const NSDefaultRunLoopMode;
```

<br/>
Swift 3

```swift
struct RunLoopMode : RawRepresentable {
  public static let defaultRunLoopMode: RunLoopMode
}

```

^ NS_EXTENSIBLE_STRING_ENUM

---

# ObjC -> Swift

- Use Nullability Annotations
- Use Objective-C Generics
- Rename (`NS_SWIFT_NAME`)
- Refine (`NS_REFINED_FOR_SWIFT`)
- Rinse (`NS_SWIFT_UNAVAILABLE`)
- Repeat

---

![fit](Resources/bridge_job.jpg)

---

## Swift -> ObjC

---

# Swift -> ObjC

- No Tuples
- No Generics
- No Typealiases
- No Swift Enums
- No Swift Structs
- No Global Variables
- No Free-standing Functions
- Only ObjC Runtime Compatible

---

![fit](Resources/nofun.jpg)

---

# Swift -> ObjC

- Subclasses of `NSObject`
- Not `private` (`fileprivate`)
- Not `@nonobjc`
- `@objc` protocols

---

# `@objc & @nonobjc`

- Subclasses of `NSObject`
- Explicit export to ObjC
- Concrete extensions
- `@objc` protocols
- `Int` enums

---

# `@objc` protocols

- Special case of protocol
- Weaker and not-Swifty
- Supports `optional`
- Extensions are inaccessible from ObjC

---

## Value Types via `_ObjectiveCBridgeable`

- Powers Foundation
- Implementation Detail Protocol
- Native ObjC Types <-> Native Swift Types

---

## `_ObjectiveCBridgeable`

```swift
public protocol _ObjectiveCBridgeable {
  associatedtype _ObjectiveCType : AnyObject

  static func _isBridgedToObjectiveC() -> Bool

  func _bridgeToObjectiveC() -> _ObjectiveCType
  
  static func _forceBridgeFromObjectiveC(_ source: _ObjectiveCType, result: inout Self?)

  static func _conditionallyBridgeFromObjectiveC(
    _ source: _ObjectiveCType,
    result: inout Self?
  ) -> Bool

  static func _unconditionallyBridgeFromObjectiveC(_ source: _ObjectiveCType?) -> Self
}
```

---

## Swift <-> ObjC

---

## Swift <-Pitfalls-> ObjC

---

### Swift <-Pitfalls-> ObjC

![right](Resources/pitfall.png)

- Closure -> Block Performance
- Type Compatibility
- Objective-C Runtime Hackery

---

## Type Compatibility

<br/>

```objectivec
NSArray<NSString *> *names = @[ @"John", @"Jane" ];
NSArray<id <NSCopying>> *array = names;
```

^ In ObjC generics are compile-time only and are type erased.

---

## Type Compatibility

<br/>

```swift
let names: [String] = ["John", "Jane"]
let array: [CustomDebugStringConvertible] = names
```

^ Let's use the same paradighm. We upcast the array of strings into an array of a protocol that is implemented by String.
^ For example, CustomDebugStringConvertible

---

## Type Compatibility

<br/>

```swift
let names: [String] = ["John", "Jane"]
let array: [CustomDebugStringConvertible] = names
```

## `fatal error: can't unsafeBitCast between types of different sizes`

^ Turns out that this fails with a runtime exception, but doesn't warn you about anything.
^ The reason for it, is that `CustomDebugStringConvertible` is a separate type in Swift, since protocols in Swift have a different memory layout
^ compared to a String. Where String has a struct memory layout, where CustomStringConvertible uses a witness table for each value on top of the storage.
^ So, in our particular case - since you are creating an array of strings - compiler doesn't create extra space for witness table, as it's not needed
^ But when you cast it to CustomDebugStringConvertible - it's actually required.

---

## Type Compatibility

<br/>

```swift
let names: [String] = ["John", "Jane"]
let array: [CustomDebugStringConvertible] = names.map({ 
  $0 as CustomDebugStringConvertible 
})
```

^ You fix it by explicitly casting every member of the array to the explicit type, which adds the witness table behind the scenes

---

## Objective-C Runtime in Swift

```swift
@objc class Animal: NSObject {
    let name = "Animal"
    override init() {
        super.init()
    }
}
```

```objectivec
@interface Plant: NSObject
@end

@implementation Plant

- (NSString *)name {
    return @"Plant";
}

@end
``` 

---

## Objective-C Runtime in Swift

```objectivec
// ObjC
SEL selector = @selector(name);

Method originalMethod = class_getInstanceMethod([Animal class], selector);
Method swizzledMethod = class_getInstanceMethod([Plant class], selector);

method_exchangeImplementations(originalMethod, swizzledMethod);

// ObjC
NSLog(@"I am %@!", name);

// Swift
print("I am \(name)!")
```


---

## Objective-C Runtime in Swift

```objectivec
// ObjC
SEL selector = @selector(name);

Method originalMethod = class_getInstanceMethod([Animal class], selector);
Method swizzledMethod = class_getInstanceMethod([Plant class], selector);

method_exchangeImplementations(originalMethod, swizzledMethod);

// ObjC
NSLog(@"I am %@!", name); // I am Plant!

// Swift
print("I am \(name)!") // I am Animal!
```

---

![fit](Resources/animal.png)

---

## Objective-C Runtime in Swift

```swift
@objc class Animal: NSObject {
    let name = "Animal"
    override init() {
        super.init()
    }
}
```

```objectivec
@interface Plant: NSObject
@end

@implementation Plant

- (NSString *)name {
    return @"Plant";
}

@end
``` 

---

## Objective-C Runtime in Swift

```swift
@objc class Animal: NSObject {
    dynamic let name = "Animal"
    override init() {
        super.init()
    }
}
```

```objectivec
@interface Plant: NSObject
@end

@implementation Plant

- (NSString *)name {
    return @"Plant";
}

@end
``` 

---

## Objective-C Runtime in Swift

```objectivec
// ObjC
SEL selector = @selector(name);

Method originalMethod = class_getInstanceMethod([Animal class], selector);
Method swizzledMethod = class_getInstanceMethod([Plant class], selector);

method_exchangeImplementations(originalMethod, swizzledMethod);

// ObjC
NSLog(@"I am %@!", name);

// Swift
print("I am \(name)!")
```

---

## Objective-C Runtime in Swift

```objectivec
// ObjC
SEL selector = @selector(name);

Method originalMethod = class_getInstanceMethod([Animal class], selector);
Method swizzledMethod = class_getInstanceMethod([Plant class], selector);

method_exchangeImplementations(originalMethod, swizzledMethod);

// ObjC
NSLog(@"I am %@!", name); // I am Plant!

// Swift
print("I am \(name)!") // I am Plant!
```

---

![inline](Resources/plant.jpg)

# Yes, this is dog!

---

## Swift <-> ObjC

- You don't need this!
- Migrate to Swift!
- OpenGL Graphics
- Interacting with C++
- System calls (server side ftw!)
- Super-über-high-performance

---

# Thank you!
## Questions?

#### github.com/nlutsenko/swift-objc-interoperability
#### facebook.com/nsunimplemented
#### twitter.com/nlutsenko
#### github.com/nlutsenko
