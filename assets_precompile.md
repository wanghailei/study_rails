# Assets Precompile



==% Assets Pipeline 讓網頁的各種素材加載更快，並確保加在最新的素材，且適應不同瀏覽器。這可以被看作是一個自動化輔助工具，跟開發其實沒什麼關係，提前設置好了就行。20231129 %==

## What does an asset pipeline do?

An asset pipeline:

* improves page loading speed, by minifying and compressing assets.
* reduces the number of browser HTTP requests, by combining multiple files into one.
* ensures browsers load the newest version of the assets, by appending a digest to filenames.
* ensuring cross-browser compatibility, by adding vendor prefixes to CSS.

In development mode, an asset pipeline

* compiles assets from languages like SCSS, Sass, or CoffeeScript into CSS and JavaScript, which browsers can understand.
* automatically compile and refresh assets as they are updated, which speeds up the development process.

Propshaft is one of the modern asset pipeline.



## Useful tips in development:

The application.js and application.css manifest files are the only JavaScript/Stylesheets included during the asset pipeline precompile step. To include additional assets, specify them using the `config.assets.precompile` configuration setting in config/initializers/assets.rb:

`config.assets.precompile += %w( admin.js admin.css )`

### Where to put my own CSS code?

`app/assets/stylesheets/application.css`

Always remember to restart your development server and potentially recompile your assets after making changes to CSS or JavaScript files.

#### Why do I get "application.css not in asset pipeline" in production?

A common issue is that your repository does not contain the output directory used by the build commands. You must have app/assets/builds available. Add the directory with a .gitkeep file, and you'll ensure it's available in production.

#### Why isn't Rails using my updated css files?

Watch out - if you precompile your files locally, those will be served over the dynamically created ones you expect. The solution:

`rails assets:clobber`



## CSS Bundling for Rails

Use Tailwind CSS, Bootstrap, Bulma, PostCSS, or Dart Sass to bundle and process your CSS, then deliver it via the asset pipeline in Rails.

This gem provides installers to get you going with the bundler of your choice in a new Rails application, and a convention to use app/assets/builds to hold your bundled output as artifacts that are not checked into source control (the installer adds this directory to .gitignore by default).

You can also use ./bin/dev, which will start both the Rails server and the CSS build watcher (along with a JS build watcher, if you're also using jsbundling-rails).

Whenever the bundler detects changes to any of the stylesheet files in your project, it'll bundle app/assets/stylesheets/application.\[bundler].css into app/assets/builds/application.css.

When you deploy your application to production, the css:build task attaches to the assets:precompile task to ensure that all your package dependencies from package.json have been installed via yarn, and then runs yarn build:css to process your stylesheet entrypoint, as it would in development. This output is then picked up by the asset pipeline, digested, and copied into public/assets, as any other asset pipeline file.

This also happens in testing where the bundler attaches to the test:prepare task to ensure the stylesheets have been bundled before testing commences.

If your setup uses jsbundling-rails (ie, esbuild + tailwind), you will also need to run javascript:build.

You can configure your bundler options in the build:css script in package.json or via the installer-generated tailwind.config.js for Tailwind or postcss.config.js for PostCSS.



### Tailwind CSS

我認為，可以用tailwind-rails。但只是為了把其特殊的一些class 提取出來。

`Error:`\
`ActionView::Template::Error (The asset "tailwind.css" is not present in the asset pipeline.)`

Solution:

`rails assets:clean assets:precompile`

==% Remember to restart the Rails server to see effect. 20230613 %

## Import Maps

Import Maps manage your application's JavaScript dependencies. ==Import Maps make using most npm packages without the need for transpiling or bundling,== by importing JavaScript modules using logical names that map to versioned/digested files – directly from the browser.

==When using Import Maps, no separate build process is required==, just start your server with `bin/rails server` and you are good to go.

==Import maps are a browser feature that allows you to control the behaviour of JavaScript imports==. They provide a way to map import specifiers (like module names) to actual URLs, enabling you to:

**Simplify Module Paths**: You can shorten or simplify paths to JavaScript modules. Instead of using long or complex URLs in your import statements, you can define a short, easy-to-remember name.

By mapping a module to a specific URL, you can easily switch versions of a module or update the path for cache busting without changing the import statement in every file where the module is used.

Polyfill or Shim Legacy Modules: Import maps allow you to redirect imports to different files. This is useful for loading polyfills or alternative implementations based on certain conditions (like browser support).

Control Dependency Loading: They give you more control over how and from where your JavaScript modules are loaded, which can be critical for performance optimization and security.

Import maps facilitate the use of native JavaScript modules (ES modules) directly in browsers without needing a build step or a module bundler.

Instead of hardcoding CDN URLs in your import statements, you can define them in an import map, making it easier to switch CDNs or move to local hosting.

Import maps are part of the larger effort to make JavaScript modules more usable and efficient in browsers.

They reduce the complexity and improve the manageability of module imports, especially in larger web applications.

==Import Maps are the default in Rails 7+.==

Import maps are a feature provided by browsers, namely, the capability to interpret and apply import maps is built into the browser. This means the browser understands the import map syntax and how to process it when loading JavaScript modules. But they are configured and used by app developers. Here's how it works:

App developers create and include import maps in their web applications.

==An import map is essentially a JSON object that defines mappings between import specifiers (like module names or paths) and the actual URLs where these modules can be found.==

Usage in Applications: When a web application using import maps is loaded in a browser that supports this feature, the browser reads the import map and uses it to resolve module paths according to the mappings defined by the developer. This process is transparent to the end user.

Flexibility and Control for Developers: Import maps give developers control over how modules are loaded and from where, without modifying the source code of the modules themselves. They can define shortcuts, fallbacks, or specific paths for their modules, which is particularly useful for large-scale applications or when integrating third-party modules.

In summary, while the ability to process import maps is a feature provided by the browser, the actual use and configuration of import maps are in the hands of the app developers. They create the import maps to manage how JavaScript modules are imported and used in their applications.
