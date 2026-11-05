# Code Generation with Super Scaffolding

Super Scaffolding is Bullet Train’s code generation engine. Its goal is to allow you to produce production-ready CRUD interfaces for your models while barely lifting a finger, and it handles a lot of other grunt-work as well.

Here’s a list of what Super Scaffolding takes care of for you each time you add a model to your application:

 - It generates a basic CRUD controller and accompanying views.
 - It generates a Yaml locale file for the views’ translatable strings.
 - It generates type-specific form fields for each attribute of the model.
 - It generates an API controller and an accompanying entry in the application’s API docs.
 - It generates a serializer that’s used by the API and when dispatching webhooks.
 - It adds the appropriate permissions for multitenancy to `config/models/roles.yml`, which Bullet Train turns into CanCanCan abilities.
 - It adds the model’s table view to the show view of its parent.
 - It adds the model to the application’s navigation (if applicable).
 - It generates breadcrumbs for use in the application’s layout.
 - It generates the appropriate routes for the CRUD controllers and API endpoints.

When adding just one model, Super Scaffolding generates ~30 different files on your behalf.

**NOTE** Super Scaffolding generates code that contains special "magic comments" that we use as targets for later Super Scaffolding operations. You shouldn't remove those comments. See [Targets for Super Scaffolding](/docs/super-scaffolding/targets.md) for more details.

## Living Templates

Bullet Train's Super Scaffolding engine is a unique approach to code generation, based on template files that are functional code instead of obscure DSLs that are difficult to customize and maintain. Super Scaffolding automates the most repetitive and mundane aspects of building out your application's basic structure. Furthermore, it does this without leaning on the magic of libraries that force too high a level of abstraction. Instead, it generates standard Rails code that is both ready for prime time, but is also easy to customize and modify.

## Prerequisites

Before getting started with Super Scaffolding, we recommend reading about [the philosophy of domain modeling in Bullet Train](/docs/modeling.md).

## Usage

Super Scaffolding is a set of Rails generators, and each one documents itself. To see the arguments, options and an example for a scaffolder, run it with `--help`, e.g. `rails generate super_scaffold --help` or `rails generate super_scaffold:field --help`.

> `bin/super-scaffold` (e.g. `bin/super-scaffold crud Project Team name:text_field`) is deprecated. It forwards to the matching `rails generate super_scaffold...` command and prints a warning.

## Available Scaffolding Types

| `rails generate` Command | Scaffolding Type |
|--------------------------|------------------|
| `rails generate super_scaffold` | Basic CRUD scaffolder |
| `rails generate super_scaffold:field` | Adds a field to an existing model |
| `rails generate super_scaffold:incoming_webhooks` | Scaffolds an incoming webhook |
| `rails generate super_scaffold:join_model` | Scaffolds a join model (must have two existing models to join before scaffolding) |
| `rails generate super_scaffold:oauth_provider` | Scaffolds logic to use OAuth2 with the provider of your choice |

The following commands are for use specifically with [Action Models](action-models).

| `rails generate` Command | Scaffolding Type |
|--------------------------|------------------|
| `rails generate super_scaffold:action_models:targets_many` | Generates an action that targets many records |
| `rails generate super_scaffold:action_models:targets_one` | Generates an action that targets one record |
| `rails generate super_scaffold:action_models:targets_one_parent` | Generates an action that targets the parent of the specified model |
| `rails generate super_scaffold:action_models:performs_import` | Generates an action that imports a list of records (e.g. from a CSV) to its parent |
| `rails generate super_scaffold:action_models:performs_export` | Generates an action that exports a list of records |

The following commands are only available when the corresponding optional gem is installed.

| `rails generate` Command | Scaffolding Type |
|--------------------------|------------------|
| `rails generate super_scaffold:audit_logs` | Adds an audit log to a model (requires the Audit Logs gem) |
| `rails generate super_scaffold:conversations` | Adds conversations to a model (requires the Conversations gem) |

