# p-shared-workflow-example

Demo layout for central, reusable GitHub Actions. Nothing here runs yet.

## Layout

```
workflows/
  ecr-retag.yml                 # Reusable workflow: validate tag -> copy manifest -> verify
actions/
  ecr-retag/
    ecr-copy-manifest/action.yml  # Composite action: retag an ECR image without pulling it
    ecr-verify-tag/action.yml     # Composite action: print digest/push time to job summary
```

- **Reusable workflows** hold whole jobs. Consumers call them in place of their own `jobs:`.
- **Composite actions** are single building blocks. The reusable workflows use them, and consumers can also use them directly as steps.

## Consuming

Call the workflow from a job in the consuming repo (see [p-parent-workflow-example-](https://github.com/mattmoulds/p-parent-workflow-example-)):

```yaml
jobs:
  promote:
    uses: org/p-shared-workflow-example/.github/workflows/ecr-retag.yml@v1
    with:
      repository: my-repo
      source_tag: abc123
      target_tag: v1.2.3
      role_to_assume: arn:aws:iam::111111111111:role/ImagePublishRole
    secrets:
      aws_access_key_id: ${{ secrets.ACCESS_KEY_ID }}
      aws_secret_access_key: ${{ secrets.SECRET_ACCESS_KEY }}
```

Or use one of the actions as a step:

```yaml
- uses: org/p-shared-workflow-example/actions/ecr-retag/ecr-verify-tag@v1
  with:
    repository: my-repo
    tag: v1.2.3
```

## Demo caveats

- `org/` is a placeholder for the real GitHub org.
- GitHub only lets other repos call workflows from `.github/workflows/`. The workflow sits in `workflows/` here to keep it readable.
- Consumers pin to `@v1`, which would be a release tag on this repo. No tag exists yet.
