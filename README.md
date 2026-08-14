# SecRandom Plugin Index

Official plugin market index for [SecRandom](https://github.com/SECTL/SecRandom).

## How it works

- Plugin authors publish `.srpx` releases in their own GitHub repositories.
- This repository keeps one metadata file per plugin under `plugins/<id>.yaml`.
- A workflow (`market-index.yml`) rebuilds the signed `index.json` + `index.json.sig`
  on every push and nightly, then uploads them to the fixed release tag `generated`.
- The SecRandom app downloads `index.json` + `index.json.sig` from that release
  (mirror-first), verifies the Ed25519 signature, and installs packages after a
  SHA-256 check.

## Contributing a plugin

1. Build your plugin with `CreateSrpx=true` to get `srpx/<Plugin>.srpx`.
2. Run `scripts/publish-plugin.ps1` from the SecRandom repository to generate
   `plugins/<id>.yaml` and the required SHA-256 block.
3. Create a GitHub release in your own repository with tag `v<version>`, upload the
   `.srpx` asset, and include this block in the release note:

   ```
   <!-- SECRANDOM_SHA256: <hex> -->
   ```

4. Fork this repository, add your `plugins/<id>.yaml`, and open a pull request.
5. The index workflow rebuilds and signs `index.json` automatically after merge.
   Future versions are picked up by the nightly run — no new PR needed.

## Plugin metadata format

Each plugin is one YAML file under `plugins/`, named `<id>.yaml`. `repoOwner` / `repoName`
must point at a real GitHub repository that already has a release with a single `.srpx` asset
and the SHA-256 block, otherwise the workflow skips the plugin with a warning:

```yaml
id: com.example.plugin
name: SecRandom Example Plugin
description: Example plugin metadata.
author: SECTL
version: 1.0.0
apiVersion: "3.0.0"
repoOwner: <your-github-owner>
repoName: <your-plugin-repo>
projectUrl: https://github.com/<your-github-owner>/<your-plugin-repo>
dependencies:
  - id: com.example.lib
    required: true
```

`repoOwner` / `repoName` tell the workflow where to query the latest release and its download count.
The download URL, package size, and SHA-256 are all derived from the GitHub release: `downloadUrl`/`size`
come from the `.srpx` asset, and `sha256` comes from the `<!-- SECRANDOM_SHA256: <hex> -->` block in the
release note. They are not stored in the metadata file.

## Signing

`index.json` is signed with Ed25519. The private key is stored only as the
`PLUGIN_MARKET_PRIVATE_KEY_PEM_BASE64` Actions secret; the embedded public key lives
in `SecRandom/Assets/Plugins/plugin-market-public-key.txt`. Never commit the private key.
