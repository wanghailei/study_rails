# Active Record Validations

This guide teaches you how to validate the state of objects before they go into the database using Active Record's validations feature.

## 1 Validations Overview

Here's an example of a very simple validation:

```
class Person < ApplicationRecord
  validates :name, presence: true
end
```

```
irb> Person.create(name: "John Doe").valid?
=> true
irb> Person.create(name: nil).valid?
=> false
```

As you can see, our validation lets us know that our `Person` is not valid without a `name` attribute. The second `Person` will not be persisted to the database.

Before we dig into more details, let's talk about how validations fit into the big picture of your application.

### 1.1 Why Use Validations?

Validations are used to ensure that only valid data is saved into your database. For example, it may be important to your application to ensure that every user provides a valid email address and mailing address. Model-level validations are the best way to ensure that only valid data is saved into your database. They are database agnostic, cannot be bypassed by end users, and are convenient to test and maintain. Rails provides built-in helpers for common needs, and allows you to create your own validation methods as well.

There are several other ways to validate data before it is saved into your database, including native database constraints, client-side validations and controller-level validations. Here's a summary of the pros and cons:

- Database constraints and/or stored procedures make the validation mechanisms database-dependent and can make testing and maintenance more difficult. However, if your database is used by other applications, it may be a good idea to use some constraints at the database level. Additionally, database-level validations can safely handle some things (such as uniqueness in heavily-used tables) that can be difficult to implement otherwise.
- Client-side validations can be useful, but are generally unreliable if used alone. If they are implemented using JavaScript, they may be bypassed if JavaScript is turned off in the user's browser. However, if combined with other techniques, client-side validation can be a convenient way to provide users with immediate feedback as they use your site.
- Controller-level validations can be tempting to use, but often become unwieldy and difficult to test and maintain. Whenever possible, it's a good idea to keep your controllers simple, as it will make your application a pleasure to work with in the long run.

Choose these in certain, specific cases. It's the opinion of the Rails team that model-level validations are the most appropriate in most circumstances.

### [1.2 When Does Validation Happen?](https://guides.rubyonrails.org/active_record_validations.html#when-does-validation-happen-questionmark)

There are two kinds of Active Record objects: those that correspond to a row inside your database and those that do not. When you create a fresh object, for example using the `new` method, that object does not belong to the database yet. Once you call `save` upon that object it will be saved into the appropriate database table. Active Record uses the `new_record?` instance method to determine whether an object is already in the database or not. Consider the following Active Record class:

```
class Person < ApplicationRecord
end
```

Copy

We can see how it works by looking at some `bin/rails console` output:

```
irb> p = Person.new(name: "John Doe")
=> #<Person id: nil, name: "John Doe", created_at: nil, updated_at: nil>

irb> p.new_record?
=> true

irb> p.save
=> true

irb> p.new_record?
=> false
```

Copy

Creating and saving a new record will send an SQL `INSERT` operation to the database. Updating an existing record will send an SQL `UPDATE` operation instead. Validations are typically run before these commands are sent to the database. If any validations fail, the object will be marked as invalid and Active Record will not perform the `INSERT` or `UPDATE` operation. This avoids storing an invalid object in the database. You can choose to have specific validations run when an object is created, saved, or updated.

There are many ways to change the state of an object in the database. Some methods will trigger validations, but some will not. This means that it's possible to save an object in the database in an invalid state if you aren't careful.

The following methods trigger validations, and will save the object to the database only if the object is valid:

- `create`
- `create!`
- `save`
- `save!`
- `update`
- `update!`

The bang versions (e.g. `save!`) raise an exception if the record is invalid. The non-bang versions don't: `save` and `update` return `false`, and `create` returns the object.

### [1.3 Skipping Validations](https://guides.rubyonrails.org/active_record_validations.html#skipping-validations)

The following methods skip validations, and will save the object to the database regardless of its validity. They should be used with caution.

- `decrement!`
- `decrement_counter`
- `increment!`
- `increment_counter`
- `insert`
- `insert!`
- `insert_all`
- `insert_all!`
- `toggle!`
- `touch`
- `touch_all`
- `update_all`
- `update_attribute`
- `update_column`
- `update_columns`
- `update_counters`
- `upsert`
- `upsert_all`

Note that `save` also has the ability to skip validations if passed `validate: false` as an argument. This technique should be used with caution.

- `save(validate: false)`

### [1.4 `valid?` and `invalid?`](https://guides.rubyonrails.org/active_record_validations.html#valid-questionmark-and-invalid-questionmark)

Before saving an Active Record object, Rails runs your validations. If these validations produce any errors, Rails does not save the object.

