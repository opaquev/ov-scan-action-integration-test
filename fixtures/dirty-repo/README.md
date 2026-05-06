# dirty-repo fixture

This is the `ov-scan-action` smoke test's dirty fixture. It contains
credential-shaped strings that `ov scan`'s detectors WILL match —
the smoke needs `ov scan` to actually find something here, otherwise
the assertion that the action exits non-zero with findings>0 is
trivially false.

`ov scan` against this directory MUST find at least one finding and
exit non-zero with the action's `--fail-on high` threshold.

If `ov scan` reports zero findings here, the smoke test fails and
opens an issue against this repo.

## Why these specific values

The fixtures use AWS's [published documentation
examples](https://docs.aws.amazon.com/IAM/latest/UserGuide/security-creds.html)
(`AKIAIOSFODNN7EXAMPLE` + the matching secret-key example string).
These are well-known fakes:

- GitHub's secret-scanning push-protection **allowlists** these by
  design — that's why this repo can commit them without push-protection
  rejecting the push.
- AWS rejects them on any API call; they don't validate against IAM.
- But `ov scan`'s detectors DO match them, exercising the
  `aws-access-key`, `aws-secret-key`, and `high-entropy` detectors.

The action repo's own `tests/fixtures/dirty-repo/` uses `XXXFAKE_`-prefixed
patterns instead because that fixture feeds a mocked `ov` binary
(`mock_ov_bin` in `tests/helpers.bash`) that returns canned JSON regardless
of input — so the prefix is for safety, not detection. **This repo runs
the REAL `ov scan` binary**, so the fixture must contain detector-matching
content.

## What NOT to do

Don't replace these with hand-invented "real-looking" fake AWS keys.
Any 16-hex-char string after `AKIA` will satisfy `ov scan`'s detector
pattern, but most won't satisfy GitHub's push-protection allowlist —
you'll get blocked from committing. Stick with the AWS docs examples
unless you have a clear reason and have verified push-protection accepts
them in a test commit.