Flags such as `--skip-migration-generation` and `--sortable` are covered in [Super Scaffolding Options](/docs/super-scaffolding/options.md).

## Examples

### 1. Basic CRUD Scaffolding

Let's implement the following feature:

> An organization has many projects.

First, run the scaffolder:
```
rails generate super_scaffold Project Team name:text_field
rake db:migrate
```

In the above example, `team` represents the model that a `Project` primarily belongs to. Also, `text_field` was selected from [the list of available field partials](/docs/field-partials.md). We'll show examples with `trix_editor` and `super_select` later.

Super Scaffolding automatically generates models for you. However, if you want to split this process, you can pass the `--skip-migration-generation` to the command.

For example, generate the model with the standard Rails generator:
```
rails g model Project team:references name:string
```

⚠️ Don't run migrations right away. It would be fine in this case, but sometimes the subsequent Super Scaffolding step actually updates the migration as part of its magic.

Then you can run the scaffolder with the flag:
```
rails generate super_scaffold Project Team name:text_field --skip-migration-generation
```

### 2. Nested CRUD Scaffolding

Building on that example, let's implement the following feature:

```
A project has many goals.
```

First, run the scaffolder:

```
rails generate super_scaffold Goal Project,Team description:text_field
rake db:migrate
```

You can see in the example above how we've specified `Project,Team`, because we want to specify the entire chain of ownership back to the `Team`. This allows Super Scaffolding to automatically generate the required permissions. Take note that this generates a foreign key for `Project` and not for `Team`.

### 3. Adding New Fields with `field`

One of Bullet Train's most valuable features is the ability to add new fields to existing scaffolded models. When you add new fields with the `field` scaffolder, you don't have to remember to add that same attribute to table views, show views, translation files, API endpoints, serializers, tests, documentation, etc.

Building on the earlier example, consider the following new requirement:

> In addition to a name, a project can have a description.

Use the `field` scaffolder to add it throughout the application:

```
rails generate super_scaffold:field Project description:trix_editor
rake db:migrate
```

As you can see, when we're using `field`, we don't need to supply the chain of ownership back to `Team`.

If you want to scaffold a new field to use for read-only purposes, add the `readonly` option. The field is left out of the form and out of strong parameters, but it is still added to the show view, table view, locale file and API serializer:
```
rails generate super_scaffold:field Project description:trix_editor{readonly}
```

Again, if you would like to automatically generate the migration on your own, pass the `--skip-migration-generation` flag:
```
rails generate super_scaffold:field Project description:trix_editor --skip-migration-generation
```

### 4. Adding Option Fields with Fixed, Translatable Options

Continuing with the earlier example, let's address the following new requirement:

> Users can specify the current project status.

We have multiple [field partials](/docs/field-partials.md) that we could use for this purpose, including `buttons`, `options`, or `super_select`.

In this example, let's add a status attribute and present it as buttons:

```
rails generate super_scaffold:field Project status:buttons
```

By default, Super Scaffolding configures the buttons as "One", "Two", and "Three", but in this example you can edit those options in the `fields` section of `config/locales/en/projects.en.yml`. For example, you could specify the following options:

```yaml
planned: Planned
started: Started
completed: Completed
```

If you want new `Project` models to be set to `planned` by default, you can add that to the migration file that was generated before running it, like so:

```ruby
add_column :projects, :status, :string, default: "planned"
```

### 5. Scaffolding `belongs_to` Associations, Team Member Assignments

Continuing with the example, consider the following requirement:

> A project has one specific project lead.

Although you might think this calls for a reference to `User`, we've learned the hard way that it's typically much better to assign resources on a `Team` to a `Membership` on the team instead. For one, this allows you to assign resources to new team members that haven't accepted their invitation yet (and don't necessarily have a `User` record yet.)

We can accomplish this like so:

