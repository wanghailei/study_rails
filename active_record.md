# Active Record

Smart models by design

* &#x20;﻿﻿"Fat model, skinny controller"
* Move most business logic to models
* Define instance methods for things an instance of a class should "know"
* Models can leverage attributes and associations

Model performs two key roles: one for representing the state of the application or business, two it communicates with the database.

State Representation: It represents the data and the rules (business logic) that govern access to and updates of this data. In other words, it maintains the state of the application.

Database Communication: The Model is responsible for all interactions with the database, namely CRUD. The Model uses an Object-Relational Mapping (ORM) system to abstract the low-level database commands into higher-level programming language methods or functions.

==The purpose of separating these responsibilities into a Model component is to isolate the business logic and data manipulation logic of an application.== Changes in the graphical user interface (handled by the View) or changes in the way user inputs are processed (handled by the Controller) do not affect the business rules or the database communication logic, and vice versa.

## Active Record

Active Record was described by Martin Fowler in his book _Patterns of Enterprise Application Architecture_: An object that wraps a row in a database table or view, encapsulates the database access, and adds domain logic on that data. An object carries both data and behaviour. Much of this data is persistent and needs to be stored in a database. Active Record uses the most obvious approach, putting data access logic in the domain object.

In Active Record, objects carry both persistent data and behaviour which operates on that data.

==Active Record automatically defines model attributes for every table column, so you don't have to declare those attributes in your model file.==

Active Record gives us several mechanisms, the most important being the ability to:

\- Represent models and their data.

\- Represent associations between these models.

\- Represent inheritance hierarchies through related models.

\- Validate models before they get persisted to the database.

\- Perform database operations in an object-oriented fashion.

### ActiveModel

ActiveRecord is an Object-Relational Mapper so that each of your objects represents a database row. ActiveModel is the interface that Rails uses to all of storage including non-relational stores like Cassandra or MongoDB, to fit particular object types into Rails.

## Associations

In Rails, an *association* is a connection between two Active Record models. Why do we need associations between models? Because they make common operations simpler and easier in your code.

Rails supports six types of associations, each with a particular use-case in mind.

- `belongs_to`
- `has_one`
- `has_many`
- `has_many :through`
- `has_one :through`
- `has_and_belongs_to_many`



### `belongs_to`

A `belongs_to` association sets up a connection with another model, such that each instance of the declaring model "belongs to" one instance of the other model. 

The `belongs_to` associations *must* use the singular term. If you used the pluralized form in the above example for the `author` association in the `Book` model and tried to create the instance by `Book.create(authors: @author)`, you would be told that there was an "uninitialized constant Book::Authors". This is because Rails automatically infers the class name from the association name. If the association name is wrongly pluralized, then the inferred class will be wrongly pluralized too.

In general, ==the `belongs_to` association should be defined on the table that holds the foreign key, while the `has_one` association should be defined on the table that holds the primary key==. This is because the foreign key is a reference to the primary key in the other table, and the `belongs_to` association is used to define this relationship.

For example, if you have a `User` model and an `Address` model, where each user has one address, you would define the `belongs_to` association on the `Address` model.

```ruby
class User < ApplicationRecord
     has_one :address
end

Class Address < ApplicationRecord
	belongs_to :user
end
```

There are 4 kinds of relationships: `belongs_to`, `has_one`, `has_many`, `has_many :through`.

###

In Rails, belongs_to is used to set up associations on the model that contains the foreign key column. belongs_to is not just for one-to-one relationships. 

In a one-to-one relationship, you will often use belongs_to and has_one together. The model that belongs_to another contains the foreign key.

```ruby
class User < ApplicationRecord
	has_one :profile
end

class Profile < ApplicationRecord
	belongs_to :user
end
```

In this example, a User has one Profile, and Profile belongs to User. The profiles table would have a user_id foreign key column.

In a one-to-many relationship, you will often use belongs_to and has_many together. The model that belongs_to another contains the foreign key.

```ruby
class Author < ApplicationRecord
       has_many :books
end

class Book < ApplicationRecord
       belongs_to :author
end
```

