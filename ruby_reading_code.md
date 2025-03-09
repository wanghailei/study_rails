# Reading Rails' source code



## `ensure`

```ruby
def a_method
    do_something
ensure
    do_others
end
```

The `ensure` block is used in conjunction with `begin`, `rescue`, and ==`def`</mark> blocks to guarantee that certain code runs regardless of whether an exception occurs. The ensure block is executed after the main body of the block (or method, if used within a method definition) and after any rescue blocks, if present. It's typically used for cleanup code that you want to execute no matter what, like closing files or releasing resources.

This method demonstrates a common pattern in Ruby (and specifically Ruby on Rails), where temporary changes are made to the state, and an ensure block is used to guarantee that the original state is restored, maintaining the integrity and consistency of the application state.



****

### Exception handling

In Ruby, begin, rescue, else, ensure, and end form a block of code for exception handling. 

```ruby
begin
	# risky code here that might cause an exception
rescue SomeExceptionType => e
	# rescue handling
else
	# code to run if no exception was raised
ensure
	# This code will always run, good for cleanup activities
end
```

The `rescue` clause catches exceptions of a certain type. You can have multiple rescue clauses for different types of exceptions. If an exception occurs in the begin block, the code in the rescue block gets executed.

The `else` clause is optional. If present, it goes after the `rescue` clause and its code is ==executed if no exceptions were raised in the begin block.==

The `ensure` clause is also optional. If present, its code gets ==executed every time, regardless of whether an exception was generated. This is typically used for cleanup activities.==

```ruby
# code from Homebrew
begin
     ...
rescue UsageError => e
	require "help"
	Homebrew::Help.help cmd, remaining_args: args.remaining, usage_error: e.message
rescue Exception => e # rubocop:disable Lint/RescueException
	onoe e
	$stderr.puts Utils::Backtrace.clean(e)
	exit 1
else
	exit 1 if Homebrew.failed?
ensure
	if ENV["HOMEBREW_STACKPROF"]
		StackProf.stop
	end
end
```

`rescue UsageError => e:` This line is catching an exception of type `UsageError`. This means that if a `UsageError` is raised within the surrounding block of code in `begin`, instead of crashing the program, the execution will jump to this `rescue` block. The instance of the error is captured into the variable `e` for further use.

****



`[2, 4, 6].select!( &:even? )` is equivalent to `[2, 4, 6].select! { |i| i.even? }`

The `&:even?` is shorthand for a block `{ |i| i.even? }`.

`:even?` is a method that returns `true` if an integer is even and `false` otherwise. The colon `:` before `even?` indicates that it is a symbol, namely, ==the `:` before `even?` converts the name of the method into a symbol.== So, `:even?` represents the symbol form of method `even?`. The `even?` is an instance method of the `Integer` class.

**`&` operator**: The ampersand (`&`) is a unary operator that converts the symbol to a block. When used in front of a symbol in a method call like this, it tries to call a method with the same name as the symbol on the objects being iterated over. 

==The `&` operator is used to convert the symbol `:even?` into a Proc object==, which is then treated as a block by the method it's passed to, and which is then passed to the `select!` method. In this case, it's calling `even?` on each element of the array.

Since `!` is used after `select`, it modifies the original array itself.

