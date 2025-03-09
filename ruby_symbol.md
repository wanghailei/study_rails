# Symbol

### Symbol 象徵 / 符號 / 標誌

Symbols are immutable. Like an integer, a symbol can't be changed.

Symbols are unique. Whenever you see :xyz, you’re seeing a representation of the same object.

The :xyz notation always represents the same object.

Because symbols are unique, there’s no point having a constructor for them; Ruby has no Symbol#new method.

Underneath, symbols are more like integers than strings. They share with integers not only immutability and uniqueness, but also immediacy: a variable to which a symbol is bound provides the actual symbol value, not a reference to it.

You can consider the colon to mean thing named, so :id is the thing named id.

Why both strings and symbols exist when they seem to be duplicating each other’s efforts in representing text? Think of symbols as integer-like entities dressed up in characters.

Ruby allows a special form of symbol representation in the hash-key position, with the colon after the symbol instead of before it and the hash-separator arrow removed.

joe\_hash = { :name => "Joe", :age => 36 }

joe\_hash = { name: "Joe", age: 36 }

The above two are actually the same.

A Symbol is the most basic Ruby object you can create. It's just a name and an internal ID.

#### Why do symbols exist Ruby?

The existence of symbols in Ruby is a matter of efficiency.

In Ruby, a symbol is basically a way to represent a name inside the interpreter.

Internally, Ruby uses symbols to keep track of all the names it created for variables, methods, and constants.

When you assign a value to a variable or constant, or create a class or write a method, the identifier you choose goes into Ruby’s internal symbol table. A number of core Ruby methods take symbols as arguments.

Symbols are used because they are more efficient in terms of memory usage when compared to strings.

A string is mutable and takes up memory each time a new string is created. A symbol, on the other hand, is immutable and is stored in memory only once. This reduces memory consumption and increases performance when you use symbols to repeatedly represent the same thing.

Most methods that take symbols can also take strings.

"abc".send(:upcase)

You can replace :upcase with "upcase" in the previous send example, and it will work.

The difference is that by supplying :upcase, you’re saving Ruby the trouble of translating the string upcase to a symbol internally on its way to locating the method.

Why Other Languages Don't Have Symbols

Many languages do not have a concept of a symbol because they choose to handle the efficiency problem in different ways. They might use different techniques for string interning or might not perceive the same need for immutability. Also, some languages like JavaScript and Lisp do have similar concepts.

In other words, the choice to include symbols (or not) is primarily a design decision made by the creators of the language based on the specific goals and priorities they have for their language.

It's also worth noting that while many languages don't have a construct explicitly called "symbols", many of them do have similar concepts. For instance, Python has the intern() function which can be used to create a pool of unique, immutable string objects. In Java, the concept of string interning is also present, where the JVM maintains a pool of strings. Each distinct string gets one entry in the pool, saving memory.

In conclusion, the symbol is a unique feature of Ruby that solves a specific problem in a certain way. Other languages may have different solutions to the same problem, or may not regard it as a problem at all, depending on their design philosophy and goals.

#### What do you do with symbols?

Ruby is letting you, the programmer, use the same symbol-storage mechanism that Ruby uses to track identifiers. Rails uses symbols to identify things. In particular, it uses them as keys when naming method parameters and looking things up in hashes.

Symbols are used for several things, but they're mainly used as hash keys or for referencing method names.

In addition, symbols are commonly used as keys in Ruby's hashes. Since symbols are immutable and unique, they are ideal for identifying hash keys.

#### What is the characteristics of Symbol?

Immutable:

Symbols are immutable, meaning they cannot be changed after they are created. If you have used the same symbol multiple times in your code, Ruby will recognize that it is the same object.

On the other hand, if you use a string "mysymbol", every time Ruby will consider it as a different object even though the content is the same. Here is an example:

puts "string".object\_id # different every time

puts "string".object\_id # different every time

puts :symbol.object\_id # same every time

puts :symbol.object\_id # same every time

WHL: Symbols in Ruby seem global, right? Does it mean all symbols exist in the same "namespace"? How about ActiveRecord models User and Post both have a symbol :id? Why they don't conflict? Explain to me in detail please.

ChatGPT:

Yes, you're right. In Ruby, all symbols exist in a single, global namespace. This means that the symbol :id in one part of your program is the exact same object as :id elsewhere in your program.

The thing that can help with the confusion is understanding that symbols in Ruby are just values, similar to integers or strings. When you use the symbol :id in your code, you're not defining anything - you're just referencing a value.

Let's dive deeper into ActiveRecord in the context of Ruby on Rails to understand this better.

