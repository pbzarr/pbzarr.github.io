# pbzarr.github.io

This repository builds the pbzarr docs and publishes them to GitHub Pages at <https://pbzarr.github.io>.

The docs are written in [pbzarr/pbzarr](https://github.com/pbzarr/pbzarr): the pages are in `docs/`, and the site settings are in `zensical.toml`. Change the docs there, in the same pull request as the code change.

This repository holds only `.github/workflows/deploy.yml`. The workflow:

1. Checks out `pbzarr/pbzarr` at a branch, tag, or commit. The default is the tag of the newest release.
2. Builds the [Zensical](https://zensical.org) site with `pixi run -e docs docs-build`.
3. Publishes the site to GitHub Pages.

## Run a deploy

- From the Actions tab: run "Build and deploy docs", and give a ref if you do not want the default.
- From the command line: `gh workflow run deploy.yml -R pbzarr/pbzarr.github.io -f ref=v0.8.0`
- From another repository: send a `repository_dispatch` event of type `docs-updated`. A `client_payload.ref` value selects the ref.

The workflow has no schedule. GitHub turns off a scheduled workflow in a repository with no commits for 60 days, and a turned-off workflow also ignores dispatch events.
