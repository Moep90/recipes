# Security

Report a vulnerability in a recipe or in `check` privately through
[GitHub private vulnerability reporting](https://github.com/kadet-hub/recipes/security/advisories/new).
Do not open a public issue for it. Without a GitHub account, write to
security@kadet-hub.org.

A vulnerability in the software a recipe deploys, such as a pinned image or
the chart the recipe was derived from, belongs to that project. Report it
there.

## Supported versions

Fixes go into the newest release of each recipe, tagged `<name>-vX.Y.Z`,
and of `check`, tagged `check-vX.Y.Z`. A recipe without a release is fixed
on `main`.

## Handling

kadet-hub has one maintainer, so reports are handled on a best-effort
basis, without a service level or a promised response time. A fixed
vulnerability is published as a
[GitHub security advisory](https://github.com/kadet-hub/recipes/security/advisories)
with the affected versions and a CVSS score.
