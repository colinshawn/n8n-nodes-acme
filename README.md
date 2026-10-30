# n8n-nodes-acme

[English](README.md) | [简体中文](README.zh-CN.md)

n8n community nodes for requesting TLS certificates through ACME, generating and inspecting certificate material, and checking DNS TXT records.

The package provides three nodes that can be combined into certificate issuance workflows. The ACME node offers Let's Encrypt staging and production endpoints and supports DNS-01 and HTTP-01 challenges. Staging is selected by default.

## Operations

| Node        | Operations                                                                                                                         |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| ACME        | Create Account, Create Order, Get Order, Get Challenge Key Authorization, Complete Challenge, Finalize Order, Download Certificate |
| Certificate | Generate Private Key, Generate CSR, Parse CSR, Parse Certificate                                                                   |
| DNS         | Query DNS Text Record, Wait for DNS Text Record                                                                                    |

The DNS node queries records and waits for propagation. Creating and deleting DNS records, serving HTTP challenge responses, scheduling renewals, and deploying certificates require other nodes or services in your workflow.

## Installation

The package name is `n8n-nodes-acme`. For installation in an n8n instance, follow the [n8n community node installation guide](https://docs.n8n.io/integrations/community-nodes/installation/).

For development from this repository, see [Development](#development).

## Account keys and data

The package currently has no custom n8n credential type. ACME operations accept an account private key as PEM text in the **Account Key** parameter. Reuse the same account key across operations for that account and directory endpoint.

The account key and certificate private key have separate roles: the account key identifies the ACME account, while the certificate key is used to generate the CSR and must be retained for deployment with the resulting certificate.

- **Generate Private Key** and **Generate CSR** return binary attachments in the configured output fields. Convert attachments to PEM text before passing them to parameters that expect strings.
- Orders, challenge parameters, parsed information, DNS query results, and downloaded certificate PEM text are written to the configured JSON output fields.
- **Complete Challenge**, **Finalize Order**, and **Wait for DNS Text Record** pass the input item onward after their operation succeeds.

Keep real account keys, certificate keys, and sensitive workflow exports out of source control. The repository's `.gitignore` covers common key and certificate file extensions and local n8n data directories.

## Example DNS-01 workflow

1. Generate and retain an account private key and a separate certificate private key.
2. Generate a CSR with the certificate key and the requested domain names.
3. Create an ACME account, then create an order using the account key and CSR.
4. Get the order's challenge parameters with **DNS-01** selected. Handle each returned domain challenge in the workflow.
5. Use your DNS provider's node or API to publish each challenge's `authorization` value as the appropriate `_acme-challenge` TXT record.
6. Use **Wait for DNS Text Record** for each record. It requires all configured DNS servers to return the expected value for the configured number of consecutive checks.
7. Run **Complete Challenge**, then **Finalize Order** with the same CSR.
8. Run **Download Certificate**, deploy the certificate with its private key, and remove the temporary TXT records through your DNS provider.

Use the staging endpoint while developing the workflow. Select the production endpoint when the workflow is ready to request production certificates.

## Development

The development environment uses Node.js 24, recorded in `.nvmrc`, and npm with the committed `package-lock.json`.

```bash
nvm use
npm ci
npm run build
npm run dev
```

If you do not use nvm, select Node.js 24 with your preferred version manager.

| Command                | Purpose                                                       |
| ---------------------- | ------------------------------------------------------------- |
| `npm run dev`          | Start the n8n node development environment with live rebuilds |
| `npm run build`        | Compile TypeScript and copy node assets into `dist/`          |
| `npm run build:watch`  | Watch and compile TypeScript only                             |
| `npm run lint`         | Run the existing n8n node lint rules                          |
| `npm run lint:fix`     | Apply available lint fixes                                    |
| `npm run format:check` | Check formatting with Prettier                                |
| `npm run format`       | Format supported project files with Prettier                  |

See [CONTRIBUTING.md](CONTRIBUTING.md) for the source layout and validation workflow.

## Compatibility

The package declares n8n Nodes API version 1. It does not yet declare a minimum n8n version or a tested n8n runtime version matrix. Validate your intended n8n runtime before using it for certificate automation.

## License

[MIT](LICENSE).