In this example, an Author can have many Books, and each Book belongs to an Author. The books table would have an author_id foreign key column.

So, belongs_to can be used to set up both one-to-one and one-to-many relationships, depending on whether the associated model uses has_one or has_many.

### `has_many`



```ruby
class Author < ApplicationRecord
  has_many :books, dependent: :destroy
end

class Book < ApplicationRecord
  belongs_to :author
end
```

### Relationships need be defined in Models

An example with Employee, Position, Department and Appointment as models.

% Appointment is a join table for Employee and Position and it actually could be named as EmployeePosition. 我認為應該創建這種 ==meaningful join-tabel==，而不是硬生生把兩個 object 連起來。 20240119 %

```ruby
# app/models/department.rb
class Department < ApplicationRecord
	has_many :employees
     has_many :positions
end
```

In general, when you're setting up a `has_many :through` association, first you create a direct one-to-many association, and then you "go through" that association to set up the many-to-many relation. So both lines are essential for establish gving a many-to-many relationship through a join model in Rails.

```ruby
# app/models/employee.rb
class Employee < ApplicationRecord
	belongs_to :department, optional: true
	has_many :appointments
	has_many :positions, through: :appointments
end
```

`has_many :appointments`: This line sets up a one-to-many relationship between `Employee` and `Appointment`. It `means` an employee can have many appointments. ==This is important because it allows access to `appointment` model's properties, and without it, the `through:` option cannot be used.==

==`has_many :positions, through: :appointments`: This line establishes the many-to-many relationship between `Employee` and `Position`.== The `through: :appointments` part tells Rails to use the `appointments` table as a join-table to facilitate this relationship.

```ruby
# app/models/position.rb
class Position < ApplicationRecord
	belongs_to :department, optional: true
	has_many :appointments
	has_many :employees, through: :appointments
end
```

The `optional: true` parameter is added to indicate that these associations are not mandatory, meaning an `employee` can exist without being related to a `position` or a `department`. If you want to enforce the presence of these associations, simply remove the `optional: true` parameter.

The `Appointment` model helps facilitate the many-to-many relationship between `Employee` and `Position`. ==% Join-tables 都是 belongs_to, belongs_to 的。 20240119 %==

```ruby
# app/models/appointment.rb
class Appointment < ApplicationRecord
	belongs_to :employee
	belongs_to :position
end
```

### Relationships also need be defined in Migration

Besides defining the relationship in ActiveRecord, you also need to define the relationships in migration files when setting up your database schema in Rails. 

```ruby
# Migration for the Employee model
class CreateEmployees < ActiveRecord::Migration[7.1]
	def change
		create_table :employees do |t|
			t.string :name
			t.references :department, null: false, foreign_key: true
			t.references :position, null: false, foreign_key: true
			t.timestamps
		end
	end
end
```

In the `Employee` migration, `t.references` adds a `department_id` and `position_id` to the `employees` table that will be used as foreign keys. The `foreign_key: true` makes these references to actually act as foreign keys.

Please note that `null: false` option in the references makes the association mandatory. If you want `department_id` and `position_id` to be optional, you would remove `null: false`.

```ruby
# migration creating the Appointment join-table
class CreateAppointments < ActiveRecord::Migration[7.1]
	def change
		create_table :appointments do |t|
			t.references :employee, null: false, foreign_key: true
			t.references :position, null: false, foreign_key: true
			t.timestamps
		end
	end
end
```

### Add references (foreign key relationships)

% 經常需要後續添加 reference（foreign key relationships）。單獨創建 migration 來做這件事。 20240120 %

```ruby
class EmployeesReferencesPositionAppointmentDepartmentAddition < ActiveRecord::Migration[7.1]
	def change
		add_reference :employees, :position, null: false, foreign_key: true
		add_reference :employees, :appointment, null: false, foreign_key: true
		add_reference :employees, :department, null: false, foreign_key: true
	end
end
```

Foreign keys additionally add an index to that column for DB performance. 

#### While it's not required, you might want to add foreign key constraints to guarantee referential integrity.

