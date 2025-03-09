# Class

Classes are, at heart, a way to organise objects and methods.\
==% 與其說 organise，毋寧說 define 或 design，可以說是「藍圖」。 20231117%==

Defining a class lets you group behaviours (methods) into convenient bundles, so that you can quickly create many objects that behave essentially the same way.

Everything you handle in Ruby is either an object or a construct that evaluates to an object, and every object is an instance of some class.

When you use the dot notation on a class, you send a message to the class. Classes can respond to messages.



Ruby is about objects.

Objects are instances of classes.

Classes are objects.

`Object` is a built-in Ruby class.

Classes are named with constants.

==% 定義一個 Class，就像「開模」來製造一個銅盤一樣。Class就是模具。20230624 %==



***

### Learned from Xavier Noria&#x20;

==The `class` and `module` keywords store classes and modules in constants.==

In Ruby, when you define a class or a module using the `class` or `module` keywords, they are indeed stored as constants. This is a unique aspect of Ruby's object model. For example, when you define `class MyClass; end`, `MyClass` is a constant that references the class object.

==Constants belong to classes and modules.==

Constants in Ruby are scoped within classes and modules. They are not global in the same sense as in some other languages. This means you can have different constants with the same name in different scopes without conflict. For instance, you could have `class A; MY_CONST = 1; end` and `class B; MY_CONST = 2; end` without any conflict between `A::MY_CONST` and `B::MY_CONST`.

==Top-level constants belong to `Object.==

In Ruby, top-level constants (those defined outside of any class or module) are treated as if they belong to the `Object` class. This is because `Object` is the default object context of Ruby at the top level. This means that any constant defined at the top-level is accessible from anywhere in the program, as all classes in Ruby are descendants of `Object`.

`Module#autoload` allows you to load constants on demand.

## Singleton Class

In Ruby, ==each object has its own singleton class== (also known as a metaclass or eigenclass). ==The singleton class is a subclass of the object's class.== This singleton class is automatically created by Ruby the first time you define a singleton method on the object.&#x20;

The singleton class is a core aspect of Ruby's object-oriented design, which emphasizes flexibility and expressiveness. Here are some key reasons why singleton classes are used:

1. **Implementing Class Methods**: ==In Ruby, class methods are actually instance methods of the class's singleton class.== This is because classes themselves are also objects in Ruby, so each class has its own singleton class where its class methods are stored.
2. **Individual Object Customisation**:  ==Singleton classes allow you to add methods to an individual object rather than to an entire class of objects.== Without singleton class, the only way would be to modify its class. However, modifying a class affects all instances of that class, which can lead to unexpected side effects. 
   Especially, when you want to add functionality to instances of core classes (like String, Array, etc.) without affecting all instances, singleton classes are the way to go. This is often used in Ruby on Rails and other frameworks to extend the capabilities of standard objects on a per-instance basis.
   &#x20;==% Why don't use mixin? 20231223 %==
3. **Supporting Metaprogramming**: Ruby's metaprogramming capabilities are largely facilitated by singleton classes. They allow dynamic addition of methods at runtime, making Ruby a very flexible and powerful language for metaprogramming.
4. **Method Look-Up Consistency**: Ruby maintains a consistent method look-up path. ==When you call a method on an object, Ruby first looks in the object's singleton class before looking in the object's actual class and its ancestors.==&#x20;





## `class << self`

This syntax is used to define class methods within the block. It opens up the singleton class of the current class, allowing you to define methods that will be available on the class itself, rather than on instances of the class.

Using the singleton class (`class << self`) to define class methods, groups them together makes the class more organised.

```ruby
class MyClass
    class << self
        def my_class_method
            # method implementation
        end
    end
end
```

A real-world code example from Rails:

```ruby
module ActiveRecord
    class CurrentAttributes
        class << self
            def instance
                current_instances[ current_instances_key ] ||= new
            end
            def attribute( *name )
            end
        end
    end
end

```



Using `self.` prefix, is another way, which is clear and concise, easy to read and understand.

```ruby
class MyClass
    def self.my_class_method
        # method implementation
    end
end
```



## `subclass = Class.new(self)`

`Class.new` is a method in Ruby that creates a new class.

* when you call `Class.new` without any arguments, it creates an anonymous class with `Object` as its superclass.
* when you provide an argument to `Class.new`, it creates a new class with the provided argument as its superclass.
* when you pass `self` as an argument to `Class.new`, it creates a new class that is a subclass of the current class (the one where this code is being executed).

This subclass inherits all the methods and properties of the original class but is a distinct class that can be modified independently.

This is a way to dynamically create a new subclass of the current class. This technique is useful in scenarios where you need to create classes dynamically based on runtime conditions or for advanced metaprogramming patterns.

