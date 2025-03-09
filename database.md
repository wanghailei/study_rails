# Database

## ==Using the same database for all environments.==

It's generally not recommended to use different databases for development and production in a Rails app. Something that works in SQLite may not work in PostgreSQL and vice versa. Using different databases can expose your app to potential database-specific bugs only in production.

## Initialise PostgreSQL on macOS

### Install PostgreSQL:

brew install postgresql@16

You will need to manage the server on its own, setting up the user(s) and database(s) before connecting them to Rails.

Homebrew starts the server and creates a default database named `postgres` automatically upon installation.

You can check your server status by:

```bash
brew services list
```

### Start, stop or restart a Postgre server:

```bash
brew services start postgresql@16
brew services stop postgresql@16
brew services restart postgresql@16
```

If you don't want/need a background service you can just run:

```bash
/opt/homebrew/opt/postgresql@16/bin/postgres -D /opt/homebrew/var/postgresql@16
```

## Create databases for a Rails app

With the "psql" tool.

```bash
psql postgres
```

#### Create databases for Rails development and test.

```pos
CREATE DATABASE appname\_development;
CREATE DATABASE appname\_test;
```

#### Create a database superuser and set password.

```po
CREATE USER dillone WITH SUPERUSER ENCRYPTED PASSWORD 'Mac2....._\_';

ALTER DATABASE appname\_development OWNER TO dillone;
ALTER DATABASE appname\_test OWNER TO dillone;
```

#### Copy data from SQLite3 to PostgreSQL with "sequel" tool.

Make sure the PostgreSQL database is empty before running the sequel command.

```bash
bundle exec sequel -C sqlite://db/development.sqlite3 postgres://dillone:Mac2Milan\_\_@localhost/depot\_development
```

### `ActiveRecord::NoDatabaseError`

在T003實驗中，rails new 和 bundle install 之後，rails sever，會收到這個錯誤信息。因為，我沒有創建 PostgreSQL數據庫。

```ruby
We could not find your database: tiny2_development. 
Which can be found in the database configuration file located at config/database.yml.
```

點擊網頁上的一個按鈕就可以了 "Create database"。

並且，有一個提示：To create your database, run: `bin/rails db:create`

我做了T003實驗，在完成T002實驗的步驟後，我運行了 `bin/rails db:create`。此時再 `bin/rails server`，網頁運行正常。

### Configuring Databases for a Rails app

The `rails db:setup` command will create the database, load the schema, and initialise it with the seed data.

database.yml for configuring database connection.

You'll need to specify your PostgreSQL username, password, host, and database name for the development, test, and production environments.

Make sure not to include sensitive data like your database password directly in the database.yml file. Instead, use environment variables to store sensitive data.

```yml	
default: \&default
	adapter: postgresql
	encoding: unicode
	username: myuser
	password: mypassword

pool: <%= ENV.fetch("RAILS\_MAX\_THREADS") { 5 } %>

development:
	<<: \*default

database: myapp\_development

test:

<<: \*default

database: myapp\_test
```

## Reset the Databases for a Rails app

The `rails db:reset` command will drop the database and set it up again. This is functionally equivalent to `rails db:drop db:setup`.

In Rails 7.1, the `db:drop` and `db:setup` tasks are part of the suite of database rake tasks that Rails provides to manage the database of a Rails application. Here's what each of these tasks does:

**`db:drop`:**

- This task is used to drop or delete the database associated with your Rails application. ==It completely removes all the tables, data, and the database schema. This is an irreversible action== and should be used with caution, especially in a production environment or if you have important data in your database.
- It's often used during the development process to reset the database state or when you want to start from scratch without any existing data or schema.

**`db:setup`:**

- This task is a combination of several tasks that are run in sequence to set up the database for your Rails application. Specifically, it runs `db:create`, `db:schema:load`, and `db:seed`.
- `db:create` creates the database based on the configuration specified in your `database.yml` file.
- `db:schema:load` loads the schema into the new database, creating all the tables and database structures as defined in your `schema.rb` (or `structure.sql`) file.
- `db:seed` loads the seed data into the database. This is defined in your `db/seeds.rb` file and typically includes a set of initial data necessary for the application.
- The `db:setup` task is typically used when setting up a new instance of the application, such as when cloning a project from a repository and preparing the development environment for the first time.

It's important to note that ==these tasks interact directly with your database, so you should always ensure you have backups of your data== (if necessary) and understand the implications of running these commands, especially in a shared or production environment.

### Drop and recreate the PostgreSQL database:

```ruby
rake db:drop
rake db:create
```

### Schema.rb

??? We should also delete db/schema.rb, since that will no longer be used. Rails will store the SQL schema in db/structure.sql.