==In Rails, foreign key constraints are not strictly required because Rails provides its own mechanisms to maintain referential integrity at the application level.== 

==ActiveRecord Associations allow Rails to infer the foreign key relationships from the model definitions, without having to explicitly enforce them at the database level.==

Not all databases support foreign key constraints, therefore Rails doesn't require them as part of its standards.

However, adding foreign key constraints at the database level provides an added level of data protection and can prevent race conditions or inadvertent errors from corrupting data relationships, such as running SQL commands directly on the database outside of Rails or concurrent writes to the database.

#### `foreign_key: true`

A Foreign Key, on the other hand, is a column or a set of columns in a table that is used to establish a link between the data in two tables. ==It's a field in one table, that uniquely identifies a row of another table. The foreign key in one table is the primary key of another table.==

For example, consider a posts table in a blog database. Each post might be associated with a user. The posts table might have a `user_id` column as a foreign key. This `user_id` is used to associate each post with a user. The `user_id` in `posts` table refers to the id `in` the `users` table.

The main purpose of foreign keys is to enforce a certain level of data integrity in the database. If a foreign key constraint is set up, it prevents actions that would destroy links between tables. For example, ==a database wouldn't allow a row containing a foreign key to be deleted if there is a row in the linked table that references it.==

==In Rails Migration, when `foreign_key: true` is specified, it means that you're adding a foreign key constraint at the database level.==

```ruby
def change
	add_reference :articles, :author, foreign_key: true
end
```

In this migration, you're adding a new column `author_id` to the `articles` table. The `author_id` will be used to link each article to a specific author. `foreign_key: true` adds a foreign key constraint to this `author_id` column.

==The foreign key constraint ensures referential integrity, which means the database will only allow you to enter an `author_id` in the `articles` table if it refers to an existing `author` record in the `authors` table.== If you try to enter an `author_id` that doesn't exist in the `authors` table, or if you try to delete an `author` record that's associated with an `article` record, the database will raise an error. ==In this way, `foreign_key: true` helps to preserve the integrity of your data.==

In Rails 5 and onward, `foreign_key: true` by default also implies an index on this column, which helps speed up queries involving this foreign key. ==Note that it's not enough just to have the foreign key column in your model - you should add the foreign key at the database level with a migration to ensure referential integrity.==

#### Potential issues that might occur:

**Nonexistent Tables**: If `employees`, `position`, `appointment`, or `department` tables do not exist, then this migration will fail during runtime. % 所以，==在運行添加 reference 的 migration 時，首先要確保相關的表都已經創建了。== 20240120 %

**Existing Columns**: If `employees` already has `position_id`, `appointment_id` or `department_id` column, the migration will fail. % 是不是在創建 migration（尤其用 rails g migration）時，分兩步走比較好？首先創建獨立的基本表，然後另起一個 migration 創建 reference。 20240120 %

**Nonexistent Values**: If `null: false` was set, ensure that you have the necessary data for all existing records. If not, existing records without these foreign key values would not be valid.

**Locking**: A very large number of indexes may negatively impact write performance. If you have a significant number of records, adding an index can lock your table for a time proportional to the number of records. 

### Setting null: false

==Setting `null: false` implies that these fields must always have a value==. By default, Rails sets `null: true` if you do not specify the `null` option.  

If you define a column with `null: true` or without specifying `null: false` in a migration in Rails, it means that the column is allowed to have `NULL` values. Therefore, in this case, the `position_id`, `appointment_id`, and `department_id` columns in the `employees` table could have `NULL` values. In other words, it would be legal for an `employee` record to not have associated `position`, `appointment`, or `department`.

#### Is setting `null: true` a good thing?

Whether to set null: true or null: false largely depends on the business requirements and the domain model of your application.

Setting null: false enforces that the column must always contain a value, i.e., it cannot be null. This is typically used when you know that a column should always have a value for every record in the table.
On the other hand, null: true is the default setting which allows NULL values in the column. This means that it's not mandatory for every record to have a value for this column.

