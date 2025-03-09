# Methods

## A method is an object

In Ruby, you can accurately say that ==a method is an object==. This is because Ruby is a language with a very consistent object model and everything in Ruby is an object, including methods.

When you reference a method in Ruby using the `method` method or the `&` operator, what you get is ==an object of the class `Method`.== This object encapsulates the method and allows you to interact with it as an object, including the ability to call the method, inspect its parameters, and so on.

For example:

```ruby
class MyClass
	def say
		"Hello, world!"
	end
end

obj = MyClass.new
method_object = obj.method :say  # Retrieves the 'say' as a Method object
p method_object.class  # => Method
p method_object.call # => "Hello, world!"
```

In this code, `method_object` is an instance of the `Method` class. You can then call this method using `method_object.call`, ==pass it around as an object==, and use other `Method` class functionalities. 

==The `Method` class is particularly useful for metaprogramming==, where methods might be manipulated programmatically, or for scenarios where you need to store references to methods for later execution.

## Defining a Method

In Ruby, parentheses in method definitions are optional unless you need to clarify the precedence of parameters. You can define a method with or without them. Both of the following definitions are valid and functionally equivalent:

```ruby
def a_method(p)
	# Method body
end

def a_method p
	# Method body
end

```

### One-line method

#### the endless method

```ruby
def a_method_name(arg) = puts arg
```



```ruby
def a_method_name; end
```





## Kinds of Methods

Methods defined inside a class and intended for use by all instances of the class, are called instance methods.

Methods that you define for one particular object are called singleton methods.

A class method belongs to the class itself.

==%. 有三種 methods：== 
==Class methods：這是 class 給自己（工廠）用的；== 
==Instance methods：這是 class 安裝進生產出來的 instance（object）上的；還有== 
==Singleton methods：這是某一個 instance（object）在被生成後，自己搞來用的。== 
==20231223 %==

### What Are Class Methods For?

Why create a class method rather than the more usual instance method? 
There are two main reasons: First, a class method can ==be used as a “ready-to-run function” without having to go to the bother of creating an object just to use it==, and second, ==it can be used on those occasions when you need to run a method before an object has been created==.

In Ruby, method names can contain characters like ?, !, and =. The convention is to end method names with a ? if they return either true or false, with ! to indicate the method will have some sort of side effect like changing the object it's called on, and with = when defining setter methods.

### The splat `*` operator

In Ruby, the `*` operator, when placed before the parameter in a method definition, is called the 'splat' operator. It's primarily used to specify that the method can accept one or more arguments. This makes it useful in scenarios where you don't know in advance how many arguments will be passed into the method. So essentially, ==the `*args` parameter allows this method to take any number of arguments.==

```ruby
def my_method( *args )
	args.each { |arg| puts arg }
end
my_method("Hello", "World", "!")
# => Hello
# => World
# => !
```

### `super()`

When `super()` is used in an instance method, ==a method of the same name but in the superclass== of the current class is called. This is often used in method overrides, where you want to augment or modify the behavior of a method inherited from the superclass, not replace it entirely.

* If you use `super()` with parentheses and no arguments, it calls the superclass method without any arguments, regardless of what arguments were passed to the child method.
* If you use `super` without parentheses and arguments, it automatically forwards the arguments that were passed to the child method.

For example:

```ruby
class Animal
    def speak
        "Some generic sound"
    end
end
class Dog < Animal
    def speak
        "#{super()} and then barks"
    end
end
dog = Dog.new
puts dog.speak # => "Some generic sound and then barks"
```

super() invokes the `speak` method of `Animal`. 



### The method name started with an underscore

`def _prepare_context`&#x20;

In Ruby, a method name beginning with an underscore (`_`) is typically and just a convention used by developers as an additional visual cue to indicate that a method is intended for internal use within the class or module. It's only a naming convention and not a language feature. The underscore does not change the visibility of the method.

For example, in Rails, methods that are not intended to be actions in a controller might be prefixed with an underscore to differentiate them from action methods.