**ActiveRecord and Symbols**

When you create an ActiveRecord model, like User or Post, you can add attributes to that model, like :id, :name, :title, and so on. These attributes correspond to columns in your database table.

When you use a symbol such as :id in ActiveRecord, it's simply a way of referring to a specific attribute. It's not that there are different :id symbols for each model, rather, it's that the :id symbol is used to refer to different things in the context of different models. For instance, User might have an :id attribute, and Post might also have an :id attribute.

The key point here is that the symbols themselves don't have any inherent meaning. They don't "contain" any data or functionality themselves. They are simply used as identifiers that are matched up with actual data or functionality elsewhere in your code.

For example, when you write a query like User.find\_by(id: 1), the :id symbol is used to construct a SQL query that looks for a row in the users table where the id column is 1. Similarly, Post.find\_by(id: 1) uses the :id symbol to construct a SQL query that looks for a row in the posts table where the id column is 1.

So in the context of ActiveRecord models, symbols like :id are not "owned" by any particular model. They are just a way of referring to the attributes of those models. The :id symbol doesn't belong to the User or Post model - it's just used by those models as a convenient and efficient way to refer to their id attributes.

**我認為：或許是每個 model 都有一個「作用域」，一個 symbol 在域中是唯一的，不與域外的衝突。 20231117**

In summary, even though symbols are global and exist in the same namespace, they don't cause conflicts because they're simply identifiers used to refer to data or functionality that's defined elsewhere in your code. The context in which a symbol is used determines what it refers to.

WHL: Suppose in the same Controller method, there are : user = User.find\_by(id: 1) and post = Post.find\_by(id: 1). How can the controller tell an :id from another :id?

ChatGPT4:

In your example:

user = User.find\_by(id: 1)

post = Post.find\_by(id: 1)

The symbols :id are both the same symbol object (remember, Ruby symbols are unique), but they are used in different contexts.

The User.find\_by(id: 1) expression tells Rails to find a User object where the id attribute is equal to 1. Here, :id is used as an identifier for the 'id' column in the 'users' table of your database. In the same way, Post.find\_by(id: 1) is telling Rails to find a Post object where the id attribute is equal to 1. In this context, :id is referring to the 'id' column in the 'posts' table of your database.

So, the same symbol :id is being used in two different contexts, but the context (either User or Post) determines how the symbol :id is interpreted. Rails is able to keep track of this context, and so it knows that when you say User.find\_by(id: 1), you're referring to the 'id' of a User, and when you say Post.find\_by(id: 1), you're referring to the 'id' of a Post.

In summary, it's not the symbol that's keeping track of which id it's referring to, it's the ActiveRecord methods (User.find\_by, Post.find\_by) and the models (User, Post) that are keeping track. The symbol :id is just an efficient way to refer to these different 'id' columns.

WHL: Tell me how Ruby keep track of the usage of the symbol :id.

ChatGPT:

In Ruby, symbols are not linked to any particular usage or meaning on their own. They are lightweight, immutable, and unique identifiers that are used across the language. The specific meaning or usage of a symbol like :id comes from how and where it is used in the context of your program, not from the symbol itself.\


