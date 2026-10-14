# Appalling Farrago

Source for [appallingfarrago.com](https://appallingfarrago.com/), a personal
blog about Ruby, Ember, Scala and whatever else I felt like writing about
(2013–2019).

## Technology

| Piece | What's used | Where |
| --- | --- | --- |
| Site generator | [Jekyll](https://jekyllrb.com/) 4.4 | `Gemfile`, `Gemfile.lock` |
| Ruby | 4.0.7 | `.ruby-version` |
| Build & deploy | GitHub Actions, publishing to [GitHub Pages](https://pages.github.com/) | `.github/workflows/pages.yml` |
| Hosting | GitHub Pages user site (`MattMSumner.github.io`) | — |
| Custom domain | `appallingfarrago.com`, DNS on Cloudflare (DNS only, not proxied) | Pages settings, `CNAME` |
| Markdown | kramdown with GFM input (Jekyll's default) | — |
| Syntax highlighting | [Rouge](https://github.com/rouge-ruby/rouge) | `_config.yml`, `_sass/_syntax.scss` |
| Styles | Sass, compiled by Jekyll with Dart Sass, using vendored copies of thoughtbot's [Bourbon](https://www.bourbon.io/), [Neat](https://neat.bourbon.io/) and Bitters-style base styles | `css/app.scss`, `_sass/` |
| Fonts | Google Fonts (Source Sans Pro, Poly, Anonymous Pro) | `css/app.scss` |
| Plugins | `jekyll-redirect-from` (keeps old post URLs working) | `_config.yml` |
| RSS | Hand-written Liquid template | `feed.xml` |

There's no JavaScript build, database or server code. It's a plain static
Jekyll site.

## Layout of the repo

```
_config.yml         Site name, description, URL, plugins, Sass options
.github/workflows/
  pages.yml         Builds the site and deploys it to GitHub Pages
_layouts/
  default.html      Page shell: header, footer, CSS
  post.html         Wraps a single post (title + date + content)
_posts/             One Markdown file per post: YYYY-MM-DD-slug.markdown
_sass/              Sass partials (site styles + vendored Bourbon/Neat)
css/app.scss        Sass entry point; Jekyll compiles it to /css/app.css
img/                Images referenced from posts
index.html          Home page: lists every post with its excerpt
feed.xml            RSS feed of the 10 most recent posts
CNAME               Custom domain (informational; see Deploying)
```

## Writing a post

Create `_posts/YYYY-MM-DD-some-slug.markdown` with front matter like this:

```yaml
---
layout: post
title: "My new post"
date: 2026-10-14 10:00:00
categories: ruby rails
excerpt: One sentence shown on the home page.
---

Post body in Markdown...
```

- `excerpt` is optional. Without it, Jekyll uses the first paragraph.
- Post URLs use Jekyll's default `date` permalink style,
  `/:categories/:year/:month/:day/:title.html`. That means changing a post's
  categories or filename changes its URL. To keep an old URL working, add it
  under `redirect_from:` in the front matter. See
  `_posts/2014-12-07-ember-cli-torii-rails.markdown` for an example.
- Put images in `img/` and reference them as `/img/filename.png`.

## Running locally

Ruby is managed with [mise](https://mise.jdx.dev/), which installs the version
in `.ruby-version`. mise only reads `.ruby-version` once that's enabled, so on
a new machine run:

```sh
brew install mise
echo 'eval "$(mise activate zsh)"' >> ~/.zshrc   # or your shell's equivalent
mise settings add idiomatic_version_file_enable_tools ruby
mise install          # installs the Ruby from .ruby-version
```

Then, in this repo:

```sh
bundle install
bundle exec jekyll serve --livereload
```

Open <http://localhost:4000>. The generated site goes to `_site/`, which is
gitignored.

The build prints a lot of Sass deprecation warnings from the vendored
Bourbon/Neat code. They're harmless for now (see Known issues).

To update dependencies, run `bundle update`. Bump `.ruby-version` to change
Ruby; CI picks it up from that file.

## Deploying

Deployment is just a push:

```sh
git push origin master
```

`.github/workflows/pages.yml` builds the site with the exact gems in
`Gemfile.lock` (with `JEKYLL_ENV=production`) and deploys it to GitHub Pages.
Pull requests run the same build without deploying, so a broken build shows up
on the PR. Progress and errors are in the repo's **Actions** tab. You can also
re-deploy by hand from there with **Run workflow**.

A few things to know:

- **Settings → Pages → Source** must be **GitHub Actions**. The old
  "Deploy from a branch" mode would ignore the workflow and build with GitHub's
  own Jekyll 3.
- Any Jekyll plugin works, since we're not limited to GitHub's allow-list.
  Add it to the `:jekyll_plugins` group in the `Gemfile` and to `plugins:` in
  `_config.yml`.
- The custom domain is configured in **Settings → Pages**. With Actions
  deploys, GitHub doesn't read the `CNAME` file, but it's harmless and documents
  the domain.
- DNS lives in Cloudflare and points straight at GitHub: four `A` and four
  `AAAA` records for the apex, and `www` as a `CNAME` to
  `mattmsumner.github.io`. All are **DNS only** (grey cloud). Turning on
  Cloudflare's proxy stops GitHub from renewing the HTTPS certificate.

## Known issues / housekeeping

- **Sass deprecations.** The vendored Bourbon 4 / Neat 1 code uses Sass
  features (`@import`, `/` division, global functions like `lighten()`) that
  Dart Sass 3 will remove. It compiles fine on Dart Sass 1.x today. The
  long-term fix is replacing Bourbon/Neat with plain CSS, since modern CSS
  covers almost everything they provide.
