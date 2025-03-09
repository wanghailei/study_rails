# Generators

In Rails, generators are simply scripts that use templates to create boilerplate code and improve your workflow saving you a quite a bit of time.

````ruby
rails generate GENERATOR [args] [options]
````

### Generate Options:

\-h, \[--help] # Print generator's options and usage

\-p, \[--pretend] # Show what files will be generated without actually generating them.

\-f, \[--force] # Overwrite files that already exist

\-s, \[--skip] # Skip files that already exist

\-q, \[--quiet] # Suppress status output

## Types of Generators

`rails generate` is often used in a shortcut form `rails g`.

### `rails g scaffold <ModelName> <attribute:type>`

A scaffold in Rails is a full set of model, database migration for that model, controller to manipulate it, views to view and manipulate the data, and a test suite for each of the above.

The following command will generate a scaffold for a single resource called Task:

```ruby
rails g scaffold Task title:string
```



### `rails g model`



### `rails g resource <ModelName> <attribute:type>`

The `rails g resource` command is provided to create a new resource in app. A resource is a term used in Rails for a collection of similar objects, such as users, posts, comments, etc.  When you run rails g resource, Rails will generate several files for you:  

- migration file to create the table in the database.
- model file.
- controller file.
- When you generate a resource using rails g resource, Rails does not generate views for each of the standard RESTful actions by default. It only creates a directory for the views.
- helper file.
- JavaScript file (if you're using the asset pipeline).
- stylesheet file (if you're using the asset pipeline).
- test file for each of the above (if you're using a test framework).

```ruby
rails g resource Post title:string body:text
```

This will generate 1) a `Post` model with a `title` attribute of type string and a `body` attribute of type `text`. 2) a `PostsController` and views for the standard actions. 3) a migration file to create the `posts` table with `title` and `body` columns. 

### Compare `rails g scaffold` and `rails g resource`

`rails generate scaffold` generates:
A full set of model, database migration for that model, controller and views.
A full test suite for that model.
Helper and Javascript files related to that model
==It also updates the Routes file with RESTful routes for the model.==

`rails generate resource` is a bit less inclusive. This generates:
The model, the corresponding migration and controller files
Test suite for that model
Resourceful declaration in the Routes file
However, it does not generate the views or helpers.

In summary, ==`scaffold` gives you a full set of ready to use (CRUD) views, while `resource` just gives you the model and controller, leaving you to manually create the views.==

### scaffold\_controller



#### `rails g controller <ModelName> <attribute:type>`

This command would create a new Controller named `BookController` and an associated `book` directory underneath the `views` directory to hold the Book controller's `view` templates.

```ruby
rails g controller <ModelName> <attribute:type>
```

When using the `rails generate controller` command, you should follow it with the name of the controller you're creating. By convention, this is usually the singular name of the resource it's going to be managing. Therefore, it's just Book, not BookController. 

```ruby
rails g controller Home index
rails g controller Book index new edit
```



#### `rails generate migration AddNewTable`



rails generate integration\_test TestName

rails generate session\_migration



controller

application\_record

benchmark

channel

generator

helper

integration\_test

jbuilder

job

mailbox

mailer





system\_test

task

active\_record:application\_record

active\_record:multi\_db

erb:controller

erb:mailer

erb:scaffold

stimulus

test\_unit:channel

test\_unit:generator

test\_unit:install

test\_unit:mailbox

test\_unit:plugin



## Creating and Customising Rails Generators & Templates

Since Rails 3.0, generators are built on top of Thor. Thor provides powerful options parsing and a great API for manipulating files.
