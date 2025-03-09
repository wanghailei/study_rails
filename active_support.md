# Active Support



## `CurrentAttributes`

Abstract super class that provides a thread-isolated attributes singleton, which resets automatically before and after each request. This allows you to keep all the per-request attributes easily available to the whole system.

The following full app-like example demonstrates how to ==use a `Current` class to facilitate easy access to the global, per-request attributes without passing them deeply around everywhere==:

```ruby
# app/models/current.rb
class Current < ActiveSupport::CurrentAttributes
	attribute :account, :user
	attribute :request_id, :user_agent, :ip_address
    
	resets { Time.zone = nil }
    
	def user=(user)
		super
		self.account = user.account
		Time.zone    = user.time_zone
	end
end

# app/controllers/concerns/authentication.rb
module Authentication
	extend ActiveSupport::Concern
    
	included do
		before_action :authenticate
	end
    
private

    def authenticate
		if authenticated_user = User.find_by(id: cookies.encrypted[:user_id])
			Current.user = authenticated_user
		else
			redirect_to new_session_url
		end
	end
end

# app/controllers/concerns/set_current_request_details.rb
module SetCurrentRequestDetails
	extend ActiveSupport::Concern
    
	included do
		before_action do
			Current.request_id = request.uuid
			Current.user_agent = request.user_agent
			Current.ip_address = request.ip
		end
	end
end

class ApplicationController < ActionController::Base
	include Authentication
	include SetCurrentRequestDetails
end

class MessagesController < ApplicationController
	def create
		Current.account.messages.create(message_params)
	end
end

class Message < ApplicationRecord
	belongs_to :creator, default: -> { Current.user }
	after_create { |message| Event.create(record: message) }
end

class Event < ApplicationRecord
	before_create do
		self.request_id = Current.request_id
		self.user_agent = Current.user_agent
		self.ip_address = Current.ip_address
	end
end
```

A word of caution: It's easy to overdo a global singleton like Current and tangle your model as a result. ==Current should only be used for a few, top-level globals, like account, user, and request details.== The attributes stuck in Current should be used by more or less all actions on all requests. If you start sticking controller-specific attributes in there, you're going to create a mess.

## Concern



```ruby
# Evaluate given block in context of base class, so that you can write class macros here.
# When you define more than one +included+ block, it raises an exception.
def included( base = nil, &block )
	if base.nil?
		if instance_variable_defined?(:@_included_block)
			if @_included_block.source_location != block.source_location
				raise MultipleIncludedBlocks
			end
		else
			@_included_block = block
		end
	else
		super
	end
end
```

==The Ruby Rails "concern" pattern encourages a more organized way of managing related functionalities==, especially in cases where an `ActiveRecord` model class has become bloated or encompasses too much functionality.

`ActiveSupport` provides the `Concern` module to allowing grouping of related methods, classes, and other encapsulated parts of a program in a neat way. ==When a module with `ActiveSupport::Concern` gets included, it extends the Concern module's instance methods in addition to the module's own methods.==

This `included` method is within the context of the Rails "concern" pattern. 

==The method `included` is used to handle the blocks that are executed in the context of the base class when modules that use `ActiveSupport::Concern` are included.== The base class is the class that includes the module. This is very useful for defining class level functionality in the included module.

The `included` method takes two arguments: an optional base and a block provided with & since its a block. 

It checks if `base` argument is `nil`. If it is `nil`, that means `included` is being used as a setter to define the block to be executed later. In such case, first it checks if an instance variable `@_included_block` has been already defined or not. `@_included_block` is supposed to hold the block that will be executed in the context of the base class.

If @_included_block has been defined earlier, it then checks if the source_location of @_included_block and the incoming block is not the same, it raises MultipleIncludedBlocks error. source_location returns the file and line number of a Proc's (block's) source. This indicates that you can't define multiple included blocks in a concern.

If @_included_block is not defined previously, it defines @_included_block with the provided block.
If base is not `nil`, it calls the super included method. This happens when the module gets included in a class, and the provided block gets executed in the class's context.