Here are a few things to consider when choosing between null: true and null: false:
Data Integrity and Consistency: Setting null: false promotes data integrity and consistency since it enforces every record to provide a value for the column.

Business Rules: If, according to the business rules, a field must always contain data, then setting null: false is the right choice. For instance, an employees table should probably have a name field that is always filled, thus null: false would be appropriate.

Flexibility: Setting null: true can provide flexibility. There might be circumstances where you do not have all the data at the time of creating a record. In such cases, allowing NULL values in certain columns could be beneficial.

Performance: Allowing NULL values (null: true) can slightly increase the performance of the database when querying data because SQL databases can optimize NULL storage and searches.

Weigh these considerations in terms of your specific requirements. For instance, if your employees can exist without associated position, appointment, or department, then setting null: true for position_id, appointment_id, and department_id would seem appropriate. However, if these associations are mandatory according to your business rules, then you should enforce null: false to ensure data integrity.

### Example 2

For three models, Position, Responsibilty and Capability. A Position shall have mulitiple responsibilities and capabilities. A responsibility might be shared by many positions, such as all directors should be able to lead team members well. A common capability might be required by different positions, such as critical thinking. How to define the relationships in ActiveRecord?

Based on the need, there are many-to-many relationships between Position and Responsibility, as well as between Position and Capability. `has_many :through` is used to set up many-to-many relationships. You'll also need *join models* (for example, PositionResponsibilities and PositionCapabilities).

```ruby
# model: position.rb
class Position < ApplicationRecord
     has_many :responsibilities, through: :position_responsibilities
	has_many :capabilities, through: :position_capabilities
     has_many :position_responsibilities
     has_many :position_capabilities
end

# model: responsibility.rb
class Responsibility < ApplicationRecord
	has_many :position_responsibilities
	has_many :positions, through: :position_responsibilities
end

# model: capability.rb
class Capability < ApplicationRecord
	has_many :position_capabilities
	has_many :positions, through: :position_capabilities
end

# model: position_responsibilities.rb
class PositionResponsibility < ApplicationRecord
	belongs_to :position
	belongs_to :responsibility
end

# model: position_capabilities.rb
class PositionCapability < ApplicationRecord
	belongs_to :position
	belongs_to :capability
end
```

Now for the migrations, you will have to create the join models `PositionResponsibility` and `PositionCapability`:

```ruby
# migration creating the PositionResponsibility join-table
class CreatePositionResponsibilities < ActiveRecord::Migration[7.1]
	def change
		create_table :position_responsibilities do |t|
			t.references :position, null: false, foreign_key: true
			t.references :responsibility, null: false, foreign_key: true
			t.timestamps
		end
	end
end

# migration creating the PositionCapability join-table
class CreatePositionCapabilities < ActiveRecord::Migration[7.1]
	def change
		create_table :position_capabilities do |t|
			t.references :position, null: false, foreign_key: true
			t.references :capability, null: false, foreign_key: true
			t.timestamps
		end
	end
end
```

These models and migrations will establish the relationships, where a position can have multiple responsibilities and capabilities, and a responsibility or capability can be associated with multiple positions.



## Other things

### Getting Column Statistics

Rails has the ability to perform statistics on the values in a column.

### Deleting Associated Objects

If you delete an article, its associated comments will also need to be deleted. Rails allows you to use the _dependent_ option of an association to achieve this.

### Scopes

As these

## Query Methods

### Query

#### `exists?`

Checks if a record exists that matches the given ID or conditions​.

```ruby
# checks if a record with id 5 exists.
Person.exists?(5)

# does the same, accepting a string as input.
Person.exists?('5')

# checks if records with ids 1, 4, and 8 exist.
Person.exists?(id: [1, 4, 8])

# checks if a record with the name 'David' exists.
Person.exists?(name: 'David')

# checks if a record with the name 'Spartacus' and rating 4 exists.
Person.where(name: 'Spartacus', rating: 4).exists
```

#### `first`

Finds the first record (or the first N records if a parameter is supplied).

