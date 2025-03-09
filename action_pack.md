# Action Pack

## ==Action Pack -- From request to response==

Action Pack is a framework for handling and responding to web requests. It provides mechanisms for _routing_ (mapping request URLs to actions), defining _controllers_ that implement actions, and generating responses. In short, Action Pack provides the controller layer in the MVC paradigm.

It consists of several modules:

* **Action Dispatch**, which parses information about the web request, handles routing as defined by the user, and does advanced processing related to HTTP such as MIME-type negotiation, decoding parameters in POST, PATCH, or PUT bodies, handling HTTP caching logic, cookies and sessions.
* **Action Controlle**r, which provides a base controller class that can be subclassed to implement filters and actions to handle requests. ==The result of an action is typically content generated from views.==

Rails users only directly interface with the Action Controller module. Necessary Action Dispatch functionality is activated by default and Action View rendering is implicitly triggered by Action Controller.

## `AbstractController`

`AbstractController` is a base class in Rails that provides the core functionalities needed for any type of controller, including rendering and layout support, view paths, and callback hooks. ==`AbstractController` alone doesn't know how to handle HTTP requests.==

==`ActionController::Base`, inherited from `AbstractController`, is specifically tailored for handling HTTP requests in a web application.== It adds functionalities like request and response handling, sessions, cookies, and much more.&#x20;

The relationship between `AbstractController` and `ActionController` is that of a **generalisation-specialisation** (or base-derived) relationship.

## `ActionController`

Action Controllers are the core of a web request in Rails. They are made up of one or more actions that are executed on request and then either it renders a template ==or redirects to another action.==

An ActionController is made up of one or more actions that are executed on request and then either it renders a template or redirects to another action. An action is defined as a public method on the controller, which will automatically be made accessible to the web-server through Rails Routes.

## `ApplicationController`

By default, only the `ApplicationController` in a Rails application inherits from `ActionController::Base`. All other controllers inherit from `ApplicationController`.

==`ApplicationController` is a central controller class that you define in your Rails application to include shared functionality across all of your controllers.== This gives you one class to configure things such as request forgery protection and filtering of sensitive request parameters.&#x20;

The `ApplicationController` source code is in a Rails application project folder. ==% 在 Rails 的source code 文件夾裡面找不到一個 ApplicationController 文件，因為它是被 rails new 生成出一個 app 代碼結構後才產生的。20231219 %==

## Action

==An action is defined as **a public method** on the controller==, which will automatically be made accessible to the web-server through Rails Routes.

Actions, by default, render a template in the app/views directory corresponding to the name of the controller and action after executing code in the action. For example, the `index` action of the `PostsController` would render the template app/views/posts/index.html.erb by default after populating the `@posts` instance variable.

Unlike `index`, the `create` action will not render a template. After performing its main purpose (creating a new post), it initiates a `redirect` instead. This `redirect` works by returning an external 302 Moved HTTP response that takes the user to the `index` action. These two methods represent the two basic action archetypes used in Action Controllers: Get-and-show and do-and-redirect. Most actions are variations on these themes.

### `respond_to do | format | ... end`

Rails apps can generate responses in different formats. `respond_to` supports specifying a common block for different formats of the template to be rendered.

## Requests

For every request, the router determines the value of the controller and action keys. These determine which controller and action are called.

The remaining request parameters, the session (if one is available), and the full request with all the HTTP headers are made available to the action through accessor methods. Then the action is performed.

## Parameters

All request parameters, whether they come from a query string in the URL or form data submitted through a POST request are available through the params method which returns a hash.

There are two kinds of parameters possible in a web application.

The first are parameters that are sent as part of the URL, called query string parameters. The query string is everything after "?" in the URL. ==% I call it "Query parameters" or even "GET parameters". %==

The second type of parameter, usually referred to as POST data, comes from an HTML form which has been filled in by the user. It's called POST data because it can only be sent as part of an HTTP POST request. ==% I call it "POST parameters". %==

Rails does not make any distinction between the two kinds of parameters, and both are available in the params hash in your controller.

``params`` may be used for more than text, but entire files can be uploaded. In HTTP, files are uploaded as a ``multipart/form-data POST`` message.

## Sessions

