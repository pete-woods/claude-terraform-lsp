# terraform-ls

A Claude Code plugin that runs HashiCorp's [terraform-ls](https://github.com/hashicorp/terraform-ls) language server for Terraform files. There's no official Terraform LSP plugin to replace.

The server starts through `go run`, so nothing needs to be installed globally:

```
go run github.com/hashicorp/terraform-ls@latest serve
```

`@latest` resolves to the latest tagged release (0.39.1 at the time of writing). Pre-releases such as `0.39.0-beta5` are excluded. Go caches the module and the build, so only the first start downloads and compiles it, which takes about 10 seconds. Later starts take under a second.

## What works

Diagnostics are pushed, so syntax and schema errors (unclosed blocks, unknown attributes, wrong label counts) show up automatically after edits. Hover, go to definition, find references, and document and workspace symbols also work.

| Extension | Language ID |
| :- | :- |
| `.tf` | `terraform` |
| `.tfvars` | `terraform-vars` |

## Requirements

- Go on your `PATH`
- Optional: `terraform` on your `PATH`. terraform-ls uses it for `terraform fmt` formatting and for validate-on-save. Navigation and diagnostics work without it.

## Install

```sh
claude plugin marketplace add ~/code/personal/claude-terraform-lsp
claude plugin install terraform-ls@claude-terraform-lsp
```

To try it without installing:

```sh
claude --plugin-dir ~/code/personal/claude-terraform-lsp
```

## Limitations

- **No call hierarchy.** terraform-ls doesn't implement `incomingCalls`/`outgoingCalls`. Use `findReferences` instead.
- **Stacks, deploy, search and policy files aren't mapped.** terraform-ls also handles `.tfcomponent.hcl`, `.tfdeploy.hcl`, `.tfquery.hcl`, `.policy.hcl` and `.policytest.hcl`. Claude Code's docs don't say whether `extensionToLanguage` can match multi-part extensions, and mapping `.hcl` would send every HCL file (Packer, Nomad and so on) to a server that, by its own docs, can't handle them.
- **No JSON configs.** terraform-ls doesn't support `.tf.json` or `.tfvars.json`.
- **Starting needs the network.** `@latest` queries the Go module proxy on every start. To work offline, pin a version in `.lsp.json`, for example `github.com/hashicorp/terraform-ls@v0.39.1`, which then starts from the module cache.
