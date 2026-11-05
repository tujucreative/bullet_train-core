# Notable Versions

## v1.45.0

The starter repository moved to Ruby 4.0.3 and Node 24.15.0 (`.ruby-version` and `.nvmrc`). Update your
local and production runtimes to match, or keep your own values when you resolve merge conflicts in those files.


## v1.42.0 / v1.42.1

Don't upgrade to `1.42.0`. It upgraded `devise` to version 5 in `Gemfile.lock` only, without pinning the new
version in the starter repository's `Gemfile`, so apps that merged it stayed on `devise` 4, which is
incompatible with the changes made in `1.42.0`. Skip to `1.42.1`, which pins `devise` to version 5.

If you have to use `1.42.0`, add this to your `Gemfile`:

```ruby
gem "devise", "~> 5.0"
```

Starting with `1.42.0` the Bullet Train gems require Rails 7.0.0 or newer.


## v1.39.0 - v1.40.x

Don't upgrade to any of these versions. Go straight to `1.41.0`.

The `1.39.0` release is not usable. The starter repository never got a `1.39.0` release, and the Tailwind CSS v4
upgrade in that release broke customized themes. The `1.40.x` releases only reverted that upgrade and got npm package
publishing working again, and they have no starter repository releases either.

`1.41.0` is a stable release that goes back to Tailwind CSS v3 and has working npm packages.


## v1.35.0

If you've ejected the theme stylesheets (with `rake bullet_train:themes:light:eject_css`) or you maintain your own
theme, compare your stylesheets with the ones in the theme gem:

* `@extend` rules were replaced with `@apply` rules, so the starter repository no longer needs the `postcss-extend-rule` package.
* The three `tailwindcss/base.css`, `components.css` and `utilities.css` files were merged into a single `tailwindcss_import.css`, which the main stylesheet now imports.


## v1.31.0

Users can now choose between dark mode, light mode or their system preference (see [Dark and Light Mode](/docs/themes)).
If your `tailwind.config.js` has a `darkMode` setting, remove it. To force a color scheme, set
`BulletTrain::Themes::Light.force_color_scheme_to` to `:light` or `:dark` in `config/initializers/theme.rb`.


## v1.4.11 / v1.5.0

In v1.5.0 we made some fairly big changes to the format of system tests. In order to ease the
transition we shipped a helper in 1.4.11 that you can use to modify your copy of the tests before
you merge in version 1.5.0. Doing this should reduce the chances for merge conflicts.

To use the helper to modify your tests run this in a console (after updating to v1.4.11):

```
bin/updates/system_tests/use_device_test
```

Then you should:

* Run the tests to make sure they still pass
* Commit the updated test files
* Update to v1.5.0

### Dealing with merge conflicts after merging v1.5.0

When you update to 1.5.0 you may have rather large merge conflicts in some system test files,
especially if you've modified those files to accommodate changes to user flows in your own app.

When you're resolving conflicts you'll mostly want to pick your own version of stuff (that is,
the code on the `HEAD` side of the conflict). If you want to avoid manually sorting a conflict
and just preserve your own version of a file you can do something like this:

```
git checkout HEAD -- test/system/the_test_file_in_question.rb
```

### About this change

We've introduced a new `device_test` helper that wraps up some of the implementation
details about running system tests on a variety of devices.

It simplifies the way that we write system tests like this:

```diff
-  @@test_devices.each do |device_name, display_details|
-    test "user can so something on a #{device_name}" do
-      resize_for(display_details)
-      # actual tests here
-    end
-  end
+  device_test "user can do something" do
+    # actual test code
+  end
```

The specific changes are:

* Remove the enclosing @@test_devices.each do block
* Remove one level of indentation from test inside those blocks
* Remove the resize_for(display_details) from tests in those blocks
* Use `device_test` helper to handle running the test on different devices


## v1.3.22

In version 1.3.22 we added an `Address` model. If your app already had an `Address` model you'll
probably want to reject some of the updates made to the starter repo in this version.

TODO: Can we offer more direction here?


## v1.3.0

Version 1.3.0 is when we started explicitly bumping the Bullet Train gems within the starter repo
every time that we release a new version of the `core` gems. Unfortunately, at that time we were
only making changes to Gemfile.lock which kind of hides the dependencies, and is often a source of
merge conflicts that can be hard to sort out.

[See the upgrade guide for getting your app to version 1.3.0](/docs/upgrades/yolo-130)
