# Part 2: Interfaces
Welcome back to Writing Testable Swift! In the first part of this series we covered dependency injection, a crucial part of testable code and a great principle for any codebase - even if you’re not writing tests. Today, let’s talk about how we can continue coding confidently with **interfaces**.

## Interfaces
If you’re like me, the first place your mind goes when you hear “interface” is objective-c and header files. Don’t worry! We’re not going to be creating any header files today.

In case you weren’t with us for part 1 of this series, we work at a fictional robot factory where we make make robots. Today, our boss power-walks into the office and straight to our desk, “We’ve got some clients that need a robot. This robot is going to need arms and legs.”

Easy, let’s use what we learned in the previous lesson and inject the parts this robot needs.

```swift

class Robot {
	let arms: Arms
	let legs: Legs

	init(arms: Arms, legs: Legs) {
		self.arms = arms
		self.legs = legs
	}
}

```

Before we could even finish, our boss comes back over and says, “Oh yea, and our clients have a few different kinds of arms and legs they want to test out, so make sure that this robot can use any arms that can grab or legs that can walk.”

Sure, nothing a little inheritance can’t solve. We write up the abstract base class that our clients should subclass for their own Arms and Legs:

```swift

class Arms {
	func grab() { fatalerror("Must override this method!") }
}

class Legs {
	func walk() { fatalerror("Must override this method!") }
}

```

We send this over to our boss and then lounge back in our chair, pleased with our robot-making prowess. About ten minutes later our email starts blowing up with angry messages from our clients:

- - - -

**“We’re using structures to represent our robot’s Arms, we can’t use inheritance!”**

And another…
**“Our robot’s Legs already have a superclass!”**

And another…
**”We’re using private, 3rd party robot arms and legs! We can’t change their implementation”**

And another…
**“Why is our robot crashing while screaming “Must override this method!””**

- - - -

Oh boy, what have we done?

Well, we know exactly what we did. We tried to enforce polymorphism through inheritance. How could this be wrong? This is how object-oriented programming works. Why is this such an issue?

That’s when we remember an [important moment](https://developer.apple.com/videos/play/wwdc2015/408/) in Swift’s history that happened just 3 years ago at WWDC 2015. [David Abrahams](https://daveabrahams.com), the Technical Lead on the Swift Standard Library team, got on stage at the Moscone Center in San Francisco and said, 

> “When we made Swift, we made the first protocol-oriented programming language.”  

There’s a lot of discussion around what protocol-oriented means, but I find it’s best described by another thing David said during the same talk,

> “We have a saying in Swift. Don’t start with a class. Start with a protocol”  

So let’s follow David’s advice and attack this problem again:

```swift

protocol Arms {
	func grab()
}

protocol Legs {
	func run()
}

```

When we use protocols in this way we’re creating an **interface**. Our interface is telling the rest of our program, “This is how we can communicate.” Now, when we inject Arms into the robot, all the robot knows is that the Arms can grab. That’s all the robot needs to know about its arms in order to function. It doesn’t matter what concrete implementation of Arms is actually conforming to this interface and getting passed into the robot.

Does this solve our clients’ problems? Let’s see:

**“We’re using structures to represent our robot’s Arms, we can’t use inheritance!”**
Great, structures and enums can both conform to protocols in Swift.

**“Our robot’s Legs already have a superclass!”**
All good, classes can conform to as many protocols as they want, even if they’re inheriting from another class.

**”We’re using private, 3rd party robot arms and legs! We can’t change their implementation”**
In Swift we can extend classes we don’t have access to and add protocol conformance. 

**“Why is our robot crashing while screaming “Must override this method!”**
This might seem silly, and we might even say, “That’s the client’s fault for not overriding the base class’s method!”. Do you think our boss cares if it’s the client’s fault? Let’s just remove the chance of error by using a protocol which will enforce conformance at _compile time_.

This protocol-oriented interface is very powerful for a number of reasons:

### Multiple Conformance
Unlike inheritance, a single type can conform to multiple protocols. This allows for there to be separate interfaces into a single type, giving us the power of [composition over inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance). Protocols can even inherit from other protocols, a technique we will start using in the next part of this series.

### Polymorphism
In Swift, any class, structure, or enum can conform to a protocol. This increases the types of entities that can provide an implementation for a certain interface.

### Loose Coupling
By using protocols, our code is now dependent on an interface instead of a concrete implementation. This means that our interface can mediate the coupling between implementations, which is much easier to modify than tight coupling between two implementations.

### Testability
Consider this situation:

```swift

class Controller {
	let network: Network

	init(network: Network) {
		self.network = network
	}

	func userName() -> String {
		return network.getUser().name
	}
}

class Network {
	func getUser() -> User {
		// Makes a network request for a user and returns that user
	}
}

```

What if I want to test the Controller’s userName method? I have to ask my network class to getUser(), which in turn makes a network request to my service to get that user, which will then (hopefully) return a user. Now I need to make sure I know that user’s name that’s on my server so I can make sure that’s the same name getting returned by the userName() method.

Why do I have to do all of this when I’m just trying to make sure my controller can return some arbitrary user’s name? I’m not trying to test my network!

With dependency injection and protocol-oriented interfaces this can be done pretty easily:

```swift

class Controller {
	let network: Network

	init(network: Network) {
		self.network = network
	}

	func userName() -> String {
		return network.getUser().name
	}
}

protocol Network {
	func getUser() -> User
}

struct TestNetwork: Network {
	func getUser() -> User {
		return User(name: "John")
	}
}

```

Now when I go to test my controller I just inject my TestNetwork and I know that my printUserName() method should be printing “John”.

- - - -

Swift protocols are an amazing, powerful tool that we can use to achieve outstanding results. Using them to create interfaces helps us to avoid some of the pitfalls of inheritance. When we combine dependency injection with interfaces, we can start to see our components become less coupled and more reusable. This will lead to code that we can _confidently_ change much faster. I encourage you to use interfaces when appropriate and keep working towards testable code.