You can also run these validations on your own. [`valid?`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Validations.html#method-i-valid-3F) triggers your validations and returns true if no errors were found in the object, and false otherwise. As you saw above:

```
class Person < ApplicationRecord
  validates :name, presence: true
end
```

Copy

```
irb> Person.create(name: "John Doe").valid?
=> true
irb> Person.create(name: nil).valid?
=> false
```

Copy

After Active Record has performed validations, any failures can be accessed through the [`errors`](https://api.rubyonrails.org/v7.1.3/classes/ActiveModel/Validations.html#method-i-errors)instance method, which returns a collection of errors. By definition, an object is valid if this collection is empty after running validations.

Note that an object instantiated with `new` will not report errors even if it's technically invalid, because validations are automatically run only when the object is saved, such as with the `create` or `save`methods.

```
class Person < ApplicationRecord
  validates :name, presence: true
end
```

Copy

```
irb> p = Person.new
=> #<Person id: nil, name: nil>
irb> p.errors.size
=> 0

irb> p.valid?
=> false
irb> p.errors.objects.first.full_message
=> "Name can't be blank"

irb> p = Person.create
=> #<Person id: nil, name: nil>
irb> p.errors.objects.first.full_message
=> "Name can't be blank"

irb> p.save
=> false

irb> p.save!
ActiveRecord::RecordInvalid: Validation failed: Name can't be blank

irb> Person.create!
ActiveRecord::RecordInvalid: Validation failed: Name can't be blank
```

Copy

[`invalid?`](https://api.rubyonrails.org/v7.1.3/classes/ActiveModel/Validations.html#method-i-invalid-3F) is the inverse of `valid?`. It triggers your validations, returning true if any errors were found in the object, and false otherwise.

### [1.5 `errors[\]`](https://guides.rubyonrails.org/active_record_validations.html#validations-overview-errors)

To verify whether or not a particular attribute of an object is valid, you can use [`errors[:attribute\]`](https://api.rubyonrails.org/v7.1.3/classes/ActiveModel/Errors.html#method-i-5B-5D). It returns an array of all the error messages for `:attribute`. If there are no errors on the specified attribute, an empty array is returned.

This method is only useful *after* validations have been run, because it only inspects the errors collection and does not trigger validations itself. It's different from the `ActiveRecord::Base#invalid?` method explained above because it doesn't verify the validity of the object as a whole. It only checks to see whether there are errors found on an individual attribute of the object.

```
class Person < ApplicationRecord
  validates :name, presence: true
end
```

Copy

```
irb> Person.new.errors[:name].any?
=> false
irb> Person.create.errors[:name].any?
=> true
```

Copy

We'll cover validation errors in greater depth in the [Working with Validation Errors](https://guides.rubyonrails.org/active_record_validations.html#working-with-validation-errors) section.

## [2 Validation Helpers](https://guides.rubyonrails.org/active_record_validations.html#validation-helpers)

Active Record offers many pre-defined validation helpers that you can use directly inside your class definitions. These helpers provide common validation rules. Every time a validation fails, an error is added to the object's `errors` collection, and this is associated with the attribute being validated.

Each helper accepts an arbitrary number of attribute names, so with a single line of code you can add the same kind of validation to several attributes.

All of them accept the `:on` and `:message` options, which define when the validation should be run and what message should be added to the `errors` collection if it fails, respectively. The `:on` option takes one of the values `:create` or `:update`. There is a default error message for each one of the validation helpers. These messages are used when the `:message` option isn't specified. Let's take a look at each one of the available helpers.

To see a list of the available default helpers, take a look at[`ActiveModel::Validations::HelperMethods`](https://api.rubyonrails.org/v7.1.3/classes/ActiveModel/Validations/HelperMethods.html).

### [2.1 `acceptance`](https://guides.rubyonrails.org/active_record_validations.html#acceptance)

This method validates that a checkbox on the user interface was checked when a form was submitted. This is typically used when the user needs to agree to your application's terms of service, confirm that some text is read, or any similar concept.

```
class Person < ApplicationRecord
  validates :terms_of_service, acceptance: true
end
```

Copy

This check is performed only if `terms_of_service` is not `nil`. The default error message for this helper is *"must be accepted"*. You can also pass in a custom message via the `message` option.

```
class Person < ApplicationRecord
  validates :terms_of_service, acceptance: { message: 'must be abided' }
end
```

Copy

It can also receive an `:accept` option, which determines the allowed values that will be considered as acceptable. It defaults to `['1', true]` and can be easily changed.

```
class Person < ApplicationRecord
  validates :terms_of_service, acceptance: { accept: 'yes' }
  validates :eula, acceptance: { accept: ['TRUE', 'accepted'] }
end
```

Copy

This validation is very specific to web applications and this 'acceptance' does not need to be recorded anywhere in your database. If you don't have a field for it, the helper will create a virtual attribute. If the field does exist in your database, the `accept` option must be set to or include `true` or else the validation will not run.

### [2.2 `confirmation`](https://guides.rubyonrails.org/active_record_validations.html#confirmation)

You should use this helper when you have two text fields that should receive exactly the same content. For example, you may want to confirm an email address or a password. This validation creates a virtual attribute whose name is the name of the field that has to be confirmed with "_confirmation" appended.

```
class Person < ApplicationRecord
  validates :email, confirmation: true
end
```

Copy

In your view template you could use something like

```
<%= text_field :person, :email %>
<%= text_field :person, :email_confirmation %>
```

Copy

This check is performed only if `email_confirmation` is not `nil`. To require confirmation, make sure to add a presence check for the confirmation attribute (we'll take a look at `presence` [later](https://guides.rubyonrails.org/active_record_validations.html#presence) on in this guide):

```
class Person < ApplicationRecord
  validates :email, confirmation: true
  validates :email_confirmation, presence: true
end
```

Copy

There is also a `:case_sensitive` option that you can use to define whether the confirmation constraint will be case sensitive or not. This option defaults to true.

```
class Person < ApplicationRecord
  validates :email, confirmation: { case_sensitive: false }
end
```

Copy

The default error message for this helper is *"doesn't match confirmation"*. You can also pass in a custom message via the `message` option.

Generally when using this validator, you will want to combine it with the `:if` option to only validate the "_confirmation" field when the initial field has changed and **not** every time you save the record. More on [conditional validations](https://guides.rubyonrails.org/active_record_validations.html#conditional-validation) later.

```
class Person < ApplicationRecord
  validates :email, confirmation: true
  validates :email_confirmation, presence: true, if: :email_changed?
end
```

Copy

### [2.3 `comparison`](https://guides.rubyonrails.org/active_record_validations.html#comparison)

This check will validate a comparison between any two comparable values.

```
class Promotion < ApplicationRecord
  validates :end_date, comparison: { greater_than: :start_date }
end
```

Copy

The default error message for this helper is *"failed comparison"*. You can also pass in a custom message via the `message` option.

These options are all supported:

- `:greater_than` - Specifies the value must be greater than the supplied value. The default error message for this option is *"must be greater than %{count}"*.
- `:greater_than_or_equal_to` - Specifies the value must be greater than or equal to the supplied value. The default error message for this option is *"must be greater than or equal to %{count}"*.
- `:equal_to` - Specifies the value must be equal to the supplied value. The default error message for this option is *"must be equal to %{count}"*.
- `:less_than` - Specifies the value must be less than the supplied value. The default error message for this option is *"must be less than %{count}"*.
- `:less_than_or_equal_to` - Specifies the value must be less than or equal to the supplied value. The default error message for this option is *"must be less than or equal to %{count}"*.
- `:other_than` - Specifies the value must be other than the supplied value. The default error message for this option is *"must be other than %{count}"*.

The validator requires a compare option be supplied. Each option accepts a value, proc, or symbol. Any class that includes Comparable can be compared.

### [2.4 `format`](https://guides.rubyonrails.org/active_record_validations.html#format)

This helper validates the attributes' values by testing whether they match a given regular expression, which is specified using the `:with` option.

```
class Product < ApplicationRecord
  validates :legacy_code, format: { with: /\A[a-zA-Z]+\z/,
    message: "only allows letters" }
end
```

Copy

Inversely, by using the `:without` option instead you can require that the specified attribute does *not*match the regular expression.

In either case, the provided `:with` or `:without` option must be a regular expression or a proc or lambda that return one.

The default error message is *"is invalid"*.

use `\A` and `\z` to match the start and end of the string, `^` and `$` match the start/end of a line. Due to frequent misuse of `^` and `$`, you need to pass the `multiline: true`option in case you use any of these two anchors in the provided regular expression. In most cases, you should be using `\A` and `\z`.

### [2.5 `inclusion`](https://guides.rubyonrails.org/active_record_validations.html#inclusion)

This helper validates that the attributes' values are included in a given set. In fact, this set can be any enumerable object.

```
class Coffee < ApplicationRecord
  validates :size, inclusion: { in: %w(small medium large),
    message: "%{value} is not a valid size" }
end
```

Copy

The `inclusion` helper has an option `:in` that receives the set of values that will be accepted. The `:in` option has an alias called `:within` that you can use for the same purpose, if you'd like to. The previous example uses the `:message` option to show how you can include the attribute's value. For full options please see the [message documentation](https://guides.rubyonrails.org/active_record_validations.html#message).

The default error message for this helper is *"is not included in the list"*.

### [2.6 `exclusion`](https://guides.rubyonrails.org/active_record_validations.html#exclusion)

The opposite of `inclusion` is... `exclusion`!

This helper validates that the attributes' values are not included in a given set. In fact, this set can be any enumerable object.

```
class Account < ApplicationRecord
  validates :subdomain, exclusion: { in: %w(www us ca jp),
    message: "%{value} is reserved." }
end
```

Copy

The `exclusion` helper has an option `:in` that receives the set of values that will not be accepted for the validated attributes. The `:in` option has an alias called `:within` that you can use for the same purpose, if you'd like to. This example uses the `:message` option to show how you can include the attribute's value. For full options to the message argument please see the [message documentation](https://guides.rubyonrails.org/active_record_validations.html#message).

The default error message is *"is reserved"*.

Alternatively to a traditional enumerable (like an Array), you can supply a proc, lambda, or symbol which returns an enumerable. If the enumerable is a numerical, time, or datetime range the test is performed with `Range#cover?`, otherwise with `include?`. When using a proc or lambda the instance under validation is passed as an argument.

### [2.7 `length`](https://guides.rubyonrails.org/active_record_validations.html#length)

This helper validates the length of the attributes' values. It provides a variety of options, so you can specify length constraints in different ways:

```
class Person < ApplicationRecord
  validates :name, length: { minimum: 2 }
  validates :bio, length: { maximum: 500 }
  validates :password, length: { in: 6..20 }
  validates :registration_number, length: { is: 6 }
end
```

Copy

The possible length constraint options are:

- `:minimum` - The attribute cannot have less than the specified length.
- `:maximum` - The attribute cannot have more than the specified length.
- `:in` (or `:within`) - The attribute length must be included in a given interval. The value for this option must be a range.
- `:is` - The attribute length must be equal to the given value.

The default error messages depend on the type of length validation being performed. You can customize these messages using the `:wrong_length`, `:too_long`, and `:too_short` options and `%{count}` as a placeholder for the number corresponding to the length constraint being used. You can still use the `:message` option to specify an error message.

```
class Person < ApplicationRecord
  validates :bio, length: { maximum: 1000,
    too_long: "%{count} characters is the maximum allowed" }
end
```

Copy

Note that the default error messages are plural (e.g., "is too short (minimum is %{count} characters)"). For this reason, when `:minimum` is 1 you should provide a custom message or use `presence: true` instead. When `:in` or `:within` have a lower limit of 1, you should either provide a custom message or call `presence` prior to `length`.

Only one constraint option can be used at a time apart from the `:minimum` and `:maximum` options which can be combined together.

### [2.8 `numericality`](https://guides.rubyonrails.org/active_record_validations.html#numericality)

This helper validates that your attributes have only numeric values. By default, it will match an optional sign followed by an integer or floating point number.

To specify that only integer numbers are allowed, set `:only_integer` to true. Then it will use the following regular expression to validate the attribute's value.

```
/\A[+-]?\d+\z/
```

Copy

Otherwise, it will try to convert the value to a number using `Float`. `Float`s are cast to `BigDecimal` using the column's precision value or a maximum of 15 digits.

```
class Player < ApplicationRecord
  validates :points, numericality: true
  validates :games_played, numericality: { only_integer: true }
end
```

Copy

The default error message for `:only_integer` is *"must be an integer"*.

Besides `:only_integer`, this helper also accepts the `:only_numeric` option which specifies the value must be an instance of `Numeric` and attempts to parse the value if it is a `String`.

By default, `numericality` doesn't allow `nil` values. You can use `allow_nil: true`option to permit it. Notice that for `Integer` and `Float` columns empty strings are converted to `nil`.

The default error message when no options are specified is *"is not a number"*.

There are also many options that can be used to add constraints to acceptable values:

- `:greater_than` - Specifies the value must be greater than the supplied value. The default error message for this option is *"must be greater than %{count}"*.
- `:greater_than_or_equal_to` - Specifies the value must be greater than or equal to the supplied value. The default error message for this option is *"must be greater than or equal to %{count}"*.
- `:equal_to` - Specifies the value must be equal to the supplied value. The default error message for this option is *"must be equal to %{count}"*.
- `:less_than` - Specifies the value must be less than the supplied value. The default error message for this option is *"must be less than %{count}"*.
- `:less_than_or_equal_to` - Specifies the value must be less than or equal to the supplied value. The default error message for this option is *"must be less than or equal to %{count}"*.
- `:other_than` - Specifies the value must be other than the supplied value. The default error message for this option is *"must be other than %{count}"*.
- `:in` - Specifies the value must be in the supplied range. The default error message for this option is *"must be in %{count}"*.
- `:odd` - Specifies the value must be an odd number. The default error message for this option is *"must be odd"*.
- `:even` - Specifies the value must be an even number. The default error message for this option is *"must be even"*.

### [2.9 `presence`](https://guides.rubyonrails.org/active_record_validations.html#presence)

This helper validates that the specified attributes are not empty. It uses the [`Object#blank?`](https://api.rubyonrails.org/v7.1.3/classes/Object.html#method-i-blank-3F)method to check if the value is either `nil` or a blank string, that is, a string that is either empty or consists of whitespace.

```
class Person < ApplicationRecord
  validates :name, :login, :email, presence: true
end
```

Copy

If you want to be sure that an association is present, you'll need to test whether the associated object itself is present, and not the foreign key used to map the association. This way, it is not only checked that the foreign key is not empty but also that the referenced object exists.

```
class Supplier < ApplicationRecord
  has_one :account
  validates :account, presence: true
end
```

Copy

In order to validate associated records whose presence is required, you must specify the `:inverse_of` option for the association:

```
class Order < ApplicationRecord
  has_many :line_items, inverse_of: :order
end
```

Copy

If you want to ensure that the association it is both present and valid, you also need to use `validates_associated`. More on that [below](https://guides.rubyonrails.org/active_record_validations.html#validates-associated)

If you validate the presence of an object associated via a `has_one` or `has_many` relationship, it will check that the object is neither `blank?` nor `marked_for_destruction?`.

Since `false.blank?` is true, if you want to validate the presence of a boolean field you should use one of the following validations:

```
# Value _must_ be true or false
validates :boolean_field_name, inclusion: [true, false]
# Value _must not_ be nil, aka true or false
validates :boolean_field_name, exclusion: [nil]
```

Copy

By using one of these validations, you will ensure the value will NOT be `nil` which would result in a `NULL` value in most cases.

The default error message is *"can't be blank"*.

### [2.10 `absence`](https://guides.rubyonrails.org/active_record_validations.html#absence)

This helper validates that the specified attributes are absent. It uses the [`Object#present?`](https://api.rubyonrails.org/v7.1.3/classes/Object.html#method-i-present-3F) method to check if the value is not either nil or a blank string, that is, a string that is either empty or consists of whitespace.

```
class Person < ApplicationRecord
  validates :name, :login, :email, absence: true
end
```

Copy

If you want to be sure that an association is absent, you'll need to test whether the associated object itself is absent, and not the foreign key used to map the association.

```
class LineItem < ApplicationRecord
  belongs_to :order
  validates :order, absence: true
end
```

Copy

In order to validate associated records whose absence is required, you must specify the `:inverse_of` option for the association:

```
class Order < ApplicationRecord
  has_many :line_items, inverse_of: :order
end
```

Copy

If you want to ensure that the association it is both present and valid, you also need to use `validates_associated`. More on that [below](https://guides.rubyonrails.org/active_record_validations.html#validates-associated)

If you validate the absence of an object associated via a `has_one` or `has_many` relationship, it will check that the object is neither `present?` nor `marked_for_destruction?`.

Since `false.present?` is false, if you want to validate the absence of a boolean field you should use `validates :field_name, exclusion: { in: [true, false] }`.

The default error message is *"must be blank"*.

### [2.11 `uniqueness`](https://guides.rubyonrails.org/active_record_validations.html#uniqueness)

This helper validates that the attribute's value is unique right before the object gets saved.

```
class Account < ApplicationRecord
  validates :email, uniqueness: true
end
```

Copy

The validation happens by performing an SQL query into the model's table, searching for an existing record with the same value in that attribute.

There is a `:scope` option that you can use to specify one or more attributes that are used to limit the uniqueness check:

```
class Holiday < ApplicationRecord
  validates :name, uniqueness: { scope: :year,
    message: "should happen once per year" }
end
```

Copy

This validation does not create a uniqueness constraint in the database, so it may happen that two different database connections create two records with the same value for a column that you intend to be unique. To avoid that, you must create a unique index on that column in your database.

In order to add a uniqueness database constraint on your database, use the [`add_index`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html#method-i-add_index) statement in a migration and include the `unique: true` option.

Should you wish to create a database constraint to prevent possible violations of a uniqueness validation using the `:scope` option, you must create a unique index on both columns in your database. See [the MySQL manual](https://dev.mysql.com/doc/refman/en/multiple-column-indexes.html) for more details about multiple column indexes or [the PostgreSQL manual](https://www.postgresql.org/docs/current/static/ddl-constraints.html) for examples of unique constraints that refer to a group of columns.

There is also a `:case_sensitive` option that you can use to define whether the uniqueness constraint will be case sensitive, case insensitive, or if it should respect the default database collation. This option defaults to respecting the default database collation.

```
class Person < ApplicationRecord
  validates :name, uniqueness: { case_sensitive: false }
end
```

Copy

Note that some databases are configured to perform case-insensitive searches anyway.

There is a `:conditions` option that you can specify additional conditions as a `WHERE` SQL fragment to limit the uniqueness constraint lookup (e.g. `conditions: -> { where(status: 'active') }`).

The default error message is *"has already been taken"*.

See [`validates_uniqueness_of`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Validations/ClassMethods.html#method-i-validates_uniqueness_of) for more information.

### [2.12 `validates_associated`](https://guides.rubyonrails.org/active_record_validations.html#validates-associated)

You should use this helper when your model has associations that always need to be validated. Every time you try to save your object, `valid?` will be called on each one of the associated objects.

```
class Library < ApplicationRecord
  has_many :books
  validates_associated :books
end
```

Copy

This validation will work with all of the association types.

Don't use `validates_associated` on both ends of your associations. They would call each other in an infinite loop.

The default error message for [`validates_associated`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Validations/ClassMethods.html#method-i-validates_associated) is *"is invalid"*. Note that each associated object will contain its own `errors` collection; errors do not bubble up to the calling model.

[`validates_associated`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Validations/ClassMethods.html#method-i-validates_associated) can only be used with ActiveRecord objects, everything up until now can also be used on any object which includes [`ActiveModel::Validations`](https://api.rubyonrails.org/v7.1.3/classes/ActiveModel/Validations.html).

### [2.13 `validates_each`](https://guides.rubyonrails.org/active_record_validations.html#validates-each)

This helper validates attributes against a block. It doesn't have a predefined validation function. You should create one using a block, and every attribute passed to [`validates_each`](https://api.rubyonrails.org/v7.1.3/classes/ActiveModel/Validations/ClassMethods.html#method-i-validates_each) will be tested against it.

In the following example, we will reject names and surnames that begin with lower case.

```
class Person < ApplicationRecord
  validates_each :name, :surname do |record, attr, value|
    record.errors.add(attr, 'must start with upper case') if /\A[[:lower:]]/.match?(value)
  end
end
```

Copy

The block receives the record, the attribute's name, and the attribute's value.

You can do anything you like to check for valid data within the block. If your validation fails, you should add an error to the model, therefore making it invalid.

### [2.14 `validates_with`](https://guides.rubyonrails.org/active_record_validations.html#validates-with)

This helper passes the record to a separate class for validation.

```
class GoodnessValidator < ActiveModel::Validator
  def validate(record)
    if record.first_name == "Evil"
      record.errors.add :base, "This person is evil"
    end
  end
end

class Person < ApplicationRecord
  validates_with GoodnessValidator
end
```

Copy

There is no default error message for `validates_with`. You must manually add errors to the record's errors collection in the validator class.

Errors added to `record.errors[:base]` relate to the state of the record as a whole.

To implement the validate method, you must accept a `record` parameter in the method definition, which is the record to be validated.

If you want to add an error on a specific attribute, pass it as the first argument, such as `record.errors.add(:first_name, "please choose another name")`. We will cover [validation errors](https://guides.rubyonrails.org/active_record_validations.html#working-with-validation-errors) in greater detail later.

```
def validate(record)
  if record.some_field != "acceptable"
    record.errors.add :some_field, "this field is unacceptable"
  end
end
```

Copy

The [`validates_with`](https://api.rubyonrails.org/v7.1.3/classes/ActiveModel/Validations/ClassMethods.html#method-i-validates_with) helper takes a class, or a list of classes to use for validation.

```
class Person < ApplicationRecord
  validates_with MyValidator, MyOtherValidator, on: :create
end
```

Copy

Like all other validations, `validates_with` takes the `:if`, `:unless` and `:on` options. If you pass any other options, it will send those options to the validator class as `options`:

```
class GoodnessValidator < ActiveModel::Validator
  def validate(record)
    if options[:fields].any? { |field| record.send(field) == "Evil" }
      record.errors.add :base, "This person is evil"
    end
  end
end

class Person < ApplicationRecord
  validates_with GoodnessValidator, fields: [:first_name, :last_name]
end
```

Copy

Note that the validator will be initialized *only once* for the whole application life cycle, and not on each validation run, so be careful about using instance variables inside it.

If your validator is complex enough that you want instance variables, you can easily use a plain old Ruby object instead:

```
class Person < ApplicationRecord
  validate do |person|
    GoodnessValidator.new(person).validate
  end
end

class GoodnessValidator
  def initialize(person)
    @person = person
  end

  def validate
    if some_complex_condition_involving_ivars_and_private_methods?
      @person.errors.add :base, "This person is evil"
    end
  end

  # ...
end
```

Copy

We will cover [custom validations](https://guides.rubyonrails.org/active_record_validations.html#performing-custom-validations) more later.

## [3 Common Validation Options](https://guides.rubyonrails.org/active_record_validations.html#common-validation-options)

There are several common options supported by the validators we just went over, let's go over some of them now!

Not all of these options are supported by every validator, please refer to the API documentation for [`ActiveModel::Validations`](https://api.rubyonrails.org/v7.1.3/classes/ActiveModel/Validations.html).

By using any of the validation methods we just mentioned, there also is a list of common options shared along with validators. We will cover these now!

- [`:allow_nil`](https://guides.rubyonrails.org/active_record_validations.html#allow-nil): Skip validation if the attribute is `nil`.
- [`:allow_blank`](https://guides.rubyonrails.org/active_record_validations.html#allow-blank): Skip validation if the attribute is blank.
- [`:message`](https://guides.rubyonrails.org/active_record_validations.html#message): Specify a custom error message.
- [`:on`](https://guides.rubyonrails.org/active_record_validations.html#on): Specify the contexts where this validation is active.
- [`:strict`](https://guides.rubyonrails.org/active_record_validations.html#strict-validations): Raise an exception when the validation fails.
- [`:if` and `:unless`](https://guides.rubyonrails.org/active_record_validations.html#conditional-validation): Specify when the validation should or should not occur.

### [3.1 `:allow_nil`](https://guides.rubyonrails.org/active_record_validations.html#allow-nil)

The `:allow_nil` option skips the validation when the value being validated is `nil`.

```
class Coffee < ApplicationRecord
  validates :size, inclusion: { in: %w(small medium large),
    message: "%{value} is not a valid size" }, allow_nil: true
end
```

Copy

```
irb> Coffee.create(size: nil).valid?
=> true
irb> Coffee.create(size: "mega").valid?
=> false
```

Copy

For full options to the message argument please see the [message documentation](https://guides.rubyonrails.org/active_record_validations.html#message).

### [3.2 `:allow_blank`](https://guides.rubyonrails.org/active_record_validations.html#allow-blank)

The `:allow_blank` option is similar to the `:allow_nil` option. This option will let validation pass if the attribute's value is `blank?`, like `nil` or an empty string for example.

```
class Topic < ApplicationRecord
  validates :title, length: { is: 5 }, allow_blank: true
end
```

Copy

```
irb> Topic.create(title: "").valid?
=> true
irb> Topic.create(title: nil).valid?
=> true
```

Copy

### [3.3 `:message`](https://guides.rubyonrails.org/active_record_validations.html#message)

As you've already seen, the `:message` option lets you specify the message that will be added to the `errors` collection when validation fails. When this option is not used, Active Record will use the respective default error message for each validation helper.

The `:message` option accepts either a `String` or `Proc` as its value.

A `String` `:message` value can optionally contain any/all of `%{value}`, `%{attribute}`, and `%{model}` which will be dynamically replaced when validation fails. This replacement is done using the i18n gem, and the placeholders must match exactly, no spaces are allowed.

```
class Person < ApplicationRecord
  # Hard-coded message
  validates :name, presence: { message: "must be given please" }

  # Message with dynamic attribute value. %{value} will be replaced
  # with the actual value of the attribute. %{attribute} and %{model}
  # are also available.
  validates :age, numericality: { message: "%{value} seems wrong" }
end
```

Copy

A `Proc` `:message` value is given two arguments: the object being validated, and a hash with `:model`, `:attribute`, and `:value` key-value pairs.

```
class Person < ApplicationRecord
  validates :username,
    uniqueness: {
      # object = person object being validated
      # data = { model: "Person", attribute: "Username", value: <username> }
      message: ->(object, data) do
        "Hey #{object.name}, #{data[:value]} is already taken."
      end
    }
end
```

Copy

### [3.4 `:on`](https://guides.rubyonrails.org/active_record_validations.html#on)

The `:on` option lets you specify when the validation should happen. The default behavior for all the built-in validation helpers is to be run on save (both when you're creating a new record and when you're updating it). If you want to change it, you can use `on: :create` to run the validation only when a new record is created or `on: :update` to run the validation only when a record is updated.

```
class Person < ApplicationRecord
  # it will be possible to update email with a duplicated value
  validates :email, uniqueness: true, on: :create

  # it will be possible to create the record with a non-numerical age
  validates :age, numericality: true, on: :update

  # the default (validates on both create and update)
  validates :name, presence: true
end
```

Copy

You can also use `on:` to define custom contexts. Custom contexts need to be triggered explicitly by passing the name of the context to `valid?`, `invalid?`, or `save`.

```
class Person < ApplicationRecord
  validates :email, uniqueness: true, on: :account_setup
  validates :age, numericality: true, on: :account_setup
end
```

Copy

```
irb> person = Person.new(age: 'thirty-three')
irb> person.valid?
=> true
irb> person.valid?(:account_setup)
=> false
irb> person.errors.messages
=> {:email=>["has already been taken"], :age=>["is not a number"]}
```

Copy

`person.valid?(:account_setup)` executes both the validations without saving the model. `person.save(context: :account_setup)` validates `person` in the `account_setup`context before saving.

Passing an array of symbols is also acceptable.

```
class Book
  include ActiveModel::Validations

  validates :title, presence: true, on: [:update, :ensure_title]
end
```

Copy

```
irb> book = Book.new(title: nil)
irb> book.valid?
=> true
irb> book.valid?(:ensure_title)
=> false
irb> book.errors.messages
=> {:title=>["can't be blank"]}
```

Copy

When triggered by an explicit context, validations are run for that context, as well as any validations *without* a context.

```
class Person < ApplicationRecord
  validates :email, uniqueness: true, on: :account_setup
  validates :age, numericality: true, on: :account_setup
  validates :name, presence: true
end
```

Copy

```
irb> person = Person.new
irb> person.valid?(:account_setup)
=> false
irb> person.errors.messages
=> {:email=>["has already been taken"], :age=>["is not a number"], :name=>["can't be blank"]}
```

Copy

We will cover more use-cases for `on:` in the [callbacks guide](https://guides.rubyonrails.org/active_record_callbacks.html).

## [4 Strict Validations](https://guides.rubyonrails.org/active_record_validations.html#strict-validations)

You can also specify validations to be strict and raise`ActiveModel::StrictValidationFailed` when the object is invalid.

```
class Person < ApplicationRecord
  validates :name, presence: { strict: true }
end
```

Copy

```
irb> Person.new.valid?
ActiveModel::StrictValidationFailed: Name can't be blank
```

Copy

There is also the ability to pass a custom exception to the `:strict` option.

```
class Person < ApplicationRecord
  validates :token, presence: true, uniqueness: true, strict: TokenGenerationException
end
```

Copy

```
irb> Person.new.valid?
TokenGenerationException: Token can't be blank
```

Copy

## [5 Conditional Validation](https://guides.rubyonrails.org/active_record_validations.html#conditional-validation)

Sometimes it will make sense to validate an object only when a given predicate is satisfied. You can do that by using the `:if` and `:unless` options, which can take a symbol, a `Proc` or an `Array`. You may use the `:if` option when you want to specify when the validation **should** happen. Alternatively, if you want to specify when the validation **should not** happen, then you may use the`:unless` option.

### [5.1 Using a Symbol with `:if` and `:unless`](https://guides.rubyonrails.org/active_record_validations.html#using-a-symbol-with-if-and-unless)

You can associate the `:if` and `:unless` options with a symbol corresponding to the name of a method that will get called right before validation happens. This is the most commonly used option.

```
class Order < ApplicationRecord
  validates :card_number, presence: true, if: :paid_with_card?

  def paid_with_card?
    payment_type == "card"
  end
end
```

Copy

### [5.2 Using a Proc with `:if` and `:unless`](https://guides.rubyonrails.org/active_record_validations.html#using-a-proc-with-if-and-unless)

It is possible to associate `:if` and `:unless` with a `Proc` object which will be called. Using a `Proc` object gives you the ability to write an inline condition instead of a separate method. This option is best suited for one-liners.

```
class Account < ApplicationRecord
  validates :password, confirmation: true,
    unless: Proc.new { |a| a.password.blank? }
end
```

Copy

As `lambda` is a type of `Proc`, it can also be used to write inline conditions taking advantage of the shortened syntax.

```
validates :password, confirmation: true, unless: -> { password.blank? }
```

Copy

### [5.3 Grouping Conditional Validations](https://guides.rubyonrails.org/active_record_validations.html#grouping-conditional-validations)

Sometimes it is useful to have multiple validations use one condition. It can be easily achieved using [`with_options`](https://api.rubyonrails.org/v7.1.3/classes/Object.html#method-i-with_options).

```
class User < ApplicationRecord
  with_options if: :is_admin? do |admin|
    admin.validates :password, length: { minimum: 10 }
    admin.validates :email, presence: true
  end
end
```

Copy

All validations inside of the `with_options` block will have automatically passed the condition `if: :is_admin?`

### [5.4 Combining Validation Conditions](https://guides.rubyonrails.org/active_record_validations.html#combining-validation-conditions)

On the other hand, when multiple conditions define whether or not a validation should happen, an `Array` can be used. Moreover, you can apply both `:if` and `:unless` to the same validation.

```
class Computer < ApplicationRecord
  validates :mouse, presence: true,
                    if: [Proc.new { |c| c.market.retail? }, :desktop?],
                    unless: Proc.new { |c| c.trackpad.present? }
end
```

Copy

The validation only runs when all the `:if` conditions and none of the `:unless` conditions are evaluated to `true`.

## [6 Performing Custom Validations](https://guides.rubyonrails.org/active_record_validations.html#performing-custom-validations)

When the built-in validation helpers are not enough for your needs, you can write your own validators or validation methods as you prefer.

### [6.1 Custom Validators](https://guides.rubyonrails.org/active_record_validations.html#custom-validators)

Custom validators are classes that inherit from [`ActiveModel::Validator`](https://api.rubyonrails.org/v7.1.3/classes/ActiveModel/Validator.html). These classes must implement the `validate` method which takes a record as an argument and performs the validation on it. The custom validator is called using the `validates_with` method.

```
class MyValidator < ActiveModel::Validator
  def validate(record)
    unless record.name.start_with? 'X'
      record.errors.add :name, "Provide a name starting with X, please!"
    end
  end
end

class Person < ApplicationRecord
  validates_with MyValidator
end
```

Copy

The easiest way to add custom validators for validating individual attributes is with the convenient [`ActiveModel::EachValidator`](https://api.rubyonrails.org/v7.1.3/classes/ActiveModel/EachValidator.html). In this case, the custom validator class must implement a `validate_each` method which takes three arguments: record, attribute, and value. These correspond to the instance, the attribute to be validated, and the value of the attribute in the passed instance.

```
class EmailValidator < ActiveModel::EachValidator
  def validate_each(record, attribute, value)
    unless URI::MailTo::EMAIL_REGEXP.match?(value)
      record.errors.add attribute, (options[:message] || "is not an email")
    end
  end
end

class Person < ApplicationRecord
  validates :email, presence: true, email: true
end
```

Copy

As shown in the example, you can also combine standard validations with your own custom validators.

### [6.2 Custom Methods](https://guides.rubyonrails.org/active_record_validations.html#custom-methods)

You can also create methods that verify the state of your models and add errors to the `errors`collection when they are invalid. You must then register these methods by using the [`validate`](https://api.rubyonrails.org/v7.1.3/classes/ActiveModel/Validations/ClassMethods.html#method-i-validate) class method, passing in the symbols for the validation methods' names.

You can pass more than one symbol for each class method and the respective validations will be run in the same order as they were registered.

The `valid?` method will verify that the `errors` collection is empty, so your custom validation methods should add errors to it when you wish validation to fail:

```
class Invoice < ApplicationRecord
  validate :expiration_date_cannot_be_in_the_past,
    :discount_cannot_be_greater_than_total_value

  def expiration_date_cannot_be_in_the_past
    if expiration_date.present? && expiration_date < Date.today
      errors.add(:expiration_date, "can't be in the past")
    end
  end

  def discount_cannot_be_greater_than_total_value
    if discount > total_value
      errors.add(:discount, "can't be greater than total value")
    end
  end
end
```

Copy

By default, such validations will run every time you call `valid?` or save the object. But it is also possible to control when to run these custom validations by giving an `:on` option to the `validate`method, with either: `:create` or `:update`.

```
class Invoice < ApplicationRecord
  validate :active_customer, on: :create

  def active_customer
    errors.add(:customer_id, "is not active") unless customer.active?
  end
end
```

Copy

See the section above for more details about [`:on`](https://guides.rubyonrails.org/active_record_validations.html#on).

### [6.3 Listing Validators](https://guides.rubyonrails.org/active_record_validations.html#listing-validators)

If you want to find out all of the validators for a given objects, look no further than `validators`.

For example, if we have the following model using a custom validator and a built-in validator:

```
class Person < ApplicationRecord
  validates :name, presence: true, on: :create
  validates :email, format: URI::MailTo::EMAIL_REGEXP
  validates_with MyOtherValidator, strict: true
end
```

Copy

We can now use `validators` on the "Person" model to list all validators, or even check a specific field using `validators_on`.

```
irb> Person.validators
#=> [#<ActiveRecord::Validations::PresenceValidator:0x10b2f2158
      @attributes=[:name], @options={:on=>:create}>,
     #<MyOtherValidatorValidator:0x10b2f17d0
      @attributes=[:name], @options={:strict=>true}>,
     #<ActiveModel::Validations::FormatValidator:0x10b2f0f10
      @attributes=[:email],
      @options={:with=>URI::MailTo::EMAIL_REGEXP}>]
     #<MyOtherValidator:0x10b2f0948 @options={:strict=>true}>]

irb> Person.validators_on(:name)
#=> [#<ActiveModel::Validations::PresenceValidator:0x10b2f2158
      @attributes=[:name], @options={on: :create}>]
```

Copy

## [7 Working with Validation Errors](https://guides.rubyonrails.org/active_record_validations.html#working-with-validation-errors)

The [`valid?`](https://api.rubyonrails.org/v7.1.3/classes/ActiveRecord/Validations.html#method-i-valid-3F) and [`invalid?`](https://api.rubyonrails.org/v7.1.3/classes/ActiveModel/Validations.html#method-i-invalid-3F) methods only provide a summary status on validity. However you can dig deeper into each individual error by using various methods from the [`errors`](https://api.rubyonrails.org/v7.1.3/classes/ActiveModel/Validations.html#method-i-errors) collection.

The following is a list of the most commonly used methods. Please refer to the[`ActiveModel::Errors`](https://api.rubyonrails.org/v7.1.3/classes/ActiveModel/Errors.html) documentation for a list of all the available methods.

### [7.1 `errors`](https://guides.rubyonrails.org/active_record_validations.html#working-with-validation-errors-errors)

The gateway through which you can drill down into various details of each error.

This returns an instance of the class `ActiveModel::Errors` containing all errors, each error is represented by an [`ActiveModel::Error`](https://api.rubyonrails.org/v7.1.3/classes/ActiveModel/Error.html) object.

```
class Person < ApplicationRecord
  validates :name, presence: true, length: { minimum: 3 }
end
```

Copy

```
irb> person = Person.new
irb> person.valid?
=> false
irb> person.errors.full_messages
=> ["Name can't be blank", "Name is too short (minimum is 3 characters)"]

irb> person = Person.new(name: "John Doe")
irb> person.valid?
=> true
irb> person.errors.full_messages
=> []

irb> person = Person.new
irb> person.valid?
=> false
irb> person.errors.first.details
=> {:error=>:too_short, :count=>3}
```

Copy

### [7.2 `errors[\]`](https://guides.rubyonrails.org/active_record_validations.html#errors)

[`errors[\]`](https://api.rubyonrails.org/v7.1.3/classes/ActiveModel/Errors.html#method-i-5B-5D) is used when you want to check the error messages for a specific attribute. It returns an array of strings with all error messages for the given attribute, each string with one error message. If there are no errors related to the attribute, it returns an empty array.

```
class Person < ApplicationRecord
  validates :name, presence: true, length: { minimum: 3 }
end
```

Copy

```
irb> person = Person.new(name: "John Doe")
irb> person.valid?
=> true
irb> person.errors[:name]
=> []

irb> person = Person.new(name: "JD")
irb> person.valid?
=> false
irb> person.errors[:name]
=> ["is too short (minimum is 3 characters)"]

irb> person = Person.new
irb> person.valid?
=> false
irb> person.errors[:name]
=> ["can't be blank", "is too short (minimum is 3 characters)"]
```

Copy

### [7.3 `errors.where` and Error Object](https://guides.rubyonrails.org/active_record_validations.html#errors-where-and-error-object)

Sometimes we may need more information about each error besides its message. Each error is encapsulated as an `ActiveModel::Error` object, and the [`where`](https://api.rubyonrails.org/v7.1.3/classes/ActiveModel/Errors.html#method-i-where) method is the most common way of access.

`where` returns an array of error objects filtered by various degrees of conditions.

```
class Person < ApplicationRecord
  validates :name, presence: true, length: { minimum: 3 }
end
```

Copy

We can filter for just the `attribute` by passing it as the first parameter to`errors.where(:attr)`. The second parameter is used for filtering the `type` of error we want by calling `errors.where(:attr, :type)`.

```
irb> person = Person.new
irb> person.valid?
=> false

irb> person.errors.where(:name)
=> [ ... ] # all errors for :name attribute

irb> person.errors.where(:name, :too_short)
=> [ ... ] # :too_short errors for :name attribute
```

Copy

Lastly, we can filter by any `options` that may exist on the given type of error object.

```
irb> person = Person.new
irb> person.valid?
=> false

irb> person.errors.where(:name, :too_short, minimum: 3)
=> [ ... ] # all name errors being too short and minimum is 2
```

Copy

You can read various information from these error objects:

```
irb> error = person.errors.where(:name).last

irb> error.attribute
=> :name
irb> error.type
=> :too_short
irb> error.options[:count]
=> 3
```

Copy

You can also generate the error message:

```
irb> error.message
=> "is too short (minimum is 3 characters)"
irb> error.full_message
=> "Name is too short (minimum is 3 characters)"
```

Copy

The [`full_message`](https://api.rubyonrails.org/v7.1.3/classes/ActiveModel/Errors.html#method-i-full_message) method generates a more user-friendly message, with the capitalized attribute name prepended. (To customize the format that `full_message` uses, see the [I18n guide](https://guides.rubyonrails.org/i18n.html#active-model-methods).)

### [7.4 `errors.add`](https://guides.rubyonrails.org/active_record_validations.html#errors-add)

The [`add`](https://api.rubyonrails.org/v7.1.3/classes/ActiveModel/Errors.html#method-i-add) method creates the error object by taking the `attribute`, the error `type` and additional options hash. This is useful when writing your own validator, as it lets you define very specific error situations.

```
class Person < ApplicationRecord
  validate do |person|
    errors.add :name, :too_plain, message: "is not cool enough"
  end
end
```

Copy

```
irb> person = Person.create
irb> person.errors.where(:name).first.type
=> :too_plain
irb> person.errors.where(:name).first.full_message
=> "Name is not cool enough"
```

Copy

### [7.5 `errors[:base\]`](https://guides.rubyonrails.org/active_record_validations.html#errors-base)

You can add errors that are related to the object's state as a whole, instead of being related to a specific attribute. To do this you must use `:base` as the attribute when adding a new error.

```
class Person < ApplicationRecord
  validate do |person|
    errors.add :base, :invalid, message: "This person is invalid because ..."
  end
end
```

Copy

```
irb> person = Person.create
irb> person.errors.where(:base).first.full_message
=> "This person is invalid because ..."
```

Copy

### [7.6 `errors.size`](https://guides.rubyonrails.org/active_record_validations.html#errors-size)

The `size` method returns the total number of errors for the object.

```
class Person < ApplicationRecord
  validates :name, presence: true, length: { minimum: 3 }
end
```

Copy

```
irb> person = Person.new
irb> person.valid?
=> false
irb> person.errors.size
=> 2

irb> person = Person.new(name: "Andrea", email: "andrea@example.com")
irb> person.valid?
=> true
irb> person.errors.size
=> 0
```

Copy

### [7.7 `errors.clear`](https://guides.rubyonrails.org/active_record_validations.html#errors-clear)

The `clear` method is used when you intentionally want to clear the `errors` collection. Of course, calling `errors.clear` upon an invalid object won't actually make it valid: the `errors` collection will now be empty, but the next time you call `valid?` or any method that tries to save this object to the database, the validations will run again. If any of the validations fail, the `errors` collection will be filled again.

```
class Person < ApplicationRecord
  validates :name, presence: true, length: { minimum: 3 }
end
```

Copy

```
irb> person = Person.new
irb> person.valid?
=> false
irb> person.errors.empty?
=> false

irb> person.errors.clear
irb> person.errors.empty?
=> true

irb> person.save
=> false

irb> person.errors.empty?
=> false
```

Copy

## [8 Displaying Validation Errors in Views](https://guides.rubyonrails.org/active_record_validations.html#displaying-validation-errors-in-views)

Once you've created a model and added validations, if that model is created via a web form, you probably want to display an error message when one of the validations fails.

Because every application handles this kind of thing differently, Rails does not include any view helpers to help you generate these messages directly. However, due to the rich number of methods Rails gives you to interact with validations in general, you can build your own. In addition, when generating a scaffold, Rails will put some ERB into the `_form.html.erb` that it generates that displays the full list of errors on that model.

Assuming we have a model that's been saved in an instance variable named `@article`, it looks like this:

```
<% if @article.errors.any? %>
  <div id="error_explanation">
    <h2><%= pluralize(@article.errors.count, "error") %> prohibited this article from being saved:</h2>

    <ul>
      <% @article.errors.each do |error| %>
        <li><%= error.full_message %></li>
      <% end %>
    </ul>
  </div>
<% end %>
```

Copy

Furthermore, if you use the Rails form helpers to generate your forms, when a validation error occurs on a field, it will generate an extra `<div>` around the entry.

```
<div class="field_with_errors">
  <input id="article_title" name="article[title]" size="30" type="text" value="">
</div>
```

Copy

You can then style this div however you'd like. The default scaffold that Rails generates, for example, adds this CSS rule:

```
.field_with_errors {
  padding: 2px;
  background-color: red;
  display: table;
}
```

Copy

This means that any field with an error ends up with a 2 pixel red border.

