# Install and Upgrade



==**Upgrade step by step, and use version control!** % Use Git to track changes after each step. %==

**Setup an up-to-date environment for developing webpps with Ruby on Rails.** First, the tools' value and relationship need to be understood.

## **工具間的關係**

- RubyGems belongs to Ruby. 
- RubyGems manages many Gems.
- Bundler is a Gem. 與某一個app有關，用於管理一個app用到的庫。
- Rails is a Gem.

#### Bundler vs. RubyGems

RubyGems 是**系統層面**的 gem 管理工具，安裝的 gem 是可OS全域使用的；而 Bundle 只是作用與某一個具體的app 範圍。

Bundler 是基於 RubyGems 的工具，並不是獨立的用來取代 RubyGems 的新系統，而應被看作是對 RubyGems 的延伸。在一個 app 範圍內，Bundler 比 RubyGems 方便。在一個 app 開發的範圍，只使用 Bundler 這一個工具。

Bundler 解決的是「依賴管理」的問題。Bundler 讓庫兼容性問題在安裝時就暴露出來；並在每個項目內鎖定所有依賴庫的版本。確保某個 Rails app 只使用特定版本的 gems，並且避免不同 app 之間的 gem 衝突，也==免受系統上其他地方安裝 gem 的影響==

Bundler 透過 Gemfile 和 Gemfile.lock 來管理一個 app 的 gem 版本。

`bundle exec` 確保當前 app 使用 Gemfile.lock 內指定的 gem 版本，而不是全局 RubyGems 內的版本。

Bundler 是工具名，而 bundle 是實際使用的命令；就像 Homebrew 是工具名，对应的命令是 brew；RubyGems是工具名，gem 是實際使用的命令。

#### **RubyGems vs. require**

require 是 Ruby 內建的函數，用來加載標準庫或已安裝的 gems。 RubyGems 是 Ruby 的庫管系統，負責安裝、管理和查找 gems，而 require 只是載入程式碼的機制。 RubyGems 讓開發者可以輕鬆安裝和管理 gem，而不需要手動下載和管理 Ruby 庫路徑，所以的確比 require 方便。RubyGems 無法取代程序中的 require 函數，只是讓它的使用更方便和準確。

## Upgrade the Ruby environment of macOS with brew

**Update Homebrew** to make sure it has the latest formulae for Ruby.&#x20;

```bash
brew update
brew upgrade
```

Upgrade Ruby to the latest version available in Homebrew.

```bash
brew upgrade ruby
brew upgrade ruby@3.3.0 // or to a specific version.
```

To lean up any outdated or unused packages and dependencies that are no longer needed.

```zsh
brew cleanup
```

To verify if your Homebrew installation is working correctly and if there are any issues

```zsh
brew doctor 
```

To list and remove formulae that were installed as dependencies but are no longer needed by any installed formula. 

```zsh
brew autoremove
```

### After Installing or Upgrading a new Ruby with brew

After brew install ruby, the .zshrc needs to be modified. 根據我的經驗教訓，.zshrc應該是這樣的：

```bash
export PATH="/opt/homebrew/bin:$PATH" 
export PATH="/opt/homebrew/sbin:$PATH" 
export PATH="/opt/homebrew/opt/ruby/bin:$PATH" 
export PATH="/opt/homebrew/lib/ruby/gems/3.3.0/bin:$PATH" 
export PATH=`gem environment gemdir`/bin:$PATH 
export LDFLAGS="-L/opt/homebrew/opt/ruby/lib" 
export CPPFLAGS="-I/opt/homebrew/opt/ruby/include" 
export PKG_CONFIG_PATH="/opt/homebrew/opt/ruby/lib/pkgconfig" 
export PATH="$PATH:/usr/local/bin" 
export PATH="$PATH:/Applications/RubyMine.app/Contents/MacOS" 
```

If you need to have ruby first in your PATH, run:

```bash
echo 'export PATH="/opt/homebrew/opt/ruby/bin:$PATH"' >> ~/.zshrc
```

For compilers to find ruby you may need to set:

```bash
export LDFLAGS="-L/opt/homebrew/opt/ruby/lib"
export CPPFLAGS="-I/opt/homebrew/opt/ruby/include"  
```

For pkg-config to find ruby you may need to set:

```bash
export PKG_CONFIG_PATH="/opt/homebrew/opt/ruby/lib/pkgconfig"
```

我認為一般用 `brew` 升級Ruby就很好。但是我發現在 Ruby 3.3 發布一個星期之後，`brew` 都沒有更新到這個最新版。所以我不得不起用`rbenv`。等到 brew 的最新版 Ruby 發布了，再重新安裝並切換回 brew 的版本。因為對我來說，不同於其他的工具——可以用brew安裝，Ruby的最新版本對我來說很重要，所以它可以被單獨管理。

