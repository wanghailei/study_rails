# Import Maps

==Import maps are a browser feature that allows you to control the behaviour of JavaScript imports==. They provide a way to map import specifiers (like module names) to actual URLs, enabling you to:

**Simplify Module Paths**: You can shorten or simplify paths to JavaScript modules. Instead of using long or complex URLs in your import statements, you can define a short, easy-to-remember name. By mapping a module to a specific URL, you can easily switch versions of a module or update the path for cache busting without changing the import statement in every file where the module is used.

**Polyfill or Shim Legacy Modules**: Import maps allow you to redirect imports to different files. This is useful for loading polyfills or alternative implementations based on certain conditions (like browser support).

**Control Dependency Loading**: They give you more control over how and from where your JavaScript modules are loaded, which can be critical for performance optimization and security. Import maps facilitate the use of native JavaScript modules (ES modules) directly in browsers without needing a build step or a module bundler. Instead of hardcoding CDN URLs in your import statements, you can define them in an import map, making it easier to switch CDNs or move to local hosting.

**Flexibility and Control for Developers**: Import maps give developers control over how modules are loaded and from where, without modifying the source code of the modules themselves. They can define shortcuts, fallbacks, or specific paths for their modules, which is particularly useful for large-scale applications or when integrating third-party modules.

Import maps are part of the larger effort to make JavaScript modules more usable and efficient in browsers. They reduce the complexity and improve the manageability of module imports, especially in larger web applications.

App developers create and include import maps in their web applications. When a web application using import maps is loaded in a browser that supports this feature, the browser reads the import map and uses it to resolve module paths according to the mappings defined by the developer. This process is transparent to the end user.

==An import map is essentially a JSON object that defines mappings between import specifiers (like module names or paths) and the actual URLs where these modules can be found.==

## Importmap in Rails

Import Maps manage your application's JavaScript dependencies. ==Import Maps make using most npm packages without the need for transpiling or bundling,== by importing JavaScript modules using logical names that map to versioned/digested files – directly from the browser. ==When using Import Maps, no separate build process is required==, just start your server with `bin/rails server` and you are good to go.

Starting from Rails 7, Importmap has become the default mechanism for handling JavaScript loading. It can fully utilize HTTP/2’s parallel download and caching mechanisms, avoiding the need to download all code every time a change is made by bundling everything into one large package.

With `bin/importmap pin something`, Importmap will download the `something.js` file from the CDN and place it in the vendor/javascript directory, automatically adding the configuration to `config/importmap.rb`. 

### Loading Problems

However, some js libraries assume developers will use bundling tools and do not package the source code into a complete bundle but instead split it into many files. In this case, using `importmap pin` will encounter problems.

有人用這樣的解決方案：first use jsbundling to bundle the dependencies, and then use importmap to import them. 但這需要刪除 `app/views/layouts/application.html.erb` 中的`javascript_include_tag`，並配置 `config/application.rb` 文件。我不認為這是個乾淨簡單的解法。

我的思路是：% 20250403 在過去兩週，經過各種嘗試，我發現在 Rails 8 中使用 Carbon Web Components，以build的方式並不自然。基於Rails框架，用最簡單的方式是最好的，哪怕是CDN。我找到的方法是用CDN把一個個的.min.js文件下載到本地，放在vendor文件夾中，用importmap引用，剩下的交給Propshaft。%

jsbundling、cssbundling、esbuild、SCSS，我都没有搞明白，而且也很难搞明白，在新一代浏览器和 Rails 8 的时代，没有必要再build了，所以DHH推出了“No Build”的口号。无论是CSS还是SVG，我想都有可能使用.js文件承载。那就在 Rails 8 的框架下，去处理从外部引入的代码，例如 web components。

## WHL's Solution

在自己的項目文件夾之外，把東西預先準備好，然後再放進去；在Rails的 `config/importmap.rb` 文件中配置，或用 `bin/rails importmap:pin` 命令生成配置。以下以 IBM Carbon Design System 舉例說明。

### 从哪里获取Carbon的最新版文件？

