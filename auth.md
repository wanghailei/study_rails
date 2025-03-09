# Auth

## Where the authorisation logic should be defined, in the Controller or the Model?

**ChatGPT**: Authorisation logic can be defined in both the Controller and the Model, depending on the complexity of your application and the level of granularity you need. However, ==it's a common practice to handle most of the authorisation logic at the Controller level.==

Controller Level: The Controller is typically where you'd check if a user is authorised to perform a certain action. For instance, before updating a resource, the Controller could check if the current user has the necessary permissions. This is because Controllers handle user inputs and interactions, making them a good place for such checks.

Model Level: Sometimes, you might need to enforce more granular or complex rules that involve specific business logic. In such cases, it might make sense to put some authorization logic in the Model. For example, you could have a method in the Model that checks if a user is allowed to perform an action based on some business rule.

A good rule of thumb is to put coarse-grained, user-focused authorization checks (like "is the user logged in?" or "is the user an admin?") in the Controller, and put fine-grained, data-focused checks (like "is this user allowed to edit this specific record?") in the Model.

In any case, it's crucial that you carefully consider where to put authorization logic to ensure security and maintainability of your application.

Moreover, many web frameworks provide libraries or modules to handle authorization. For example, in Rails, you might use a gem like Pundit or CanCanCan, which provide a structured way to handle authorization and can help ensure that you don't miss any important checks.

## Authentication

### Authentication with Devise

Devise is a flexible authentication solution for Rails based on Warden. It is Rack based, complete MVC solution based on **Rails engines** and allows you to have multiple models signed in at the same time.

It's based on a modularity concept: use only what you really need. The following are its 10 modules:

\- Database Authenticatable: hashes and stores a password in the database to validate the authenticity of a user while signing in. The authentication can be done both through POST requests or HTTP Basic Authentication.

\- Omniauthable: adds OmniAuth (https://github.com/omniauth/omniauth) support.

\- Confirmable: sends emails with confirmation instructions and verifies whether an account is already confirmed during sign in.

\- Recoverable: resets the user password and sends reset instructions.

\- Registerable: handles signing up users through a registration process, also allowing them to edit and destroy their account.

\- Rememberable: manages generating and clearing a token for remembering the user from a saved cookie.

\- Trackable: tracks sign in count, timestamps and IP address.

\- Timeoutable: expires sessions that have not been active in a specified period of time.

\- Validatable: provides validations of email and password. It's optional and can be customized, so you're able to define your own validations.

\- Lockable: locks an account after a specified number of failed sign-in attempts. Can unlock via email or after a specified time period.
