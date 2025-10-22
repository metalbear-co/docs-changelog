# mirrord docs: changelog

This repo contains the source for the mirrord docs site [Changelog tab](https://metalbear.com/mirrord/docs/changelog). It uses [GitBook](https://gitbook.com/) to generate documentation from markdown.

> **NOTE** that the mirrord docs' other tabs are synced to the [docs](https://github.com/metalbear-co/docs) and [docs-configuration](https://github.com/metalbear-co/docs-configuration) repos.

### Structure

The root of the GitBook docs is `./changelog`, as defined in `.gitbook.yaml`. See [the GitBook docs](https://gitbook.com/docs/getting-started/git-sync/content-configuration) for more info.

### Development

**When you open a PR with changes**, GitBook will publish a preview and display the link under the `Checks` section at the bottom of the page that will show your changes - it does so via the [GitBook bot](https://github.com/gitbook-bot) installed in this repo. Use the preview labelled "*GitBook - metalbear.com*".

Changes to **content and basic structure** are possible in the repo, and other changes must be made via the [GitBook platform](https://app.gitbook.com). To do so, you will need access to the MetalBear organisation.

When **changing the structure** of the docs, like creating a new page, you must update `./changelog/SUMMARY.md` to reflect this. New folders should have a `README.md` as their overview page.

### GitBook

Each tab on the docs site corresponds to a GitBook "space". Adding a space is done through GitBook, and GitHub sync must be set up to sync changes between the space's repo and the website. Note that changes made in GitBook may be live.
