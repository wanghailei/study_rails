# Active Record Callbacks

This guide teaches you how to hook into the life cycle of your Active Record objects.

## 1 The Object Life Cycle

During the normal operation of a Rails application, objects may be created, updated, and destroyed. Active Record provides hooks into this *object life cycle* so that you can control your application and its data.

Callbacks allow you to trigger logic before or after an alteration of an object's state.

```
class Baby < ApplicationRecord
  after_create -> { puts "Congratulations!" }
end
```

Copy

```
irb> @baby = Baby.create
Congratulations!
```

Copy

As you will see, there are many life cycle events and you can choose to hook into any of these either before, after, or even around them.

## [2 Callbacks Overview](https://guides.rubyonrails.org/active_record_callbacks.html#callbacks-overview)

Callbacks are methods that get called at certain moments of an object's life cycle. With callbacks it is possible to write code that will run whenever an Active Record object is created, saved, updated, deleted, validated, or loaded from the database.

### [2.1 Callback Registration](https://guides.rubyonrails.org/active_record_callbacks.html#callback-registration)

In order to use the available callbacks, you need to register them. You can implement the callbacks as ordinary methods and use a macro-style class method to register them as callbacks:

```
class User < ApplicationRecord
  validates :login, :email, presence: true

  before_validation :ensure_login_has_a_value

  private
    def ensure_login_has_a_value
      if login.blank?
        self.login = email unless email.blank?
      end
    end
end
```

Copy

The macro-style class methods can also receive a block. Consider using this style if the code inside your block is so short that it fits in a single line:

```
class User < ApplicationRecord
  validates :login, :email, presence: true

  before_create do
    self.name = login.capitalize if name.blank?
  end
end
```

Copy

Alternatively you can pass a proc to the callback to be triggered.

```
class User < ApplicationRecord
  before_create ->(user) { user.name = user.login.capitalize if user.name.blank? }
end
```

Copy

