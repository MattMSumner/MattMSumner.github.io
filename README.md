# Appalling Farrago

Source for [appallingfarrago.com](https://appallingfarrago.com/), a personal
blog about Ruby, Ember, Scala and whatever else I felt like writing about
(2013–2019).

## Technology

| Piece | What's used | Where |
| --- | --- | --- |
| Site generator | [Jekyll](https://jekyllrb.com/) 3.8.5, pulled in by the [`github-pages`](https://github.com/github/pages-gem) gem (v198) | `Gemfile`, `Gemfile.lock` |
| Hosting | [GitHub Pages](https://pages.github.com/) user site (`MattMSumner.github.io`) | — |
| Custom domain | `appallingfarrago.com` | `CNAME` |
| Ruby | 2.6.1 | `.ruby-version` |
| Markdown | kramdown (the GitHub Pages default) | — |
| Syntax highlighting | [Rouge](https://github.com/rouge-ruby/rouge) | `_config.yml`, `_sass/_syntax.scss` |
| Styles | Sass, compiled by Jekyll, with vendored copies of thoughtbot's [Bourbon](https://www.bourbon.io/), [Neat](https://neat.bourbon.io/) and Bitters-style base styles | `css/app.scss`, `_sass/` |
| Fonts | Google Fonts (Source Sans Pro, Poly, Anonymous Pro) | `css/app.scss` |
| Plugins | `jekyll-redirect-from` (keeps old post URLs working) | `_config.yml` |
| RSS | Hand-written Liquid template | `feed.xml` |
| Analytics | Google Analytics (Universal Analytics, `UA-44510916-1`) | `_layouts/default.html` |

There's no JavaScript build, database or server code. It's a plain static
Jekyll site.

## Layout of the repo

```
_config.yml         Site name, description, URL, highlighter, plugins
_layouts/
  default.html      Page shell: header, footer, CSS, analytics
  post.html         Wraps a single post (title + date + content)
_posts/             One Markdown file per post: YYYY-MM-DD-slug.markdown
_sass/              Sass partials (site styles + vendored Bourbon/Neat)
css/app.scss        Sass entry point; Jekyll compiles it to /css/app.css
img/                Images referenced from posts
index.html          Home page: lists every post with its excerpt
feed.xml            RSS feed of the 10 most recent posts
CNAME               Custom domain for GitHub Pages
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

```sh
bundle install
bundle exec jekyll serve --livereload
```

Then open <http://localhost:4000>. The generated site goes to `_site/`, which
is gitignored.

**Expect some friction after 7 years.** The lockfile pins Ruby 2.6 era gems,
for example `nokogiri 1.10.3` and `github-pages 198`. These may not compile on
a current Ruby or on Apple Silicon. The easiest fix is usually to refresh the
lockfile:

```sh
bundle update github-pages
```

GitHub Pages ignores `Gemfile.lock` anyway (see below). Updating it locally
just brings your preview closer to what production actually builds.

## Deploying

Deployment is just a push:

```sh
git push origin master
```

GitHub Pages builds every push to `master` with its own Jekyll and publishes
the result to `appallingfarrago.com`. There's no CI config and no build step
to run yourself. Build status and errors show up under the repo's
**Settings → Pages** and in the **Actions** tab on GitHub.

A few things to know:

- GitHub builds with **its own** pinned versions of Jekyll and plugins, listed
  at <https://pages.github.com/versions/>. Your `Gemfile.lock` is not used.
- Only plugins on the GitHub Pages
  [allow-list](https://pages.github.com/versions/) work.
  `jekyll-redirect-from` is on it.
- The `CNAME` file controls the custom domain. Don't delete it.

## Known issues / housekeeping

Things I noticed while writing this README, not yet fixed:

- **Google Analytics is dead.** Universal Analytics (`UA-…` IDs) stopped
  collecting data in 2023. Either remove the snippet from
  `_layouts/default.html` or replace it with GA4 or another analytics tool.
- **Mixed-content font imports.** `css/app.scss` loads Google Fonts over
  `http://`, which browsers block on an HTTPS page. Switch them to `https://`.
- **`site.url` is `http://www.…`** in `_config.yml`, but the site is served
  from `https://appallingfarrago.com/`. That affects absolute links in
  `feed.xml`.
- **Double slashes in the RSS feed.** `feed.xml` builds links as
  `{{ site.url }}/{{ post.url }}`, but both pieces already contain a slash.
  `jekyll-feed`, which GitHub Pages supports, could replace the hand-written
  feed.
- **Old Ruby.** `.ruby-version` pins 2.6.1, which reached end of life in 2022.