### `rails db:drop` vs `rake db:drop`

In older versions of Rails (before Rails 5), `rake` was used for running tasks, including database-related tasks. In modern versions of Rails, it's recommended to interact with the database and other Rails tasks through the `rails` command.

## PostgreSQL vs MySQL?

==% 停止探索、掙扎用哪個數據庫，就用 PostgreSQL。20231129 %==

### Which database to use, for a group of enterprise applications, like ERP, CRM, PLM, etc., for running a franchise of hundreds of bakery shops and two factories, and used by hundreds of people daily?

==**Bard**: Based on the information you've provided, I would highly recommend PostgreSQL for your scenario.== Here's why:

**PostgreSQL's strengths for your case:**

* **Complex data models:** Enterprise applications like ERP, CRM, WMS, TMS, PLM often have intricate data models with complex relationships and queries. PostgreSQL's advanced data types, functions, and indexing capabilities are well-suited for handling these challenges effectively.
* **Scalability:** With hundreds of users accessing the system daily and potential for further growth, your database needs to handle increased activity efficiently. PostgreSQL scales very well, offering features like partitioning and replication to manage larger datasets and concurrent access.
* **Data integrity:** Enterprise applications require stringent data integrity to ensure accurate information and reliable operations. PostgreSQL's ACID transactions with multiversion concurrency control (MVCC) provide stronger data consistency and error protection compared to MySQL's strict ACID mode.
* **Advanced features:** PostgreSQL offers features like materialized views, full-text search, and JSONB data type, which can be very useful for improving query performance and handling various data formats in your enterprise applications.
* **Rails integration:** Rails 7 has further optimized its integration with PostgreSQL, making it easier to leverage its advanced features and benefit from its performance enhancements.

**While MySQL might be tempting due to its familiarity and ease of use, its limitations could become increasingly evident for your project:**

* **Complex queries:** MySQL can struggle with intricate queries and joins as your data model grows, potentially impacting performance and developer agility.
* **Scalability:** As your user base and data volume increase, MySQL could bottleneck under heavy load, requiring more frequent optimization or even sharding, which adds complexity.
* **Data integrity:** Although MySQL offers ACID transactions, its strict mode can impact performance. PostgreSQL's MVCC provides a more flexible and performant approach to data consistency.
* **Lack of advanced features:** Some features crucial for complex enterprise applications, like ==materialized views== and advanced data types, are lacking in MySQL.

**ChatGPT**: Given that you are developing multiple business applications including an ERP, CRM, MES, PLM, and franchise store sales management, it sounds like your system might be complex and could benefit from PostgreSQL's advanced features and robustness.

==Active Record supports PostgreSQL's `array` and `hstore` data types==, which allow you to store arrays and hashes directly in the database. This can simplify your data model and make certain types of queries more efficient.

==Full-Text Search: PostgreSQL's full-text search is supported by Active Record, providing powerful and efficient text search capabilities without needing to use a separate search server.==

PostGIS Extension: If you're working with geospatial data, Active Record can take advantage of the PostGIS extension in PostgreSQL.

## WHL: Data integrity is more important than query efficiency in an enterprise app.

**ChatGPT**: Absolutely, ==in an enterprise and business context, data integrity is paramount. The cost of data corruption or loss can be immense==, potentially leading to incorrect business decisions, regulatory violations, loss of customer trust, and other serious consequences. Therefore, maintaining data integrity should always be a top priority.

==Query efficiency is also important. However, it should not come at the expense of data integrity.== A well-designed database and properly optimized queries can usually provide both good performance and strong data integrity.

### Normalization Principles

Normalisation is a database design technique that reduces data redundancy and eliminates undesirable characteristics like Insertion, Update and Deletion Anomalies. ==The goal is to divide your database schema into smaller tables and define relationships between them to increase the clarity, efficiency, and scalability of your database.==

### About using array in database

*WHL*: You know in Ruby, you use array to store multiple objects, and a common place is to iterate the array to manipulate each element in it. You don't use Array for storage but for operation, right? Yet, in a database, the main purpose is to store data with integrity as the first consideration. Therefore, ==I don't think to have array as a datatype in a database is a good idea.==

**ChatGPT**: You bring up an excellent point. The data storage needs of an application are often quite different from the data manipulation needs within the application logic, and this is a key distinction to keep in mind when considering whether to use array data types in a database.

As you noted, arrays are used extensively in Ruby (and most other programming languages) for manipulating collections of objects in memory. ==They're great for transient operations where you need to process a list of items and then discard it.==&#x20;

==Storing arrays in a database can violate normalization principles, leading to potential data redundancy and anomalies.==

