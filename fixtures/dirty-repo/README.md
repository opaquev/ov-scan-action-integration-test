# dirty-repo fixture

This is the `ov-scan-action` smoke test's dirty fixture. It contains
several credential-shaped strings, all prefixed with `XXXFAKE_` per
the action's safe-prefix discipline (R3-LOW in the action's plan doc).

`ov scan` against this directory MUST find at least one finding and
exit non-zero with the action's `--fail-on high` threshold.

If `ov scan` reports zero findings here, the smoke test fails and
opens an issue against this repo.

## Safe-prefix discipline

Every credential-looking string in this directory uses `XXXFAKE_` to:

- Prevent accidental match against real provider regexes (Stripe,
  AWS, GitHub, etc.) so secret scanners don't flag this fixture as
  a real leak.
- Prevent GitHub's secret-scanning push-protection from blocking
  this repo's own commits.
- Make grep auditing easy.
