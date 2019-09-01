# Automatic Reference Counting
*ARC will not deallocate an instance as long as at least one active reference to that instance still exists*

---
## How ARC Works
To make sure that instances don't disappear while they are still needed, ARC tracks how many properties, constants and variables are currently referring to each class instance.
The **reference** is called a **"Strong"** reference because it
- keeps a firm hold on that instance
- doesn't now allow it to be deallocated for as long as that strong reference remains.

---
## ARC in Action
*Example*
```Swift
class Person {
    let name: String
    
    init(name: String) {
        self.name = name
        print("\(name) is being initialized")
    }
    
    deinit {
        print("\(name) is being deinitialized")
    }
}
```

```Swift
var reference1: Person?     // init with nil
var reference2: Person?     // init with nil
var reference3: Person?     // init with nil
```
Three variables 
- are initialized with `nil`
- do not currently reference a `Person` instance


```Swift
reference1 = Person(name: "John Appleseed")
```
`John Appleseed is being initialized` is printed
`Person` class's initialization has taken place.

The new `Person` instance has heen assigned to the `reference1` variable
A **strong** reference from `reference1` to the new `Person` instance.
- ARC makes sure that this `Person` is kept in memory & is not deallocated

```Swift
reference2 = reference1
reference3 = reference1
```
- Two more strong references are established.

```Swift
reference1 = nil        // 2 Strong reference still remains
reference2 = nil        // 1 String reference still remains
// Person instance is not deallocated
reference3 = nil     // It's clear that you're no longer using the Person instance
```
'John Appleseed is being deinitialized'

---
## Strong Reference Cycles
> It's possible that an instance of class **never** has zero strong references.
This can happen if **two class instances** hold a strong reference to each other.
This is known as a **Strong reference cycle**

#### How to resolve strong reference cycle?
defining some of the relationship between classes as one of the below instead strong
- weak
- unowned

*Example*
```Swift
class Person {
    let name: String
    var apartment: Apartment?       // initially nil
    
    init(name: String) { self.name = name }
    deinit { print("\(name) is being deinitialized") }
}

class Apartment {
    let unit: String
    var tenant: Person?             // initially nil
    
    init(unit: String) { self.unit = unit }
    deinit { print("Apartment \(unit) is being deinitialized") }
}
```

```Swift
var john: Person?       // nil
var unit4A: Apartment?  // nil

john = Person(name: "John Appleseed")   // assign Person instance
unit4A = Apertment(unit: "4A")          // assign Apartment instance
```


| john | | unit4A |
| ----- | ----- | ----- |
| **strong** | | **strong** |
| <**Person** instance> | | <**Apartment** instance> |
| **name**: "John Appleseed" | | **unit**: "4A" |
| **apartment**: nil | | **tenant**: nil |

```Swift
john!.apartment = unit4A
unit4A!.tenant = john
```
| john | | unit4A |
| ----- | ----- | ----- |
| **strong** | | **strong** |
| <**Person** instance> | | <**Apartment** instance> |
| **name**: "John Appleseed" | **-- strong -- >** | **unit**: "4A" |
| **apartment**: <Apartment instance> | **< -- strong --** | **tenant**: <Person instance> |

This is a **Strong Reference Cycle**.
When you break the strong references held by the `john` and `unit4A` variables, the reference counts do NOT drop to zero & the instances are NOT deallocated.  It causes a **memory leak** in app.
```Swift
john = nil      // deinitializer wasn't called
unit4A = nil    // deinitializer wasn't called
```
| john | | unit4A |
| ----- | ----- | ----- |
|  | |  |
| <**Person** instance> | | <**Apartment** instance> |
| **name**: "John Appleseed" | **-- strong -- >** | **unit**: "4A" |
| **apartment**: <Apartment instance> | **< -- strong --** | **tenant**: <Person instance> |

### Resolving Strong Reference Cycle
1. Weak: 
    Use when the other instance has a shorter lifetime(= can be deallocated first)
2. Unowned:
    Use when the other instance has the same lifetime or a longer.

#### Weak References
Indicating: Place the `weak` before a property or variable declaration.
ARC automatically sets a weak reference to `nil` when the refered instance is deallocated.  And, because weak references need to allow their value to be changed to `nil` at runtime, they are always declared as *optional variables*, NOT constants.

> **NOTE**
Property observers aren't called when ARC sets a weak reference to `nil`

```Swift
class Person {
    let name: String
    var apartment: Apartment?
    
    init(name: String) { self.name = name }
    deinit { print("\(self.name) is being deinitialized")
}

class Apartment {
    let unit: String
    weak var tenant: Person?    // Weak Reference
    
    init(unit: String) { self.unit = unit }
    deinit { print("Apartment \(self.unit) is being deinitialized")
}
```
Create instances as same as before
```Swift
var john: Person?
var unit4A: Apartment?

john = Person(name: "John Appleseed")
unit4A = Apartment(unit: "4A")

john!.apartment = unit4A
unit4A!.tenant = john
```
| john | | unit4A |
| ----- | ----- | ----- |
| **strong** | | **strong** |
| <**Person** instance> | | <**Apartment** instance> |
| **name**: "John Appleseed" | **-- strong -- >** | **unit**: "4A" |
| **apartment**: <Apartment instance> | **< -- weak --** | **tenant**: <Person instance> |

`Apartment` instance now has a `weak` reference to the `Person` instance.
When set the `john` to `nil` that breaks the string reference held by the `john`, there are no more strong references to the `Person` instance:
```Swift
john = nil
// Prints "John Appleseed is being deinitialized"
```
> No more strong references to the `Person` instance, it is deallocated & `tenant = nil` automatically