```
rails generate super_scaffold:field Project lead_id:super_select{class_name=Membership}
rake db:migrate
```

There are three important things to point out here:

1. The scaffolder automatically adds a foreign key for `lead` to `Project`.
2. When adding this foreign key the `references` column is generated under the name `lead`, but when we're specifying the _field_ we want to scaffold, we specify it as `lead_id`, because that's the name of the attribute on the form, in strong parameters, etc.
3. We have to specify the model name with the `class_name` option so that Super Scaffolding can fully work its magic. We can't reflect on the association, because at this point the association isn't properly defined yet. With this information, Super Scaffolding can handle that step for you.

Finally, Super Scaffolding will prompt you to edit `app/models/project.rb` and implement the required logic in the `valid_leads` method. This is a template method that will be used to both populate the select field on the `Project` form, but also enforce some important security concerns in this multi-tenant system. In this case, you can define it as:

```ruby
def valid_leads
  team.memberships.current_and_invited
end
```

(The `current_and_invited` scope just filters out people that have already been removed from the team.)

**NOTE:** It's possible to define the source for `valid_leads` in your super scaffolding command.
You need to wrap the bits inside the curly brackets with double quotes, and then add
`,source=team.memberships.current_and_invited` inside the quotes:

```
{"class_name=Membership,source=team.memberships.current_and_invited"}
```

So the whole command would be:

```
rails generate super_scaffold:field Project lead_id:super_select{"class_name=Membership,source=team.memberships.current_and_invited"}
```

### 6. Scaffolding Has-Many-Through Associations with `join_model`

Finally, working from the same example, imagine the following requirement:

> A project can be labeled with one or more project-specific tags.

We can accomplish this with a new model, a new join model, and a `super_select` field.

First, let's create the tag model:

```
rails generate super_scaffold Projects::Tag Team name:text_field
```

Note that project tags are specifically defined at the `Team` level. The same tag can be applied to multiple `Project` models.

Now, let's create a join model for the has-many-through association.

We're not going to scaffold this model with the typical `rails generate super_scaffold` scaffolder, but some preparation is needed before we can use it with the `field` scaffolder, so we need to do the following:

```
rails generate super_scaffold:join_model Projects::AppliedTag project_id{class_name=Project} tag_id{class_name=Projects::Tag}
```

All we're doing here is specifying the name of the join model, and the two attributes and class names of the models it joins. Note again that we specify the `_id` suffix on both of the attributes.

Now that the join model has been prepared, we can use the `field` scaffolder to create the multi-select field:

```
rails generate super_scaffold:field Project tag_ids:super_select{class_name=Projects::Tag}
rake db:migrate
```

Just note that the suffix of the field is `_ids` plural, and this is an attribute provided by Rails to interact with the `has_many :tags, through: :applied_tags` association.

The `field` step will ask you to define the logic for the `valid_tags` method in `app/models/project.rb`. You can define it like so:

```ruby
def valid_tags
  team.projects_tags
end
```

Honestly, it's crazy that we got to the point where we can handle this particular use case automatically. It seems simple, but there is so much going on to make this feature work.

### 7. Scaffolding image upload attributes

Bullet Train comes with two different ways to handle image uploads.

* Cloudinary - This option allows your app deployment to be simpler because you don't need to ship any image manipulation libraries. But it does introduce a dependence on a 3rd party service.
* ActiveStorage - This option doesn't include reliance on a 3rd party service, but you do have to include image manipulation libraries in your deployment process.

#### Scaffolding images with Cloudinary

When you scaffold your model a `string` is generated where Cloudinary can store a reference to the image.

Make sure you have the `CLOUDINARY_URL` environment variable to use Cloudinary for your images.

For instance to scaffold a `Project` model with a `logo` image upload.
Use `image` as a field type for super scaffolding:

```
rails generate super_scaffold Project Team name:text_field logo:image
rake db:migrate
```

