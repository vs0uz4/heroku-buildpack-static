# heroku-buildpack-static
This is a discovery project for using a buildpack for handling static sites and single page web apps (I'm looking at you Sam Phippen).

## Features
* serving static assets
* gzip on by default
* error/access logs support in `heroku logs`
* custom [configuration](#configuration)

## Deploying
The directory structure expected is that you have a `public_html/` directory containing all your static assets.

1. Set the app to this buildpack: `$ heroku buildpacks:set git://github.com/hone/heroku-buildpack-static.git`.
2. Deploy: `$ git push heroku master`

### Configuration
You can configure different options for your static application by writing a `static.json` in the root folder of your application.

#### Root
This allows you to specify a different asset root for the directory of your application. For instance, if you're using ember-cli, it naturally builds a `dist/` directory, so you might want to use that intsead.

```json
{
  "root": "dist/"
}

```

By default this is set to `public_html/`

#### Clean URLs
For SEO purposes, you can drop the `.html` extension from URLs for say a blog site. This means users could go to `/foo` instead of `/foo.html`.


```json
{
  "clean_urls": true
}
```

By default this is set to `false`.

#### Custom Routes
You can define custom routes that combine to a single file. This allows you to preserve routing for a single page web application. The following operators are supported:

* `*` supports a single path segment in the URL. In the configuration below, `/baz.html` would match but `/bar/baz.html` would not.
* `**` supports any length in the URL.  In the configuration below, both `/route/foo` would work and `/route/foo/bar/baz`.

```json
{
  "routes": {
    "/*.html": "index.html",
    "/route/**": "bar/baz.html"
  }
}
```

#### Custom Redirects
With custom redirects, you can move pages to new routes but still preserve the old routes for SEO purposes. By default, we return a `301` status code, but you can specify the status code you want.

```json
{
  "redirects": {
    "/old/gone/": {
      "url": "/",
      "status": 302
    }
  }
}
```

#### Custom Error Pages
You can replace the default nginx 404 and 500 error pages by defining the path to one in your config.

```json
{
  "error_page": "errors/error.html"
}
```

#### Proxy Backends
For single page web applications like Ember, it's common to back the application with another app that's hosted on Heroku. The down side of separating out these two applications is that now you have to deal with CORS. To get around this (but at the cost of some latency) you can have the static buildpack proxy apps to your backend at a mountpoint. For instance, we can have all the api requests live at `/api/` which actually are just requests to our API server.

```json
{
  "proxies": {
    "/api/": {
      "origin": "https://hone-ember-todo-rails.herokuapp.com/"
    }
  }
}
```