Lastly, you can define your own custom callback object, which we will cover later in more detail [below](https://guides.rubyonrails.org/active_record_callbacks.html#callback-classes).

```
class User < ApplicationRecord
  before_create MaybeAddName
end

class MaybeAddName
  def self.before_create(record)
    if record.name.blank?
      record.name = record.login.capitalize
    end
  end
end
```

Callbacks can also be registered to only fire on certain life cycle events, this allows complete control over when and in what context your callbacks are triggered.

```ruby
class User < ApplicationRecord
	before_validation :normalize_name, on: :create
	# :on takes an array as well
	after_validation :set_location, on: [ :create, :update ]
     
private
     
	def normalize_name
		self.name = name.downcase.titleize
	end
	def set_location
		self.location = LocationService.query(self)
	end
end
```

Copy

It is considered good practice to declare callback methods as private. If left public, they can be called from outside of the model and violate the principle of object encapsulation.

Avoid calls to `update`, `save` or other methods which create side-effects to the object inside your callback. For example, don't call `update(attribute: "value")` within a callback. This can alter the state of the model and may result in unexpected side effects during commit. Instead, you can safely assign values directly (for example, `self.attribute = "value"`) in `before_create` / `before_update` or earlier callbacks.

## 3 Available Callbacks

Here is a list with all the available Active Record callbacks, listed in the same order in which they will get called during the respective operations:

### [3.1 Creating an Object](https://guides.rubyonrails.org/active_record_callbacks.html#creating-an-object)

- [`before_validation`](https://api.rubyonrails.org/v7.1.3/classes/ActiveModel/Validations/Callbacks/ClassMethods.html#method-i-before_validation)
- [`after_validation`](https://api.rubyonrails.org/v7.1.3/classes/ActiveModel/Validations/Callbacks/ClassMethods.html#method-i-after_validation)
- [`before_save`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Callbacks/ClassMethods.html#method-i-before_save)
- [`around_save`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Callbacks/ClassMethods.html#method-i-around_save)
- [`before_create`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Callbacks/ClassMethods.html#method-i-before_create)
- [`around_create`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Callbacks/ClassMethods.html#method-i-around_create)
- [`after_create`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Callbacks/ClassMethods.html#method-i-after_create)
- [`after_save`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Callbacks/ClassMethods.html#method-i-after_save)
- [`after_commit`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Transactions/ClassMethods.html#method-i-after_commit) / [`after_rollback`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Transactions/ClassMethods.html#method-i-after_rollback)

### [3.2 Updating an Object](https://guides.rubyonrails.org/active_record_callbacks.html#updating-an-object)

- [`before_validation`](https://api.rubyonrails.org/v7.1.3/classes/ActiveModel/Validations/Callbacks/ClassMethods.html#method-i-before_validation)
- [`after_validation`](https://api.rubyonrails.org/v7.1.3/classes/ActiveModel/Validations/Callbacks/ClassMethods.html#method-i-after_validation)
- [`before_save`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Callbacks/ClassMethods.html#method-i-before_save)
- [`around_save`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Callbacks/ClassMethods.html#method-i-around_save)
- [`before_update`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Callbacks/ClassMethods.html#method-i-before_update)
- [`around_update`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Callbacks/ClassMethods.html#method-i-around_update)
- [`after_update`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Callbacks/ClassMethods.html#method-i-after_update)
- [`after_save`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Callbacks/ClassMethods.html#method-i-after_save)
- [`after_commit`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Transactions/ClassMethods.html#method-i-after_commit) / [`after_rollback`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Transactions/ClassMethods.html#method-i-after_rollback)

`after_save` runs both on create and update, but always *after* the more specific callbacks `after_create` and `after_update`, no matter the order in which the macro calls were executed.

### [3.3 Destroying an Object](https://guides.rubyonrails.org/active_record_callbacks.html#destroying-an-object)

- [`before_destroy`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Callbacks/ClassMethods.html#method-i-before_destroy)
- [`around_destroy`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Callbacks/ClassMethods.html#method-i-around_destroy)
- [`after_destroy`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Callbacks/ClassMethods.html#method-i-after_destroy)
- [`after_commit`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Transactions/ClassMethods.html#method-i-after_commit) / [`after_rollback`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Transactions/ClassMethods.html#method-i-after_rollback)

`before_destroy` callbacks should be placed before `dependent: :destroy`associations (or use the `prepend: true` option), to ensure they execute before the records are deleted by `dependent: :destroy`.

`after_commit` makes very different guarantees than `after_save`, `after_update`, and `after_destroy`. For example if an exception occurs in an `after_save` the transaction will be rolled back and the data will not be persisted. While anything that happens `after_commit` can guarantee the transaction has already completed and the data was persisted to the database. More on [transactional callbacks](https://guides.rubyonrails.org/active_record_callbacks.html#transaction-callbacks) below.

### 3.4 `after_initialize` and `after_find`

Whenever an Active Record object is instantiated the [`after_initialize`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Callbacks/ClassMethods.html#method-i-after_initialize) callback will be called, either by directly using `new` or when a record is loaded from the database. It can be useful to avoid the need to directly override your Active Record `initialize` method.

When loading a record from the database the [`after_find`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Callbacks/ClassMethods.html#method-i-after_find) callback will be called. `after_find` is called before `after_initialize` if both are defined.

The `after_initialize` and `after_find` callbacks have no `before_*`counterparts.

They can be registered just like the other Active Record callbacks.

```
class User < ApplicationRecord
  after_initialize do |user|
    puts "You have initialized an object!"
  end

  after_find do |user|
    puts "You have found an object!"
  end
end
```

Copy

```
irb> User.new
You have initialized an object!
=> #<User id: nil>

irb> User.first
You have found an object!
You have initialized an object!
=> #<User id: 1>
```

### 3.5 `after_touch`

The [`after_touch`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Callbacks/ClassMethods.html#method-i-after_touch) callback will be called whenever an Active Record object is touched.

```
class User < ApplicationRecord
  after_touch do |user|
    puts "You have touched an object"
  end
end
```

```
irb> u = User.create(name: 'Kuldeep')
=> #<User id: 1, name: "Kuldeep", created_at: "2013-11-25 12:17:49", updated_at: "2013-11-25 12:17:49">

irb> u.touch
You have touched an object
=> true
```

It can be used along with `belongs_to`:

```
class Book < ApplicationRecord
  belongs_to :library, touch: true
  after_touch do
    puts 'A Book was touched'
  end
end

class Library < ApplicationRecord
  has_many :books
  after_touch :log_when_books_or_library_touched

  private
    def log_when_books_or_library_touched
      puts 'Book/Library was touched'
    end
end
```

```
irb> @book = Book.last
=> #<Book id: 1, library_id: 1, created_at: "2013-11-25 17:04:22", updated_at: "2013-11-25 17:05:05">

irb> @book.touch # triggers @book.library.touch
A Book was touched
Book/Library was touched
=> true
```

## 4 Running Callbacks

The following methods trigger callbacks:

- `create`
- `create!`
- `destroy`
- `destroy!`
- `destroy_all`
- `destroy_by`
- `save`
- `save!`
- `save(validate: false)`
- `save!(validate: false)`
- `toggle!`
- `touch`
- `update_attribute`
- `update`
- `update!`
- `valid?`

Additionally, the `after_find` callback is triggered by the following finder methods:

- `all`
- `first`
- `find`
- `find_by`
- `find_by_*`
- `find_by_*!`
- `find_by_sql`
- `last`

The `after_initialize` callback is triggered every time a new object of the class is initialized.

The `find_by_*` and `find_by_*!` methods are dynamic finders generated automatically for every attribute. Learn more about them at the [Dynamic finders section](https://guides.rubyonrails.org/active_record_querying.html#dynamic-finders)

## [5 Skipping Callbacks](https://guides.rubyonrails.org/active_record_callbacks.html#skipping-callbacks)

Just as with validations, it is also possible to skip callbacks by using the following methods:

- `decrement!`
- `decrement_counter`
- `delete`
- `delete_all`
- `delete_by`
- `increment!`
- `increment_counter`
- `insert`
- `insert!`
- `insert_all`
- `insert_all!`
- `touch_all`
- `update_column`
- `update_columns`
- `update_all`
- `update_counters`
- `upsert`
- `upsert_all`

These methods should be used with caution, however, because important business rules and application logic may be kept in callbacks. Bypassing them without understanding the potential implications may lead to invalid data.

## [6 Halting Execution](https://guides.rubyonrails.org/active_record_callbacks.html#halting-execution)

As you start registering new callbacks for your models, they will be queued for execution. This queue will include all your model's validations, the registered callbacks, and the database operation to be executed.

The whole callback chain is wrapped in a transaction. If any callback raises an exception, the execution chain gets halted and a ROLLBACK is issued. To intentionally stop a chain use:

```
throw :abort
```

Copy

Any exception that is not `ActiveRecord::Rollback` or `ActiveRecord::RecordInvalid` will be re-raised by Rails after the callback chain is halted. Additionally, may break code that does not expect methods like `save` and `update` (which normally try to return `true` or `false`) to raise an exception.

If an `ActiveRecord::RecordNotDestroyed` is raised within `after_destroy`, `before_destroy` or `around_destroy` callback, it will not be re-raised and the `destroy` method will return `false`.

## [7 Relational Callbacks](https://guides.rubyonrails.org/active_record_callbacks.html#relational-callbacks)

Callbacks work through model relationships, and can even be defined by them. Suppose an example where a user has many articles. A user's articles should be destroyed if the user is destroyed. Let's add an `after_destroy` callback to the `User` model by way of its relationship to the `Article`model:

```
class User < ApplicationRecord
  has_many :articles, dependent: :destroy
end

class Article < ApplicationRecord
  after_destroy :log_destroy_action

  def log_destroy_action
    puts 'Article destroyed'
  end
end
```

Copy

```
irb> user = User.first
=> #<User id: 1>
irb> user.articles.create!
=> #<Article id: 1, user_id: 1>
irb> user.destroy
Article destroyed
=> #<User id: 1>
```

Copy

## [8 Association Callbacks](https://guides.rubyonrails.org/active_record_callbacks.html#association-callbacks)

Association callbacks are similar to normal callbacks, but they are triggered by events in the life cycle of a collection. There are four available association callbacks:

- `before_add`
- `after_add`
- `before_remove`
- `after_remove`

You define association callbacks by adding options to the association declaration. For example:

```
class Author < ApplicationRecord
  has_many :books, before_add: :check_credit_limit

  def check_credit_limit(book)
    # ...
  end
end
```

Copy

Rails passes the object being added or removed to the callback.

You can stack callbacks on a single event by passing them as an array:

```
class Author < ApplicationRecord
  has_many :books,
    before_add: [:check_credit_limit, :calculate_shipping_charges]

  def check_credit_limit(book)
    # ...
  end

  def calculate_shipping_charges(book)
    # ...
  end
end
```

Copy

If a `before_add` callback throws `:abort`, the object does not get added to the collection. Similarly, if a `before_remove` callback throws `:abort`, the object does not get removed from the collection:

```
# book won't be added if the limit has been reached
def check_credit_limit(book)
  throw(:abort) if limit_reached?
end
```

Copy

These callbacks are called only when the associated objects are added or removed through the association collection:

```
# Triggers `before_add` callback
author.books << book
author.books = [book, book2]

# Does not trigger the `before_add` callback
book.update(author_id: 1)
```

Copy

## [9 Conditional Callbacks](https://guides.rubyonrails.org/active_record_callbacks.html#conditional-callbacks)

As with validations, we can also make the calling of a callback method conditional on the satisfaction of a given predicate. We can do this using the `:if` and `:unless` options, which can take a symbol, a `Proc` or an `Array`.

You may use the `:if` option when you want to specify under which conditions the callback **should**be called. If you want to specify the conditions under which the callback **should not** be called, then you may use the `:unless` option.

### [9.1 Using `:if` and `:unless` with a `Symbol`](https://guides.rubyonrails.org/active_record_callbacks.html#using-if-and-unless-with-a-symbol)

You can associate the `:if` and `:unless` options with a symbol corresponding to the name of a predicate method that will get called right before the callback.

When using the `:if` option, the callback **won't** be executed if the predicate method returns **false**; when using the `:unless` option, the callback **won't** be executed if the predicate method returns **true**. This is the most common option.

```
class Order < ApplicationRecord
  before_save :normalize_card_number, if: :paid_with_card?
end
```

Copy

Using this form of registration it is also possible to register several different predicates that should be called to check if the callback should be executed. We will cover this [below](https://guides.rubyonrails.org/active_record_callbacks.html#multiple-callback-conditions).

### [9.2 Using `:if` and `:unless` with a `Proc`](https://guides.rubyonrails.org/active_record_callbacks.html#using-if-and-unless-with-a-proc)

It is possible to associate `:if` and `:unless` with a `Proc` object. This option is best suited when writing short validation methods, usually one-liners:

```
class Order < ApplicationRecord
  before_save :normalize_card_number,
    if: Proc.new { |order| order.paid_with_card? }
end
```

Copy

As the proc is evaluated in the context of the object, it is also possible to write this as:

```
class Order < ApplicationRecord
  before_save :normalize_card_number, if: Proc.new { paid_with_card? }
end
```

Copy

### [9.3 Multiple Callback Conditions](https://guides.rubyonrails.org/active_record_callbacks.html#multiple-callback-conditions)

The `:if` and `:unless` options also accept an array of procs or method names as symbols:

```
class Comment < ApplicationRecord
  before_save :filter_content,
    if: [:subject_to_parental_control?, :untrusted_author?]
end
```

Copy

You can easily include a proc in the list of conditions:

```
class Comment < ApplicationRecord
  before_save :filter_content,
    if: [:subject_to_parental_control?, Proc.new { untrusted_author? }]
end
```

Copy

### [9.4 Using Both `:if` and `:unless`](https://guides.rubyonrails.org/active_record_callbacks.html#using-both-if-and-unless)

Callbacks can mix both `:if` and `:unless` in the same declaration:

```
class Comment < ApplicationRecord
  before_save :filter_content,
    if: Proc.new { forum.parental_control? },
    unless: Proc.new { author.trusted? }
end
```

Copy

The callback only runs when all the `:if` conditions and none of the `:unless` conditions are evaluated to `true`.

## [10 Callback Classes](https://guides.rubyonrails.org/active_record_callbacks.html#callback-classes)

Sometimes the callback methods that you'll write will be useful enough to be reused by other models. Active Record makes it possible to create classes that encapsulate the callback methods, so they can be reused.

Here's an example where we create a class with an `after_destroy` callback to deal with the clean up of discarded files on the filesystem. This behavior may not be unique to our `PictureFile` model and we may want to share it, so it's a good idea to encapsulate this into a separate class. This will make testing that behavior and changing it much easier.

```
class FileDestroyerCallback
  def after_destroy(file)
    if File.exist?(file.filepath)
      File.delete(file.filepath)
    end
  end
end
```

Copy

When declared inside a class, as above, the callback methods will receive the model object as a parameter. This will work on any model that uses the class like so:

```
class PictureFile < ApplicationRecord
  after_destroy FileDestroyerCallback.new
end
```

Copy

Note that we needed to instantiate a new `FileDestroyerCallback` object, since we declared our callback as an instance method. This is particularly useful if the callbacks make use of the state of the instantiated object. Often, however, it will make more sense to declare the callbacks as class methods:

```
class FileDestroyerCallback
  def self.after_destroy(file)
    if File.exist?(file.filepath)
      File.delete(file.filepath)
    end
  end
end
```

Copy

When the callback method is declared this way, it won't be necessary to instantiate a new `FileDestroyerCallback` object in our model.

```
class PictureFile < ApplicationRecord
  after_destroy FileDestroyerCallback
end
```

Copy

You can declare as many callbacks as you want inside your callback classes.

## [11 Transaction Callbacks](https://guides.rubyonrails.org/active_record_callbacks.html#transaction-callbacks)

### [11.1 `after_commit` and `after_rollback`](https://guides.rubyonrails.org/active_record_callbacks.html#after-commit-and-after-rollback)

There are two additional callbacks that are triggered by the completion of a database transaction: [`after_commit`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Transactions/ClassMethods.html#method-i-after_commit) and [`after_rollback`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Transactions/ClassMethods.html#method-i-after_rollback). These callbacks are very similar to the `after_save`callback except that they don't execute until after database changes have either been committed or rolled back. They are most useful when your Active Record models need to interact with external systems which are not part of the database transaction.

Consider, for example, the previous example where the `PictureFile` model needs to delete a file after the corresponding record is destroyed. If anything raises an exception after the `after_destroy` callback is called and the transaction rolls back, the file will have been deleted and the model will be left in an inconsistent state. For example, suppose that `picture_file_2` in the code below is not valid and the `save!` method raises an error.

```
PictureFile.transaction do
  picture_file_1.destroy
  picture_file_2.save!
end
```

Copy

By using the `after_commit` callback we can account for this case.

```
class PictureFile < ApplicationRecord
  after_commit :delete_picture_file_from_disk, on: :destroy

  def delete_picture_file_from_disk
    if File.exist?(filepath)
      File.delete(filepath)
    end
  end
end
```

Copy

The `:on` option specifies when a callback will be fired. If you don't supply the `:on` option the callback will fire for every action.

When a transaction completes, the `after_commit` or `after_rollback` callbacks are called for all models created, updated, or destroyed within that transaction. However, if an exception is raised within one of these callbacks, the exception will bubble up and any remaining `after_commit` or `after_rollback` methods will *not* be executed. As such, if your callback code could raise an exception, you'll need to rescue it and handle it within the callback in order to allow other callbacks to run.

The code executed within `after_commit` or `after_rollback` callbacks is itself not enclosed within a transaction.

In the context of a single transaction, if you interact with multiple loaded objects that represent the same record in the database, there's a crucial behavior in the `after_commit` and `after_rollback` callbacks to note. These callbacks are triggered only for the first object of the specific record that undergoes a change within the transaction. Other loaded objects, despite representing the same database record, will not have their respective `after_commit` or `after_rollback` callbacks triggered. This nuanced behavior is particularly impactful in scenarios where you expect independent callback execution for each object associated with the same database record. It can influence the flow and predictability of callback sequences, leading to potential inconsistencies in application logic following the transaction.

### [11.2 Aliases for `after_commit`](https://guides.rubyonrails.org/active_record_callbacks.html#aliases-for-after-commit)

Since using the `after_commit` callback only on create, update, or delete is common, there are aliases for those operations:

- [`after_create_commit`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Transactions/ClassMethods.html#method-i-after_create_commit)
- [`after_update_commit`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Transactions/ClassMethods.html#method-i-after_update_commit)
- [`after_destroy_commit`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Transactions/ClassMethods.html#method-i-after_destroy_commit)

```
class PictureFile < ApplicationRecord
  after_destroy_commit :delete_picture_file_from_disk

  def delete_picture_file_from_disk
    if File.exist?(filepath)
      File.delete(filepath)
    end
  end
end
```

Copy

Using both `after_create_commit` and `after_update_commit` with the same method name will only allow the last callback defined to take effect, as they both internally alias to `after_commit` which overrides previously defined callbacks with the same method name.

```
class User < ApplicationRecord
  after_create_commit :log_user_saved_to_db
  after_update_commit :log_user_saved_to_db

  private
    def log_user_saved_to_db
      puts 'User was saved to database'
    end
end
```

Copy

```
irb> @user = User.create # prints nothing

irb> @user.save # updating @user
User was saved to database
```

Copy

### [11.3 `after_save_commit`](https://guides.rubyonrails.org/active_record_callbacks.html#after-save-commit)

There is also [`after_save_commit`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Transactions/ClassMethods.html#method-i-after_save_commit), which is an alias for using the `after_commit` callback for both create and update together:

```
class User < ApplicationRecord
  after_save_commit :log_user_saved_to_db

  private
    def log_user_saved_to_db
      puts 'User was saved to database'
    end
end
```

Copy

```
irb> @user = User.create # creating a User
User was saved to database

irb> @user.save # updating @user
User was saved to database
```

Copy

### [11.4 Transactional Callback Ordering](https://guides.rubyonrails.org/active_record_callbacks.html#transactional-callback-ordering)

By default, callbacks will run in the order they are defined. However, when defining multiple transactional `after_` callbacks (`after_commit`, `after_rollback`, etc), the order could be reversed from when they are defined.

```
class User < ActiveRecord::Base
  after_commit { puts("this actually gets called second") }
  after_commit { puts("this actually gets called first") }
end
```

Copy

This applies to all `after_*_commit` variations too, such as `after_destroy_commit`.

This order can be set via configuration:

```
config.active_record.run_after_transaction_callbacks_in_order_defined = false
```

Copy

When set to `true` (the default from Rails 7.1), callbacks are executed in the order they are defined. When set to `false`, the order is reversed, just like in the example above.

### Feedback

You're encouraged to help improve the quality of this guide.

Please contribute if you see any typos or factual errors. To get started, you can read our [documentation contributions](https://edgeguides.rubyonrails.org/contributing_to_ruby_on_rails.html#contributing-to-the-rails-documentation) section.

You may also find incomplete content or stuff that is not up to date. Please do add any missing documentation for main. Make sure to check [Edge Guides](https://edgeguides.rubyonrails.org/) first to verify if the issues are already fixed or not on the main branch. Check the [Ruby on Rails Guides Guidelines](https://guides.rubyonrails.org/ruby_on_rails_guides_guidelines.html) for style and conventions.

If for whatever reason you spot something to fix but cannot patch it yourself, please [open an issue](https://github.com/rails/rails/issues).

And last but not least, any kind of discussion regarding Ruby on Rails documentation is very welcome on the [official Ruby on Rails Forum](https://discuss.rubyonrails.org/c/rubyonrails-docs).

This work is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-sa/4.0/) License

"Rails", "Ruby on Rails", and the Rails logo are trademarks of David Heinemeier Hansson. All rights reserved.