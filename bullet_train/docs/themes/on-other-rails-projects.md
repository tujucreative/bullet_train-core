# Installing Bullet Train Themes on Other Rails Projects

Bullet Train themes can be installed on Vanilla Rails projects.

Our main theme, called `Light`, uses `erb` partials to give you native Rails views with Hotwire-powered components. It's built on `tailwindcss` (these instructions target Tailwind CSS v3, `tailwindcss` ^3.4), uses `postcss` to allow for local CSS overrides and uses `esbuild` for fast javascript compilation and to support javascript-side CSS imports.

In addition to providing a nice set of UI components, you'll get access to [`nice_partials`](https://github.com/bullet-train-co/nice_partials), Bullet Train's own lightweight answer for creating `erb` partials with ad-hoc named content areas, which we think is just the right amount of magic for making `erb`-based components.

Note: we have [special instructions for installing themes on Jumpstart Pro projects](on-jumpstart-pro-projects.md).

**Contents:**

1. Installation Instructions
2. Optional Configurations for switching colors, theme gems
3. Using Locales for fields on new models
4. Partials that require special instructions, exclusions
5. Modifying ejected partials

## 1. Installation Instructions

### Ensure your Rails Project uses `esbuild` and `tailwindcss` with `postcss`

You'll need to make sure your Rails project is set up to use `esbuild`, `tailwindcss` and `postcss`.

The Light theme requires Tailwind CSS v3 (`tailwindcss` ^3.4) installed from npm and run through `postcss`. The starter does not use the `tailwindcss-rails` gem (which `rails new --css tailwind` sets up); it installs `tailwindcss` from npm and builds the stylesheets with `postcss`.

