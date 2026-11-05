# Authentication
Bullet Train uses [Devise](https://github.com/heartcombo/devise) for authentication and we've done the work of making the related views look pretty and well-integrated with the look-and-feel of the application template.

## Customizing Controllers
Bullet Train registers its own slightly customized registration and session controllers for Devise. If you want to customize them further, you can simply eject those controllers from the framework and override them locally, like so:

```
bin/resolve RegistrationsController --eject --open
bin/resolve SessionsController --eject --open
```

## Customizing Views
You can customize Devise views using the same workflow you would use to customize any other Bullet Train views.

## Invite-Only Mode and Disabling Registration
If you would like to stop users from signing up for your application without an invitation code or without an invitation to an existing team, set `INVITATION_KEYS` to one or more comma-delimited values in `config/application.yml` (or however you configure your environment values in production.) Once invitation keys are configured, you can invite people to sign up with one of your keys at the following URL:

```
https://example.com/invitation?key=ONE_OF_YOUR_KEYS
```

If you want to disable new registrations completely, put an unguessable value into `INVITATION_KEYS` and keep it secret.

Note that in both of these scenarios that existing users will still be able to invite new collaborators to their teams and those collaborators will have the option of creating a new account, but no users in the application will be allowed to create a new team without an invitation code and following the above URL.

## Enabling Two-Factor Authentication (2FA)
Two-factor authentication is enabled by default in Bullet Train, but you must have Rails built-in encrypted secrets and Active Record Encryption configured.

To do this, first run:

```
bin/secrets
```

That will generate credentials files for development, test and production, including the Active Record Encryption keys (`active_record_encryption.primary_key`, `deterministic_key` and `key_derivation_salt`), along with the matching key files in `config/credentials/` (which are git-ignored, so store them somewhere safe). To inspect or edit the credentials, run:

```
EDITOR='code --wait' bin/rails credentials:edit --environment development
```

If you ever need to regenerate the keys manually, run `bin/rails db:encryption:init` and paste the output into the credentials file.

If you'd rather set them via environment variables you could add something like this to `config/application.rb`:

```
config.active_record.encryption.primary_key = ENV['ACTIVE_RECORD_ENCRYPTION_PRIMARY_KEY']
config.active_record.encryption.deterministic_key = ENV['ACTIVE_RECORD_ENCRYPTION_DETERMINISTIC_KEY']
config.active_record.encryption.key_derivation_salt = ENV['ACTIVE_RECORD_ENCRYPTION_KEY_DERIVATION_SALT']
```

And then populate those ENV variables by whatever means you use. (Maybe setting them in `.env` or possibly exporting them directly.)

After you have things working in development you'll need to follow the same process for your production environment, and any others.
