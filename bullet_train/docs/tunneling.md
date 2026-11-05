# HTTP Tunneling with ngrok

Before your application can take advantage of features that depend on incoming webhooks, you'll need to setup an HTTP tunnel using a service like [ngrok](https://ngrok.com).

## Use a Paid Plan

You should specifically sign up for a paid account. Although ngrok's free plan includes one assigned development domain, a paid plan (see their [pricing page](https://ngrok.com/pricing)) lets you reserve your own domain for reuse each time you spin up your tunnel. This is a critical productivity improvement, because in practice you'll end up configuring your tunnel URL in a bunch of different places like `config/application.yml` but also in external systems like when you [configure payment providers to deliver webhooks to you](/docs/billing/stripe.md).

## Usage

Once you have ngrok installed, you can start your tunnel like so, replacing `YOUR-DOMAIN` with the full domain you reserved or were assigned in your ngrok account (for example `your-name.ngrok-free.app`):

```
ngrok http --domain=YOUR-DOMAIN 3000
```

## Updating Your Configuration

Before your Rails application will accept connections on your tunnel hostname, you need to update `config/application.yml` with:

```yaml
BASE_URL: https://YOUR-DOMAIN
```

If ngrok is already running, `bin/set-ngrok-url` will read the tunnel URL and update `BASE_URL` in `config/application.yml` for you. Run it again whenever the URL changes.

You'll also need to restart your Rails server:

```
rails restart
```
