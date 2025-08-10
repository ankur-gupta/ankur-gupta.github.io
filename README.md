# ankur-gupta.github.io

[http://ankur-gupta.github.io](http://ankur-gupta.github.io) or
[https://www.perfectlyrandom.org](https://www.perfectlyrandom.org).

This website is made using a Jekyll theme Laplacian (https://github.com/ankur-gupta/laplacian),
which in turn, is based on the Lagrange theme (https://lenpaul.github.io/Lagrange/).

## Jekyll Build & Deployment

| Environment       | Gemfile Required? | Notes                          |
| ----------------- | ----------------- | ------------------------------ |
| GitHub Actions    | No                | Uses built-in Jekyll + plugins |
| Local Development | Yes               | Use `bundle` to manage gems    |

### GitHub Actions

- Uses the official [`actions/jekyll-build-pages`](https://github.com/actions/jekyll-build-pages) action.
- **No `Gemfile` needed** — GitHub Actions uses a preconfigured Jekyll environment with supported plugins built-in (you do get a build warning which you can ignore for now).
- The workflow builds and deploys your site automatically on push.

### Local Development

- Requires a `Gemfile` to specify Jekyll and plugin versions for consistent builds.
- To set up locally:
  1. Create a `Gemfile` with your desired gems (e.g., `github-pages`, `jekyll-paginate`).
  2. Run `bundle install` to install dependencies.
  3. Use `bundle exec jekyll serve` to build and preview your site locally.

## License

All original content (including text and images) belongs to Ankur Gupta and is not licensed for reproduction. This original content includes these files and folders:

- `_posts`
- `_drafts`
- `assets`

The code for the layout and design of the website are based on the Jekyll theme Trio (https://github.com/ankur-gupta/trio) and are available for reuse under the MIT License as specified in https://github.com/ankur-gupta/trio/blob/master/LICENSE.

## History

- Moved away from [Trio](https://github.com/ankur-gupta/trio) theme on April 20, 2019
- Moved away from [Pixyll theme](http://www.pixyll.com) on December 24, 2015
- Moved away from [Jekyll version of the Tactile Theme](https://github.com/ankur-gupta/jekyll-tactile-theme) on September 19, 2014
