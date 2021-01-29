# What the schnack?

Schnack is an open source commenting system written in JavaScript.
- Tiny! It takes only ~**8 KB!!!** to embed Schnack.
- **Open source** and **self-hosted**.
- Ad-free and Tracking-free. Schnack will **not disturb your users**.
- It's simple to moderate, with a **minimal** and **slick UI** to allow/reject comments or trust/block users.
- **[webpush protocol](https://tools.ietf.org/html/draft-ietf-webpush-protocol-12) to notify the site owner** about new comments awaiting for moderation.
- **Third party providers for authentication** like Github, Twitter, Google and Facebook. Users are not required to register a new account on your system and you don't need to manage a user management system.

# Quickstart

This is the fastest way to setup *schnack*.


**Requirements**:
- Node.js (>= v8)
- npm (>= v5)

Create a new folder in which you want to install Schnack:

```bash
> mkdir schnack
> cd schnack
```

Download the [config file template](https://github.com/schn4ck/schnack/blob/master/schnack.tpl.json) as `schnack.json` and edit it according to [configuration](#configuration) section:

```bash
> curl -o schnack.json
> vim schnack.json  # or open with any editor of your choice
```

Make sure you remove all plugin configs that you don't need. 
Then install Schnack and the configured plugins via

```bash
> npm init schnack
```

Run the server:

```bash
> npm start
```

Embed in your HTML page:

```html
<div class="comments-go-here"></div>
<script src="https://comments.yoursite.com/embed.js"
    data-schnack-slug="post-slug"
    data-schnack-target=".comments-go-here">
</script>
```

# Configuration

*schnack* will try to read the configuration from the `schnack.json` file. The minimal configuration requires the following fields: *schnack_host*, *admins*, *oauth.secret*  and at least one oauth provider (id and secret key) and one notification provider.
The fields *schnack_host* and *page_url* should be hosted on the **same domain**. If your blog is running at *https://blog.example.com*, then your *schnack* instance should be reachable at any **subdomain** of *example.com*.


| option                                    | description                                                                                                                                               |
|-------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------|
| schnack_host                              | the hostname where the schnack server runs (e.g. *https://schnack.example.com*)                                                                            |
| page_url                                  | the page where schnack is going to be embeded. The `%SLUG%` placeholder should be replaceable with you tags (e.g. *https://blog.example.com/posts/%SLUG%*) |
| database                                  |                                                       |
| &nbsp;&nbsp;comments                                  | the filename of the embeded SQLite database where the user comments are going to be stored (e.g. *comments.db*)                                                         |
|  &nbsp;&nbsp;sessions                                  | the filename of the embeded SQLite database where OAuth session data is going to be stored (e.g. *sessions.db*)                                                         |
| port                                      | the port where the schnack server is going to run (e.g. 3000)                                                                                             |
| admins                                    | an array of userIDs which can login as admin (e.g. *[1, 245]*)                                                                                            |
| oauth                                     |                                                                                                                                                           |
| &nbsp;&nbsp;secret                        | the secret passed to [express-session](https://github.com/expressjs/session#secret)                                                                       |
| plugins                                     | additional config options for plugins (see below)                                                                                                                                                     |
| date_format                               | how to display dates (e.g. *MMMM DD, YYYY - h:mm a*)                                                                                                      |
| trust                                     | a list of trusted users (see [Trust your friends](#trust-your-friends))                                                                                   |


# Plugins

*schnack* supports various [authentication](#authentication) and [notification](#notifications) providers which can be configured via plugins. Once you added the plugin config to your `schnack.json`, they will be installed automatically via:

```bash
> npm init schnack
```



## Authentication

*schnack* uses authentication plugins to authenticate the users of your comment platform, in order to prevent spam without having to implement and manage an own user management system. Most of these plugins use external services as OAuth providers.

You should configure at least one authentication plugin in order to allow users to login and write comments.

When a user logs in through an OAuth provider, the session informations are stored into a cookie. In order to allow this action, your *schnack* instance and the page where you are embedding *schnack* should reside on **subdomains of the same domain**.

The `secret` provided in `schnack.json` will be used by [express-session](https://github.com/expressjs/session#secret) to sign the session ID cookie.

### Mastodon

If you activate the plugin `auth-mastodon` users can log in through any Mastodon instance account.

- All you have to do is to define `app_name` and `app_website` in the `plugins.auth-mastodon` section of your `schnack.json`. Schnack will then create OAuth applications at Mastodon instances whenever a user tries to sign in (the users are asked to enter a Mastodon domain before signing in).

```json
// schnack.json
{
    "plugins": {
        "auth-mastodon": {
            "app_name": "your website name",
            "app_website": "https://blog.example.com/"
        }
    }
}
```

### Twitter

If you activate the plugin `auth-twitter` users can log in using their Twitter account.

- Create a new OAuth App on [apps.twitter.com](https://apps.twitter.com/)
    - *Name*: the name of your blog
    - *Description*: the description of your blog
    - *Website*: the URL of your *schnack* instance (e.g. https://schnack.example.com)
    - *Callback URL*: the URL of your *schnack* instance followed by `/auth/twitter/callback` (e.g. https://schnack.example.com/auth/twitter/callback)
    - Check the checkbox "Allow this application to be used to Sign in with Twitter"
- Copy the Consumer Key and the Consumer Secret from "Keys and Access Tokens" to `plugins.auth-twitter.consumer_key` and `plugins.auth-twitter.consumer_secret` in `schnack.json`

```json
// schnack.json
{
    "plugins": {
        "auth-twitter": {
            "consumer_key": "xxxxx",
            "consumer_secret": "xxxxx"
        }
    }
}
```



### GitHub

If you activate the plugin `auth-github` users can log in using their Github account.

- Create a new GitHub [OAuth App](https://github.com/settings/applications/new)
    - *Application name*: the name of your blog
    - *Homepage URL*: the URL of your *schnack* instance (e.g. https://schnack.example.com)
    - *Application description*: the description of your blog
    - *Authorization callback URL*: the URL of your *schnack* instance followed by `/auth/github/callback` (e.g. https://schnack.example.com/auth/github/callback)

```json
// schnack.json
{
    "plugins": {
        "auth-github": {
            "client_id": "xxxxx",
            "client_secret": "xxxxx"
        }
    }
}
```


### Facebook

If you activate the plugin `auth-facebook` users can log in using their Facebook account.

```json
// schnack.json
{
    "plugins": {
        "auth-facebook": {
            "client_id": "xxxxx",
            "client_secret": "xxxxx"
        }
    }
}
```


### Google


If you activate the plugin `auth-google` users can log in using their Google account.

```json
// schnack.json
{
    "plugins": {
        "auth-google": {
            "client_id": "xxxxx",
            "client_secret": "xxxxx"
        }
    }
}
```

## Notifications

When new comments are awaiting for approval, *schnack* can notify the administrators via **notification plugins**.

### web-push

Web-push is a [protocol](https://tools.ietf.org/html/draft-ietf-webpush-protocol-12) designed to send push notifications to browsers on mobile and desktop devices. Using a [Service Worker](https://caniuse.com/#feat=serviceworkers) it is possible to register a component which is always listening for push notifications, even when the tab or the browser are closed. This allows to send end-to-end notifications from the *schnack* server to the admins.

In order to configure web-pushes, you should follow these steps:
- Generate a VAPID key pairs using the web-push package:

```bash
node_modules/.bin/web-push generate-vapid-keys
```

- Copy the VAPID keys in `schnack.json`
```json
// schnack.json
{
    "plugins": {
        "notify-webpush": {
            "vapid_public_key": "xxxxx",
            "vapid_private_key": "xxxxx"
        }
    }
}
```
- Add your user ID to the *admin* array in `schnack.json`
- Copy the [sw.js](https://github.com/schn4ck/schnack/blob/master/sw.js) into your website's root path, so that this will be accessible at https://comments.example.com/sw.js
- Login to your *schnack* instance and you will be asked to grant the permission for push notifications.

When a new comment is posted, you will be notified with a notification. In order to avoid flooding, *schnack* will send only a notification every 5 minutes, highlighting the number of comments awaiting for approval.

We strongly reccommend to subscribe to push notifications using Chrome on your mobile device.



### Slack

*schnack* can also send a message to a slack channel when a new comment is awaiting for approval. In order to configure this service just create a slack [webhook](https://api.slack.com/incoming-webhooks) and paste its URL to `notify.slack.webhook_url` in `schnack.json`.

```json
// schnack.json
{
    "plugins": {
        "notify-slack": {
            "webhook_url": "xxxxx"
        }
    }
}
```

### PushOver

[PushOver](https://pushover.net/) is a service to send notifications to your devices. You can use it to receive *schnack* notifications. In order to configure it you should first register for an account and download a [client](https://pushover.net/clients). Then you can create an app and copy the token and the key to `notify.pushover.app_token` and `notify.pushover.user_key` in `schnack.json`.

```json
// schnack.json
{
    "plugins": {
        "notify-pushover": {
            "app_token": "xxxxx",
            "user_key": "xxxxx"
        }
    }
}
```

### sendmail

If you configure the plugin `notify-sendmail` *schnack* will use your web servers `sendmail` program to send an email to you every time a comment is awaiting your approval. Note that you may need to [configure sendmail on your server](https://duckduckgo.com/?q=configure+sendmail&) before this is working.

```json
// schnack.json
{
    "plugins": {
        "notify-sendmail": {
            "to": "your-email@example.com",
            "from": "schnack@blog.example.com"
        }
    }
}
```

### RSS

If none of the notification services fits your needs, you can still use the RSS feed provided at `/feed` to stay updated about the latest comments posted. You can also use this endpoint in combination with services like [IFTTT](https://ifttt.com).

## Localization

The language used in the Schnack form and comments ("Send comment", "Reply", etc.) can now be customized! Just pass any wording you want to change as `data-schnack-partials-` attribute to the script tag:

```html
<script src="https://comments.yoursite.com/embed.js"
    data-schnack-slug="post-slug"
    data-schnack-target=".comments-go-here"
    data-schnack-partials-reply="Reply to this comment"
    data-schnack-partials-sign-in-via="To post a comment sign in via">
</script>
```

Here's a list of all the supported attributes (make sure to add `data-schnack-partial-` in to the keys:

| key    | default value                 |
|---------------|---------------------|
| `sign-in-via`  | _To post a comment you need to sign in via_ |
| `login-status`  | _signed in as @%USER% (&lt;a class='schnack-signout' href='#'&gt;sign out&lt;/a&gt;)_ |
| `or`  | <i>or</i> |
| `edit`  | _Edit_ |
| `preview`  | _Preview_ |
| `cancel`  | _Cancel_ |
| `reply`  | _Reply_ |
| `send-comment`  | _Send comment_ |
| `post-comment`  | _Write your comment here (feel free to use markdown). Please be nice :)_ |
| `mute`  | _mute notifications_ |
| `unmute`  | _unmute_ |
| `admin-approval`  | _This comment is still waiting for your approval_ |
| `waiting-for-approval`  | _Your comment is still waiting for approval by the site owner_ |


# Administration

Administrators are managed adding or removing their *schnack* user ID to the `admins` array in `config.js`. When a user logs in as administrator, the moderation UI will appear in the certain comment section. By default it's set to `[1]`. So the first user will be an admin.

## Moderation

It is possible to approve or reject single comments, but it is also possible to trust or block a certain user.
An approved comment will be displayed to all users visiting your site, while a rejected comment will be deleted. Comments posted by a trusted users are approved automatically, while comments posted by blocked users will be automatically deleted.

*schnack* also prevents users from posting the same comment twice.

## Trust your friends

You can provide a list of user IDs of people you trust for each authentication provider. For instance, you could use the Twitter API to [get a list of all the people you follow](https://apigee.com/console/twitter?req=%7B%22resource%22%3A%22friends_ids%22%2C%22params%22%3A%7B%22query%22%3A%7B%22stringify_ids%22%3A%22true%22%2C%22cursor%22%3A%22-1%22%7D%2C%22template%22%3A%7B%7D%2C%22headers%22%3A%7B%7D%2C%22body%22%3A%7B%22attachmentFormat%22%3A%22mime%22%2C%22attachmentContentDisposition%22%3A%22form-data%22%7D%7D%2C%22verb%22%3A%22get%22%7D) and drop that into the `trust.twitter` in `config.js`.

```json
"trust": {
	"twitter": [
		"916586732845400064",
		"902094599329591296"
	],
	"github": [
		1639, 2931, 2946, 3602, 4933
	]
}
```

## Backups

The most effective way to keep a backup of your data is to take a copy of your `comments.db` file, which is actually including all necessary data. If you cannot find this file then you probably set another name to `database.comments` in `schnack.json`.

# Import comments

It is possible to import [disqus XML export](https://help.disqus.com/customer/portal/articles/472149-comments-export) and [Wordpress XML export](https://en.blog.wordpress.com/2006/06/12/xml-import-export/).
In order to be able to import your comments, a database should already exist and schnack shouldn't be running.
The importer requires **Node.js >= v9**.

## Wordpress

You can export your data from Wordpress following [this guide](https://en.blog.wordpress.com/2006/06/12/xml-import-export/). Then you can run the following to import the comments into schnack's database:

```bash
npm run import -- wordpress.xml
```

## Disqus

You can [export](https://help.disqus.com/customer/portal/articles/472149-comments-export) your disqus comments and import them into schnack running:

```bash
npm run import -- disqus.xml
```

# Docker

You can build a Docker image for the schnack server running:

```bash
docker build -t schn4ck/schnack .
```

The image will contain everything in the project folder and can be started with:

```bash
docker run -p 3000:3000 -d schn4ck/schnack
```

In order to be able to edit your config file and your SQL database files, you may want to share the project folder with the docker container:

```bash
docker run -p 3000:3000 -v $(pwd):/usr/src/app -d schn4ck/schnack
```
# Try schnack

Here you can leave us some comments. Let us know what you think about schnack!
