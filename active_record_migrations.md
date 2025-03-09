# Active Record Migrations

This is an annotated version of the official Rails Guide.

Migrations are a feature of Active Record that allows you ==to evolve your database schema over time==. Rather than write schema modifications in pure SQL, migrations allow you to use a Ruby DSL to describe changes to your tables.

In Rails, you typically ==define the attributes of a model in a database migration, rather than in the model file itself.== When you create a new `LineItem` object, Rails will check the `line_items` table definition and automatically give the new `LineItem` object attributes that match the columns of the table. So even though you don't see a definition of `price` in the `LineItem` model file, Rails provides that attribute for you based on the database schema. You can access and modify these attributes just like any other property of the object.

For example, `price` and `quantity` would be columns in the `line_items` table in the database. ==Rails then automatically creates getter and setter methods for those columns, namely, Rails will infer their existence from the database schema.==

Migrations are subclasses of the Rails class `ActiveRecord::Migration`.

## 1 Migration Overview

Migrations are a convenient way ==to alter your database schema over time in a consistent way==. They use a Ruby DSL so that you don't have to write SQL by hand, allowing your schema and changes to be database independent.

==You can think of each migration as being a new 'version' of the database.== A schema starts off with nothing in it, and each migration modifies it to add or remove tables, columns, or entries. Active Record knows how to update your schema along this timeline, bringing it from whatever point it is in the history to the latest version. Active Record will also update your `db/schema.rb` file to match the up-to-date structure of your database.

Here's an example of a migration:

```ruby
class CreateProducts < ActiveRecord::Migration[7.1]
	def change
		create_table :products do |t|
			t.string :name
			t.text :description
			t.timestamps
		end
	end
end
```

This migration adds a table called `products` with a string column called `name` and a text column called `description`. A primary key column called `id` will also be added implicitly, as it's the default primary key for all Active Record models. The `timestamps` macro adds two columns, `created_at` and `updated_at`. These special columns are automatically managed by Active Record if they exist.

Note that we define the change that we want to happen moving forward in time. ==Before this migration is run, there will be no table. After, the table will exist.== Active Record knows how to reverse this migration as well: if we roll this migration back, it will remove the table.

On databases that support transactions with statements that change the schema, ==each migration is wrapped in a transaction.== If the database does not support this then when a migration fails the parts of it that succeeded will not be rolled back. You will have to rollback the changes that were made by hand.

There are certain queries that can't run inside a transaction. If your adapter supports DDL transactions you can use `disable_ddl_transaction!` to disable them for a single migration.

### 1.1 Making the Irreversible Possible

If you wish for a migration to do something that Active Record doesn't know how to reverse, you can use `reversible`:

```ruby
class ChangeProductsPrice < ActiveRecord::Migration[7.1]
	def change
		reversible do |direction|
			change_table :products do |t|
				direction.up   { t.change :price, :string }
				direction.down { t.change :price, :integer }
			end
		end
	end
end
```

This migration will change the type of the `price` column to a string, or back to an integer when the migration is reverted. Notice the block being passed to `direction.up` and `direction.down` respectively.

Alternatively, you can use `up` and `down` instead of `change`:

```ruby
class ChangeProductsPrice < ActiveRecord::Migration[7.1]
  def up
    change_table :products do |t|
      t.change :price, :string
    end
  end

  def down
    change_table :products do |t|
      t.change :price, :integer
    end
  end
end
```

### % Migration Naming %

==Migrations 也是延用了 Convention over Configuration 的原則。==

If the migration name is of the form "AddColumnToTable" or "RemoveColumnFromTable" and is followed by a list of column names and types, then a migration containing the appropriate `add_column` and `remove_column` statements will be created.

The name of the migration class (CamelCased version) should match the latter part of the file name. For example, `20080906120000_create_products.rb` should define class `CreateProducts` and `20080906120001_add_details_to_products.rb` should define class `AddDetailsToProducts`.

==% 我認為 migration的命名，應該用名詞而非動詞，因為這會產生一個 Class name。 例如：ProductsCreation，ProductsDetailsAddition。也就是TableColumnAction三部分。20240119 %==

## 2 Generating Migrations



The `model`, `resource`, and `scaffold` generators will create migrations appropriate for adding a new model.

```ruby
rails generate migration AddPartNumberToProducts part_number:string price:decimal
```

This will generate the following class:

```ruby
class AddPartNumberToProducts < ActiveRecord::Migration[7.0]
	def change
		add_column :products, :part_number, :string
		add_column :products, :price, :decimal
	end
end
```

References are a shorthand for creating columns, indexes, foreign keys, or even polymorphic association columns.

==% 一般地，我就統一使用 `change`，不使用 `up` 和 `down`。%==

### 2.1 Creating a Standalone Migration

Migrations are stored as files in the `db/migrate` directory, one for each migration class. The name of the file is of the form `YYYYMMDDHHMMSS_create_products.rb`, that is to say a UTC timestamp identifying the migration followed by an underscore followed by the name of the migration. The name of the migration class (CamelCased version) should match the latter part of the file name. For example `20080906120000_create_products.rb` should define class `CreateProducts` and `20080906120001_add_details_to_products.rb` should define `AddDetailsToProducts`. Rails uses this timestamp to determine which migration should be run and in what order, so if you're copying a migration from another application or generate a file yourself, be aware of its position in the order.

Of course, calculating timestamps is no fun, so Active Record provides a generator to handle making it for you:

