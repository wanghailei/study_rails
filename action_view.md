# Action View

Action View encapsulates all the functionality needed to render templates, most commonly generating HTML, XML, or JavaScript back to the user.

## Templates

The `render` method expects to find templates in the app/views directory of the current application.

Within this directory, the convention is to have a separate subdirectory for the views of each controller. Each subdirectory typically contains templates named after the actions in the corresponding controller.

Templates contain a mixture of fixed text and code. The code in the template adds dynamic content to the response. That code runs in an environment that gives it access to the information set up by the controller:

All instance variables of the controller are also available in the template. This is how actions communicate data to the templates.

The path to the base directory of the templates is stored in the attribute base_path.

Jbuilder templates generate JSON responses.

In an ERB template, the ``debug`` method can be used to display the contents of the session, the details of the parameters, and the current response.

### Expose one instance variable per Action

The way Rails makes data from controllers available to views is ==by copying the *instance variables of the controller* into the view code as *instance variables with the same name*==.

The way to get the most of Rails’ design without creating a problem is to adopt two conventions:

- ==Expose exactly one instance variable from any given action, ideally named for the resource or resources being manipulated by the route to that action.== For example, the `widget` `show` page should only expose `@widget`. ==The key situation to avoid is exposing multiple instance variables that collectively represent the resource rather than creating a single instance variable (and perhaps a new model class) to do so.==

- There are there exceptions: when a view requires access to reference data, like a list of country codes, when the view needs access to global context, like the currently logged-in user, or when there is UI state that is persisted across page refreshes, such as the currently selected tab in a tab navigation control.

	





## Layout





## Partial



***

