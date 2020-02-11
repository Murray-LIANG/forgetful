# forgetful

This repo is used to generate the site hosted on GitHub Pages.

It is a Hugo site with `LoveIt` theme.

The markdown documents are put under `content/posts`.

Because GitHub Pages only supports building site from `master` branch or `docs` folder of `master` branch, `hugo` generates the site and puts files under `docs`.

## Steps to update GitHub Pages
1. Add `.md` file to `content/posts`.
2. Run `hugo --theme LoveIt -d docs`.
3. Push the new files generated to GitHub.
4. TODO: enable Travis to do this automatically.

## Notes
1. The `baseURL` in `config.toml` is set to `baseURL = "https://murray-liang.github.io/forgetful"`.
2. GitHub Pages of `forgetful` repo is set to built from `master branch /docs folder`.
