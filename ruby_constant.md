# Constant

Constants are used for values that aren’t supposed to change, yet Ruby doesn’t prevent you from changing them.

Constants in Ruby are much more than just fixed values. ==Ruby's use of constants is a fundamental aspect of its design and operation. They are an integral part of the language's architecture==, contributing to its organisation, flexibility, and dynamic capabilities.

==In Ruby, classes and modules are constants.== This means ==when you define a class like `class MyClass; end`, `MyClass` is actually a constant that holds a reference to the class object.== This is a key aspect of Ruby's object model. The constant becomes the class name. Array, String and Hash are constants because the first letter is uppercase. 

The most likely reason that you are seeing an “uninitialized constant” error is that you forgot to `require` a file or gem that defines the constant, or maybe you just misspelled the name of the constant.

**Immutable Nature**: Constants in Ruby are supposed to be immutable. (While Ruby allows reassignment of constants, it will generate a warning.) This immutability is essential for maintaining consistent references throughout a program.

**Reflection and Metaprogramming:** Ruby's dynamic nature is partly due to its handling of constants. Through reflection, you can programmatically list, access, and modify constants, classes, and modules. This is a powerful feature used in metaprogramming.

**Constant Lookup:** Ruby has a sophisticated constant lookup mechanism. When you reference a constant, Ruby searches for it in the current module, then in its ancestors (modules included into the current module or class), and finally in the global scope (`Object`). This lookup path supports Ruby's flexible and dynamic module and class composition.

==**Ruby's standard library and core classes** (like `Array`, `Hash`， `String`, etc.) are all organised as constants within the Ruby interpreter.== This makes them readily available to Ruby programs without requiring explicit imports.

**Lazy Loading and Memory Management**: By using constants and autoload, Ruby can manage memory more efficiently, loading only the necessary parts of the standard library or gems when they are actually needed.

**Conventions Over Configuration**: Ruby, and particularly frameworks like Ruby on Rails, use constants and their naming conventions to provide a lot of functionality automatically (like class loading in Rails), which adheres to the principle of "Convention Over Configuration."

**Extensibility**: The constant-based architecture of Ruby makes it easy to extend or modify existing classes and modules. This is commonly used in Ruby on Rails with monkey patching and opening up classes.

## Constants can only be defined outside of methods.

A constant cannot be defined inside a method, otherwise a "dynamic constant assignment" error is shown and it's considered as a syntax error by the Ruby interpreter.

```ruby
def a_method
     ABC = 1
end
# "dynamic constant assignment"
```

A constant of a class is accessed using ``ClassName::CONST``.

A constant is a type of variable which always starts with a capital letter. 

Run ``puts Cla`` in IRB, an error "``uninitialized constant Cla (NameError)``" will show.

## Constant Scope

When you create a constant outside of any class, at the top-level of your code, that constant will be available anywhere. Constants are also available in child classes. Constants defined outside a nested module or class are also available inside the nested classes. 

Constants from mixed-in modules are also available. Notice that this works when including the module, it won’t work if you are extending it. When you use a method that has been included from a module, Ruby will look for constants starting where that method is defined.

## Constants could but shouldn't be changed

There is no way to prevent a constant from changing because variables in Ruby are not containers, they are simply pointers or labels to objects. 

The best you can do is to use immutable objects.

```ruby
AUTHOR = "Jesus Castello".freeze
AUTHOR << "o"
# RuntimeError: can't modify frozen String
```

In this example, you can still change what the `AUTHOR` constant is pointing to, the only thing `freeze` protects you from is from changing the object itself.

**Related article**: [Ruby mutability & the freeze method](https://www.rubyguides.com/2016/01/ruby-mutability/).

When a constant is changed, a warning will will be raised:

```ruby
warning: already initialized constant ABC
```

## Constant Methods

There are a few methods dedicated to working with constants:

| METHOD               | DESCRIPTION                                                  |
| :------------------- | :----------------------------------------------------------- |
| **constants**        | Returns an array of symbols that represent the constants defined in a class |
| **const_get**        | Returns value for a constant. Takes a symbol or string as parameter |
| **const_set**        | Sets the value for a constant.                               |
| **const_missing**    | Same as `method_missing` but for constants                   |
| **const_defined?**   | Returns true if a given constant (as a symbol) has been defined |
| **remove_const**     | Removes a constant                                           |
| **private_constant** | Makes a constant unaccessable outside the class with `Class::ABC` syntax |