| john | | unit4A |
| ----- | ----- | ----- |
| **strong** | | **strong** |
| <**Person** instance> | | <**Apartment** instance> |
| **name**: "John Appleseed" |  | **unit**: "4A" |
| **apartment**: <Apartment instance> |  | **tenant**: <Person instance> |

```Swift
unit4A = nil
// Prints "Apartment 4A is being deinitialized"
```
> No more strong references to the `Apartment` instance, it is deallocated too.

| john | | unit4A |
| ----- | ----- | ----- |
|  | |  |
| ~~<**Person** instance>~~ | | ~~<**Apartment** instance>~~ |
| ~~**name**: "John Appleseed"~~ |  | ~~**unit**: "4A"~~ |
| ~~**apartment**: <Apartment instance>~~ |  | ~~**tenant**: <Person instance>~~ |

> **NOTE**
In systems that use garbage collection, weak pointers are sometimes used to implement a simple caching mechanism because objects with no strong references are deallocated only when memory pressure triggers garbage collection. However, with ARC, values are deallocated as soon as their last strong reference is removed, making weak references unsuitable for such a purpose.

---

## Unowned References

Unowned reference doesn't keep a strong hold on the instance it referes to.
**Unlike** a week reference an unowned reference is used when the other instance has the same lifetime or a longer lifetime.

Unowned references always have a **value**, so ARC never sets its value to `nil` (non-optional types)


> **Important:** Use an unowned reference only when you sure that the reference **always** refers to an instance that has **never been deallocated**

```Swift
class Customer {
	let name: String
	var card: CreditCard?
	init(name: String) {
		self.name = name
	}
	deinit { print("\(name) is being deinitialized") }
}

class CreditCard {
	let number: UInt64
	unowned let customer: Customer
	init(number: UInt64, customer: Customer) {
		self.number = number
		self.custimer = customer
	}
	deinit { print("Card #\(number) is being deinitialized") }
```

> Note:
> `CreditCard.number` is defined with a type of `UInt64` rather than `Int` to ensure that the `number` capacity is large enough to store a 16-digit card # on both 32bit & 64bit systems.

```Swift
var john: Customer?     // john = nil

john = Customer(name: "John Appleseed")
john!.card = CreditCard(number: 1234_5678_9012_3456, customer: john!)
```

| john | |  |
| ----- | ----- | ----- |
| Strong | |  |
| <**Customer** instance> | | <**CreditCard** instance> |
| **name**: "John Appleseed" | -- Strong --> | **number**: 1234_5678_9012_3456 |
| **card**: <CreaditCard instance> | <-- unowned -- | **customer**: <Customer instance> |

```Swift
john = nil
// "John Appleseed is being deinitialized"
// "Card #1234567890123456 is being deinitialized"
```

Because there are no more strong references to the Customer instance, it's deallocated. After then, there are **no more strong references to the CreditCard instance**, and it's deallocated too.

> `unowned(unsafe)`:
If you try to access an unsafe unowned reference after the instance that it refers to is deallocated, your porgram will try to access the memory location where the instance used to be, which is an unsafe opeeration.

---
## Unowned References and Implicitly Unwrapped Optional Properties

`Person` & `Apartment` shows a situation where two properties have the potential to cuase a strong reference cycle: both allowed to be 'nil'

`Customer` & `CreditCard` shows a situation where two properties have the potential to cuase a strong reference cycle: one property allowed to be `nil` and another property *never*

Third Scenario:
Both properties should always have a value and never be `nil` once initailization is complete.
Solution: It is useful to combine an `unwoned` property on one class with an `!` property on the other class.
Both properties enble to be accessed directly once initialization is complete, while still avoiding the cycle.

**Interdependency**
```Swift
class Country {
    let name: String
    var capitalCity: City!
    init(name: String, capitalName: String) {
        self.name = name
        self.capitalCity = City(name: capitalName, country: self)
    }
}

class City {
    let name: String
    unowned let country: Country
    
    init(name: String, country: Country) {
        self.name = name
        self.country = country
    }
}
```

To set up the interdependency between the 2 classes
- the initializer for `City` takes a `Country` instance
`City.init(country: Country)`
- stores the `Country` instance in its `Country` property
`self.country = country`

`City.init()` is called from within `Country.init()`
However, `Country.init()` can't pass `self` to the `City.init()` until a new `Country` instance is fully initialized.
`Country.init()` --> `City(country: self)`

Related Link >> [Two-Phase Initialization]("https://docs.swift.org/swift-book/LanguageGuide/Initialization.html#ID220")

Declare the  `Country.capitalCity: City!`. This means that the `capitalCity = nil` as a default like other optional, but can be accessed without the need to unwrap its value.

Because `capitalCity = nil`, a new `Country` instance is considered fully initialized as soon as the `Country` instance sets its `name` property within its initializer.
(`newCountry = Country(name: "new", capitalCity: "cap")`)
- This means that `Country.init()` can start to reference 
- Then, pass around the implicit `self` as soon as `name` is set
- Therfore `Country.init()` can pass `self` as one of the parameters for `City.init()` when `Country.init(capitalCity: )` is setting its own `capitalCity`

** You can create the `Country` and `City` instances in a single statement** without creating a strong reference cycle.
`capitalCity` can be accessed directly without needing to user `!`mark to unwrap its optional value:

```Swift
var country = Country(name: "Canada", capitalName: "Ottawa")
print("\(country.name)'s capital city is called \(country.capitalCity.name)")
// Prints "Canada's capital City is called Ottawa"
```




