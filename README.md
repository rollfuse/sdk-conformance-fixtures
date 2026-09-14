# sdk-conformance-fixtures

The authoritative cross-language conformance fixtures every rollfuse client
library must pass: [`rollfuse/go-sdk`](https://github.com/rollfuse/go-sdk)
and [`rollfuse/js-sdk`](https://github.com/rollfuse/js-sdk) each carry a
checked-in copy of the two files here, and each repository's own CI verifies
its copy against this one on every run.

This repository exists because the reference implementation and the vectors
it generates live in `apps/api`'s evaluation domain, inside the private
`rollfuse/rollfuse` monorepo — a client repository's CI cannot read a
private repository's raw file content without extra credentials, but it can
read this one's, since it is public and carries nothing except these two
JSON files, their combined digest manifest, and this document.

## Files

- **`bucketing-vectors.json`** — the stable `(flag_key, subject_key) →
  bucket` contract: FNV-1a (32-bit) of `flag_key + ":" + subject_key`,
  modulo 10000. Generated from and verified against
  `apps/api/internal/evaluation/domain/bucketing.go`'s `Bucket()`.
- **`rollout-outcome-vectors.json`** — rollout outcome resolution: given a
  flag's percentage-split rollout and a subject's bucket, which variation
  is served, with explicit coverage of every split boundary (the bucket
  value exactly at a cumulative split threshold, and the values immediately
  on either side of it). Generated from and verified against
  `apps/api/internal/evaluation/domain/configuration.go`'s `Evaluate()` /
  `Outcome.resolve()`.
- **`manifest.json`** — the SHA-256 digest of each file above, so a drift
  check can compare a small manifest fetch rather than always diffing full
  file content (either works; the manifest is the faster path).

## Why a fixture, not just a shared library

The three implementations (the platform server, `go-sdk`, `js-sdk`) do not
share code — they cannot, being different languages — so nothing stops them
from silently drifting apart one edit at a time. A shared, versioned fixture
with an automated drift check is what turns "we verified parity once, by
hand, during an audit" into a property every CI run re-asserts.

## Who updates this repository

Only `rollfuse/rollfuse`'s own tooling does, when the source-of-truth
fixtures in `apps/api/internal/evaluation/domain/testdata/` change. If you
find yourself about to hand-edit a file here: don't — the private monorepo
is the source of truth, and an edit here that isn't also made there is
exactly the drift this repository exists to prevent. See
`scripts/update-manifest.sh` for the digest-regeneration step that must
accompany any content change.

## Versioning

This repository has no version numbers or releases: `go-sdk`/`js-sdk`'s
drift checks always compare against the content on `main`, and a change
here is expected to land in the same pull request (or the immediately
following one) as the corresponding fix in whichever client repository the
new vector exposed a divergence in.