```bash
$ bin/rails generate migration AddPartNumberToProducts
```

This will create an appropriately named empty migration:

```ruby
class AddPartNumberToProducts < ActiveRecord::Migration[7.1]
	def change
	end
end
```

This generator can do much more than prepend a timestamp to the file name. Based on naming conventions and additional (optional) arguments it can also start fleshing out the migration.

### 2.2 Adding New Columns

If the migration name is of the form "AddColumnToTable" or "RemoveColumnFromTable" and is followed by a list of column names and types then a migration containing the appropriate [`add_column`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html#method-i-add_column) and [`remove_column`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html#method-i-remove_column) statements will be created.

```bash
$ bin/rails generate migration AddPartNumberToProducts part_number:string
```

This will generate the following migration:

```ruby
class AddPartNumberToProducts < ActiveRecord::Migration[7.1]
	def change
		add_column :products, :part_number, :string
	end
end
```

If you'd like to add an index on the new column, you can do that as well.

```bash
$ bin/rails generate migration AddPartNumberToProducts part_number:string:index
```

This will generate the appropriate [`add_column`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html#method-i-add_column) and [`add_index`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html#method-i-add_index) statements:

```ruby
class AddPartNumberToProducts < ActiveRecord::Migration[7.1]
	def change
		add_column :products, :part_number, :string
		add_index :products, :part_number
	end
end
```

You are **not** limited to one magically generated column. For example:

```bash
$ bin/rails generate migration AddDetailsToProducts part_number:string price:decimal
```

Will generate a schema migration which adds two additional columns to the `products` table.

```ruby
class AddDetailsToProducts < ActiveRecord::Migration[7.1]
  def change
    add_column :products, :part_number, :string
    add_column :products, :price, :decimal
  end
end
```

### 2.2+ `add_index`

In a database, an **index** is a data structure that improves the speed of data retrieval operations on a table. Think of it as a “lookup table” that allows the database to find rows more quickly, much like an index in a book helps you locate information faster.

Without an index, the database has to perform a full table scan, which means it checks each row to find matches for a query. With an index, it can go directly to the relevant rows, making retrieval faster, especially in large datasets.

In Rails, `add_index` is used to create an index on one or more columns in your database table, improving query performance by making data retrieval faster for those columns. ==Indexes improve read performance but slow down insert, update, and delete operations==, so use them strategically where data retrieval is most frequent. Rails automatically generates a name for the index based on the table and column names. 

**Adding an index to a single column.** 

```ruby
add_index :table_name, :column_name
```

**Adding an Index to Multiple Columns.**

You can add an index on multiple columns to support compound queries. This is useful when queries commonly filter by both column1 and column2.

```ruby
add_index :table_name, [:column1, :column2]
```

**Unique Indexes**

If you want to ensure that the values in the indexed column(s) are unique, you can add a unique index to prevent duplicated values from being inserted into the column.

```ruby
add_index :table_name, :column_name, unique: true
```

**Conditional (Partial) Indexes**

For databases that support partial indexes (like PostgreSQL), you can create an index with a condition. This is useful if you want to index only a subset of records, like only those that are active.

```ruby
add_index :table_name, :column_name, where: "active = true"
```

**7. Index with if_not_exists Option (Rails 5+)**

To avoid errors if the index already exists, you can use if_not_exists:

add_index :table_name, :column_name, if_not_exists: true



**Example of a Migration Using add_index**

Here’s how you might add several indexes in a migration:

```ruby
class AddIndexesToTable < ActiveRecord::Migration[8.0]
	def change
		add_index :users, :email, unique: true
		add_index :posts, [:user_id, :created_at]
		add_index :orders, :status, where: "status = 'completed'"
		add_index :customers, "LOWER(email)", name: "index_customers_on_lower_email"
    end
end
```



**Dropping an Index**

To remove an index, use remove_index with similar syntax:

remove_index :table_name, :column_name



### 2.3 Removing Columns

Similarly, you can generate a migration to remove a column from the command line:

```
$ bin/rails generate migration RemovePartNumberFromProducts part_number:string
```

This generates the appropriate [`remove_column`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html#method-i-remove_column) statements:

```
class RemovePartNumberFromProducts < ActiveRecord::Migration[7.1]
  def change
    remove_column :products, :part_number, :string
  end
end
```

### [2.4 Creating New Tables](https://guides.rubyonrails.org/active_record_migrations.html#creating-new-tables)

If the migration name is of the form "CreateXXX" and is followed by a list of column names and types then a migration creating the table XXX with the columns listed will be generated. For example:

```
$ bin/rails generate migration CreateProducts name:string part_number:string
```

generates

```ruby
class CreateProducts < ActiveRecord::Migration[7.1]
  def change
    create_table :products do |t|
      t.string :name
      t.string :part_number

      t.timestamps
    end
  end
end
```

As always, what has been generated for you is just a starting point. You can add or remove from it as you see fit by editing the `db/migrate/YYYYMMDDHHMMSS_add_details_to_products.rb` file.

### [2.5 Creating associations using references](https://guides.rubyonrails.org/active_record_migrations.html#creating-associations-using-references)

Also, the generator accepts ==column type as `references`== (also available as `belongs_to`). For example,

```bash
$ bin/rails generate migration AddUserRefToProducts user:references
```

generates the following [`add_reference`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html#method-i-add_reference) call:

```ruby
class AddUserRefToProducts < ActiveRecord::Migration[7.1]
	def change
		add_reference :products, :user, foreign_key: true
	end
end
```

This migration will create a user_id column. References are a shorthand for creating columns, indexes, foreign keys, or even polymorphic association columns.

There is also a generator which will produce join tables if `JoinTable` is part of the name:

```bash
$ bin/rails generate migration CreateJoinTableCustomerProduct customer product
```

will produce the following migration:

```ruby
class CreateJoinTableCustomerProduct < ActiveRecord::Migration[7.1]
	def change
		create_join_table :customers, :products do |t|
			# t.index [:customer_id, :product_id]
			# t.index [:product_id, :customer_id]
		end
	end
end
```

### 2.6 Model Generators

The model, resource, and scaffold generators will create migrations appropriate for adding a new model. This migration will already contain instructions for creating the relevant table. If you tell Rails what columns you want, then statements for adding these columns will also be created. For example, running:

```
$ bin/rails generate model Product name:string description:text
```

This will create a migration that looks like this:

```
class CreateProducts < ActiveRecord::Migration[7.1]
  def change
    create_table :products do |t|
      t.string :name
      t.text :description

      t.timestamps
    end
  end
end
```

You can append as many column name/type pairs as you want.

### 2.7 Passing Modifiers

Some commonly used type modifiers can be passed directly on the command line. They are enclosed by curly braces and follow the field type:

For instance, running:

```bash
$ bin/rails generate migration AddDetailsToProducts 'price:decimal{5,2}' supplier:references{polymorphic}
```

will produce a migration that looks like this

```ruby
class AddDetailsToProducts < ActiveRecord::Migration[7.1]
	def change
		add_column :products, :price, :decimal, precision: 5, scale: 2
		add_reference :products, :supplier, polymorphic: true
	end
end
```

Have a look at the generators help output (`bin/rails generate --help`) for further details.

## 3 Writing Migrations

Once you have created your migration using one of the generators it's time to get to work!

### 3.1 Creating a Table

The `create_table` method is one of the most fundamental, but most of the time, will be generated for you from using a model, resource, or scaffold generator. A typical use would be

```ruby
create_table :products do |t|
	t.string :name
end
```

This method creates a `products` table with a column called `name`.

==By default, `create_table` will implicitly create a primary key called `id` for you. You can change the name of the column with the `:primary_key` option, or pass an array to `:primary_key` for a composite primary key. If you don't want a primary key at all, you can pass the option `id: false`.==

If you need to pass database-specific options you can place an SQL fragment in the `:options` option. For example:

```ruby
create_table :products, options: "ENGINE=BLACKHOLE" do |t|
	t.string :name, null: false
end
```

This will append `ENGINE=BLACKHOLE` to the SQL statement used to create the table.

An index can be created on the columns created within the `create_table` block by passing `index: true` or an options hash to the `:index` option:

```ruby
create_table :users do |t|
	t.string :name, index: true
	t.string :email, index: { unique: true, name: 'unique_emails' }
end
```

Also, you can pass the `:comment` option with any description for the table that will be stored in the database itself and can be viewed with database administration tools, such as MySQL Workbench or PgAdmin III. It's highly recommended to specify comments in migrations for applications with large databases as it helps people to understand the data model and generate documentation. Currently only the MySQL and PostgreSQL adapters support comments.

### 3.2 Creating a Join Table

The migration method `create_join_table` creates an HABTM (has and belongs to many) join table. A typical use would be:

```ruby
create_join_table :products, :categories
```

This migration will create a `categories_products` table with two columns called `category_id` and `product_id`.

==These columns have the option `:null` set to `false` by default, meaning that you **must** provide a value in order to save a record to this table.== This can be overridden by specifying the `:column_options` option:

```ruby
create_join_table :products, :categories, column_options: { null: true }
```

By default, the name of the join table comes from the union of the first two arguments provided to create_join_table, in alphabetical order.

To customize the name of the table, provide a `:table_name` option:

```ruby
create_join_table :products, :categories, table_name: :categorization
```

This ensure the name of the join table is `categorization` as requested.

Also, `create_join_table` accepts a block, which you can use to add indices (which are not created by default) or any additional columns you so choose.

```ruby
create_join_table :products, :categories do |t|
	t.index :product_id
	t.index :category_id
end
```

### 3.3 Changing Tables

If you want to change an existing table in place, there is `change_table`.

It is used in a similar fashion to `create_table` but the object yielded inside the block has access to a number of special functions, for example:

```ruby
change_table :products do |t|
	t.remove :description, :name
	t.string :part_number
	t.index :part_number
	t.rename :upccode, :upc_code
end
```

This migration will remove the `description` and `name` columns, create a new string column called `part_number` and adds an index on it. Finally it renames the `upccode` column to `upc_code`.

### 3.4 Changing Columns

Similar to the `remove_column` and `add_column` methods, Rails also provides the `change_column` migration method.

```ruby
change_column :products, :part_number, :text
```

This changes the column `part_number` on products table to be a `:text` field.

The `change_column` command is **irreversible**. You should provide your own `reversible` migration.

==Besides `change_column`, the `change_column_null` and `change_column_default` methods are used specifically to change a null constraint and default values of a column.==

```ruby
change_column_null :products, :name, false
change_column_default :products, :approved, from: true, to: false
```

This sets `:name` field on products to a `NOT NULL` column and the default value of the `:approved` field from true to false. Both of these changes will only be applied to future transactions, any existing records do not apply.

When setting the null constraint to true, this means that column will accept a null value, otherwise the `NOT NULL` constraint is applied and a value must be passed in order to persist the record to the database.

You could also write the above `change_column_default` migration as `change_column_default :products, :approved, false`, but unlike the previous example, this would make your migration irreversible.

### 3.5 Column Modifiers

Column modifiers can be applied when creating or changing a column:

- `comment` Adds a comment for the column.
- `collation` 校對 Specifies the collation for a `string` or `text` column.
- `default` Allows to set a default value on the column. Note that if you are using a dynamic value (such as a date), the default will only be calculated the first time (i.e. on the date the migration is applied). Use `nil` for `NULL`.
- `limit` Sets the maximum number of characters for a `string` column and the maximum number of bytes for `text/binary/integer` columns.
- `null` Allows or disallows `NULL` values in the column.
- `precision` Specifies the precision for `decimal/numeric/datetime/time` columns.
- `scale` Specifies the scale for the `decimal` and `numeric` columns, representing the number of digits after the decimal point.

For `add_column` or `change_column` there is no option for adding indexes. They need to be added separately using `add_index`.

Some adapters may support additional options; see the adapter specific API docs for further information.

`null` and `default` cannot be specified via command line when generating migrations.

### 3.6 References

The `add_reference` method allows the creation of an appropriately named column acting as the connection between one or more associations.

```ruby
add_reference :users, :role
```

==This migration will create a `role_id` column in the users table. It creates an index for this column as well==, unless explicitly told not to with the `index: false` option.

==The method `add_belongs_to` is an alias of `add_reference`.==

```ruby
add_belongs_to :taggings, :taggable, polymorphic: true
```

The polymorphic option will create two columns on the `taggings` table which can be used for polymorphic associations: `taggable_type` and `taggable_id`.

A foreign key can be created with the `foreign_key` option.

```ruby
add_reference :users, :role, foreign_key: true
```

References can also be removed:

```ruby
remove_reference :products, :user, foreign_key: true, index: false
```

### 3.7 Foreign Keys

==While it's not required, you might want to add foreign key constraints to guarantee referential integrity.==

```ruby
add_foreign_key :articles, :authors
```

This `add_foreign_key` call adds a new constraint to the articles table. The constraint guarantees that a row in the `authors` table exists where the `id` column matches the `articles.author_id`.

If the `from_table` column name cannot be derived from the `to_table` name, you can use the `:column` option. Use the `:primary_key` option if the referenced primary key is not `:id`.

For example, to add a foreign key on `articles.reviewer` referencing `authors.email`:

```ruby
add_foreign_key :articles, :authors, column: :reviewer, primary_key: :email
```

This will add a constraint to the `articles` table that guarantees a row in the `authors` table exists where the `email` column matches the `articles.reviewer` field.

Several other options such as `name`, `on_delete`, `if_not_exists`, `validate`, and `deferrable` are supported by `add_foreign_key`.

Foreign keys can also be removed using `remove_foreign_key`:

```ruby
# let Active Record figure out the column name
remove_foreign_key :accounts, :branches

# remove foreign key for a specific column
remove_foreign_key :accounts, column: :owner_id
```

Active Record only supports single column foreign keys. `execute` and `structure.sql` are required to use composite foreign keys. 

#### Relfect on Foreign Keys

Foreign keys are a bit like that well-intentioned friend who insists on double-checking everything you do. They’re often recommended as a must-have for enforcing referential integrity checks in your database, yet their impact on performance, migrations, and complexity is hard to ignore.

When a record in a referenced table is updated or deleted, databases must wait for any locks on the corresponding rows in the referencing table to be released to validate the integrity. This process can slow down write operations, especially in systems with high transaction volumes. To speed up these referential integrity checks, you might add an index on the foreign key column. But index additional overhead - every insert, update, or delete on the referencing table now requires updating the index. 

Foreign keys have a more direct impact on write operations than on reads. If you care about ensuring the relationship between the the tables are intact, but say just during writes. You can ensure them from a `before_create` hook in Rails. 

As you start on designing a new project, it’s fine to initially incorporate foreign keys. Plan to revisit this decision a few months later to evaluate their usefulness. 

==% 但是，我认为加不加 Foreign Keys，首先是由业务逻辑决定的，而不是由performance决定的。 这些检查，就是为了保证反映业务逻辑的数据是没出错的。 20240202 %==

### 3.8 Composite Primary Keys

==Sometimes a single column's value isn't enough to uniquely identify every row of a table, but a combination of two or more columns *does* uniquely identify it.== This can be the case when using a legacy database schema without a single `id` column as a primary key, or when altering schemas for sharding or multitenancy.

You can create a table with a composite primary key by passing the `:primary_key` option to `create_table` with an array value:

```ruby
class CreateProducts < ActiveRecord::Migration[7.1]
	def change
		create_table :products, primary_key: [:customer_id, :product_sku] do |t|
			t.integer :customer_id
			t.string :product_sku
			t.text :description
		end
	end
end
```

Tables with composite primary keys require passing array values rather than integer IDs to many methods. 

### [3.9 When Helpers aren't Enough](https://guides.rubyonrails.org/active_record_migrations.html#when-helpers-aren-t-enough)

If the helpers provided by Active Record aren't enough you can use the [`execute`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/ConnectionAdapters/DatabaseStatements.html#method-i-execute) method to execute arbitrary SQL:

```ruby
Product.connection.execute("UPDATE products SET price = 'free' WHERE 1=1")
```

For more details and examples of individual methods, check the API documentation. In particular the documentation for `ActiveRecord::ConnectionAdapters::SchemaStatements`, which provides the methods available in the change, up and down methods.

For methods available regarding the object yielded by `create_table`, see [`ActiveRecord::ConnectionAdapters::TableDefinition`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/ConnectionAdapters/TableDefinition.html).

And for the object yielded by `change_table`, see [`ActiveRecord::ConnectionAdapters::Table`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/ConnectionAdapters/Table.html).

### [3.10 Using the `change` Method](https://guides.rubyonrails.org/active_record_migrations.html#using-the-change-method)

The `change` method is the primary way of writing migrations. It works for the majority of cases in which Active Record knows how to reverse a migration's actions automatically. Below are some of the actions that `change` supports:

- add_check_constraint
- add_column
- add_foreign_key
- add_index
- add_reference
- add_timestamps
- change_column_comment (must supply :from and :to options)
- change_column_default (must supply :from and :to options)
- change_column_null
- change_table_comment (must supply :from and :to options)
- create_join_table
- create_table
- disable_extension
- drop_join_table
- drop_table (must supply table creation options and block)
- enable_extension
- remove_check_constraint (must supply original constraint expression)
- remove_column (must supply original type and column options)
- remove_columns (must supply original type and column options)
- remove_foreign_key (must supply other table and original options)
- remove_index (must supply columns and original options)
- remove_reference (must supply original options)
- remove_timestamps (must supply original options)
- rename_column
- rename_index
- rename_table

`change_table` is also reversible, as long as the block only calls reversible operations like the ones listed above. If you're going to need to use any other methods, you should use reversible or write the up and down methods instead of using the change method.

### [3.11 Using `reversible`](https://guides.rubyonrails.org/active_record_migrations.html#using-reversible)

Complex migrations may require processing that Active Record doesn't know how to reverse. You can use [`reversible`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Migration.html#method-i-reversible) to specify what to do when running a migration and what else to do when reverting it. For example:

```
class ExampleMigration < ActiveRecord::Migration[7.1]
  def change
    create_table :distributors do |t|
      t.string :zipcode
    end

    reversible do |direction|
      direction.up do
        # create a distributors view
        execute <<-SQL
          CREATE VIEW distributors_view AS
          SELECT id, zipcode
          FROM distributors;
        SQL
      end
      direction.down do
        execute <<-SQL
          DROP VIEW distributors_view;
        SQL
      end
    end

    add_column :users, :address, :string
  end
end
```



Using `reversible` will ensure that the instructions are executed in the right order too. If the previous example migration is reverted, the `down` block will be run after the `users.address`column is removed and before the `distributors` table is dropped.

### [3.12 Using the `up`/`down` Methods](https://guides.rubyonrails.org/active_record_migrations.html#using-the-up-down-methods)

You can also use the old style of migration using `up` and `down` methods instead of the `change`method.

The `up` method should describe the transformation you'd like to make to your schema, and the `down` method of your migration should revert the transformations done by the `up` method. In other words, the database schema should be unchanged if you do an `up` followed by a `down`.

For example, if you create a table in the `up` method, you should drop it in the `down` method. It is wise to perform the transformations in precisely the reverse order they were made in the `up` method. The example in the `reversible` section is equivalent to:

```
class ExampleMigration < ActiveRecord::Migration[7.1]
  def up
    create_table :distributors do |t|
      t.string :zipcode
    end

    # create a distributors view
    execute <<-SQL
      CREATE VIEW distributors_view AS
      SELECT id, zipcode
      FROM distributors;
    SQL

    add_column :users, :address, :string
  end

  def down
    remove_column :users, :address

    execute <<-SQL
      DROP VIEW distributors_view;
    SQL

    drop_table :distributors
  end
end
```

### [3.13 Throwing an error to prevent reverts](https://guides.rubyonrails.org/active_record_migrations.html#throwing-an-error-to-prevent-reverts)

Sometimes your migration will do something which is just plain irreversible; for example, it might destroy some data.

In such cases, you can raise `ActiveRecord::IrreversibleMigration` in your `down` block.

If someone tries to revert your migration, an error message will be displayed saying that it can't be done.

### [3.14 Reverting Previous Migrations](https://guides.rubyonrails.org/active_record_migrations.html#reverting-previous-migrations)

You can use Active Record's ability to rollback migrations using the [`revert`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Migration.html#method-i-revert) method:

```
require_relative "20121212123456_example_migration"

class FixupExampleMigration < ActiveRecord::Migration[7.1]
  def change
    revert ExampleMigration

    create_table(:apples) do |t|
      t.string :variety
    end
  end
end
```

The `revert` method also accepts a block of instructions to reverse. This could be useful to revert selected parts of previous migrations.

For example, let's imagine that `ExampleMigration` is committed and it is later decided that a Distributors view is no longer needed.

```
class DontUseDistributorsViewMigration < ActiveRecord::Migration[7.1]
  def change
    revert do
      # copy-pasted code from ExampleMigration
      reversible do |direction|
        direction.up do
          # create a distributors view
          execute <<-SQL
            CREATE VIEW distributors_view AS
            SELECT id, zipcode
            FROM distributors;
          SQL
        end
        direction.down do
          execute <<-SQL
            DROP VIEW distributors_view;
          SQL
        end
      end

      # The rest of the migration was ok
    end
  end
end
```

The same migration could also have been written without using `revert` but this would have involved a few more steps:

1. Reverse the order of `create_table` and `reversible`.
2. Replace `create_table` with `drop_table`.
3. Finally, replace `up` with `down` and vice-versa.

This is all taken care of by `revert`.

## 4 Running Migrations

Rails provides a set of commands to run certain sets of migrations.

The very first migration related rails command you will use will probably be `bin/rails db:migrate`. In its most basic form it just runs the `change` or `up` method for all the migrations that have not yet been run. If there are no such migrations, it exits. It will run these migrations in order based on the date of the migration.

Note that running the `db:migrate` command also invokes the `db:schema:dump` command, which will update your `db/schema.rb` file to match the structure of your database.

If you specify a target version, Active Record will run the required migrations (`change`, `up`, `down`) until it has reached the specified version. The version is the numerical prefix on the migration's filename. For example, to migrate to version 20080906120000 run:

```bash
$ bin/rails db:migrate VERSION=20080906120000
```

If version 20080906120000 is greater than the current version (i.e., it is migrating upwards), this will run the `change` (or `up`) method on all migrations up to and including 20080906120000, and will not execute any later migrations. If migrating downwards, this will run the `down` method on all the migrations down to, but not including, 20080906120000.

### 4.1 Rolling Back

Active Record knows how to reverse the migration as well: if we roll this migration back, it will remove the table. 

A common task is to rollback the last migration. If you made a mistake in it and wish to correct it, rather than tracking down the version number associated with the previous migration you can run:

```bash
$ bin/rails db:rollback
```

==The `rails db:rollback`  will rollback the latest migration==, either by reverting the `change` method or by running the `down` method. If you need to undo several migrations you can provide a `STEP` parameter:

```bash
$ bin/rails db:rollback STEP=3
```

The last 3 migrations will be reverted.

The `db:migrate:redo` command is a shortcut for doing a rollback and then migrating back up again. As with the `db:rollback` command, you can use the `STEP` parameter if you need to go more than one version back, for example:

```bash
$ bin/rails db:migrate:redo STEP=3
```

Neither of these rails commands do anything you could not do with `db:migrate`. They are there for convenience, since you do not need to explicitly specify the version to migrate to.

### 4.2 Setup the Database

The `bin/rails db:setup` command will create the database, load the schema, and initialize it with the seed data.

### 4.3 Preparing the Database

The `bin/rails db:prepare` command is similar to `bin/rails db:setup`, but it operates idempotently.

- If the database has not been created yet, the command will run as the `bin/rails db:setup` does.
- If the database exists but the tables have not been created, the command will load the schema, run any pending migrations, dump the updated schema, and finally load the seed data.
- If both the database and tables exist but the seed data has not been loaded, the command will only load the seed data.
- If the database, tables, and seed data are all in place, the command will do nothing.

Once the database, tables, and seed data are all established, the command will not try to reload the seed data, even if the previously loaded seed data or the existing seed file have been altered or deleted. To reload the seed data, you can manually run `bin/rails db:seed`.

### [4.4 Resetting the Database](https://guides.rubyonrails.org/active_record_migrations.html#resetting-the-database)

==The `bin/rails db:reset` command will drop the database and set it up again. This is functionally equivalent to `bin/rails db:drop db:setup`.==

This is not the same as running all the migrations. It will only use the contents of the current db/schema.rb or db/structure.sql file. If a migration can't be rolled back, bin/rails db:reset may not help you. To find out more about dumping the schema see Schema Dumping and You section.

### [4.5 Running Specific Migrations](https://guides.rubyonrails.org/active_record_migrations.html#running-specific-migrations)

If you need to run a specific migration up or down, the `db:migrate:up` and `db:migrate:down`commands will do that. Just specify the appropriate version and the corresponding migration will have its `change`, `up` or `down` method invoked, for example:

```
$ bin/rails db:migrate:up VERSION=20080906120000
```

By running this command the `change` method (or the `up` method) will be executed for the migration with the version "20080906120000".

First, this command will check whether the migration exists and if it has already been performed and will do nothing if so. If the version specified does not exist, Rails will throw an exception.

```bash
$ bin/rails db:migrate VERSION=zomg
rails aborted!
ActiveRecord::UnknownMigrationVersionError:

No migration with version number zomg.
```

### [4.6 Running Migrations in Different Environments](https://guides.rubyonrails.org/active_record_migrations.html#running-migrations-in-different-environments)

==By default running `bin/rails db:migrate` will run in the `development` environment.== To run migrations against another environment you can specify it using the `RAILS_ENV` environment variable while running the command. For example to run migrations against the `test` environment you could run:

```bash
$ bin/rails db:migrate RAILS_ENV=test
```

### 4.7 Changing the Output of Running Migrations

By default migrations tell you exactly what they're doing and how long it took. A migration creating a table and adding an index might produce output like this

```ruby
==  CreateProducts: migrating =================================================
-- create_table(:products)
   -> 0.0028s
==  CreateProducts: migrated (0.0028s) ========================================
```

Several methods are provided in migrations that allow you to control all this:

| Method                                                       | Purpose                                                      |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [`suppress_messages`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Migration.html#method-i-suppress_messages) | Takes a block as an argument and suppresses any output generated by the block. |
| [`say`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Migration.html#method-i-say) | Takes a message argument and outputs it as is. A second boolean argument can be passed to specify whether to indent or not. |
| [`say_with_time`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Migration.html#method-i-say_with_time) | Outputs text along with how long it took to run its block. If the block returns an integer it assumes it is the number of rows affected. |

For example, take the following migration:

```ruby
class CreateProducts < ActiveRecord::Migration[7.1]
	def change
		suppress_messages do
			create_table :products do |t|
				t.string :name
				t.text :description
				t.timestamps
			end
		end
          
		say "Created a table"
		suppress_messages { add_index :products, :name }
		say "and an index!", true

		say_with_time 'Waiting for a while' do
			sleep 10
			250
		end
	end
end
```

This will generate the following output:

```ruby
==  CreateProducts: migrating =================================================
-- Created a table
   -> and an index!
-- Waiting for a while
   -> 10.0013s
   -> 250 rows
==  CreateProducts: migrated (10.0054s) =======================================
```

If you want Active Record to not output anything, then running `bin/rails db:migrate VERBOSE=false` will suppress all output.

## 5 Changing Existing Migrations

Occasionally you will make a mistake when writing a migration. If you have already run the migration, then you cannot just edit the migration and run the migration again: Rails thinks it has already run the migration and so will do nothing when you run `bin/rails db:migrate`. ==You must rollback the migration (for example with `bin/rails db:rollback`), edit your migration, and then run `bin/rails db:migrate` again to run the corrected version.==

In general, editing existing migrations is not a good idea. You will be creating extra work for yourself and your co-workers and cause major headaches if the existing version of the migration has already been run on production machines.

Instead, you should write a new migration that performs the changes you require. ==Editing a freshly generated migration that has not yet been committed to source control (or, more generally, which has not been propagated beyond your development machine) is relatively harmless.==

The `revert` method can be helpful when writing a new migration to undo previous migrations in whole or in part.

## 6 Schema Dumping and You

### 6.1 What are Schema Files for?

Migrations, mighty as they may be, are not the authoritative source for your database schema. ==Your database remains the source of truth.== By default, Rails generates `db/schema.rb` which attempts to capture the current state of your database schema.

It tends to be faster and less error prone to create a new instance of your application's database by loading the schema file via `bin/rails db:schema:load` than it is to replay the entire migration history. [Old migrations](https://guides.rubyonrails.org/active_record_migrations.html#old-migrations) may fail to apply correctly if those migrations use changing external dependencies or rely on application code which evolves separately from your migrations.

Schema files are also useful if you want a quick look at what attributes an Active Record object has. This information is not in the model's code and is frequently spread across several migrations, but the information is nicely summed up in the schema file.

### 6.2 Types of Schema Dumps

The format of the schema dump generated by Rails is controlled by the `config.active_record.schema_format` setting defined in config/application.rb. By default, the format is `:ruby`, or alternatively can be set to `:sql`.

#### 6.2.1 Using the default :ruby schema

==When `:ruby` is selected, then the schema is stored in `db/schema.rb`.== If you look at this file you'll find that it looks an awful lot like one very big migration:

```ruby
ActiveRecord::Schema[7.1].define(version: 2008_09_06_171750) do
	create_table "authors", force: true do |t|
		t.string   "name"
		t.datetime "created_at"
		t.datetime "updated_at"
	end
	create_table "products", force: true do |t|
		t.string   "name"
		t.text     "description"
		t.datetime "created_at"
		t.datetime "updated_at"
		t.string   "part_number"
	end
end
```

In many ways this is exactly what it is. ==This file is created by inspecting the database and expressing its structure using `create_table`, `add_index`, and so on.==

#### [6.2.2 Using the `:sql` schema dumper](https://guides.rubyonrails.org/active_record_migrations.html#using-the-sql-schema-dumper)

However, `db/schema.rb` cannot express everything your database may support such as triggers, sequences, stored procedures, etc.

While migrations may use `execute` to create database constructs that are not supported by the Ruby migration DSL, these constructs may not be able to be reconstituted by the schema dumper.

If you are using features like these, you should set the schema format to `:sql` in order to get an accurate schema file that is useful to create new database instances.

When the schema format is set to `:sql`, the database structure will be dumped using a tool specific to the database into `db/structure.sql`. For example, for PostgreSQL, the `pg_dump` utility is used. For MySQL and MariaDB, this file will contain the output of `SHOW CREATE TABLE` for the various tables.

To load the schema from `db/structure.sql`, run `bin/rails db:schema:load`. Loading this file is done by executing the SQL statements it contains. By definition, this will create a perfect copy of the database's structure.

### 6.3 Schema Dumps and Source Control

Because schema files are commonly used to create new databases, it is strongly recommended that you check your schema file into source control.

Merge conflicts can occur in your schema file when two branches modify schema. To resolve these conflicts run `bin/rails db:migrate` to regenerate the schema file.

Newly generated Rails apps will already have the migrations folder included in the git tree, so all you have to do is be sure to add any new migrations you add and commit them.

## 7 Active Record and Referential Integrity

==The Active Record way claims that intelligence belongs in your models, not in the database. As such, features such as triggers or constraints, which push some of that intelligence back into the database, are not recommended.==

Validations such as `validates :foreign_key`, `uniqueness: true` are one way in which models can enforce data integrity. The `:dependent` option on associations allows models to automatically destroy child objects when the parent is destroyed. Like anything which operates at the application level, these cannot guarantee referential integrity and so some people augment them with foreign key constraints in the database.

Although Active Record does not provide all the tools for working directly with such features, the `execute` method can be used to execute arbitrary SQL.

## 8 Migrations and Seed Data

==The main purpose of Rails' migration feature is to issue commands that modify the schema using a consistent process.== Migrations can also be used to add or modify data. This is useful in an existing database that can't be destroyed and recreated, such as a production database.

```ruby
class AddInitialProducts < ActiveRecord::Migration[7.1]
	def up
		5.times do |i|
			Product.create(name: "Product ##{i}", description: "A product.")
		end
	end
	def down
		Product.delete_all
	end
end
```

To add initial data after a database is created, Rails has a built-in 'seeds' feature that speeds up the process. This is especially useful when reloading the database frequently in development and test environments, or when setting up initial data for production.

To get started with this feature, open up `db/seeds.rb` and add some Ruby code, then run `bin/rails db:seed`.

The code here should be idempotent so that it can be executed at any point in every environment.

```ruby
["Action", "Comedy", "Drama", "Horror"].each do |genre_name|
      MovieGenre.find_or_create_by!(name: genre_name)
end
```

This is generally a much cleaner way to set up the database of a blank application.

### % Using gem seed\_dump %

https://github.com/rroblak/seed_dump

Add it to your Gemfile with: `gem 'seed_dump'`

```ruby
#  Dump all data directly to db/seeds.rb:
rake db:seed:dump

# Append to db/seeds.rb instead of overwriting it:
rake db:seed:dump APPEND=true

# Dump only data from the users table and dump a maximum of 1 record:
$ rake db:seed:dump MODELS=User LIMIT=1

# Use another output file instead of db/seeds.rb:
rake db:seed:dump FILE=db/seeds/users.rb
```

## 9 Old Migrations

The `db/schema.rb` or `db/structure.sql` is a snapshot of the current state of your database and is the authoritative source for rebuilding that database. This makes it possible to delete or prune old migration files.

When you delete migration files in the `db/migrate/` directory, any environment where `bin/rails db:migrate` was run when those files still existed will hold a reference to the migration timestamp specific to them inside an internal Rails database table named `schema_migrations`. This table is used to keep track of whether migrations have been executed in a specific environment.

If you run the `bin/rails db:migrate:status` command, which displays the status (up or down) of each migration, you should see `********** NO FILE **********` displayed next to any deleted migration file which was once executed on a specific environment but can no longer be found in the `db/migrate/` directory.

### 9.1 Migrations from Engines

There's a caveat, though with [Engines](https://guides.rubyonrails.org/engines.html). Rake tasks to install migrations from engines are idempotent, meaning they will have the same result no matter how many times they are called. Migrations present in the parent application due to a previous installation are skipped, and missing ones are copied with a new leading timestamp. If you deleted old engine migrations and ran the install task again, you'd get new files with new timestamps, and `db:migrate` would attempt to run them again.

Thus, you generally want to preserve migrations coming from engines. They have a special comment like this:

```ruby
# This migration comes from blorgh (originally 20210621082949)
```







### Purpose of [8.0] in `ActiveRecord::Migration[8.0]`

In Rails, migration syntax and functionality can vary between major versions. To ensure consistent behaviour and avoid issues when upgrading Rails, you specify the migration version. 

The [8.0] in ActiveRecord::Migration[8.0] specifies the **version of the ActiveRecord migration class** that the migration should use, and it is indeed legal syntax in Ruby. This syntax is known as a **parameterised constant** or **generic class**, allowing you to reference a specific version of ActiveRecord’s migration behaviour. Here’s why and how it works:

​	1.**Version-Specific Behaviour**

Each version of ActiveRecord::Migration may include different methods or behaviours, tailored to the version of Rails you’re working with. For example, Rails 7 and Rails 8 migrations might handle certain types or validations differently. By specifying [8.0], you lock the migration to Rails 8 behaviour.

​	2.**Backward Compatibility**

If you upgrade a Rails application from, say, version 6 to version 8, previous migrations can still rely on ActiveRecord::Migration[6.0] or ActiveRecord::Migration[7.0]. This ensures that older migrations run as expected even on newer Rails versions.

​	3.**Migration Stability**

It helps prevent errors that could arise if the internal migration methods change across Rails versions. By locking the migration to a specific version, you ensure that it remains stable and unaffected by changes in future Rails releases.



**Is [8.0] Legal Syntax in Ruby?**

Yes, [8.0] is valid in Ruby when used with classes, thanks to Ruby’s **class bracket syntax**. This allows a constant (class or module) to accept parameters inside square brackets. In the case of ActiveRecord::Migration, Rails defines [] as a class method to handle versioning.

When creating a migration with Rails 8, it might look like this:

```ruby
class AddIndexToUsersEmail < ActiveRecord::Migration[8.0]
	def change
		add_index :users, :email, unique: true
	end
end
```

Here, [8.0] tells Rails to use ActiveRecord migration methods and behaviour from Rails 8.

**How Does This Work Internally?**

Internally, ==ActiveRecord::Migration defines a [] method that takes the version number as an argument and returns a specific migration class, essentially like a factory for versioned migration classes.== So, when you write ActiveRecord::Migration[8.0], it returns the correct migration class for version 8.0, which Ruby interprets without issues.