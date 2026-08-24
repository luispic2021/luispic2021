# Luispe.me

Personal website for Luis P. Perez, a technical product leader working across AI/ML, data products, compliance and risk, and fintech. The site is a static Jekyll project published through GitHub Pages at [luispe.me](https://luispe.me).

## Project structure

- `index.html` — main portfolio page
- `cv.md` — detailed experience profile at `/cv/`
- `posts.html` and `_posts/` — post index and dated posts
- `_layouts/` and `_includes/` — shared page structure
- `assets/css/style.scss` — site design system and responsive styles
- `assets/js/site.js` — accessible mobile navigation
- `assets/images/` — site and post imagery

## Setup

1. Install a Ruby version compatible with GitHub Pages.
2. Install Bundler: `gem install bundler`
3. Install dependencies: `bundle install`
4. Start the local site: `bundle exec jekyll serve`
5. Open `http://127.0.0.1:4000`.

The repository contains no required secrets. If future integrations require credentials, store them in a local `.env` file; `.env` is ignored by Git.

## Usage

- Edit homepage copy and sections in `index.html`.
- Add a post as `_posts/YYYY-MM-DD-title.md` with `layout`, `title`, `date`, and `categories` in its front matter.
- Edit site-wide metadata and social links in `_config.yml`.
- Keep `CNAME` unchanged so the GitHub Pages custom domain continues to resolve.

## Expected output

`bundle exec jekyll build` generates a static `_site/` directory containing the homepage, experience page, post index, individual posts, feed, sitemap, and 404 page.

## Troubleshooting

- **Bundler version error:** install the Bundler version shown at the bottom of `Gemfile.lock`, or run `bundle update --bundler` intentionally and commit the lockfile change.
- **Styles are missing:** confirm the generated page links to `/assets/css/style.css` and restart Jekyll after editing `_config.yml`.
- **Custom domain is missing:** verify `CNAME` contains only `luispe.me`.
- **GitHub Pages build differs locally:** use the repository’s locked `github-pages` dependency instead of a globally installed Jekyll version.
