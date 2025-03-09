# Variable 承值容器

## Variable 承值容器（可更换值的）

\#AppleScript: Variables are what we use to store objects when we aren’t working with them. Think of a variable as a box. Whenever a script has an object that it will need to use again later, it can put it in this box for safe storage while it goes off and does other things. Later on, when it needs to use that object again, it can retrieve it just by finding the box it put it in earlier. To make these storage boxes easy to identify, each one has its own name, or identifier.

WHL: If a variable is like container, say a bottle, I think, this bottle has two info on it: a name of the value, and a memory address of the value. Right?

GitHub Copilot: Yes, you are correct. In most programming languages, including C and Ruby, a variable can be thought of as a container that holds a value in memory.

So in your analogy, the bottle can be thought of as a container that holds a value, and it has two pieces of information associated with it: the name of the value (which is like the variable name), and the memory address where the value is stored (which is like the location of the bottle).

## Instance Variable 實例變量 / 某實體中的一個值容器

Ruby objects/instances come with their own storage and retrieval mechanisms for values: instance variables. The instance variable enables individual objects to remember state.

Instance variables are only visible to the instance to which they belong.

An instance variable initialised in one method inside a class can be used by any instance method defined within that class.

Instance variable names always start with a single @ (at sign).

An instance variable retains the value assigned to it even after the method in which it was initialised has terminated. This property of instance variables - their survival across method calls — makes them suitable for maintaining state in an object.

You can change any instance variable’s value in any method.

### `initialize`

The `initialize` method is used to define instance variables. The `initialize` method is called when a new instance of the class is created.&#x20;

```ruby
class MyClass
    def initialize(instance_variable)
        @instance_variable = instance_variable
    end
end
```

##

## Class Variables

<mark style="background-color:orange;">A class variable must be given a value when it is first declared</mark>.

GitHub Copilot: <mark style="background-color:orange;">Class variables are typically defined at the class level, outside of any methods</mark>.&#x20;

```ruby
class MyClass
    @@class_variable = "Hello, world!"
end
```



GitHub Copilot:

In Ruby, an instance variable defined outside of the `initialize` method is not a class instance variable, but rather an instance variable that is shared by all methods of the class. This is in contrast to class instance variables, which are defined at the class level and are shared by all instances of the class.

Here's an example of how you could define an instance variable and a class instance variable in Ruby:

class MyClass

@class\_instance\_variable = "Hello, world!"

def initialize(instance\_variable)

@instance\_variable = instance\_variable

end

def say\_hello

puts @class\_instance\_variable

puts @instance\_variable

end

end

\# Create two instances of the class

my\_instance\_1 = MyClass.new("Instance 1")

my\_instance\_2 = MyClass.new("Instance 2")

\# Call the method on each instance

my\_instance\_1.say\_hello # Output: "Hello, world!" and "Instance 1"

my\_instance\_2.say\_hello # Output: "Hello, world!" and "Instance 2"

So to answer your question, an instance variable defined outside of the initialize method is not a class instance variable, but rather an instance variable that is shared by all methods of the class. An instance variable defined inside the initialize method is a regular instance variable that is unique to each instance of the class.

GitHub Copilot: In Ruby, an instance variable defined outside of the initialize method is not a class instance variable, but rather an instance variable that is shared by all methods of the class. This is in contrast to class instance variables, which are defined at the class level and are shared by all instances of the class.

Here's a table that compares class variables, instance variables, and class instance variables in Ruby:

<table data-header-hidden><thead><tr><th width="113"></th><th width="215"></th><th width="208"></th><th width="127"></th><th></th></tr></thead><tbody><tr><td>Variable Type</td><td>Definition</td><td>Scope</td><td>Shared by Subclasses</td><td>Prefix</td></tr><tr><td>Class<br>Variable</td><td>A variable that is shared by all instances of a class and its subclasses.</td><td>Defined at the class level, outside of any methods.</td><td>Yes</td><td>@@</td></tr><tr><td>Instance Variable</td><td>A variable that is unique to each instance of a class.</td><td>Defined within methods, such as the initialize method.</td><td>No</td><td>@</td></tr><tr><td>Class<br>Instance Variable</td><td>A variable that is shared by all instances of a class, but is not shared by subclasses of the class.</td><td>Defined at the class level, outside of any methods.</td><td>No</td><td>@</td></tr></tbody></table>