Symbols in Ruby don't carry state or context themselves. They are simply identifiers. When we say that Ruby "keeps track" of a symbol like :id, it's not that the symbol is being tracked individually, but rather that the overall program (or specific libraries or frameworks like Rails' ActiveRecord) interprets and uses the symbol based on its context.

For example, in your ActiveRecord User.find\_by(id: 1) scenario, Ruby doesn't need to keep track of :id specifically. Instead, the ActiveRecord find\_by method takes a hash argument {id: 1} where :id is the key. The find\_by method knows that it should generate a SQL query looking for a record where the column named 'id' matches the given value 1.

The responsibility of understanding what :id means in this context falls on the ActiveRecord find\_by method, not on the symbol :id itself. Ruby's role here is to ensure the :id symbol is created and reused efficiently. The specific usage and tracking of the :id symbol in the context of ActiveRecord queries is handled by the ActiveRecord library itself.

In conclusion, Ruby doesn't "track" the usage of :id or any other symbol. Instead, the context in which the symbol is used (in this case, ActiveRecord) is responsible for giving the symbol its specific meaning and functionality.

WHL: I still couldn't understand it. Can you mimic it with a real world example to help me understand symbol?

ChatGPT: Of course, let's think about symbols as if they were name tags at a conference.

At the conference, each person is given a name tag with their name on it. These name tags are like symbols - they're unique identifiers. When you see a name tag with "Alice" written on it, you know it refers to the person named Alice, and when you see a "Bob" name tag, you know it refers to Bob.

Now, imagine there are multiple conference rooms within the same event, each room hosting a different topic discussion. Each person can join any discussion in any room. Alice can be in Room 1 discussing "Programming Languages", while Bob can be in Room 2 talking about "Database Design".

Now, if you come across a note that says "Alice made a great point", without additional context, you can't tell what the point was about. Was it about Programming Languages or Database Design or something else entirely? The name tag "Alice" is just an identifier; it doesn't carry information about where it was used or what was discussed.

However, if you know that the note came from Room 1, you can infer that Alice's point was about Programming Languages. In this scenario, Room 1 provides the context for understanding the note about Alice, similar to how the User model provides the context for understanding :id in User.find\_by(id: 1).

So in this analogy:\


The conference attendees are like Ruby objects (like User or Post). The name tags are like symbols (like :id). They are just identifiers. The conference rooms are like the context in which the symbols are used (like the find\_by method in ActiveRecord).

It's important to understand that the name tag (or symbol) itself doesn't know anything about who it identifies or where it's used. That context comes from elsewhere (the person wearing the name tag or the method using the symbol).\


So in your ActiveRecord User.find\_by(id: 1) example, :id is like a name tag. It doesn't know by itself that it's referring to the 'id' column of the 'users' table in the database. That context is provided by the User.find\_by method in which it's used. The :id symbol is just an efficient way for your code to refer to that 'id' column.

WHL: How can I know is "id" a symbol or a variable in a code, I'm not familiar with, like User.id?

ChatGPT: When reading Ruby code, the usage of symbols and variables can indeed be confusing. However, there are some rules and conventions that can help.

For your example User.id, id is neither a symbol nor a regular variable. In the context of an ActiveRecord model (like User), id is a method name that represents an attribute of the User model. It's like calling a method to access the id attribute of the User model.\


user = User.find(1) # Find a User with an id of 1

puts user.id # Prints the id of the user

In this code, user.id is calling the id method on the user object to get the value of the id attribute for that user.

In a plain Ruby context, the User.id expression is interpreted as calling the id method on the User object. User is the receiver of the method call. id is the method name.

class User

def self.id

"This is the User's ID."

end

end

puts User.id # => "This is the User's ID."

In this example, id is a class method defined on the User class. We call this method by using the dot notation User.id.\


In Ruby, method names and local variable names are written in the same way. Both are case sensitive and can include alphanumeric characters and underscores, but they must begin with a lowercase letter or an underscore.

The way to distinguish them is based on context and usage:

If it's on the left-hand side of an assignment operation, or if it's used in an expression without a receiver and a dot before it, it's a variable.

If it's preceded by a receiver and a dot, like User.id, it's a method call.

WHL: With the help of mimicking symbols to "name tag", I think a symbol in Ruby is just like a kind of "conceptual identifier", it's an existing concept psychologically. It's like "James" is a name, an identifier and a concept. Anyone can call himself James, and there are many James in different classrooms. Am I right?

ChatGPT: Yes, you're absolutely right! Your understanding is spot on.

A Ruby symbol like :name or :id is very much like a conceptual identifier. It's a unique and immutable piece of data that serves as an identifier within the code.

Just like the name "James" can be used to refer to many different people in different contexts (or classrooms, as per your example), a symbol in Ruby such as :id can be used in different contexts to refer to different things. For example, in the context of a User model in a Rails application, :id might refer to the unique identifier of a user record. In the context of a Post model, :id might refer to the unique identifier of a blog post. The symbol :id itself is just an identifier—it doesn't inherently know anything about users or blog posts. The meaning comes from how it's used in the code.

So, your analogy of symbols being like names, and the way names can refer to different individuals in different contexts, is a very good way to understand how symbols work in Ruby.

WHL: Let's just comparing a symbol and an immutable variable. I know a variable is a container, and a symbol is just like an identifier that could be put on a container. Isn't a "variable name" also an identifier, or it's not? How do you represent a "variable name" in Ruby interpreter? Can I say a symbol is a variable name when it's used as a variable name?

ChatGPT: Yes, a variable name is indeed an identifier, but it's important to distinguish between a variable name and the value it refers to.

In Ruby, a variable name is a label that refers to a value. For example, in name = "Alice", name is the variable name, and it refers to the string "Alice". When we refer to name later in the code, we're accessing the value it points to.

A symbol in Ruby, like :name, is also an identifier, but it's an immutable, unique value in its own right.

While a variable name refers to a value, a symbol is a value. So, while both variable names and symbols act as identifiers in Ruby, they're used in different ways:\


Variable names are used to label and refer to values. Symbols are used as unique, immutable values themselves, often to represent concepts or identities in the code.

Regarding your question about the Ruby interpreter, variable names are used as keys in a symbol table to look up values. So in the Ruby interpreter's perspective, variable names have a role similar to symbols as they both act as keys in various internal data structures.

However, it wouldn't be accurate to say a symbol is a variable name when it's used as a variable name. While they have similarities and can sometimes be used in similar ways, they have distinct roles in Ruby's language design and implementation. A symbol is a unique, immutable value, while a variable name is a label that refers to a value.

WHL: what is the data type of the "name" from the Ruby interpreter's perspective?

ChatGPT: From the perspective of the Ruby interpreter, the name of a variable, like name in name = "Alice", is represented internally as a symbol. This means that in the underlying C code of the Ruby interpreter, the variable name name is stored as a symbol.

When the Ruby interpreter parses your code, it converts variable names into symbols and uses these symbols as keys in its internal data structures to keep track of variable values. For instance, it might use a hash-like structure where the keys are symbols representing variable names and the values are the corresponding variable values. So, from the Ruby interpreter's perspective, the variable name name is a symbol.

WHL: From a programmer's perspective, why don't I just use "name" wherever needed rather than using ":name"?

ChatGPT: There are several reasons you might want to use a symbol (:name) instead of a variable (name):

Immutability: Symbols in Ruby are immutable, meaning they can't be changed once they're created. This makes them safe to use as keys in a Hash or anywhere you need a consistent, unchangeable identifier.

Identity: Each Symbol is a unique object, regardless of its content. For example, :name is the same object everywhere in your Ruby program. This is not the case with strings or variables, where "name" or name in different parts of your program could refer to different objects or values.

Performance: Because all uses of :name refer to the same object, symbols can be more efficient than strings or variables. They take up less memory and comparing them for equality is faster because it's essentially comparing object IDs rather than the content.

Let's consider an example in a Rails application where you define a new route:

get 'photos/show', to: 'photos#show', as: :show\_photo

Here, :show\_photo is used as a unique identifier for the route. This symbol is then used in the application to generate paths and URLs for the route. It's safe to use :show\_photo here because it's an immutable symbol. If it were a variable and its value changed, it could break URL generation in parts of the application.

In short, symbols are used when you care about the identity of the object (the fact that it's the same object everywhere), not what it points to or its contents. In contrast, variables are used when you want to refer to changing or dynamic data.

WHL: I could use a variable show\_photo and I discipline my code to never change its value. Therefore show\_photo is the same as :show\_photo, right?

ChatGPT: Conceptually, you could indeed do that. You could define a variable show\_photo and use it in a way that it never changes its value throughout your code. However, there are a few important things to remember.

Firstly, the guarantee that symbols provide in Ruby is enforced by the language itself, not by programmer discipline. A symbol is immutable and unique throughout the Ruby program, whereas the immutability of a variable like show\_photo is not enforced by the language, but by the programmer's discipline. This opens the door to potential bugs if the variable is accidentally modified.

Secondly, a symbol in Ruby is a value. It is its own thing, not a variable pointing to something. When you use :show\_photo, you're not referencing a value elsewhere, you're using the symbol value itself. This distinction becomes important in certain contexts. For example, when used as keys in a Hash, symbols provide identity — you can be sure that every :show\_photo is exactly the same object, which isn't necessarily the case with string or variable equivalents.

So while you can use a never-modified variable to mimic a symbol, doing so would require more caution, might introduce potential bugs, and would be less performant than using symbols. It also wouldn't be idiomatic Ruby code, which could make your code harder for other Rubyists to understand.

WHL: I can assign a value, a method or an object to a variable. Can I "assign" something to a symbol? say :name = "James"?

ChatGPT: No, you can't assign a value to a symbol in Ruby. A symbol is not a variable; it's an immutable, unique identifier in the system. You can think of a symbol as a label that exists on its own.

However, you can use a symbol as a key in a hash and associate it with a value, like this:

person = {:name => "James"} Or, using the newer syntax: person = {name: "James"}

:name is a symbol used as a key in the person hash, and "James" is the value associated with that key. This does not mean that the symbol :name has been assigned the value "James"; rather, it means that in the context of the person hash, the key :name is associated with the value "James".

WHL: Well, in a hash, :name is "connected" to a value "James", right? through what mechanism? And what is the internal mechanism of assigning a value to a variable?

ChatGPT: In Ruby, both assigning a value to a variable and associating a value with a key in a hash are operations that involve storing references to objects.

When you assign a value to a variable with a statement like name = "James", the Ruby interpreter creates a string object with the value "James", then creates a reference from the variable name to that object. You can think of this as name pointing to the "James" object. When you use name in your code, the interpreter follows that reference to get the object it refers to.

When you associate a key with a value in a hash with a statement like person = {name: "James"}, the Ruby interpreter creates a hash object. Inside that hash object, it creates a reference from the key (:name in this case) to the value ("James"). You can think of this as :name pointing to "James" within the person hash. When you access person\[:name], the interpreter looks inside the person hash, finds the :name key, and follows the reference to get the associated object.

The important difference between these two cases is that with variable assignment, the reference is from a variable to a value, while with hash keys, the reference is from a key to a value within a specific hash. The scope of a variable is typically the entire program or a specific part of it (depending on where the variable is defined), whereas the scope of a hash key is just that particular hash.

WHL: Suppose, in a controller method, there are two lines: user = User.find\_by(id: 1) post = Post.find\_by(id: 1) How can I tell one :id from another? Won't they conflicts?

GitHub Copilot: In the example you provided, both User and Post models have an :id attribute, which represents the primary key of their corresponding database tables. However, since each model has its own table, the :id attribute of each model represents a unique identifier for that model's records.

In the controller method you provided, the find\_by method is used to retrieve a single record from each model's table based on the id attribute. Since each model has its own table, the find\_by method will only return a single record from the corresponding table.

To differentiate between the :id attributes of the User and Post models, you can use the model name as a prefix. For example, you could use user\_id to refer to the id attribute of the User model, and post\_id to refer to the id attribute of the Post model. This can help make your code more readable and avoid confusion between the two attributes.

Here's an example of how you could modify the controller method to use these naming conventions:

user = User.find\_by(id: 1)

post = Post.find\_by(id: 1)

\# Use the model name as a prefix to differentiate between the id attributes

user\_id = user.id

post\_id = post.id

By using these naming conventions, you can avoid conflicts between the :id attributes of the User and Post models.

WHL: Define "reference". Are the two kind of referencing different in a variable scenario and a symbol scenario?

ChatGPT: In the context of programming, a reference can be thought of as a way to access or 'refer to' a value stored in memory. It's like a pointer or an address that tells the system where to find the actual data.\


When you create a variable in Ruby and assign it a value, what you're really doing is creating a reference from that variable to the value's object. Here's an example:

name = "James"

In this code, name is a reference to the string object "James". The variable name doesn't contain the string itself; it contains a reference to the string.\


Similarly, in a hash, the keys (which can be symbols) refer to their corresponding values:

`person = {name: "James"}`

Here, :name is a reference to the string "James" within the person hash.

The key difference in these scenarios is scope and usage:

In the variable scenario, name is a variable that refers to a value. It's a reference you can use anywhere within its scope to refer to that value.

In the symbol scenario, :name is a symbol used as a key in a hash. It refers to a value within that specific hash.

In both cases, you're dealing with references, but the way those references are used is different.

And from a technical perspective, the referencing in Ruby is largely handled for you by the language's memory management system.

WHL: Define referencing from an interpreter's perspective please.

ChatGPT: From the perspective of an interpreter (such as the one that executes Ruby code), a reference is a way to access data stored somewhere in memory.

When a program is executed, the data it manipulates (numbers, strings, arrays, objects, etc.) are stored in the computer's memory. However, instead of working directly with the raw data, the interpreter works with references, which you can think of as the address of where the data is stored in memory.

When you assign a value to a variable in Ruby, like so:

name = "Alice"

The interpreter does the following:

\- Allocates memory to store the string "Alice".

\- Stores the string "Alice" in that allocated memory.

\- Creates a reference from the variable name to the memory location where "Alice" is stored.

So, the variable name is essentially holding the reference (the memory address), not the actual data "Alice". When you use the variable name in your program, the interpreter uses that reference to access the memory location where the string "Alice" is stored.\


When it comes to a hash, the mechanism is similar but slightly more complex due to the nature of the hash data structure. Here's a simple example:

person = {name: "Alice"}

In this case, the interpreter:

\- Allocates memory for the hash person.

\- Within this hash, it allocates memory to store the string "Alice".

\- Creates a reference from the symbol :name to the memory location where "Alice" is stored.

So when you access person\[:name], the interpreter uses the reference from :name to find the associated string "Alice" in the hash's memory.

In both cases, references are used as a way to find and access data in memory. However, remember that as a Ruby programmer, you're rarely (if ever) dealing with these low-level details. Ruby's memory management system handles all of this for you, letting you focus on writing your code at a higher level of abstraction.
