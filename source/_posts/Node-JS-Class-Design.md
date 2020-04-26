title: "Node JS OOP - Class Design"
date: 2015-03-02 22:45:29
tags:
- Node
- Javascript
- OOP
- Basic
categories:
- Programming
---

Class design in Node JS is pretty interesting. There is no dedicated syntax to declare a class. Unlike Java, C#, and other static-typed language, there is no identifier like _class_, _extends_, _static_, or _private_ which make object abstraction easier to create and understand.

Javascript implements OOP using prototype-based paradigm. As everything in javascript is an object, you can declare an object's behaviors (class methods and attributes) and reuse it later.

<!-- more -->

Let's make an example. Imagine we have a __Person__ class which have _firstName_ and _lastName_ attributes. Each Person instance could _sayHello_ and _revealIdentity_.

Below is an example of __Person__ class implementation inside __person.js__:

``` javascript
// Constructor
var Person = function(firstName, lastName){
    // Attributes
    this.firstName = firstName;
    this.lastName = lastName;
    console.log("%s %s is born in the world!", firstName, lastname);
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

Let's play around with __Person__ class inside __main.js__.

``` javascript
// Import Person Class.
var Person = require("./person");

// Create two Person instances
var firstPerson = new Person("Joe", "Melon");
var secondPerson = new Person("Mela", "Latina");

firstPerson.sayHello();
firstPerson.revealIdentity();

// Changes species static attribute
Person.species = "Homo erectus";

firstPerson.revealIdentity();
secondPerson.revealIdentity();

// All instances have the same prototype function
var helloFunction = secondPerson.sayHello

console.log(firstPerson.sayHello === helloFunction); // true

helloFunction.call(firstPerson);
helloFunction.apply(secondPerson);
```

To run the example above:

``` bash
node main.js
```

In the example, we create two Person instances by calling __new Person__. As we supply different name, each instance's _sayHello_ and _revealIdentity_ will yield different output.

We declare static attribute by putting new variable inside our class object. You can refer the static attribute inside class implementation by calling __Person.species__. When __species__ value is changed, _revealIdentity_ method will give different output. This affect all Person's instances.

The last part of code tell us about class method type across different instances. All Person intances are proven to have the same _sayHello_. Thus we are confident that all Person instances have uniform behavior.

As a side note, you can call class method using different pattern. Instead of, __Instance__.__method__, you can use __method__.call(__Instance__) or __method__.apply(__Instance__). In method execution, the first argument (which is __Instance__) will replace __this__ context. This trick will be useful for inheritance concept later.

Wonderful references:

- [Mozilla Developer Network (MDN) -- Introduction to Object-Oriented JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Introduction_to_Object-Oriented_JavaScript#Prototype-based_programming)
- [Mixu's Blog -- Essential Node JS Patterns and Snippets](http://blog.mixu.net/2011/02/02/essential-node-js-patterns-and-snippets/)