If no order is defined, it returns the first record in the database according to the database's natural order. If an order is defined, it returns the first record according to that order.

```ruby
# returns the first object fetched by SELECT * FROM people
Person.first

# returns the first object from OFFSET 3 (which is OFFSET 2)
Person.offset(3).first

Person.where(["user_name = :u", { u: user_name }]).first
```

Please note that the natural order in most databases is based on the order of insertion, but this is not guaranteed across all databases and can be affected by factors such as the table's primary key and certain database settings. Therefore, ==it's recommended to specify an order if the order is important to your application​​.==

#### `fifth`

Returns the fifth record. If no order is defined it will order by primary key​.

```ruby
# returns the fifth object.
Person.fifth

# returns the fifth object that matches the where clause.
Person.where( ["user_name = :u", { u: user_name }] ).fifth
```

#### `offset`

This is used to skip a certain number of records.

```ruby
# Skip the first 10 users and then get the next 5.
User.offset(10).limit(5)

# returns the fifth object from OFFSET 3 (which is OFFSET 7).
Person.offset( 3 ).fifth

# Skip the first 5 users and then get the next 5:
User.offset(5).limit(5)

# Skip the first 10 products and then get the next 10:
Product.offset(10).limit(10)

# Skip the first 3 orders and then get the next 3:
Order.offset(3).limit(3)
```

#### `find`

This is used to retrieve a record by its primary key, namely finds a record by its ID. This can be a specific ID, a list of IDs, or an array of IDs.

If one or more records cannot be found for the requested IDs, then `ActiveRecord::RecordNotFound` will be raised​​.

find with a lock: This is used to handle concurrent transactions to prevent conflicts. For example, if two transactions are each adding 1 to a value, by locking the row, the second transaction will have to wait until the first is finished​​.

```ruby
# Find the user with primary key (id) of 1.
User.find(1)

# Find the product with primary key (id) of 100.
Product.find(100)

# finds the objects with IDs 7 and 17.
Person.find([7, 17])

```

#### `find_by`

Finds the first record matching the specified conditions. If no record is found, it returns `nil`​​. This is similar to find but allows you to specify any column you want, not just the primary key.

```ruby
# Assuming username is a unique column
User.find_by( username: 'john_doe' )

# Finds the first user from a specific city:
User.find_by city: 'New York'

# finds the first Post with the name 'Spartacus' and rating 4.
Post.find_by name: 'Spartacus', rating: 4

# finds the first Post that was published less than 2 weeks ago.
Post.find_by "published_at < ?", 2.weeks.ago

# Finds the first order that was placed today:
Order.find_by 'created_at >= ?', Date.today

# Finds the first product with a certain SKU:
Product.find_by sku: 'ABC123'
```

#### `find_or_initialize_by`

Returns the first item or a new instance (which is not persisted to the database)​1​.

#### `find_or_create_by`

Returns the first item or creates it and returns it​1.

### Sorting

#### `order`

This is used to sort the records according to a certain column.

```ruby
# Get all users, ordered by their name.
User.order :name

# Get all products, ordered by their price in ascending order.
Product.order price: :asc

# Get all users, ordered by their signup date in descending order.
User.order created_at: :desc

# Get all orders, ordered first by user_id, then by created_at.
Order.order user_id: :asc, created_at: :desc
```

#### `where`

Used to filter records based on certain conditions​​.

```ruby
# Finds all users who are active.
User.where active: true

# Finds all products in a certain category.
Product.where category: 'Electronics'

# Finds all users who signed up in the past week:
User.where 'created\_at >= ?', 1.week.ago

# Finds all orders that have been shipped and are not cancelled:
Order.where status: ['shipped', 'not cancelled']

# finds all books with a category of "Ruby".
Book.where category: "Ruby"

# finds all books with a category of "Ruby" and an author of "Jesus Castello".
Book.where category: "Ruby", author: "Jesus Castello"

# finds all books that do not have a category of "Java".
Book.where.not category: "Java"

# finds all books with an id of 1, 2, or 3.
Book.where id: [1,2,3]

# finds all books that have a comment with an id of.
Book.joins(:comments).where(comments: { id: 2 })

# finds the first object with "administrator = 1", ordered by "created_on" in descending order.
Person.where("administrator = 1").order("created_on DESC").find(1)
```