==Sessions allow you to store objects in between requests.== This is useful for objects that are not yet ready to be persisted, such as a Signup object constructed in a multi-paged process, or objects that don't change much and are needed all the time, such as a User object for a system that requires login.&#x20;

The session should not be used, however, as a cache for objects where it's likely they could be changed unknowingly. It's usually too much work to keep it all synchronised – something databases already excel at.

==By default, sessions are stored in an encrypted browser cookie== (see `ActionDispatch::Session::CookieStore`). Thus the user will not be able to read or edit the session data. However, the user can keep a copy of the cookie even after it has expired, so ==you should avoid storing sensitive information in cookie-based sessions==.

## Responses

==Each action results in a `response`==, which holds the headers and document to be sent to the browser. The actual response object is generated automatically through the use of renders and redirects and requires no user intervention.

## Strict Locals

Rails 7.1 introduces a way to opt into "strict" mode for locals in renders. Why?

When you're rendering partials and views, it's easy to introduce bugs when you mistakenly refer to instance variables that haven't been initialized. Normally, Rails would return nil. With "strict" mode enabled, Rails will raise an error if you attempt to refer to an instance variable that hasn't been initialized.

The strict locals mode encourages developers to pass locals explicitly to the render calls rather than relying on instance variables, which can be helpful for making views more predictable and easier to reason about.	

```ruby
# Without strict mode
class UsersController < ApplicationController
	def show
		@user = User.find(params[:id])
	end
end

<p><%= @user.name %></p>
```

In the typical approach of Rails, there's no need to explicitly pass the ``@user`` instance variable to the ``render`` method, but it can be prone to errors when instance variables are ``nil``, which is why strict mode was introduced in Rails 7.1.

```ruby
# With strict mode
class UsersController < ApplicationController
	def show
		user = User.find(params[:id])
		render :show, locals: { user: user }
	end
end

<p><%= user.name %></p>
```

==% 我認為用標準模式比用嚴格模式好，因為更簡潔。我所需要做的是如何在標準模式下消除這個「實例變量未初始化就被調用了」（instance variable non-initialized errors）的隱患。有三種解法，我認為應該用第一種————在 controller methods 裡面完成實例變量初始化。20231231 %==

1. Always initialize the instance variables, ideally with a default value, in the controller, even a variable may not be necessarily used in a view.
	==The ``||=`` operator works by checking if the object on its left side is nil. If it is, it assigns the right-hand side. This makes it a succinct way to give variables a default value.==

	``` ruby
	class UsersController < ApplicationController 
	     def show 
	          # Try to find the user with the given id first. If it cannot find a user, 
	          # it will initialize a new user instance and assign it to the @user instance variable.
			@user = User.find_by( id: params[:id] ) || User.new
	     end 
	end 
	```

2. Assign local variables in ERB templates before using them.

	```ruby
	<% user = @user || User.new %> 
	<p><%= user.name %></p> 
	```

3. Use ==safe navigation operator(``&.``)== of Ruby to prevent ``nil`` errors. This operator only sends the method to the object if the object is not ``nil``. This approach helps to prevent ``NoMethodError`` on ``nil`` values but it doesn't solve the root cause of the problem (uninitialized variables).

	``` ruby
	<%= @user&.name %>
	```

To enable the strict locals mode:

```ruby
# in config/initializers/action_view.rb
Rails.application.config.action_view.raise_on_missing_locals = true
```

Using strict mode can have both advantages and disadvantages.

Advantages:

- It encourages the explicit declaration of locals, which can make code cleaner and easier to maintain. 
- It helps prevent issues where you expected an instance variable to be assigned but it isn't, a situation that could result in unexpected behavior when nil is used instead.

Disadvantages:

- It requires you to explicitly pass all locals to partials, which can be a bit more verbose especially when there are many locals to pass.
- It could cause your application to crash during development if a developer accidentally refers to an unassigned instance variable.



## Routes

Routes' primary purpose is to connect view to controller, and they also serve as a UI element--URL.

Rules:

* Always use canonical routes that conform to Rails’ defaults.
* Avoid custom actions in favor of creating new resources that use Rails’\
	default actions.
* User-friendly URLs should be added in addition to the canonical routes.

==Using resources to define routes:== ======`resources :widgets`====. It's the best way.==

`bin/rails routes -g widgets`

If the app’s routes are made up entirely of calls to resources, it becomes easy to understand the app at a high level.

