# Releases

Configure the npm package's Trusted Publisher settings for GitHub Actions:

- Owner: rbanis92
- Repository: vue3-select2
- Workflow: publish.yml
- Environment: leave empty
- Allow direct publishing with npm publish.

No npm token is required in GitHub. See https://docs.npmjs.com/trusted-publishers/.

Before each release, check published versions with `npm view vue3-select2 versions --json`, update package.json and package-lock.json together with `npm version patch --no-git-tag-version`, install dependencies, and run `npm run build:publish`. The legacy Vue CLI build may require `NODE_OPTIONS=--openssl-legacy-provider`.

Inspect `npm pack --dry-run`, commit and push to main, then create a GitHub release with tag `v` followed by the exact package version. Publishing a release triggers npm publishing. Do not create a release for a version already present on npm.

The migration itself does not publish a package. Keep the npm package name vue3-select2.