#### `select`

This is used to specify only the columns you are interested in, reducing the amount of data that needs to be read from the database.

```ruby
# Select only the name and email of all users
User.select(:name, :email)

# Select only the product name and price:
Product.select(:name, :price)

# Select only the order number and status:
Order.select(:number, :status)
```



### Filtering

#### `group`

This is used to group the records by a certain column, which can be useful for aggregate queries.

```ruby
# Group users by their account status
User.group(:account_status)

# Group users by their city:
User.group(:city)
```

#### `having`

This is used in conjunction with group to filter the groups.

```ruby
# Group users by account status, and select only those groups with more than 5 users
User.group(:account_status).having("count(*) > 5")

# Group users by their city and select only those cities with more than 10 users:
User.group(:city).having('count(*) > 10')

# Group products by their category and select only those categories with more than 5 products:
Product.group(:category).having('count(*) > 5')

# Group orders by their user_id and select only those users with more than 1 order:
Order.group(:user_id).having('count(*) > 1')
```



#### `limit`

This is used to limit the number of records returned.

```ruby
# Get the first 5 users
User.limit(5)

# Get the first 10 products:
Product.limit(10)

# Get the first 3 orders:
Order.limit(3)
```



#### `joins`

Gluing tables together with the joins method. This method allows us to tell `ActiveRecord` to perform a SQL `join` on its associations.

`Person.all.joins(:role)` which generates SQL like this:

```sql
SELECT "people".\*
FROM "people"
INNER JOIN "roles"
ON "roles.id" = "people"."role\_id";
```



```ruby
# Joins the users table with the orders table based on the user_id and 
# gets all the users who have placed an order.
User.joins(:orders)

# Joins the products table with the order_items table and 
# gets all the products that have been ordered:
Product.joins(:order_items)

# Joins the orders table with the users table and 
# gets all the orders placed by users from a specific city:
Order.joins(:user).where(users: { city: 'New York' })
```



## Reference



**WHL: How is the Model communicate with the Database specifically, in detail?**

The Model interacts with the database through an Object-Relational Mapping (ORM) system, like ActiveRecord. Here's a more detailed breakdown of the process:

==When a Controller needs to interact with data, it asks the Model to do so. The Model uses ActiveRecord methods to create SQL queries.==

* ActiveRecord then sends the SQL query to the database for execution.
* The database returns the data, and ActiveRecord converts this raw data into Ruby objects that the application can work with. These objects are instances of the Model class. For example, each row in the users table is represented as an instance of the User model.
* If the data needs to be modified, the Controller makes changes to the Model's objects, and then it calls a method (like save or update) to write these changes back to the database.
* ActiveRecord generates the appropriate SQL queries for these operations as well.

ActiveRecord also handles relationships between different models (like has\_many, belongs\_to, etc.). This lets you manage related data across different tables in a more intuitive and Ruby-like way, rather than writing complex SQL queries.

Before writing data back to the database, ActiveRecord performs any validations defined in the Model. If the data is invalid (for example, if a required field is missing), the save or update operation is cancelled, and the invalid data is not written to the database.

==So while it seems like the Model is communicating directly with the database, it's actually ActiveRecord that does most of the work behind the scenes.==



**WHL: Are the states of the business represented by the Model stored in the RAM of the server?**

ChatGPT: The Model in an MVC architecture doesn't typically store business state in server RAM in the way that you might be imagining. Instead, the Model represents the structure of the data and the rules for interacting with that data. ==The actual data, or "state", is usually stored in a database.==

However, when your application is running and an instance of a Model is created, that instance and its data are temporarily stored in the server's RAM. This data remains in RAM only for the duration of the request. Once the request is completed and the response has been sent, the data for that request is discarded to free up memory for other requests. Persistent storage is managed by the database.

It's important to note that some applications do store certain data in server memory for quick access, such as session data or cache data.