對於沒有 bundle 過的第三方module，某一個component的.js文件是分拆成多個的，不是只有一個，例如 button.min.js中只是一組簡單的調用命令。我需要的至少是一個完整的carbon_button.min.js，這裡面包含了所有的內容，甚至包括了字體的設置和SVG icons文件。

原始文件獲取路徑有：

1. GitHub repo 的最新版，這個可以下載到所有文件。但這些文件是開發用的，並非producton用的，所以文件分散嚴重。不好用。
2. NPM 或 Yarn，下載的是node_modules文件夾，其中/dist文件夾中有所有production狀態的文件。
3. 官方CDN（e.g., IBM's s18c.com）。下載的只是第一個文件，很簡單，其他文件沒有。

### How to bundle, the NPM way.

esbuid!





### How to bundle, the JSMP API way.



## 處理好的.js應該放在哪裡？`app/javascript`/ and `vendor/` ？

It seems likely that in a Rails application, the **vendor** directory is for third-party code and libraries, while **app/javascript** is for your application's JavaScript code, managed by tools like Webpacker or import maps. The evidence leans toward ==using the root directory for managing dependencies, with "vendor" for external assets and "app/javascript" for application-specific JavaScript.==

### app/javascript



**Purpose:**

**Primary place for your app’s JavaScript code.** Meant for **your own JS code**, Stimulus controllers, or code closely tied to your app’s functionality. But you won’t use `app/javascript` to manage external libraries like Carbon unless you are bundling.

Rails uses this directory especially when you set up **Webpacker** (Rails 6) or **jsbundling-rails** (Rails 7+).	

**app/javascript Directory**: This is for your application's own JavaScript code in modern Rails (6 and later). It's managed by tools like Webpacker or import maps, where you write custom scripts, like for Carbon Web Components integration. 

Purpose if "app/javascript"**: this directory is where you place your application’s own JavaScript code, managed by a JavaScript bundler like Webpacker or import maps. It’s for custom scripts, components, or integrations, such as those for BOS.

**Usage in Rails**: You write your JavaScript files here, typically in subdirectories like `app/javascript/packs` for Webpacker or directly in `app/javascript` for import maps, and it’s processed for serving in your application, aligning with the user’s integration of Carbon Web Components.

**Example structure:**

```
app/javascript/
├── controllers/   # Stimulus controllers
│   └── hello_controller.js
├── packs/         # (for Webpacker, rarely used now)
├── my_custom_logic.js
└── application.js # Main entry point
```

### vendor/ (Specifically vendor/assets/)

**Purpose:** 

For third-party assets (JS, CSS, fonts, images) NOT managed by Ruby gems or package managers. Basically, **external libraries’ raw files** that you want to serve yourself. You control their updates, and not auto-managed like gems or npm.

The "vendor" directory is used ==to store third-party code or libraries that are not part of the Rails framework itself.== This includes:

- Unpacked gems.
- Other external dependencies, such as third-party libraries or scripts, where it’s used for assets from gems.
- Assets like JavaScript, CSS, or images from these libraries, placed in `vendor/assets`, which are then processed by the asset pipeline.

**When to use:**

- Third-party **JavaScript libraries**, **CSS frameworks**, or **fonts** you want to bundle manually.
- Libraries that **aren’t available as gems** or Rails engines.
- Assets you want to **version-control directly** in your project without depending on npm or RubyGems.

**Usage in Rails**: It ensures that your application can use specific versions of dependencies independently of system-wide installations, which is useful for consistency across development and production environments. For example, you might place a third-party JavaScript library in `vendor/assets/javascripts` to make it available for your application.



**Example structure:**

```
vendor/assets/
├── javascripts/
│   └── carbon/
│       └── carbon-web-components.js  # Manually copied JS bundle
├── stylesheets/
│   └── carbon/
│       └── carbon-tokens.css
└── fonts/
    └── IBM-Plex/
```



**Why it’s useful for you:**

Since you’re:

•	**Avoiding npm in production**

•	Using **importmap** for JS

•	Managing all Carbon assets **locally and manually**



**→ vendor/assets/ is PERFECT** for storing:

•	Carbon Web Components compiled JavaScript

•	Carbon CSS/SCSS (if not using it via gem)

•	Carbon icons

