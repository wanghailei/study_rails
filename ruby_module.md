# Module

Modules are a way of grouping together methods, classes, and constants. Modules give you two major benefits:&#x20;

* Modules provide a namespace and prevent name clashes.&#x20;
* Modules can be included in other classes, a facility known as a mixin.

Like a class, a module name is also global constants.&#x20;

Module methods are defined like class methods, using the `def self.method_name` syntax.As with class methods, you call a module method is called by preceding its name with the module’s name and a period.

As with class methods, you call a module method by preceding its name with the module’s name and a period, like `ActiveSupport.run_load_hooks(:active_record, Base)`

Module constants are referenced using the module name followed by two colons, which is called the scope resolution operator, like Active

A module can’t have instances, because a module isn’t a class. &#x20;

You can `include` a module within a class definition. When this happens, all the module’s instance methods are suddenly available as instance methods in the class as well. This si called mixin.





***

## `Module#autoload`

==`Module#autoload` allows you to load constants on demand.==

==`Module#autoload` is a method in Ruby that allows you to defer the loading of a module or class until it is first used.== This can be beneficial for reducing the startup time of a Ruby program.

Here's how `Module#autoload` works:

1. **Definition:** You specify a constant name and the name of a file to be loaded. The constant serves as a placeholder for the module or class you want to autoload.
2. **Usage:** When the constant is first accessed in the code, Ruby automatically loads the specified file. This is done only once, the first time the constant is accessed.
3. **Syntax:** The method is used like this: `autoload :ModuleName, 'path/to/file'`. Here, `:ModuleName` is the name of the constant that will be used as a placeholder for the module or class, and `'path/to/file'` is the file path (relative or absolute) where the actual definition of the module or class is located.

Here is an example:

```ruby
module MyModule
	autoload :MySubmodule, 'path/to/my_submodule'
end
# At this point, MySubmodule is not yet loaded.
# Later in the program...
MyModule::MySubmodule.do_something # This line will trigger the loading of 'path/to/my_submodule'
```

In this example, `MySubmodule` will only be loaded when it is first accessed via `MyModule::MySubmodule.do_something`.

Ruby 3.0 introduced improvements to make constant autoloading thread-safe. ==Zeitwerk handles code loading in a more sophisticated way, making the explicit use of `autoload` less common in modern Rails applications.==



***

In Ruby, the `extend` keyword is used to add methods from a module to a single object, typically an instance of a class or the class itself.&#x20;

==When `extend` a module in a class, the module's methods are added as class methods of the target class. When `include` a module into a class, the module's methods are added ==as instance methods of the class==.



