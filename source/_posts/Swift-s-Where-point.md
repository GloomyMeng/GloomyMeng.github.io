---
title: Swift's 'Where' point
date: 2019-02-25 10:29:56
tags: 
    - Swift
    - Objc
categories: iOS
keywords: 
    - Swift
---

I found it's different for call method in extension with **Where**
code is here 

```
// A basic protocol
protocol ProtocolC: class { }

// A protocol extension basic protocol
protocol ProtocolDForSomeClassA: ProtocolC { }

// A work class's super class
class SomeClassBaseClass { }
// A work class
class SomeClassA: SomeClassBaseClass, ProtocolDForSomeClassA { }

// A tool class for control work class with protocol
class InstanceD<T: ProtocolC> {
    
    // a instance method for call other method in extension
    func callMethodD() { self.methodD() }
    // a class method for call other method in extension
    class func callClassMethodD() { self.classMethodD() }
}

// Add methods in extension
extension InstanceD {  func methodD() { print("methodD-default") } }
extension InstanceD {  class func classMethodD() { print("classMethodD-default") } }

// Add methods in extension with Where for check protocol
extension InstanceD where T: SomeClassBaseClass { func methodD() { print("methodD-someClassBaseClass") } }
extension InstanceD where T: SomeClassBaseClass { class func classMethodD() { print("classMethodD-someClassBaseClass") } }

// Direct call extension methods, result is what i wanted
InstanceD<SomeClassA>.init().methodD()      //  result:  methodD-someClassBaseClass
InstanceD<SomeClassA>.classMethodD()        //  result:  classMethodD-someClassBaseClass

// Call extension methods for instance methods, result isn't what i wanted
InstanceD<SomeClassA>.init().callMethodD()  //  result:  methodD-default
InstanceD<SomeClassA>.callClassMethodD()    //  result:  classMethodD-default
```

---

it seem like when i call **callMethodD** and **callMethodD** call extension methods. it won't found extension with **Where** .
any one can tell me where?

---

i found a way to achieve what i wanted. add this code i will got what i wanted. but maybe not very good,(sad

```
// Add methods in extension with Where for check protocol
extension InstanceD where T: SomeClassBaseClass {
    // a instance method for call other method in extension
    func callMethodD() { self.methodD() }
    // a class method for call other method in extension
    class func callClassMethodD() { self.classMethodD() }
}
```