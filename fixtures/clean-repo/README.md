# clean-repo fixture

This is the `ov-scan-action` smoke test's clean fixture. It contains no
credential-shaped strings. `ov scan` against this directory MUST report
zero findings.

If `ov scan` finds anything in this directory, the smoke test fails and
opens an issue against this repo.
