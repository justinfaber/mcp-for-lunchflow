# Publishing guide (maintainer notes)

This is a one-time setup, then a short checklist for each release. The file is not included in the npm package.

## One-time setup

### 1. Secure your accounts

- **npm:** create an account (username `faberjustin`) at npmjs.com and turn on two-factor authentication: Account → Two-Factor Authentication, then "Authorization and writes".
- **GitHub:** make sure two-factor authentication is on.

### 1b. Keep your email out of the public history

Git history and npm metadata are permanent once public, so set this up before the first commit and publish.

- **GitHub:** Settings → Emails. Tick **Keep my email addresses private** and **Block command line pushes that expose my email**. Copy your noreply address (`<id>+justinfaber@users.noreply.github.com`), then, inside the repo folder after `git init`, run:

  ```bash
  git config user.email "<id>+justinfaber@users.noreply.github.com"
  git config user.name "Justin Faber"
  ```

- **npm:** the registry shows the npm account's email publicly in the package metadata (`maintainers`). Before publishing, set it to an address you're happy to be public, under npmjs.com → Account → Email.

### 2. Create the GitHub repository

1. Create a **public** repository named `justinfaber/mcp-for-lunchflow`. Don't add a README, license or .gitignore, because these files already include them.
2. Push the code:

   ```bash
   cd mcp-for-lunchflow
   git init -b main
   git config user.email "<id>+justinfaber@users.noreply.github.com"   # see 1b
   git add .
   git commit -m "Initial public release 1.2.0"
   git remote add origin https://github.com/justinfaber/mcp-for-lunchflow.git
   git push -u origin main
   ```

3. Check that the **CI** workflow passes under the Actions tab.

### 3. Protect the repository

In repository **Settings**:

- **Branches:** add a protection rule for `main` that requires a pull request and the CI checks to pass. You can allow yourself to bypass it if you work alone.
- **Environments:** create an environment named **`npm-publish`**.
  - Add yourself as a **required reviewer**. Every publish will then wait for your approval.
  - Under deployment branches and tags, restrict it to tags matching `v*.*.*`.
- **Code security:** enable **Private vulnerability reporting** and **Dependabot alerts**.
- **Actions → General:** set workflow permissions to "Read repository contents".

### 4. Reserve the npm name with a placeholder

npm only allows trusted publishing for a package that already exists, so the first publish has to be done by hand. Publishing a tiny placeholder means the real 1.2.0 release still gets provenance.

```bash
mkdir -p /tmp/placeholder && cd /tmp/placeholder
cat > package.json <<'EOF'
{
  "name": "@faberjustin/mcp-for-lunchflow",
  "version": "0.0.1",
  "description": "Placeholder. Use version 1.2.0 or later.",
  "license": "MIT",
  "repository": { "type": "git", "url": "git+https://github.com/justinfaber/mcp-for-lunchflow.git" }
}
EOF
echo "Placeholder release. Install 1.2.0 or later." > README.md
npm login            # opens the browser; the session lasts 2 hours
npm publish --access public
```

### 5. Connect npm to GitHub (trusted publishing)

On npmjs.com, open the package, go to **Settings → Trusted Publisher → GitHub Actions**, and enter:

| Field | Value |
|---|---|
| Organization or user | `justinfaber` |
| Repository | `mcp-for-lunchflow` |
| Workflow filename | `publish.yml` |
| Environment | `npm-publish` |
| Allowed actions | tick **npm publish**; leave dist-tag management off |

Make sure **npm publish** is ticked. Configurations created since September 2026 default to staged publishing only, and the workflow's `npm publish` would then be rejected.

You can do the same from the command line with npm 11.15 or later:

```bash
npm trust github @faberjustin/mcp-for-lunchflow --file publish.yml \
  --repo justinfaber/mcp-for-lunchflow --env npm-publish --allow-publish
```

Then, on the same Settings page under **Publishing access**, choose **"Require two-factor authentication and disallow tokens"**. Trusted publishing keeps working with this setting, and nobody can publish with a stolen token.

## Releasing a version

**First release (1.2.0):** it's already committed, so just tag it:

```bash
git tag v1.2.0
git push origin v1.2.0
```

**Later releases**, using 1.2.1 as an example:

1. Update `version` in `package.json` and add an entry to `CHANGELOG.md`.
2. Run `npm install --ignore-scripts` so `npm-shrinkwrap.json` picks up the new version, then run `npm test`.
3. Commit, tag and push:

   ```bash
   git commit -am "Release 1.2.1"
   git tag v1.2.1
   git push && git push origin v1.2.1
   ```

Use plain `vX.Y.Z` tags only. A prerelease tag like `v1.3.0-rc.1` would also trigger the workflow, and would be published as `latest` unless you add `--tag`.

Then, for any release:

1. In the Actions tab, the **Publish to npm** run waits for your approval. Review it, then approve it.
2. Check the release:
   - The npm page should show a provenance badge linking to the workflow run.
   - Run `npm view @faberjustin/mcp-for-lunchflow` to see the new version.

After the first real release, deprecate the placeholder:

```bash
npm deprecate @faberjustin/mcp-for-lunchflow@0.0.1 "Placeholder release. Use 1.2.0 or later."
```

## Dependency updates

Dependabot opens monthly pull requests for GitHub Actions and npm dependencies. For npm updates, look at the changelog of anything in the MCP SDK or zod, and let CI run before merging. Each merged update only reaches users with your next tagged release.

## If something goes wrong

- **A bad version was published:** run `npm deprecate @faberjustin/mcp-for-lunchflow@<version> "<reason>"` and release a fix. npm allows unpublishing only within 72 hours and under conditions, so deprecating is the normal route.
- **You suspect an account compromise:**
  1. Revoke npm sessions and tokens.
  2. Check the package's trusted publisher settings.
  3. Rotate your GitHub credentials.
  4. Review recent workflow runs.
