# Using SVG icons in ERB

## Prepare SVG icons.

Use SVG icons instead of the old-fashioned icon fonts. 

Instead of using inline SVG icons, which mess the HTML or ERB, keeping all SVG icons in one `icons.svg` file, which means minimal network requests(single `icons.svg`).  

### Always preload the icons.svg file

Preload the SVG File for Faster Rendering. Since the SVG file is static, preload it in your `<head>` for instant rendering, to ensure zero flickering on first load, and makes icons render instantly in Turbo-enabled pages.

`app/views/layouts/application.html.erb`

```erb
<link rel="preload" as="image" href="<%= asset_path('icons.svg') %>" type="image/svg+xml">
```

## Use SVG ioncs in ERB

### Method A: With a Partial

Here’s the best approach that leverages Rails’ Asset Pipeline efficiently while keeping ERB clean.

```erb
<%= render "icon", name: "star" %>
<%= render "icon", name: "heart" %>
```

#### 1. Store All Icons in a Single SVG Sprite

Use the Asset Pipeline to serve one static `icons.svg` file. This file is precompiled and cached by Rails, and it contains all your icons in one place, `app/assets/images/icons.svg`

```xml
<svg xmlns="http://www.w3.org/2000/svg" style="display: none;">
    <symbol id="star" viewBox="0 0 24 24">
        <path d="M12 2L15 8H21L16 12L18 18L12 14L6 18L8 12L3 8H9L12 2Z"/>
    </symbol>
    <symbol id="heart" viewBox="0 0 24 24">
        <path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"/>
    </symbol>
</svg>
```
#### 2. Create a Partial to Render Icons

This partial will be used to generate the SVG reference dynamically.

`app/views/application/_icon.html.erb`

```erb
<svg class="icon" aria-hidden="true">
    <use xlink:href="<%= asset_path('icons.svg') %>#<%= name %>"></use>
</svg>
```
- Uses `asset_path("icons.svg")` to load the precompiled file.

#### 3. Define Icon Styles in CSS

`app/assets/stylesheets/icons.css`

```css
.icon {
    width: 24px;
    height: 24px;
    fill: currentColor;
}
```

#### 4. Make sure this file is included in your application layout:

`<%= stylesheet_link_tag "icons", media: "all" %>`

#### 5. Use It in ERB

Now you can call icons with minimal code.
```erb
<%= render "icon", name: "star" %>
<%= render "icon", name: "heart" %>
```
No repeated `class` attributes.  
Rails Asset Pipeline serves the SVG sprite efficiently.  
One `icons.svg` file means fewer network requests.

#### Final Thoughts: Best Balance for Rails + Hotwire

| Approach                     | Uses Asset Pipeline? | Short ERB? | Good Performance? |
| ---------------------------- | -------------------- | ---------- | ----------------- |
| Inline SVGs                  | No                   | Yes        | Yes (fast)        |
| Partial for Each Icon        | Yes                  | Yes        | Very Fast         |
| Single `icons.svg` + Partial | Yes                  | Yes        | Very Fast         |

Now you have the best of both worlds:
- Short ERB references (`<%= render "icon", name: "star" %>`).
- Rails Asset Pipeline optimizations.
- Minimal network requests (single `icons.svg`).
- Works well with Hotwire/Turbo for fast updates.



### Method B: With an Icon Helper

You want to keep using the Rails Asset Pipeline but keep ERB calls minimal, like:

```erb
<%= icon "star" %>
<%= icon "heart" %>
```

Here’s the best approach that leverages Rails’ Asset Pipeline efficiently while keeping ERB clean.

#### 1. Store All Icons in a Single SVG Sprite

It contains all your icons in one place. Use the Asset Pipeline to serve one static `icons.svg` file. This file is precompiled and cached by Rails. 

`app/assets/images/icons.svg`

```xml
<svg xmlns="http://www.w3.org/2000/svg" style="display: none;">
    <symbol id="star" viewBox="0 0 24 24">
        <path d="M12 2L15 8H21L16 12L18 18L12 14L6 18L8 12L3 8H9L12 2Z"/>
    </symbol>
    <symbol id="heart" viewBox="0 0 24 24">
        <path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"/>
    </symbol>
</svg>
```
#### 2. Create a Helper Method for Short ERB Usage

Since partials (`render "icon"`) are still too long, create a helper method. ==Uses `asset_path("icons.svg")` to load the precompiled file.== Defaults to `icon` CSS class so you don’t need to set a class each time. % 我寫了一個最簡潔的版本，就一種默認的 CSS classe ———— icon。%

`app/helpers/icon_helper.rb`

```ruby
module IconHelper
        def icon( name )
                content_tag :svg, class: "icon", "aria-hidden": true do
                        tag.use "xlink:href": asset_path( "icons.svg##{name}" )
                end
        end
end
```

#### 3. Use It in ERB

Now you can call icons with minimal code, very short and clean.
```erb
<%= icon "star" %>
<%= icon "heart" %>
```
No repeated `class` attributes.  
Rails Asset Pipeline serves the SVG sprite efficiently.  
One `icons.svg` file means fewer network requests.

#### Final Thoughts: Best Balance for Rails + Hotwire

| Approach                    | Uses Asset Pipeline? | Short ERB? | Good Performance? |
| --------------------------- | -------------------- | ---------- | ----------------- |
| Inline SVGs                 | No                   | Yes        | Yes (fast)        |
| Partial for Each Icon       | No                   | No         | Yes (fast)        |
| Single `icons.svg` + Helper | Yes                  | Yes        | Very Fast         |

Now you have the best of both worlds:
- Short ERB references (`<%= icon "star" %>`).
- Rails Asset Pipeline optimizations.
- 
- Works well with Hotwire/Turbo for fast updates.