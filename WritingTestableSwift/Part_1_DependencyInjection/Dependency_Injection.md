# Part 1: Dependency Injection
Let’s say we work at a robot factory. Our boss asks us to create a class for a robot. She says that the robot needs a wire, an antenna, and a power supply in order to function, and we can’t change out parts once it’s made.

``` swift

class Robot {
	let wire: Wire = Wire(color: .red)
	let antenna: Antenna = Antenna(length: 3)
	let powerSupply: PowerSupply = PowerSupply(watts: 300)
}

let robot = Robot()

```

Great! This meets all of our boss’s requirements.

So we bring this to our boss and we say “Boss, here you go. The robot will have these gorgeous red wires, the antenna will be 3 meters, and it will have a 300 watt power supply. Everyone’s going to love this.”

Our boss replies, “Okay, but we need to be able to build all different kinds of robots. Some of our customers like green wires and 5,000 watt power supplies.”

Dang. So back to the drawing board. How can we dynamically create this robot’s parts when it’s being manufactured? Easy! We’ll use an initializer that takes in the custom values for the parts:

``` swift

class Robot {

	let wire: Wire
	let antenna: Antenna
	let powerSupply: PowerSupply

	init(wireColor: UIColor, antennaLength: Double, powerSupplyWattage: Int) {
		self.wire = Wire(color: wireColor)
		self.antenna = Antenna(length: antennaLength)
		self.powerSupply = PowerSupply(watts: powerSupplyWattage)
	}
}

let robot = Robot(wireColor: .red, antennaLength: 3, powerSupplyWattage: 300)

```

Alright, looking good! We run to our boss’s office and pronounce, “Look at this. Now you can make _any_ kind of robot that you want.”

Our boss looks slightly amused as she says, “Good, but we actually just decided to move our parts manufacturing overseas so our robot needs to be able to use the parts it’s provided”

Agh! We sulk out of her office, thinking about how to handle this new constraint. While walking down the hallway, we run into a friend who works in the robot testing department. We tell him about the issues that we’re having with this robot class, then we explain the solutions we’ve come up with so far and why they won’t work anymore. 

“Why don’t you use **dependency injection**?”, our friend asks.

“Dependsiwhatsit?”, we reply. Our friend quotes James Shore,

> "Dependency injection means giving an object its instance variables. Really. That's it."  
> 		- James Shore  

That’s it! We run back to our computer and quickly come up with a solution using dependency injection:

``` swift

class Robot {

	let wire: Wire
	let antenna: Antenna
	let powerSupply: PowerSupply

	init(wire: Wire, antenna: Antenna, powerSupply: PowerSupply) {
		self.wire = wire
		self.antenna = antenna
		self.powerSupply = powerSupply
	}
}

let wire = Wire(color: .red)
let antenna = Antenna(length: 3)
let powerSupply = PowerSupply(watts: 300)
let robot = Robot(wire: wire, antenna: antenna, powerSupply: powerSupply)

```

Dependency Injection is very, very simple in theory. Implementing it can take some practice however.

Luckily, in our example, it’s actually _simpler to use dependency injection_. To see how, let’s look at how our instance variables were assigned in our first implementation:

``` swift
let wire: Wire = Wire(color: .red)
let antenna: Antenna = Antenna(length: 3)
let powerSupply: PowerSupply = PowerSupply(watts: 300)
```

Look at all of that complexity and specificity. And what did it get us? One kind of robot. Let’s look at how our instance variables were assigned in our second implementation:

``` swift
self.wire = Wire(color: wireColor)
self.antenna = Antenna(length: antennaLength)
self.powerSupply = PowerSupply(watts: powerSupplyWattage)
```

Better, but look at what is still happening - our robot knows how to create its own parts. This means that our robot **depends** on the implementation of Wire, Antenna and PowerSupply. If the initializer for any one of those types changed, we would end up having to also change the implementation of robot’s initializer. That doesn’t seem right, does it?

Now let’s look at how our instance variables are set when using dependency injection:

``` swift
self.wire = wire
self.antenna = antenna
self.powerSupply = powerSupply
```

It’s stupidly simple.

Now you might be asking, “Where’s the implementation for Wire, Antenna, and PowerSupply? How do we know this is really so simple without knowing how these other components are created?”

This is the greatest part about dependency injection: _we don’t care_. We don’t care how Wire, Antenna, PowerSupply, or any other dependency of robot is created. We’re robot engineers; our **only** job is to create a robot class. All we care about is that we get our parts when we need them (at initialization time).

Let’s point out some of the big advantages of dependency injection:

## Transparency
The responsibilities and requirements (dependencies) of an object become much more clear. By seeing what our objects need to function, and **only** what our objects need to function, we can more easily assume what our objects do.

## Separation of Concerns
We want our objects to only know what they absolutely need to know in order to function. Did our robot need to know how to create its parts in order to function? As we saw using dependency injection, no it did not. Even though our robot uses its parts, it’s not _responsible_ for creating those parts.

## Loose Coupling
When pieces of our software are **coupled**, they depend on eachother in order to function. Tight coupling kills reusability and maintainability. When we combine dependency injection with the use of Swift’s protocols, which we’ll see later in this series, we can achieve reduced coupling between our components. By having our objects depend on the interfaces of other components, not their implementations, we will be able to make code changes much faster when architecture or requirements change.

## Testability
This is arguably the **most important** reason to use dependency injection when writing software. The ability to test your code is _directly related_ to how your components get their dependencies. As we’ll soon see, if your objects are creating their own dependencies it can be extremely difficult to control their behavior, which means it can be extremely difficult or impossible to test their behavior.

- - - -
**Think about this:**

How would the conversations with our boss have gone had we implemented dependency injection in the first place?

These conversations aren’t completely fictional - the requirements and constraints of our projects are constantly changing and evolving. It’s our job to make sure that change is not a cause for panic. Using dependency injection is the first step towards being _certain_ and _confident_ in the changes we make to our code.
