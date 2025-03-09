# `rails new`

The `rails new` command creates a new Rails application with a default directory structure and configuration at the path you specify.

You can specify extra command-line arguments to be used every time 'rails new' runs in the .railsrc configuration file in your home directory, or in $XDG\_CONFIG\_HOME/rails/railsrc if XDG\_CONFIG\_HOME is set.

==But tailoring rails new command should be as little as possible, since it can be hard to add back parts of Rails you initially skip==, and for the most part, the parts of Rails you don’t use can sit there, inert, not bothering anyone.

```bash
rails new appname -a=propshaft
```

## ```rails new``` Options:

```bash
-d, --database=DATABASE # Preconfigure for selected database (sqlite3/mysql/postgresql)
-a, --asset-pipeline=ASSET_PIPELINE # Choose asset pipeline (sprockets, propshaft)
-j, --javascript=JAVASCRIPT # Choose JavaScript approach [importmap, webpack, esbuild, rollup]
-c, --css=CSS # Choose CSS processor [options: tailwind, bootstrap, bulma, postcss, sass]

--api # Preconfigure smaller stack for API only apps
--minimal # Preconfigure a minimal rails app

-s, --skip-keeps # Skip source control .keep files
-f, --force # Overwrite files that already exist
-p, --pretend # Run but do not make any changes
```



## Generate a new Rails app with --main

`--dev` uses your locally checked-out Rails repository to create the app. 

`--edge` will use the latest stable version from the Github repository, instead of from your locally installed Rails gem.

`--main` generate a new Rails app that runs on Rails main branch from GitHub. 
`rails new appname --main` will generate a `Gemfile` that has something like this inside:

```
gem "rails", github: "rails/rails", branch: "main"
```

## .railsrc

```bash
--asset-pipeline=propshaft
```

## Rails

$ sudo gem install rails

$ rails new my-app

### How to run a Rails deposit on a new Mac.

$ rails db:migrate

$ rails db:seed

$ rake assets:precompile



### Bundler Errors when rails new appname

```bash
Bundler::DirectoryRemovalError: Could not delete previous installation of 
`/opt/homebrew/lib/ruby/gems/3.3.0/gems/websocket-driver-0.7.6`.
An error occurred while installing websocket-driver (0.7.6), and Bundler cannot continue.
```

我的經驗是，用這個來嘗試解決：

`gem pristine websocket-driver --version 0.7.6`

如果提示沒有寫權限，則加 sudo。