## Install or Upgrade Ruby with `rbenv`

在使用rbenv更新 Ruby 版本前，要先用 brew 更新 rbenv，否则最新版的 ruby 可能不会列出来。

### Upgrade Ruby with `rbenv`

```bash
# list latest stable versions:
rbenv install -l

# list all local versions:
rbenv install -L

# install a Ruby version:
rbenv install 3.3.0
rbenv install 3.4.0-preview2

# make 3.3.0 the defaul version on macOS
rbenv global 3.3.0
```

==After `rbenv install` and `rbenv global`, the ~/.zshrc need to be modified for `ruby` to take effect:==

```bash
if command -v rbenv > /dev/null; then
	eval "$(rbenv init -)"
fi
```

It can be down in CLI with the following code:

```zsh
echo 'if command -v rbenv > /dev/null; then eval "$(rbenv init -)"; fi' >> ~/.zshrc
source ~/.zshrc
```

To remove a Ruby version:

````bash
rbenv uninstall 3.4.0
````



## RubyGems

RubyGems is a package management framework for Ruby. Commonly gems are used to distribute **reusable functionality** that is shared with other Rubyists for use in their applications and libraries. RubyGems的存在主要是為了分享。

The RubyGems software allows you to easily download, install, and use ruby software packages on your system. The software package is called ==a “gem” which contains a packaged Ruby application or library.==

#### Update RubyGems itself to the latest version on macOS

```bash
gem update --system
````

`gem update --system` upgrades RubyGems itself whereas `gem update` upgrades all gems including both user-installed ones and default ones but not RubyGems. The `--system` flag cuts off execution of the regular `update` command.

#### Upgrade all installed gems to the latest version on macOS

```bash
gem update
```

#### Upgrade a particular gem installed on macOS

```bash
gem update rails // updates a  gem
```

#### Error: can't find gem bundler (= 2.4.13) with executable bundle

```bash
gem i bundler -v 2.4.13
````

#### To install a gem (Ruby package), run:

```bash
gem install <gemname>
<<<<<<< HEAD

// This will install the latest pre-release version of Rails, it could be a beta.
gem install rails --pre 
=======
>>>>>>> ecde517b3863b9b891b64c90b253ee32b52728bc
```

#### List installed gems

```bash
gem list
```

#### To check if any installed gems are outdated:

```bash
gem outdated
```

#### Remove old gem versions

RubyGems keeps old versions of gems, so feel free to do some cleaning after updating:

```bash
gem cleanup
```

### RubyGems Versioning and Directories

- **Ruby Version**: 3.3.2 indicates the specific version of Ruby you have installed.
- **Installation Directory**: `/opt/homebrew/lib/ruby/gems/3.3.0` is used for all 3.3.x versions, not just 3.3.2.

The discrepancy you're seeing between the Ruby version (3.3.2) and the installation directory (3.3.0) is a result of how RubyGems manages its versioning and directories. This setup is standard for RubyGems and helps in managing gems more efficiently across minor version updates.

1. **Minor Version Directories**:
   - RubyGems uses the minor version number for its directories. This means that even if your Ruby version is 3.3.2, the directory for gems is based on the minor version `3.3.0`. ==This approach helps in maintaining compatibility and reducing the need for multiple directories for patch-level changes.==

2. **Installation Directory**:
   - ==The installation directory `/opt/homebrew/lib/ruby/gems/3.3.0` is used for any Ruby 3.3.x installation.== This includes all patch versions such as 3.3.1, 3.3.2, etc. It prevents redundancy and ensures that gems installed for one patch version can be used by another patch version within the same minor version.

3. **Consistency Across Patch Levels**:
   - By using the minor version directory (3.3.0) for all 3.3.x versions, RubyGems ensures that gems do not need to be reinstalled for each patch update. This approach is efficient and saves disk space.

## Bundler

Bundler: a gem to bundle gems. 

==It manages the gems that the application depends on.== Given a list of gems, it can automatically download and install those gems, as well as any other gems needed by the gems that are listed. 

Before installing gems, it checks the versions of every gem ==to make sure that they are compatible, and can all be loaded at the same time==. After the gems have been installed, Bundler can help you update some or all of them when new versions become available. 

Finally, it records the exact versions that have been installed, so that others can install the exact same gems.==% 應該說，Bundler 是團隊開發協作用的。20230619 %== Bundler makes sure a Ruby app run the same code on every development, staging, and production machine. 

Bundler is not a technology specific to Rails. ==Bundler is a dependency management tool for Ruby.== ==Bundler is a gem to bundle gems.== It is an exit from dependency hell.

### How to install gems in your project?

要在自己的程序里使用一顆 gem（Ruby語言的程序包），總共兩步：
第一步，往項目的 Gemfile 里添加 gem "some-library"。這是“聲明”或提出要求。
第二步：執行 bundle install 命令來安裝。