### Best practices of migration

**WHL: Shall I rewrite migrations to be simpler, unified after a period of development since there were many adjustments, modifications or amendments back and force?**

ChatGPT: Rewriting migrations can be a tempting idea, especially when your application has a long history with many changes to the database schema over time. It might feel like a good idea to simplify and unify all of these migrations into a single, more manageable set. However, it's generally recommended not to rewrite old migrations for a few reasons:

\- Database Integrity: Running new migrations on existing databases can risk data integrity, especially if the new migrations don't account for all past states of the data.

\- Deployment Risks: If you have a live, production application, rewriting migrations might create discrepancies between your development environment and the production environment, leading to potentially serious issues during deployment.

\- Historical Record: Migrations serve as a historical record of changes to your database schema. Rewriting them erases this history, which might be useful for understanding past decisions or debugging issues.

\- Collaboration: If you are working in a team, rewriting migrations might disrupt your colleagues' work as they would need to reset and recreate their databases to match the new migrations.

Instead of rewriting old migrations, consider these best practices:

\- If your schema is getting complex, use the schema.rb or structure.sql file that Rails automatically generates and updates whenever migrations are run. This file gives you the current state of the database schema.

\- If you're just starting a new feature and realize you want to make changes to a migration you just created, and this migration hasn't been pushed or merged into your main or development branch, then it's usually safe to roll it back, edit it, and then rerun it.

\- If you need to change the database schema for a feature or bug fix, create new migrations to alter the required tables.

\- If you have migrations that are no longer relevant (for example, if you have a CreateUsers migration and later a DropUsers migration), you could consider squashing these migrations to reduce the number of migration files. But be very careful with this approach; ensure the squashed migration still works as expected on all your environments.

\- If you're working on a personal project or starting a new project from scratch, you might find it cleaner to reset your database and recreate the schema with fewer, more organized migrations. But for most existing, shared, or production applications, it's best to leave old migrations as they are and make changes with new ones.

**WHL: Therefore it's not a good practice to manipulate data with migrations.**

ChatGPT: You're correct in observing that data manipulation in migrations should be done cautiously. While it's technically possible to perform data manipulations within migrations, it's not always the best approach and can lead to complications. This is especially true for applications with large datasets, complex business logic, or in production environments.

Migrations in Rails are primarily intended to evolve your database schema over time, making structural changes like creating and modifying tables or columns. When it comes to changing actual data, there are some reasons you might want to be cautious:

1\. Irreversibility: Data changes in migrations may be irreversible. If you change the data in a migration and then need to roll it back, you might lose data or end up in an inconsistent state.

2\. Performance: For large datasets, running data transformations within a migration could be very slow or even cause timeouts, particularly if the migration is run as part of a deployment process.

3\. Testing: Migrations typically aren't tested in the same way that your application code is, so bugs in data manipulations might not be caught until they've already affected your live data.

4\. No Access to Application Logic: Migrations run at the database level and don't have access to your application logic. This means you can't use models or services that might be part of your data manipulation needs.

So while the email lower-casing example is a simple one, in a real-world scenario you'd likely want to use a different approach, like a Rake task, a background job, or a script that you can test and run independently of your database migration process. This way, you can ensure your data is manipulated safely and efficiently.

**WHL: Is it a common design pattern in ORMs using migrations to change database rather than modifying schema?**

Yes, using migrations to make changes to the database is a common practice in many ORMs (Object-Relational Mappers), not just in Rails. Other frameworks like Django in Python, Hibernate in Java, or Sequelize in JavaScript also use a similar migration system.

The main reason for this approach is that changes to the database often need to be done incrementally and should be tracked over time, just like changes to the codebase. Migrations allow for versioning of the database schema, which makes it possible to move forward to a newer schema or backward to an older one. They provide a consistent way to create, change, and drop database tables across different development environments.

