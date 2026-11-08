# Issue #2162 evidence dependencies

This records the inputs of the two CPU audits and report generators in this
branch. It describes the benchmark record, not a production acceptance. The
RTX 3090 result directory contains the three earlier QKVO experiments; the
MLP directory contains diagnostic A, its recovery, and allocation B. Later
5090 and second-3090 campaigns are outside this record.

## Files read

`issue2162_audit.py` reads `issue2162-rtx3090-v1/manifest.json` and checks the
SHA-256 of all 58 paths in `published_files_sha256`. It interprets these files:

- `reference-updates.json` and `census-v1/seq2048.json`, `seq4096.json`;
- `exactness-v1/freeze.json`, `ledger.json`, `processes.jsonl`;
- for each of `timing-v1`, `reallocation-v1`, and `scoped-v2-resume1`:
  `freeze.json`, `ledger.json`, `original-audit.json`, `processes.jsonl`,
  `resources.jsonl`, `timings.jsonl`, `source-hashes.json`, and
  `source-blobs.json`;
- for `scoped-v2-resume1`: `prior-attempt.json`, `recovery-amendment.json`, and
  `stage-a-decision.json`.

With `--verify-git-sources`, it also reads the three `source-blobs.json` files
and verifies each recorded blob, commit tree path, frozen SHA-256, and newline
conversion. `issue2162_report.py` consumes the *generated audit output* and
the three experiment `freeze.json` files. It does not read the checked-in
`independent-audit.json`.

The other 25 RTX 3090 paths in `published_files_sha256` are checked by hash
but are not parsed for the result: census audit/log/resource/status; environment
and model material; exactness logs/audit/resources; the three `RULE.md` files;
experiment logs, material preflight/preservation and state; and the rejected
preflight CPU/state plus scoped readiness. These retain provenance and failed
attempts. They are not interchangeable with the timing `resources.jsonl`,
which the audit reads sample by sample. The checked-in
`independent-audit.json` is a historical output, not an input.

`issue2162_mlp_record.py` reads `issue2162-mlp-v1/manifest.json`, checks the
SHA-256 of its 19 `files_sha256` paths, then reads `objects.json`, the three
`*/files.json` maps and the RTX 3090 `reference-updates.json`. It restores
281 original files into a temporary directory, verifies every restored byte
hash and runs `issue2162_mlp_diagnostic_audit.py` and
`issue2162_mlp_budget_audit.py` against those files. The restored independent
audit outputs must equal the recomputed ones. With `--verify-git-sources`, it
also reads `issue2162-mlp-v1/source-blobs.json` and validates its frozen hashes
against Git blobs. `issue2162_mlp_report.py` invokes the record audit and reads
`objects.json`, `mlp-budget-b-v1/files.json`, and the shared reference file to
render the report. The five `frozen/*.txt` files, three `RULE.md` files,
three frozen `freeze.json` files, two `original-audit.json` files, and
`validation.json` are hash-checked historical context in the published tree;
the corresponding original material consumed by the independent audits is
decoded from `objects.json`.

The MLP record does not add 281 uncompressed receipts to Git: `objects.json`
contains their lossless encoding, indexed by the three `files.json` maps.
The original source ZIP archives and full closure bundles are preserved outside
Git. No result directory here is a substitute for those original archives.

## Reachability and reproduction

The four source-binding files contain 6,581 path bindings to 2,010 distinct
Git blobs. A clean, single-branch remote clone of the predecessor head
`cbae329a97b919474076ee55dfe57db1f0e87b3d` contains all 2,010 blobs in
`git rev-list --objects HEAD`; the four source commits are ancestors of that
head. Both audit commands below passed against that clone, including 2,927
QKVO and 3,654 MLP source bindings. This branch retains those source commits
as ancestors, so the verification does not rely on extra objects in the
author's working repository.

```sh
python benchmarks/harness/issue2162_audit.py \
  --evidence benchmarks/results/issue2162-rtx3090-v1 \
  --verify-git-sources . --output audit-2162.json
python benchmarks/harness/issue2162_report.py \
  --evidence benchmarks/results/issue2162-rtx3090-v1 \
  --audit audit-2162.json --output report-2162.md
python benchmarks/harness/issue2162_mlp_record.py \
  --evidence benchmarks/results/issue2162-mlp-v1 \
  --verify-git-sources . --output audit-2162-mlp.json
python benchmarks/harness/issue2162_mlp_report.py \
  --evidence benchmarks/results/issue2162-mlp-v1 \
  --output report-2162-mlp.md
```

The four `source-blobs.json` files now put one binding on each line, and the
shared `reference-updates.json` puts one tensor record on each line. Their JSON
values are unchanged; the manifest hashes were updated to bind the new
serialization. Raw telemetry, original receipts, protocols, verdicts, and
report text are unchanged.
