---
title: Introduction
weight: 10
menu:
  notes:
    name: Introduction
    identifier: notes-go-basics-intro
    parent: notes-go-basics
    weight: 10
---

<!-- Object Oriented Programming -->
{{< note title="Object Oriented Programming">}}
The four pillars for OOP are Abstraction, Encapsulation, Inheritance, Polymorphism.

Abstraction : Abstraction is the process of showing only essential/necessary features of an entity/object to the outside world and hide the other irrelevant information. For example to open your TV we only have a power button, It is not required to understand how infra-red waves are getting generated in TV remote control.

Encapsulation : Encapsulation means wrapping up data and member function (Method) together into a single unit i.e. class. Encapsulation automatically achieve the concept of data hiding providing security to data by making the variable as private and expose the property to access the private data which would be public.

Inheritance : The ability of creating a new class from an existing class. Inheritance is when an object acquires the property of another object. Inheritance allows a class (subclass) to acquire the properties and behavior of another class (super-class). It helps to reuse, customize and enhance the existing code. So it helps to write a code accurately and reduce the development time.

Polymorphism: Polymorphism is derived from 2 Greek words: poly and morphs. The word "poly" means many and "morphs" means forms. So polymorphism means "many forms". A subclass can define its own unique behavior and still share the same functionalities or behavior of its parent/base class. A subclass can have their own behavior and share some of its behavior from its parent class not the other way around. A parent class cannot have the behavior of its subclass.
{{< /note >}}

<!-- A Sample Program -->
{{< note title="Hello World">}}
A sample go program is show here.
  
```go
package main

import "fmt"

func main() {
  message := greetMe("world")
  fmt.Println(message)
}

func greetMe(name string) string {
  return "Hello, " + name + "!"
}
```

Run the program as below:

```bash
$ go run hello.go
```
{{< /note >}}

<!-- Declaring Variables -->

{{< note title="Variables" >}}
**Normal Declaration:**
```go
var msg string
msg = "Hello"
```

---

**Shortcut:**
```go
msg := "Hello"
```
{{< /note >}}


<!-- Declaring Constants -->

{{< note title="Constants" >}}
```go
const Phi = 1.618
```
{{< /note >}}