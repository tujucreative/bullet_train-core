# Onboarding
Bullet Train provides an easy to understand and modifiable structure for defining required onboarding steps and forms. This system makes it easy for users to complete required steps before seeing your application's full account interface. The code for this feature is well documented in the code with comments, so rather than duplicating that document here, we'll simply direct you to the relevant files.

## Included Onboarding Steps

### Collect User Details

The included "user details" onboarding step is intended to collect any fields that are required for a user account to be "complete" while not requiring those fields to be collected on the initial sign up form. This is a deliberate UX decision to try and increase conversions on the initial form.

### Collect User Email

The "user email" onboarding step is specifically used in situations where a user signs up with an OAuth provider that either doesn't supply their email address to the application or doesn't have a verified email address for the user. In this situation, we want to have an email address on their account, so we prompt them for it.

### Invite Team Members (optional)

When bulk invitations are enabled (`BulletTrain::Configuration.enable_bulk_invitations = true`), a user who has just created a team is shown a form to invite several collaborators at once after completing their user details. It is off by default.

## Relevant Files

Each of the following files exist within their own respective Bullet Train packages. Make sure you navigate to the correct package when searching for the following files.

### Controllers
 - `ensure_onboarding_is_complete` in `bullet_train-core/bullet_train/app/controllers/concerns/account/controllers/base.rb` (the framework's steps) and in your app's `app/controllers/account/application_controller.rb` (where you add your own)
 - `bullet_train-core/bullet_train/app/controllers/account/onboarding/user_details_controller.rb`
 - `bullet_train-core/bullet_train/app/controllers/account/onboarding/user_email_controller.rb`

### Views
 - `bullet_train-core/bullet_train/app/views/account/onboarding/user_details/edit.html.erb`
 - `bullet_train-core/bullet_train/app/views/account/onboarding/user_email/edit.html.erb`

### Models
 - `user#details_provided?` in `bullet_train/app/models/concerns/users/base.rb`

### Routes
 - `namespace :onboarding` in `bullet_train-core/bullet_train/config/routes.rb` and `bullet_train/config/routes.rb`

## Adding Additional Steps
Although you can implement onboarding steps from scratch, we always just copy and paste one of the existing steps as a starting point, like so:

1. Copy, rename, and modify all of the existing onboarding controllers.
2. Copy, rename, and modify the corresponding `edit.html.erb` view.
3. Add a route for your new step inside the `namespace :onboarding` block in your app's `config/routes.rb` (model it on the entries in `bullet_train-core/bullet_train/config/routes.rb`).
4. Add the appropriate gating logic in `ensure_onboarding_is_complete` in your app's `app/controllers/account/application_controller.rb`.

Onboarding steps aren't limited to targeting `User` models. It's possible to add onboarding steps to help flesh out team `Membership` records or `Team` records as well. You can use this pattern for setting up any sort of required data for either the user or the team.
