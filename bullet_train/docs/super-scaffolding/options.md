# Super Scaffolding Options

There are different flags you can pass to the Super Scaffolding command which gives you more flexibility over creating your model. Add the flag of your choice to **the end** of the command for the option to take effect.
```
rails generate super_scaffold Project Team description:text_field --sortable
```

Most of these include skipping particular functionalities, so take a look at what's available here and pass the flag that applies to your use-case.

You can also run `rails generate super_scaffold --help` to see the flags of the scaffolder you are using.

## `rails generate super_scaffold`

| Option | Description |
|--------|-------------|
| `--skip-migration-generation` | Don't generate the model migration (generate the model yourself first) |
| `--sortable` | [Details here](/docs/super-scaffolding/sortable.md) |
| `--namespace=customers` | [Details here](/docs/namespacing.md). Also implies `--skip-api`, `--skip-model` and `--skip-locales`. |
| `--navbar="ti-world"` | Pass the Themify icon or Font Awesome icon of your choice to automatically add it to the navbar* |
| `--only-index` | Only scaffold the index view for a model |
| `--skip-form` | Don't generate a new/edit form |
| `--skip-locales` | Don't generate locale files |
| `--skip-api` | Don't generate API files |
| `--skip-model` | Don't generate a model file |
| `--skip-controller` | Don't generate a controller file |
| `--skip-routes` | Don't generate routes |
| `--skip-parent` | Don't add the model to the show page of its parent (and skip the navbar prompt) |

*This option is only available for top-level models, which are models that are direct children of the `Team` model. The value must start with `ti` (Themify) or `fa` (Font Awesome). If you omit the option for a top-level model you are asked interactively.

## Other scaffolders

`rails generate super_scaffold:field` and `rails generate super_scaffold:join_model` accept `--skip-migration-generation`, `--skip-form`, `--skip-show`, `--skip-table`, `--skip-locales`, `--skip-api` and `--skip-model`.

`rails generate super_scaffold:oauth_provider` accepts `--icon=ti-...`.