Under the hood, Bullet Train will generate your model with the following command:
```
bin/rails generate model Project team:references name:string logo:string
```

#### Scaffolding images with ActiveStorage

When you scaffold your model we generate an `attachment` type attribute.

For instance to scaffold a `Project` model with a `logo` image upload.
Use `image` as a field type for super scaffolding:

```
rails generate super_scaffold Project Team name:text_field logo:image
rake db:migrate
```

Under the hood, Bullet Train will generate your model with the following command:
```
bin/rails generate model Project team:references name:string logo:attachment
```

## Attribute Options

Each attribute can take options in curly braces after its type, e.g. `lead_id:super_select{class_name=Membership}`. Separate several options with commas and wrap the whole value in double quotes, as in the `source` example in section 5 above.

| Option | Description |
|--------|-------------|
| `{class_name=Foo}` | The model on the other side of an `_id` or `_ids` attribute. Required when it can't be inferred from the attribute name. |
| `{source=expr}` | The body of the generated `valid_*` method for an `_id` or `_ids` attribute. The expression can't contain a comma or `=`, because options are split on both. |
| `{vanilla}` | Treat an attribute ending in `_id` or `_ids` as a plain attribute, with no association magic. |
| `{multiple}` | Allow multiple values for `buttons`, `options`, `super_select` (stored as `jsonb`), and for `file_field` and `image` (stored as `attachments`). Not supported for `image` when Cloudinary is enabled. |
| `{required}` | Adds `validates :attribute, presence: true` to the model. For `_id` attributes it drops `optional: true` from the `belongs_to` instead, and `file_field` is never required. The first attribute of a newly scaffolded model is always required. |
| `{readonly}` | Leaves the field out of the form and strong parameters. See the `readonly` example in section 3. |
| `{label=method}` | The method called on the associated records to build option labels. Defaults to `label_string`. |
| `{language=ruby}` | The language used by a `code_editor` field. |
| `{attribute=partial}` | Overrides the `shared/attributes/*` partial used to display the attribute on show and table views. |

The first attribute of a newly scaffolded model can't be a `trix_editor`, `ckeditor`, `text_area` or `code_editor` field.

## Additional Notes

### `TangibleThing` and `CreativeConcept`

In order to properly facilitate this type of code generation, Bullet Train includes two models in the `Scaffolding` namespace as a parent and child model:

1. `Scaffolding::AbsolutelyAbstract::CreativeConcept`
2. `Scaffolding::CompletelyConcrete::TangibleThing`

Their peculiar naming is what's required to ensure that their corresponding view and controller templates can serve as the basis for any combination of different model naming or [namespacing](https://blog.bullettrain.co/rails-model-namespacing/) that you may need to employ in your own application. There are [a ton of different potential combinations of parent and child namespaces](https://blog.bullettrain.co/nested-namespaced-rails-routing-examples/), and these two class names provide us with the fidelity we need when transforming the templates to represent any of these scenarios.

Only the files associated with `Scaffolding::CompletelyConcrete::TangibleThing` actually serve as scaffolding templates, so we also take advantage of `Scaffolding::AbsolutelyAbstract::CreativeConcept` to demonstrate other available Bullet Train features. For example, we use it to demonstrate how to implement resource-level collaborators.

### Hiding Scaffolding Templates

You won't want your end users seeing the Super Scaffolding templates in your environment, so you can disable their presentation by setting `HIDE_THINGS` in your environment. For example, you can add the following to `config/application.yml`:

```yaml
HIDE_THINGS: true
```

## Advanced Examples
 - [Super Scaffolding Options](/docs/super-scaffolding/options.md)
 - [Super Scaffolding with Delegated Types](/docs/super-scaffolding/delegated-types.md)
 - [Super Scaffolding with the `--sortable` option](/docs/super-scaffolding/sortable.md)

## Additional Info

 - [Targets for Super Scaffolding](/docs/super-scaffolding/targets.md)