The easiest way to see what your project should include is to look at the [Bullet Train starter repository](https://github.com/bullet-train-co/bullet_train). It uses `jsbundling-rails` with `esbuild` and `cssbundling-rails` with `postcss`, configured in its `package.json`, `esbuild.config.js` and `postcss.config.js`. The steps below copy those files into your project.

### Add the theme gem

These instructions assume you're installing the `Light` theme bundled with Bullet Train.

```
bundle add bullet_train-themes-light
```

Or add the following to your `Gemfile`:

```
gem "bullet_train-themes-light"
```

And then run:

```
bundle install
```

### Add `npm` packages

The `Light` theme requires the following npm packages to be installed

```
yarn add @bullet-train/bullet-train @bullet-train/fields tailwindcss@^3.4 @tailwindcss/forms @tailwindcss/typography postcss postcss-cli postcss-import postcss-nested postcss-css-variables autoprefixer glob @rails/actiontext
```

Update your `app/javascript/controllers/index.js` with the following lines:

```js
import { controllerDefinitions as bulletTrainControllers } from "@bullet-train/bullet-train"
import { controllerDefinitions as bulletTrainFieldControllers } from "@bullet-train/fields"

application.load(bulletTrainControllers)
application.load(bulletTrainFieldControllers)
```

### Overwrite tailwind and esbuild config files, add bin stubs from Bullet Train

```
curl -L "https://raw.githubusercontent.com/bullet-train-co/bullet_train/main/esbuild.config.js" -o esbuild.config.js
curl -L "https://raw.githubusercontent.com/bullet-train-co/bullet_train/main/postcss.config.js" -o postcss.config.js
curl -L "https://raw.githubusercontent.com/bullet-train-co/bullet_train/main/tailwind.config.js" -o tailwind.config.js
curl -L "https://raw.githubusercontent.com/bullet-train-co/bullet_train/main/bin/theme" -o bin/theme
curl -L "https://raw.githubusercontent.com/bullet-train-co/bullet_train/main/bin/link" -o bin/link
chmod +x bin/theme bin/link
mkdir -p postcss_mailer_config
curl -L "https://raw.githubusercontent.com/bullet-train-co/bullet_train/main/postcss_mailer_config/postcss.config.js" -o postcss_mailer_config/postcss.config.js
```

### Update `build:css` in `package.json`

In `package.json`, replace the `build` and `build:css` entries under `scripts` with:

```json
"build": "THEME=\"light\" node esbuild.config.js",
"build:css": "bin/link; yarn light:build:css; yarn light:build:mailer:css",
"light:build:css": "THEME=\"light\" NODE_PATH=./node_modules postcss ./app/assets/stylesheets/application.css -o ./app/assets/builds/application.light.css --config ./postcss.config.js",
"light:build:mailer:css": "THEME=\"light\" NODE_PATH=./node_modules postcss $(bundle exec bin/theme tailwind-stylesheet light) -o ./app/assets/builds/application.mailer.light.css --config ./postcss_mailer_config"
```

In development, `yarn light:build:css --watch` and `yarn light:build:mailer:css --watch` rebuild the stylesheets when files change. The starter's `Procfile.dev` runs both as `light-css` and `light-mailer-css`.

### Update esbuild.config.js

Remove or comment out the following line from `esbuild.config.js`:

```js
"intl-tel-input-utils": path.join(process.cwd(), "app/javascript/intl-tel-input-utils.js"),
```

### Define `current_theme` helper

In your `app/helpers/application_helper.rb`, define:

```
def current_theme
  :light
end
```

### Add `stylesheet_link_tag` to `<head>`

Make sure you have the following four lines in your `<head>`, which should be defined in `app/views/layouts/application.html.erb`:

```erb
<%= javascript_include_tag 'application', 'data-turbo-track': 'reload' %>
<%= stylesheet_link_tag 'application', media: 'all', 'data-turbo-track': 'reload' %>
<%= javascript_include_tag 'application.light', 'data-turbo-track': 'reload' %>
<%= stylesheet_link_tag 'application.light', media: 'all', 'data-turbo-track': 'reload' %>
```

### Import the Theme Style Sheet

In `app/assets/stylesheets/application.css`, add the following line:

```css
@import "$ThemeStylesheet";
```

The theme's stylesheet already includes Tailwind's base, components and utilities layers, so don't add `@tailwind` directives or `tailwindcss/*` imports yourself. The `$ThemeStylesheet` alias is resolved by `postcss-import` using the `postcss-import-config.js` that ships with the theme gem (see `postcss.config.js`).

### Add Themify Icons and trix editor support

```
yarn add @icon/themify-icons
```

To your `application.js`, add the following lines:

```js
require("@icon/themify-icons/themify-icons.css")

import { trixEditor } from "@bullet-train/fields"
trixEditor()
```

### Add Locale Strings

Add these to your `config/locales/en.yml` under `en:`

```yml
  date:
    formats:
      date_field: "%m/%d/%Y"
      date_and_time_field: "%m/%d/%Y %l:%M %p"
      date_controller: "MM/DD/YYYY"
  time:
    am: AM
    pm: PM
    formats:
      date_field: "%m/%d/%Y"
      date_and_time_field: "%m/%d/%Y %l:%M %p"
      date_controller: "MM/DD/YYYY h:mm A"
  daterangepicker:
    firstDay: 1
    separator: " - "
    applyLabel: "Apply"
    cancelLabel: "Cancel"
    fromLabel: "From"
    toLabel: "To"
    customRangeLabel: "Custom"
    weekLabel: "W"
    daysOfWeek:
    - "Su"
    - "Mo"
    - "Tu"
    - "We"
    - "Th"
    - "Fr"
    - "Sa"
    monthNames:
    - "January"
    - "February"
    - "March"
    - "April"
    - "May"
    - "June"
    - "July"
    - "August"
    - "September"
    - "October"
    - "November"
    - "December"
  date_range_controller:
    today: Today
    yesterday: yesterday
    last7Days: Last 7 Days
    last30Days: Last 30 Days
    thisMonth: This Month
    lastMonth: Last Month
  global:
    buttons:
      other: Other
      cancel: Cancel
    bulk_select:
      all: All
```

## 2. Optional Configurations for switching colors, theme gems

### For Setting the Active Color

```
curl -L "https://raw.githubusercontent.com/bullet-train-co/bullet_train/main/config/initializers/theme.rb" -o config/initializers/theme.rb
```

Add the following classes to your `html` tag for your layout:

```erb
<html class="theme-<%= BulletTrain::Themes::Light.color %> <%= "theme-secondary-#{BulletTrain::Themes::Light.secondary_color}" if BulletTrain::Themes::Light.secondary_color %> <%= BulletTrain::Themes::Light.force_color_scheme_to.to_s.presence %>">
```

### For Switching Between Installed Themes

If you'd like to create your own theme but would still like to build on top of `:light`, you'll need to have both gems installed and you'll be able to switch the current theme this way.

Change the `current_theme` value in `app/helpers/application_helper.rb`

```
def current_theme
  :super_custom_theme
end
```

To change to use a different theme:

1. Change the value returned by `current_theme` to the new theme name
2. Change the name of the `THEME` env var defined in `build` and `build:css` in `package.json`
3. Change the theme name in both the `javascript_include_tag` and `stylesheet_link_tag` (`application.<theme>`) in the `<head>`.

## 3. Using Locales for fields on new models

The theme's field partials work best with locale strings that are defined for the model you're creating.

Example: you've created a Project model. Here we'll create a `projects.en.yml`

1. Run `curl -L "https://raw.githubusercontent.com/bullet-train-co/bullet_train-core/main/bullet_train-super_scaffolding/config/locales/en/scaffolding/completely_concrete/tangible_things.en.yml" -o config/locales/projects.en.yml`
2. Search and replace `projects.en.yml` for `scaffolding/completely_concrete/tangible_things`, `Tangible Things`, `Tangible Thing`, `tangible_things` and `tangible_thing`. Replace with `projects`, `Projects`, `Project`, `projects` and `project` respectively.
3. Remove strings you won't be using. In particular, look for comments with "skip" or "scaffolding".

Some fields use locale strings to drive their `options`. In the `tangible_things.en.yml` template file, look for `super_select_value`, `multiple_option_values` and others.

You'll notice `&` and `*` symbols prefixing some special keys in the `yml` file. Those are anchors and aliases and they help you reduce repetition in your locale strings.

To learn more about how these locales are generated in Bullet Train, see the documentation on [Bullet Train's Super Scaffolding](/docs/super-scaffolding.md)

## 4. Partials that require special instructions, exclusions

### For using boolean-type fields (options, buttons)

In `ApplicationController`, add this:

```ruby
include Fields::ControllerSupport
```

### For the file_field partial

```ruby
# in the model
has_one_attached :file_field_value
after_validation :remove_file_field_value, if: :file_field_value_removal?
attr_accessor :file_field_value_removal
def file_field_value_removal?
def remove_file_field_value
```

```ruby
# in the controller's strong_params
:file_field_value,
:file_field_value_removal,
```

### For `image`, `active_storage_image`

See [`account/users_helper` in BT core repo](https://github.com/bullet-train-co/bullet_train-core/blob/main/bullet_train/app/helpers/account/users_helper.rb) for implementing `photo_url_for_active_storage_attachment`

## 5. Modifying ejected partials

### For ejecting a theme partial and modifying it

We recommend firing up a Bullet Train project and using its `bin/resolve` (see docs on [Indirection](/docs/indirection)) to get a copy of the partial field locally to modify.