`bundle install` installs the dependencies specified in your Gemfile.

==The Bundler update steps should be performed **within the Rails app's directory**, not in the home (`~`) directory.== This is important because Bundler ne·eds to access the `Gemfile` and `Gemfile.lock` specific to your Rails application. 

### How Bundler works

First, given the gems listed in the Gemfile of a Ruby app, Bundler checks the versions of every gem to make sure that they are compatible, and can all be loaded at the same time.

Then, Bundler downloads and installs those gems, as well as any other gems needed by the gems that are listed.  And ==it creates a file called Gemfile.lock with the list of the gems installed along with their respective versions.==

It will require them while booting.

After the gems have been installed, Bundler can help you update them when new versions become available. Bundler is also an easy way to create new gems.&#x20;





## Install Rails

==🚫 Avoid sudo gem install rails. Always install Rails in **user space** when using rbenv.== You **should NOT** use `sudo gem install rails` unless you are installing Rails system-wide and fully understand the implications. Instead, install Rails without `sudo`, using Bundler and rbenv.

### Why You Should Avoid sudo

**Permission Issues:** Installing gems with sudo means they will be placed in system directories (e.g., /usr/local/lib/ruby/gems). This can lead to permission conflicts when managing gems later.

**Conflicts with rbenv:** ==If you are using rbenv, Ruby gems should be installed in your **user space** (~/.rbenv/versions/<ruby-version>/lib/ruby/gems), not system-wide.==

**Security Risks:** Running sudo gives gems full system access, which is risky if a malicious gem is installed.

### **The Correct Way to Install Rails**

Using rbenv, follow these steps:

1. **Ensure rbenv and ruby-build are updated**

```bash
rbenv install --list  # Check available Ruby versions
rbenv install 3.4.2    # Ensure you're using the latest version
rbenv global 3.4.2     # Set it as the default
```

2. **Install Bundler (if not already installed)**

```bash
gem install bundler
```

3. **Install Rails (without sudo)**

```bash
gem install rails
```

4. **Rehash rbenv (if necessary)**

```bash
rbenv rehash # Regenerate shims for all known Ruby executables
```

運行這個命令是必要的，否則系統識別不了 `rails` 。

5. **Check Rails Installation**

```bash
rails -v
```

### What If You Already Used sudo?

If you mistakenly installed Rails with sudo, it might cause permission problems. To fix it:

1. Uninstall Rails and reset gem permissions:

```bash
sudo gem uninstall rails
sudo rm -rf /usr/local/lib/ruby/gems/*
sudo chown -R $(whoami) ~/.gem
```

2. Then, reinstall Rails **without** sudo using the steps above.

### Where to install Rails

If you want to create multiple Rails projects using the same Rails version, installing Rails globally is fine. If you work on multiple projects requiring different Rails versions, a global install might cause version conflicts.

There is **no functional difference** in how `gem install rails` behaves when run in ~ (your home directory) versus inside a Rails project folder. 

#### `gem install rails` in the Home Directory (~)

If you run `gem install rails` in your home directory (or any non-Rails project directory), it installs the **latest** version of Rails **globally for your current Ruby version**, puts Rails in your GEM_HOME (likely `~/.rbenv/versions/3.4.2/lib/ruby/gems` if using rbenv), and makes `rails` available **system-wide** for the current Ruby version. If you later start a new Rails project, it will use this globally installed Rails version.

#### `gem install rails` in a Rails Project Folder

If you are inside a Rails project folder and run `gem install rails`,it **still installs Rails globally**, not just for that project. It does **not** modify Gemfile or Gemfile.lock of your project, and it has **no direct effect** on the project unless the project is already configured to use that version.

#### Correct way to install Rails

**For global installs** (e.g., when setting up a new development machine), `gem install rails` is fine.

**For individual projects**, specify Rails in Gemfile and use `bundle install` to avoid conflicts.

## Upgrade a Rails app

首先，最好先升級一下這個 Rails app 用的打包工具 Bundler。當然，這只是個幹活工具，不是是app本身的一部分。

### Upgrade the Bundler version of an app

It's important to ensure that the newer Bundler version is compatible with your Rails app and other gem dependencies. ==就用最新版的 Bundler！==

1. If the `Gemfile` explicitly specifies a Bundler version (like `gem 'bundler', '~> 2.4.13'`), update this line to allow the newer version. 我看，好像一般也不會在這裡寫吧？

2. Install the latest stable release of Bundler, or install a specific version: 

	```bash
	gem install bundler
	gem install bundler -v 2.5.5
	```
	==% 用 brew install ruby 後，一定要 gem install bundler 安裝 bundler。否則就會報錯LoadError。20240119 %==