For example, if you're storing a list of tags as an array in a blog post record, and you want to rename a tag, you would have to find and update every post that contains that tag in its array. This operation could be slow and error-prone, especially if you have a large number of posts. If you had a separate tags table with a many-to-many relationship to posts, you could simply update the tag in one place.

==So while PostgreSQL's support for array data types can provide some convenience and flexibility for developers, it's not always the best choice from a data integrity and database design perspective.== As always, the best approach depends on the specific needs and constraints of your application.

### About using Arrays in PostgreSQL

**WHL: Let's consent it together with Rails in mind. Since Ruby has array and Rails use array a lot. Is it more compatible to support array in database?**

**ChatGPT**: From a programming perspective, the ability to use arrays natively in PostgreSQL can be attractive to Rails developers due to the ease and familiarity of working with arrays in Ruby. Here are some factors to consider:

Convenience: Ruby is very array-oriented, and Rails developers often use arrays to handle collections of objects. Being able to store and retrieve arrays directly from the database can be convenient and intuitive.

Easy Mapping: Arrays in PostgreSQL map directly to arrays in Ruby, which can make certain types of operations easier and more efficient. For example, you can store a multi-select form input (which comes in as an array) directly to a PostgreSQL array field.

Array Operations: PostgreSQL supports many powerful operations on array fields, such as intersection, union, and difference, which can make certain types of queries easier and more efficient.

Simplicity: Storing arrays directly in a column can simplify your data model, especially for one-to-many relationships that don't require a separate table.

Advanced Querying: PostgreSQL allows for sophisticated querying on array fields, such as checking if an array contains a certain value or querying on array length.

**However, there are also potential downsides:**

Database Agnosticism: One of the key principles of Rails is to be database-agnostic, and using arrays in PostgreSQL could tie your application more closely to a specific database system. % 我不認為這是一個實際問題，既然我們選擇了一種成熟的數據庫，我們在未來的多少年裡就可以一直使用它，形成自己的 best practice。 20230724 %

Complexity with Associations: If you need to associate other data with the elements in an array (e.g., if you need to track when each item was added or who added it), you'll likely need a separate table anyway.

Search and Indexing: ==Searching for a specific element in an array or indexing an array column can be more complex and less performant than using a normalised schema, especially for large arrays.==

Limited Indexing and Performance: While you can index array fields in PostgreSQL, the performance benefits might not be as good as with normalised data, especially for large arrays. Querying and updating large array fields can also be slow.

% 使用場景：一個員工的緊急聯繫信息，可以放在一個 Arrary裡，裏面包含：聯繫人姓名、聯繫人手機號碼、聯繫人地址、聯繫人關係等。因為緊急聯繫信息不是企業環境裡的主要數據，所以，完全可以做一個開放式的 Arrary存儲。%

### Is it a good practice that using JSON accordingly but forbidding array strictly in databases?

**ChatGPT**: JSON can be a good choice for storing semi-structured data, or data with a flexible schema that doesn't fit neatly into a relational model. For example, you might use a JSON field to store custom attributes for a user profile or product listing, where different records might have different sets of attributes.

==Avoid arrays for atomicity and normalization==: Storing arrays in a database can violate the principles of atomicity and normalization. It can lead to data redundancy and anomalies, and it can make certain types of queries and updates more difficult and less efficient. Therefore, ==in many cases, it's better to represent many-to-many relationships with a separate join table, rather than storing arrays of values in a single field.==

Performance considerations: ==JSON fields and array fields can have performance implications. For example, querying and indexing JSON data or array data can be less efficient than querying and indexing traditional scalar data types.== This might not be a concern for small datasets, but it can become a bottleneck for larger datasets or more complex queries.

## SQLite

SQLite is an in-process library that implements a self-contained, serverless, zero-configuration, transactional SQL database engine.

Unlike most other SQL databases, SQLite does not have a separate server process. SQLite reads and writes directly to ordinary disk files.

A database in SQLite is a single disk file. The database file format is cross-platform and make SQLite a popular choice as an Application File Format. SQLite database files are a recommended storage format by the US Library of Congress.

SQLite is a compact library. With all features enabled, the library size can be less than 600KiB.

SQLite reads and writes small blobs (for example, thumbnail images) 35% faster than the same blobs can be read from or written to individual files on disk. Furthermore, a single SQLite database holding 10-kilobyte blobs uses about 20% less disk space than storing the blobs in individual files.

SQLite is very carefully tested prior to every release and has a reputation for being very reliable.

SQLite responds gracefully to memory allocation failures and disk I/O errors. Transactions are ACID even if interrupted by system crashes or power failures.

SQLite does not need to be "installed" before it is used. There is no "setup" procedure. Nothing needs to be done to tell the system that SQLite is running. No actions are required to recover after a system crash or power failure.
