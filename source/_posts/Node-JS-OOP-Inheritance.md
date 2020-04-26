title: "Node JS OOP - Inheritance"
date: 2015-03-05 22:45:29
tags:
- Node
- Javascript
- OOP
- Basic
categories:
- Programming
---

Once you create a class, you may need to extends it, i.e. add more methods, attributes, and override behaviors. In OOP, this concept is know as inheritance. Using inheritance, your code base will stay tidy, slim, and reasonable.

Inheritance introduces new terminologies such as __superclass__ and __subclass__. Subclass inherits all superclass method and attributes. By extending superclass, subclass have more property or more specific implementations.

<!-- more -->

Let’s consider __Person__ class which acts as superclass in __person.js__ file.

``` javascript
// Constructor
var Person = function(firstName, lastName){
    // Attributes
    this.firstName = firstName;
    this.lastName = lastName;
    console.log("%s %s is born in the world!", firstName, lastName);
};

// Methods
Person.prototype.sayHello = function(){
    console.log("%s say hello to you!", this.firstName);
};

Person.prototype.revealIdentity = function(){
    console.log("%s is a %s", this.firstName, Person.species);
}

// Static attribute
Person.species = "Homo Sapiens";

module.exports = Person;
```

We will create __PowerRanger__ class which acts as subclass. Everyone knows that power ranger is also a normal person. So it’s safe to assume that power ranger __could do__ what normal person able to.

But, of course, power ranger is different with normal person. They fight evil, has cool costume and megazord. File __power_ranger.js__:

``` javascript
// Import class Person
var Person = require("./person");

// Constructor
function PowerRanger(firstName, lastName, color){
    // You can call superclass's constructor
    Person.call(this, firstName, lastName);

    // New attribute
    this.color = color;
}

// Inherits superclass behaviors
PowerRanger.prototype = Object.create(Person.prototype);

// New methods
PowerRanger.prototype.fight = function(){
    console.log("%s ranger is fighting evil", this.color);
};

PowerRanger.prototype.callMegazord = function(){
    console.log("%s ranger call the mighty megazord", this.color);
};

// Override superclass methods
var personSayHello = PowerRanger.prototype.sayHello;
PowerRanger.prototype.sayHello = function(){
    personSayHello.call(this);
    console.log("As a %s power ranger!", this.color);
}

module.exports = PowerRanger;
```

If I allowed to sum up inheritance in javascript, there are 3 steps you should do to extends a superclass:

1. Chain superclass constructor
2. Copy superclass methods
3. Add or override superclass implementation

## Chain superclass constructor

You can chain superclass constructor using __call__ or __apply__ method. When __Superclass.apply__ is executed, there are two things happening:

1. Superclass’s constructor is called
2. __Call__‘s first argument will be injected. Replacing __this__ context inside caller object.

Inside Person class’s constructor, __this__ context won’t refer to Person instance anymore. Instead, it will be replaced by PowerRanger instance. PowerRanger instance then has it’s firstName and lastName attributes initialized inside Person’s constructor.

## Copy superclass methods

Subclass can inherit superclass methods by cloning it’s prototype. This is effectively done by

```javascript
PowerRanger.prototype = Object.create(Person.prototype);
```

Declared methods inside Person class will be inherited to PowerRanger. Thus, you can call sayHello and revealIdentity method from PowerRanger instance.

Warning: If you simply referring superclass prototype, instead of cloning, superclass will be similar to subclass. Subclass alteration will affect superclass too.

## Add or override superclass implementation

Adding new method in superclass is simple. Declare new method inside Superclass.prototype and voila.

Overriding superclass implementation is tricky. You need to consider if the behavior changes is total or partial. Total change means subclass implementation is completely different with superclass. While partial change means you want to retain superclass implementation and add a little bit pre-processing or post-processing

If your changes is total, redeclare __Superclass.prototype.superclass_method__ is sufficient.

Partial changes, however, require you to save superclass method first.

```javascript
var personSayHello = PowerRanger.prototype.sayHello;
```

In the example, the Person’s sayHello method is saved inside a variable called __personSayHello__. We then redeclare __Superclass.prototype.superclass_method__ and call superclass method later.

Let’s play around with PowerRanger and Person class inside main_power_ranger.js.

```javascript
var PowerRanger = require("./power_ranger")
var Person = require("./person")

var p1 = new PowerRanger("Adam", "Bodam", "White");
var p2 = new PowerRanger("Justin", "Putin", "Blue");
var p3 = new Person("Desi", "Durna");

p3.sayHello();
p1.sayHello();
p1.fight();
p2.callMegazord();

console.log(p1 instanceof PowerRanger); // true
console.log(p1 instanceof Person); // true
console.log(p3 instanceof Person); // true
console.log(p3 instanceof PowerRanger); // false

// Changing Person's static attribute also affects PowerRanger's instance
p3.revealIdentity();
p1.revealIdentity();

Person.species = "Home Genus";

p3.revealIdentity();
p1.revealIdentity();
```

To run the example above:

```shell
node main_power_ranger.js
```

That’s it guys. Feel free to give correction or ask. Hope this helps :)

Wonderful references:

- [Mozilla Developer Network (MDN) -- Introduction to Object-Oriented JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Introduction_to_Object-Oriented_JavaScript#Prototype-based_programming)
- [Mixu's Blog -- Essential Node JS Patterns and Snippets](http://blog.mixu.net/2011/02/02/essential-node-js-patterns-and-snippets/)
