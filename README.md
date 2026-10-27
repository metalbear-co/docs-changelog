# mirrord docs: changelog

This repo contains the source for the mirrord docs site [Changelog tab](https://metalbear.com/mirrord/docs/changelog). It uses [GitBook](https://gitbook.com/) to generate documentation from markdown.

> **NOTE** that the mirrord docs' other tabs are synced to the [docs repo](https://github.com/metalbear-co/docs) and [the mirrord repo docs branch](https://github.com/metalbear-co/mirrord/tree/docs).

### Structure

The root of the GitBook docs is `./changelog`, as defined in `.gitbook.yaml`. See [the GitBook docs](https://gitbook.com/docs/getting-started/git-sync/content-configuration) for more info.

### Development

**When you open a PR with changes**, GitBook will publish a preview and display the link under the `Checks` section at the bottom of the page that will show your changes - it does so via the [GitBook bot](https://github.com/gitbook-bot) installed in this repo. Use the preview labelled "*GitBook - metalbear.com*".

Changes to **content and basic structure** are possible in the repo, and other changes must be made via the [GitBook platform](https://app.gitbook.com). To do so, you will need access to the MetalBear organisation.

When **changing the structure** of the docs, like creating a new page, you must update `./changelog/SUMMARY.md` to reflect this. New folders should have a `README.md` as their overview page.

### GitBook

Each tab on the docs site corresponds to a GitBook "space". Adding a space is done through GitBook, and GitHub sync must be set up to sync changes between the space's repo and the website. Note that changes made in GitBook may be live.

### Changelog publication

The Changelog space contains four sibling pages. Operator keeps the space's root page and existing route; the other histories are separate pages in the same space.

| Component | Source repository and file | Website page |
| --- | --- | --- |
| Operator | `metalbear-co/operator`, `public/CHANGELOG.md` | [Operator](changelog/README.md) |
| mirrord | `metalbear-co/mirrord`, `CHANGELOG.md` | [mirrord](changelog/mirrord.md) |
| VS Code | `metalbear-co/mirrord-vscode`, `CHANGELOG.md` | [VS Code](changelog/mirrord-vscode.md) |
| IntelliJ | `metalbear-co/mirrord-intellij`, `CHANGELOG.md` | [IntelliJ](changelog/mirrord-intellij.md) |

These pages retain the complete source histories, including the CLI releases inherited by the IDE plugins. `down` is outside this publication scope and has no changelog.

The shared [release-changelog-docs action](https://github.com/metalbear-co/release-changelog-docs) owns source resolution, checkout, public rendering and publication. Every source repository calls the same action at a reviewed full commit SHA. Source repositories own their changelogs, release timing and `.github/workflows/changelog-update.yaml`; this repository owns the website pages and navigation. Successful release jobs pass the exact release checkout SHA to the publisher, without a cross-repository workflow dispatch. The action removes Internal sections and known private Operator issue references while preserving public text and links. Its README documents inputs, metadata, safeguards and maintenance checks.

To recover a failed website update, run the source repository's manual `changelog-update.yaml` workflow. With `ref` empty, it selects GitHub's latest published non-prerelease release and resolves its tag to a commit once; a missing release or ref fails instead of falling back to `main`. An explicit `ref` selects a commit, tag or branch for recovery. The publisher skips older versions, so this cannot roll back a newer website history. The product release remains valid when only docs publication fails; recover the docs update without rerunning product publication.

Identical refreshes preserve the page and its `lastmod`, even on a later day. Changed content at the same version requires matching source provenance in the latest page commit. Existing pages without that provenance need a separately reviewed page correction or a newer release before a changed same-version refresh can succeed. Fix source release notes in their owning repository and refresh from the intended source; unexplained website edits can block a later refresh.

### Publisher rollout

The local implementation alone does not activate publication. Before enabling the source callers:

1. Review and publish the shared action, then pin every caller to the full SHA of that available revision. The final action SHA must be bound after review; an existing base revision does not establish action availability.
2. Confirm source `contents: read` access, organization Actions policy and permission to use the shared action. Operator uses its existing Cubby GitHub App credentials `CUBBY_CLIENT_ID` and `CUBBY_PRIVATE_KEY` to issue a `docs-changelog` installation token; each other source retains `CHANGELOG_PAT`. Confirm each destination token can write contents to `metalbear-co/docs-changelog` and that branch policies permit the publisher's non-force direct pushes to `main`. Secret names alone do not establish grants or usability, and absence from repository secret lists does not rule out organization or environment secrets.
3. Migrate Operator off its old writer and verify its refresh before activating the other publishers. The old writer lacks the shared action's collision safeguards. Land the new pages and SUMMARY entries together, then use the GitBook preview described above to verify all four labels, their order, full histories and the unchanged Operator route. Confirm hosted workflow execution and live GitBook sync during rollout.

Publication changes only the selected component page and retries competing writes without force. A same-version conflict or rejected write remains a visible failure to investigate. See the action README for the detailed recovery and testing contract.