3. Go to your Rails application's root directory to update Gemfile.lock.

	```bash
	bundle update --bundler
	```

	This command updates the `BUNDLED WITH` section in your app's `Gemfile.lock` to the new Bundler version. 此命令告訴這個Rails app用哪一個版本的Bundler打包。

4. After installing the required Bundler version, you can then run it specifically for your project. This command tells Bundler to use version 2.5.5 to manage your gems as specified in your `Gemfile`.

	```bash
	bundle _2.5.5_ install
	```



Use the Terminal commands only **insde a Rails app's directory**. Otherwise, the commands work on the Rails on your system instead of the Rails used by a certain app.

### Verify the Rails version of an app

Inside a Rails app's directory: 

1\. Via the Gemfile.lock: ==The Gemfile.lock in a Rails application specifies the exact versions of each gem that the application is using.== Look for the line specifying the rails gem version in this file.

2\. Via the command line: In the terminal, navigate to the directory of your Rails app and use the command `rails -v` . This will display the current version of Rails the application is using.

3\. Via the Rails console: Open the Rails console with the command `rails console` or `rails c`, then print the Rails version with `Rails.version`.

### Upgrade the Rails version of an app

**Step 1**: to update the Rails gem and its dependencies, by modify the Rails gem version in your Gemfile, then run:

```bash
bundle update rails
```

Bundler will update the Rails gem to the latest version specified in the Gemfile, along with all its dependencies. ==But it does not make any changes to the configuration files, initializers, or other Rails-specific files in the app.== 

做了這一步，只是把app需要的dependency的最新款，都搬運過來了而已，但是app還並沒有配置「啟用」它們。

**Step 2**: to make sure the app's configuration and boilerplate code are up-to-date with the new Rails version, by running:

```bash
bin/rails app:update
```

The command `bin/rails app:update` is used to update the Rails application with any new changes or updates that have been made in the Rails framework since the application was first created or last updated. 

==When you run this command, Rails will create a new set of configuration files, initializers, and other necessary files for the updated version of Rails. It will also create a new `config/routes.rb` file and a new `config/application.rb` file.== (It won't just overwrite your existing files. Instead, it will create new files with a `.new` extension (for example, `config/routes.rb.new`). This allows you to see what the new default files look like so you can decide whether to keep your old files, use the new ones, or merge the changes from the new files into your old ones.)

==It's recommended to always backup your application before running `bin/rails app:update`==, thoroughly review and test the changes, and have a rollback plan in case something goes wrong.

### Gemfile

A Gemfile describes the gem dependencies required to execute associated Ruby code. ==The Gemfile is indeed related to Bundler and not directly to RubyGems itself.== Gem是跟具體某一個app有關係的，是管理一個app的dependency的。

Place the Gemfile in the root of the directory containing the associated code.

A Gemfile is evaluated as Ruby code. The `gem` method is used to declare a gem dependency for your application. 

```ruby
gem "rails", "~> 8.1.0"
```

This line says that the application depends on the "rails" gem, and requires a version that's compatible with version 8.1.0.

Remember to run `bundle install` after you update your Gemfile to install the required versions of each gem.

#### Gem Version Specifier

==The "`~> 7.1.2`" is a version specifier, also known as a pessimistic version constraint. This means that Bundler will update the Rails gem to the latest minor or patch version==, but not to a new major version (which could potentially include breaking changes). For example, with this constraint, Bundler could update Rails to version 8.1.0 or 7.2.0, but not to 8.0.0.

```ruby
gem "pg"
```

==The `gem` command without a version specifier tells Bundler to install the most up-to-date version of the gem that's compatible with your project.== Bundler will try to get the latest version of the gem that is compatible with all other gems in the Gemfile, taking into account all the requirements specified in your application's codebase.

`gem 'pg', '>= 1.1.0'` : install any version greater than or equal to 1.1.0
`gem 'pg', '1.1.0'`    : install exactly version 1.1.0
`gem 'pg', '~> 1.1.0'` : install the latest version of the 1.x series 
			(greater than or equal to version 1.1 but less than version 2.0) .

### Upgrade the gems versions of an app&#x20;

1. Start by updating your `Gemfile`. For each gem, specify the latest version compatible with Rails 8. You might need to check each gem's documentation or repository for compatibility information.
   ==Update gems incrementally. Avoid updating all gems at once==, as this can introduce multiple breaking changes that are hard to debug. ==Update one or a few gems at a time==, then test your application to ensure it still works as expected.
2.  Once you've updated your `Gemfile`, ==to update your `Gemfile.lock` with the new versions.==

    ```bash
    bundle update
    ```


3.  To ensure all gems are installed correctly with the new Bundler version. This step will install the necessary gems based on your updated `Gemfile.lock`.

    ```bash
    bundle install
    ```



`bundle exec` 確保當前 app 使用 Gemfile.lock 內指定的 gem 版本，而不是全局 RubyGems 內的版本。
