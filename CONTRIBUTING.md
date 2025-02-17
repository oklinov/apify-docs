# Contributing to Apify Documentation

## Architecture

Currently, there are 6 separate projects outside of this repo:

- apify-client-js
- apify-client-python
- apify-sdk-js
- apify-sdk-python
- apify-cli
- apify-docs (this repository)

The main documentation content for Platform docs and Academy is inside the `./sources` directory. Every project repository then has its own docusaurus instance and is available on a URL prefix (used as the `baseUrl` in docusaurus) that is routed via nginx reverse proxy to the main domain. All those docusaurus instances are deployed to GH pages on push.

We use a shared docusaurus theme published to NPM as `@apify/docs-theme`, that is automatically synced in all the repositories via CI.

## Local setup

If you want to work only on the main documentation content, cloning this repository is enough, Once you install and run `npm start`, the main portal will open on <http://localhost:3000>. All the links in navbar and footer need to be absolute, and they will use a different hostname, configured to `docs.apify.loc` - to use that, follow the steps below and set up the nginx server.

Alternatively, you can skip the nginx part and navigate to <http://localhost:3000/academy> or <http://localhost:3000/platform> manually instead of using links in navbar. All relative links should work fine there, the problem with absolute links is only with shared components. The nginx server is needed only for testing the whole setup and mapping all the different ports to a single one.

Clone all the repositories, checkout the `docs-v2` branch (if still not merged to `master`). Then you can start the docusaurus instances in them.

| repo                | branch  | port |
|---------------------|---------|------|
| apify-docs          | master  | 3000 |
| apify-client-js     | master  | 3001 |
| apify-client-python | docs-v2 | 3002 |
| apify-sdk-js        | master  | 3003 |
| apify-sdk-python    | docs-v2 | 3004 |
| apify-cli           | master  | 3005 |

> To run docusaurus on a specific port, use `npm start -- --port XXXX`.

To route them, you will need nginx server with following config:

```nginx
server {
  listen       80;
  server_name  docs.apify.loc;
  location / {
    proxy_pass http://localhost:3000;
  }
  location /api/client/js {
    proxy_pass http://localhost:3001;
  }
  location /api/client/python {
    proxy_pass http://localhost:3002;
  }
  location /sdk/js {
    proxy_pass http://localhost:3003;
  }
  location /sdk/python {
    proxy_pass http://localhost:3004;
  }
  location /cli {
    proxy_pass http://localhost:3005;
  }
}
```

And add a record to `/etc/hosts` to map the docs.apify.loc hostname to localhost:

```text
127.0.0.1 docs.apify.loc
```

## Deployment

Current nginx deployment config:

```nginx
server {
  listen       80;
  server_name  docs.apify.com;
  location / {
    proxy_pass https://apify.github.io/apify-docs/;
  }
  location /api/client/js {
    proxy_pass https://apify.github.io/apify-client-js/;
  }
  location /api/client/python {
    proxy_pass https://apify.github.io/apify-client-python/;
  }
  location /sdk/js {
    proxy_pass https://apify.github.io/apify-sdk-js/;
  }
  location /sdk/python {
    proxy_pass https://apify.github.io/apify-sdk-python/;
  }
  location /cli {
    proxy_pass https://apify.github.io/apify-cli/;
  }
}
```

## @apify/docs-theme

The `@apify/docs-theme` is a Docusaurus theme package with custom components and styles to be used in all the Apify Docuaurus instances.
Aside from the regular Docusaurus theme interface, it also exports the common parts of the Docusaurus config, such as the navbar contents, url, `og:image`, etc.

The theme is available on npm as `@apify/docs-theme` and can be installed in any Docusaurus instance by running `npm install @apify/docs-theme`.

### Publishing the theme

There is a GitHub Action that automatically publishes the theme to npm whenever any changes are pushed to the `master` branch. However, this only happens if you update the version in the `package.json` file manually - if the current version already exists on npm, the publish will be skipped.

Additionally, if there are any changes to the `apify-docs-theme` folder detected, the GitHub action will invoke docs builds in all the subprojects to make sure that all the pages are using the latest theme version. This is done in the `rebuild-docs` job. This job utilizes a matrix strategy to run the builds in parallel. The actual rebuild is initiated by the `workflow_dispatch` event in the respective repositories. Because of this, the `GITHUB_TOKEN` envvar has to be replaced by the PAT token stored in the `GH_TOKEN` secret - the original token does not have the necessary permissions to trigger the workflows in other repositories.

## Interesting links

- <https://github.com/facebook/docusaurus/discussions/6086>
- <https://docusaurus.io/docs/docs-multi-instance>
- <https://docusaurus.io/docs/advanced/routing#escaping-from-spa-redirects>
