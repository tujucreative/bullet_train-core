# Automated Test Suite
All of Bullet Train’s core functionality is verifiable using the provided test suite. This foundation of headless browser integration tests took a ton of time to write, but they can give you the confidence and peace of mind that you haven't broken any key functionality in your application before a deploy.

You can run the test suite with the following commands in your shell:

```
rails test
rails test:system
```

## Fixing Broken Tests

### 1. Run Chrome in Non-Headless Mode

When debugging tests, it's important to be able to see what Capybara is seeing. You can disable the headless browser mode by prefixing `rails test:system` like so:

```shell
MAGIC_TEST=1 rails test:system
```

When you run the test suite with `MAGIC_TEST` set in your environment like this, the browser will appear on your screen after the first Capybara test starts. (This may not be the first test that runs.) Be careful not to interact with the window when it appears, as sometimes your interactions can cause the test to fail needlessly. You can also set `HEADLESS=0`.

### 2. Insert `binding.pry`.

Open the failing test file and insert `binding.pry` right before the action or assertion that is causing the test to fail. After doing that, when you run the test, it will actually stop and open a debugging console while the browser is still open to the appropriate page where the test is beginning to fail. You can use this console and the open browser to try and figure out why the test is failing. When you're done, hit <kbd>Control</kbd> + <kbd>D</kbd> to exit the debugger and continue letting the test run.

## Super Scaffolding Test Suite
In addition to the standard application test suite, there are separate tests, not run by default, that you can run in CI to ensure you haven't accidentally broken Super Scaffolding. Each directory in `test/system/super_scaffolding/` (for example `project` or `insight`) contains a `setup.rb` that runs a number of Super Scaffolding commands, a test, and a `teardown.rb` that removes the generated code. Because setup generates many files and teardown cleans up by brute force, you should never run them in anything but a clean working copy that can be reset when you're done.

Run the setup script, the test, and then the teardown script:

```
./test/system/super_scaffolding/project/setup.rb
rails test test/system/super_scaffolding/project/
./test/system/super_scaffolding/project/teardown.rb
```

(Setup refuses to run in a dirty git workspace unless you pass `--allow-dirty-workspace`.)