•	Fonts like IBM Plex



#### Table: Directory Purposes and Usage

| Directory          | Purpose                                                      | Usage in Rails                                               |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **vendor**         | Stores third-party code, libraries, or assets (e.g., unpacked gems). | Use for external dependencies, including their assets (e.g., `vendor/assets`). |
| **app/javascript** | Stores your application's JavaScript code (modern Rails 6+). | Use for custom JavaScript files, managed by Webpacker or import maps. |

#### Considerations and Limitations
- **Vendor Assets**: Ensure assets in `vendor/assets` are included in the asset pipeline, possibly requiring configuration in `config/initializers/assets.rb.
- **JavaScript Management**: For BOS, ensure "app/javascript" is set up for your chosen bundler, and manage dependencies like `@carbon/web-components` via `npm` in the root directory.





Excellent question — and it’s super relevant since you’re using **Rails 8**, **Propshaft**, and **importmap**, and managing **Carbon Web Components locally without npm**.



Let’s go over:



------



**📁 Where to Put 3rd Party JS Like Carbon Web Components?**



**✅ app/assets/javascripts**

​	•	This is **legacy sprockets-style**.

​	•	Used in Rails <7 with //= require directives.

​	•	**Not used** or recommended in Rails 7+ (with importmap or esbuild).

​	•	Not automatically added to the asset load paths in Rails 8 with Propshaft.



💡 **Don’t use this in modern Rails (8 with Propshaft).**



------



**✅ app/javascript**

​	•	Default folder used **only if you’re using JS bundlers** (esbuild, webpacker, Vite).

​	•	Irrelevant if you use **importmap** (like you do).

​	•	Rails 8 will **not auto-serve** files here unless you explicitly move them into a proper load path.



💡 **Ignore this if you don’t use bundlers.**



------



**✅ vendor/javascript**

​	•	✅ This **is the correct place** for non-Rails-managed 3rd party libraries (like Carbon’s compiled .js files).

​	•	This folder **is on the asset load path** with importmap in Rails 8.

​	•	Anything here can be referenced in config/importmap.rb.



💡 **Use this for your Carbon Web Components JS bundle.**



------



**✅ app/assets (general)**

​	•	Safe place for fonts, images, or SCSS files.

​	•	Not for JS if you use importmap — JS here won’t be picked up unless you explicitly add paths.



------



**🧩 Why You See JS Loading Errors in ERB**



If you’re getting 404s or Uncaught ReferenceError, it’s often one of these:



**1. Wrong Path in <script>**

​	•	You might be pointing to /javascript/carbon.js, but it should be /assets/carbon.js (or whatever path Propshaft maps).



Check via browser:

```
http://localhost:3000/assets/carbon.js
```





------



**2. Not Added to importmap**



For Carbon Web Components, you must register it in config/importmap.rb:

```
pin "carbon-components-wc", to: "carbon-components-wc.js"
```

(assuming the file is in vendor/javascript/carbon-components-wc.js)



------



**3. Loading JS Before DOM Is Ready**



In ERB layouts, if you load JS too early (especially custom elements), you might get errors like:



> Uncaught TypeError: customElements.define is not a function



✅ Make sure to load your <script> tag:

```
<%= javascript_importmap_tags %>
```

at the **end of <body>**, or wrap JS inside a DOMContentLoaded event listener.



------



**🧵 Summary Recommendation for You (PGMeet)**



You’re using:

​	•	Rails 8 with Propshaft

​	•	Importmap

​	•	No bundlers

​	•	Carbon Web Components locally



🧷 **Do this:**

​	1.	Put carbon-components-wc.js in:

```
vendor/javascript/carbon-components-wc.js
```



​	2.	Add this to config/importmap.rb:

```
pin "carbon-components-wc", to: "carbon-components-wc.js"
```



​	3.	In your layout (usually application.html.erb), add:

```
<%= javascript_importmap_tags %>
```



​	4.	In your app JS (like application.js), import it:

```
import "carbon-components-wc"
```





✅ Then you can start using <cds-button> etc. in your ERB views.



------



Let me know if you’d like a minimal working example for this setup — I can give you one with all folders and config in place.
