# Terraform Cloud (HCP Terraform) Monorepo Example

A minimal monorepo layout showing how to connect multiple environments
in one repo to separate HCP Terraform workspaces.

## Layout

```
.
├── modules/
│   └── vpc/                  # shared module, used by both environments
├── environments/
│   ├── dev/                  # maps to HCP Terraform workspace "dev"
│   └── prod/                 # maps to HCP Terraform workspace "prod"
```

Each environment directory is a **fully standalone Terraform root module**
with its own `cloud` block — this is what HCP Terraform's VCS-driven
workflow expects (plain Terraform, not Terragrunt).

## One-time setup in HCP Terraform

1. **Create two workspaces**, e.g. `myorg-dev` and `myorg-prod`, using the
   "Version control workflow" and connecting both to this same repo.
2. For each workspace, set **Terraform Working Directory**:
   - `myorg-dev`  → `environments/dev`
   - `myorg-prod` → `environments/prod`
3. Update the `organization` and `workspaces.name` values in each
   `environments/*/main.tf` to match what you created.

## Making the shared module trigger both workspaces

By default, HCP Terraform only re-runs a workspace when files change
*inside its working directory*. A change to `modules/vpc` won't trigger
`environments/dev` or `environments/prod` unless you tell it to.

In each workspace's **Settings → Version Control → Automatic Run Triggering**
(or the newer VCS trigger patterns), add:

```
modules/vpc/**/*
```

Now a commit touching the shared module will trigger runs in both
environments, in addition to changes made directly in their own directories.

## Running locally (optional)

Since these use the `cloud` block, you can still run Terraform locally and
have it execute (or stream) through HCP Terraform:

```bash
cd environments/dev
terraform login      # once, to authenticate the CLI
terraform init
terraform plan
```

## Notes

- This structure is **plain Terraform**, not Terragrunt. If you're using
  Terragrunt, HCP Terraform's native VCS workflow won't process
  `generate`/`dependency` blocks — see the earlier discussion on running
  `terragrunt run-all` from your own CI instead, with these `cloud` blocks
  generated dynamically by Terragrunt rather than committed as static files.
- Workspace execution mode is left as default (**Remote**). Switch to
  **Local** execution mode in workspace settings if you want HCP Terraform
  to act purely as a state backend while runs happen in your own CI.