Directly modifying the schema (for example, by editing a \`schema.sql\` file) is not typically recommended because:

1\. It's error-prone. Directly editing a schema file can easily lead to syntax errors, and unlike migrations, these errors are not easy to track or revert.

2\. It doesn't capture the history of changes. With migrations, you can see when each change was made and in what order changes were applied.

3\. It can lead to inconsistencies. If different developers are making changes to the schema file at different times, it can be hard to keep everyone's local databases in sync.

That said, in some limited cases, directly modifying a database schema might be used. For example, some lightweight, file-based databases like SQLite allow you to edit the database file directly. Some developers might also manually write SQL scripts to change the database schema. However, these approaches don't provide the same benefits as a migration-based system and are generally not recommended for larger, collaborative projects.

**WHL: what is a good granularity of defining a migration? Shall it be covering just one table or doing as more change as possible?**

The granularity of migrations depends on the context and the nature of the changes you're making. However, here are a few best practices:

1\. One Logical Change Per Migration: Each migration should make one logical change to the database schema. For example, if you are adding a new feature that requires adding a table and adding a column to an existing table, these should be two separate migrations. This makes it easier to understand what each migration is doing and allows for more granular control when applying and rolling back changes.

2\. Keep Them Small: Small migrations are easier to understand and less likely to cause problems. If you have a big change to make, consider whether it can be broken down into smaller, incremental steps.

3\. Don't Mix Schema Changes with Data Manipulation: As discussed earlier, mixing schema changes with data manipulation in the same migration can lead to problems and should generally be avoided. It's usually better to handle data manipulation separately, for example, using a rake task or a script. <mark style="color:blue;">% 根本就不要用 migration 去改數據。20230621 %==

4\. Avoid Dependencies Between Migrations: Each migration should be able to run independently of other migrations. This means you shouldn't write a migration that depends on a specific column being present in a table unless that column is added in the same migration.

In the end, the key is to create migrations that are clear, understandable, and easily reversible, so that they can be managed effectively over the lifetime of your application.

**WHL: What is a good initial or the first migration? Shall it be as comprehensive or considerate as possible?**

ChatGPT: The first migration in a Rails application is often used to create the initial database schema. This might include creating the tables that your application will use, as well as defining any indices or constraints that should be in place.

A good first migration should:

1\. Create the necessary tables: Your first migration should create all the tables you know you'll need right away. Each table should have all the columns it needs, with the correct data types and any necessary constraints.

2\. Be as simple as possible: While it might be tempting to try and anticipate all the changes you'll need to make in the future and include them in your first migration, it's usually better to start with the minimum schema you need to get your application working. You can always add more tables, columns, indices, etc., in future migrations as your application's requirements evolve.\


Keep in mind that over time, your database schema will evolve and change. That's completely normal and to be expected. The goal of your initial migration should be to create a solid foundation that you can build on, not to predict every change you'll need to make in the future.

**WHL: If I use '`rails generate migration`' command to generate the initial migration, it might look like a very long command line. Is this a good practice? Or I'd better generate with multiple commands, each for only one table/model?**

ChatGPT: While you could technically generate an entire set of tables with one \`rails generate migration\` command, ==it's generally better practice to split your schema creation into multiple commands, each focused on one model or table.==

This has several advantages:

1\. Ease of Reading and Understanding: Having one migration per table makes the purpose of each migration file clear and easier to understand. Someone reading your code (including your future self) can quickly see what each migration does without having to parse a very long file.

2\. Granularity of Control: Each migration in Rails can be run, reversed, or re-run independently. If you generate separate migrations for each table, you can modify, rollback or remove tables independently without affecting other tables.

3\. Easier Troubleshooting: If there's an issue with your schema, it's easier to debug and fix when your changes are split across multiple files. If you make all changes in one file and there's a problem, you'll have to comb through that entire file to find it.

So, while generating a long migration command might work, it's generally better practice to create separate migrations for each table. You can use the \`rails generate model\` command to generate both the model and associated migration for each table, which will simplify your workflow.

For example, instead of:

rails generate migration CreateUsersAndPostsAndComments ...

You might do:

rails generate model User ...

rails generate model Post ...

rails generate model Comment ...

Each of these commands will create a separate migration file, along with the corresponding model file.

