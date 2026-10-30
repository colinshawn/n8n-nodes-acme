# Contributing

## Local setup

Use Node.js 24 (`nvm use`) and install the locked dependencies:

```bash
npm ci
npm run build
npm run dev
```

## Project layout

```text
BaseAction.ts          Shared execution context, parameter access, and wait helper
nodes/
  ACME/                ACME node, account and order actions, and challenge actions
  Certificate/         Certificate node, key generation, CSR, and parsing actions
  DNS/                 DNS node, TXT queries, and propagation polling
```

Each `*.node.ts` file declares the node's parameters and dispatches the selected operation. Each `*.action.ts` file contains an operation or the common helpers for its node. New operations should follow this structure and be registered in the corresponding node definition.

## Formatting and validation

Use UTF-8, LF line endings, and tabs with a display width of two for TypeScript and JSON, consistent with `.editorconfig` and `.prettierrc.js`. Markdown and YAML use spaces.

Run the checks relevant to your change:

```bash
npm run format:check
npm run lint
npm run build
```

Use `npm run format` or `npm run lint:fix` to apply available fixes, then review the diff. The repository does not currently define an automated test script. Report existing check failures separately from new failures introduced by a change.

For changes to issuance behavior, validate a representative workflow against the Let's Encrypt staging endpoint and include the result in the pull request. Never commit real private keys, account details, environment secrets, or sensitive workflow exports.

## Dependencies and packaging

Use npm and commit `package-lock.json` alongside dependency changes. Generated files under `dist/` are ignored by Git; `npm run build` recreates the code and icons used by the package.

The `files` allowlist in `package.json` includes compiled JavaScript, source maps, type declarations, JSON files, and icons from `dist/`. It excludes TypeScript build caches and certificate material. npm also includes package metadata, the README, and the license. Preview the package without publishing:

```bash
npm run build
npm pack --dry-run --ignore-scripts
```

The release and prepublish scripts are publication steps, so they are not part of routine local validation.
