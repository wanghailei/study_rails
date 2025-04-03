# Assets Pipeline



==% Assets Pipeline 讓網頁的各種素材加載更快，並確保加在最新的素材，且適應不同瀏覽器。這可以被看作是一個自動化輔助工具，跟開發其實沒什麼關係，提前設置好了就行。20231129 %==

## What does an asset pipeline do?

An asset pipeline:

* improves page loading speed, by minifying and compressing assets.
* reduces the number of browser HTTP requests, by combining multiple files into one.
* ensures browsers load the newest version of the assets, by appending a digest to filenames.
* ensuring cross-browser compatibility, by adding vendor prefixes to CSS.

In development mode, an asset pipeline

* ==compiles assets from languages like SCSS, Sass, or CoffeeScript into CSS and JavaScript==, which browsers can understand.
* automatically compile and refresh assets as they are updated, which speeds up the development process.

Propshaft is one of the modern asset pipeline.

## Useful tips in development:

The `application.js` and `application.css` manifest files are the only JavaScript/Stylesheets included during the asset pipeline precompile step. ==To include additional assets, specify them using the `config.assets.precompile` configuration setting in `config/initializers/assets.rb`==:

```ruby
config.assets.precompile += %w( admin.js admin.css )
```

### Where to put my own CSS code?

`app/assets/stylesheets/application.css`

Always remember to restart your development server and potentially recompile your assets after making changes to CSS or JavaScript files.

% Just use `bin/dev`. %

#### Why do I get "*application.css not in asset pipeline*" in production?

A common issue is that your repository does not contain the output directory used by the build commands. You must have `app/assets/builds` available. Add the directory with a `.gitkeep` file, and you'll ensure it's available in production.

#### Why isn't Rails using my updated css files?

Watch out - if you precompile your files locally, those will be served over the dynamically created ones you expect. The solution:

`rails assets:clobber`

## CSS Bundling for Rails

Use Tailwind CSS, Bootstrap, Bulma, PostCSS, or Dart Sass to bundle and process your CSS, then deliver it via the asset pipeline in Rails.

This gem provides installers to get you going with the bundler of your choice in a new Rails application, and a convention to use app/assets/builds to hold your bundled output as artifacts that are not checked into source control (the installer adds this directory to .gitignore by default).

==Use `./bin/dev`, which will start both the Rails server, the CSS build watcher, along with a JS build watcher, if you're also using jsbundling-rails.==

Whenever the bundler detects changes to any of the stylesheet files in your project, ==it'll bundle app/assets/stylesheets/application.\[bundler].css into app/assets/builds/application.css.==

When you deploy your application to production, the `css:build` task attaches to the `assets:precompile` task to ensure that all your package dependencies from `package.json` have been installed via `yarn`, and then runs `yarn build:css` to process your stylesheet entrypoint, as it would in development. This output is then picked up by the asset pipeline, digested, and copied into `public/assets`, as any other asset pipeline file.

This also happens in testing where the bundler attaches to the `test:prepare` task to ensure the stylesheets have been bundled before testing commences.

If your setup uses `jsbundling-rails` (ie, esbuild + tailwind), you will also need to run `javascript:build`.

You can configure your bundler options in the `build:css` script in `package.json` or via the installer-generated tailwind.config.js for Tailwind or postcss.config.js for PostCSS.



### Tailwind CSS

我認為，可以用tailwind-rails。但只是為了把其特殊的一些class 提取出來。

`Error:`\
`ActionView::Template::Error (The asset "tailwind.css" is not present in the asset pipeline.)`

Solution:

`rails assets:clean assets:precompile`

% Remember to restart the Rails server to see effect. 20230613 %

算了吧，不使用這些勞什子了————各領風騷三五年，而且使用這一套DSL，挺無聊的。就用CSS3！
