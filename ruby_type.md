# Type

## Ruby's Approach to Types

Ruby is a dynamically typed language, meaning that it does not require or enforce type annotations in its syntax. Variables in Ruby can hold references to objects of any type, and the type of a variable can change over its lifetime. This provides a great deal of flexibility and is a hallmark of Ruby's design philosophy, focusing on simplicity and productivity.

==Ruby does not have syntax for types.==

```ruby
Project.find params[:id] # Project is a constant, not a type.
```



****

### Type 形態 / 形式 / 模樣 / 形狀

我認為 type 翻譯成「形態」/「型態」比較合適，甚至叫「形狀」或「模樣」都很好。因為一個 type 確實表現的是一段代碼的構成樣式、形狀、構造，如果這種樣式經常被看到，就可以抽象出來，並起一個名字，也就是稱之為「某型代碼」或「某種形態代碼」。這跟虎形、鶴形；可愛型、性感型，其實是一個道理。

形，外形、姿態、容貌、樣式等意，例如：形勢、形色、形式等。型，器物的模樣，引申為某種標準，型式、典範、態度、風格之意。例如：模型、典型。二字意思相近，有字典謂兩字相通。所以，凡與外貌有關的，用「形」字；凡人工鑄造出來的，或引申事物表現的格調，用「型」字。

英文的「形」為shape；英文的「型」為type、style。

In any programming language there are two main categories of types: primitive and composite.

Primitive types are atomic, i.e., they are not formed by the combination of other types. 基本的型態

Composite types are made of other types, either primitive or composite. 複合的型態

A data type is the type of data (value) a variable or constant can store in it.

Most programming languages deal with typed values, i.e., integers, booleans, vehicles, etc.

There are however, programming languages that have no types at all.

BCPL has only one data type, a word. Different operations treat each word as a different type.

A type is, in essence, a set of values. Some of these sets have a finite number of elements, while others are infinite.

For instance, the boolean data type is a set with two elements: true and false; however, the string data type is a set with an infinite number of elements.

Not every operation can be applied on every data type.

Types exist so that developers can represent entities from the real world in their programs. However, types are not the entities that they represent.

Types prevent the program from entering into undefined states.

Types are a form of documentation that the compiler can check.

Types classify programming languages along three most important dimensions: Statically vs Dynamically typed; Strongly vs Weakly typed; Structurally vs Nominally typed.

### Value Types

A value type is a type whose value is copied when it’s assigned to a variable or constant, or when it’s passed to a function.

### Reference Types

Unlike value types, reference types are not copied when they are assigned to a variable or constant, or when they are passed to a function. Rather than a copy, a reference to the same existing instance is used.

==Classes are reference types.==

#### Size of a Data Type ( a data's type / a value's type :-))

The size of a data type specifies the size of data that can be stored in a given variable or constant.

A Type’s size is measured in terms of bit and can store values up to 2 bits.

For a n bit system, it can store a maximum of 2n values in it.\


Think of bit as a value that is either 0 or 1.

## Number

In Ruby, numbers are objects.

Keep in mind that most of the arithmetic operators you see in Ruby are methods. They don’t look that way because of the operator-like syntactic sugar that Ruby gives them.

If you define, say, a method called + in a class of your own, you can use the operator’s syntactic sugar.\
And if you see arithmetic operators behaving weirdly, it may be that someone has redefined their underlying methods.

### 5 is an integer and an object. How can that be? With a language background of Java or Rust, it's hard to understand.

The idea that everything is an object in Ruby, including primitives like integers or booleans, can be a little challenging to understand if you're coming from a language like Java or Rust where primitives are not objects.

In languages like Java or Rust, primitives such as integers or booleans are not objects, they're basic data types with a fixed size in memory and a specific set of operations that can be performed on them. They don't have methods or properties like objects do.

Ruby, however, takes a different approach. In Ruby, everything is an object. This includes integers, strings, arrays, and even classes and modules themselves. Each of these objects is an instance of a class, which means it has a set of methods it can call and it can have instance variables to hold state.

In this way, Ruby simplifies its model of the world by treating everything as an object, each with its own set of behaviours and data. This allows for a consistent way of interacting with all elements in the language and makes the language more flexible and expressive.

An integer in Ruby is an instance of the Integer class, which means it has methods that can be called on it, like `even?` to check if the number is even, or `times` to repeat an action a certain number of times.

```ruby
5.times do |i|
	p "We are at number #{i}!"
end
```

## String



```ruby
# frozen_string_literal: true
```

This line is a magic comment in Ruby. It tells Ruby to freeze all string literals in the file. Freezing a string makes it immutable, meaning it can't be modified.

