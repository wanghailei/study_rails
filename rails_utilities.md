# Rails Utilities



# ActiveSupport

==% ActiveSupport 是個武器庫啊！是個寶庫，慢慢挖掘！20231227 %==

Active Support is a collection of utility classes and Ruby standard library extensions. Active Support is responsible for ==providing Ruby language extensions and utilities==. It offers a richer bottom-line at the language level.

These additions reside in this package so they can be loaded as needed in Ruby projects outside of Rails.

## Concerns

Concerns are a way to make large controllers or models easier to understand and manage. This also has the advantage of reusability when multiple models (or controllers) share the same concerns.

A concern is only responsible for a focused subset of the model's responsibility.

The ActiveSupport::Concern gives us a simpler way to include them.



# Zeitwerk

Rails uses a Gem called Zeitwerk for autoloading. When Rails encounters an undefined constant in the code, it uses Zeitwerk which uses file-naming conventions to find and require the needed Ruby script.

Zeitwerk has the concept of autoload paths, and the default autoload paths include the base directories of just about anywhere you would think of adding code to your Rails application.



# ActionCable - WebSocket

Rails offers an abstraction over WebSockets: ActionCable.





****

# bin/dev

The \``bin/dev`\` script is a new feature of Rails 7 to make it easier to start up your development environment. This script allows you to start the Rails server along with other services your application might depend on.

`bin/dev` will start both the Rails server and the JS build watcher (along with a CSS build watcher, if you're also using cssbundling-rails).

The `bin/dev` script isn't automatically created when you generate a new Rails 7 application, so you'll have to create it yourself.

==% 在 `rails new` 時，如果我安裝了 tailwind-rails 後，就有了 `bin/dev`；但如果我沒安裝，就沒有。如果運行 bin/dev，就會得到報錯：`zsh: no such file or directory: bin/dev`。這種情況下，用 `rails server` 就行了。%==

`jsbundling-rails`, `cssbundling-rails`, and `tailwindcss-rails` all introduce a new `bin/dev` and `Procfile.dev` file to the app when they are installed. This is a good thing as it allows developers to run one command to execute their Rails server and frontend build tools.

### **Here's an example of how to create a simple \`bin/dev\` script:**

1\. In your Rails root directory, create a new file \`bin/dev\` and open it in a text editor.

2\. Add the following content:

\#!/bin/bash

foreman start -f Procfile.dev

This will start all the services defined in your \`Procfile.dev\` using the Foreman tool.

If you're using another process manager like Overmind, replace \`foreman\` with \`overmind\`.

3\. Save and close the file.

4\. Make the script executable by running this command in your terminal:

$ chmod +x bin/dev

Now you can start your development environment by running \`./bin/dev\` from your Rails root directory. This will run all the commands specified in your \`Procfile.dev\`.

Depending on your needs, you might want to add more commands to it, like commands to check if necessary services like PostgreSQL or Redis are running, or commands to run setup scripts.

### Procfile.dev

A Procfile defines various services that your application is composed of, such as a web server and a CSS builder.

`web: bin/rails server -p 3000` is a command to start the Rails server on port 3000.

`css: bin/rails tailwindcss:watch` is a command that presumably watches for changes in your Tailwind CSS files and rebuilds them.

