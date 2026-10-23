# Learnings

After each phase, write 3 lines: what broke, how I fixed it, what I'd say in an interview.

## Phase 0 — Setup
- Broke: The pipeline was broke
- Fixed: we fixed it
- Interview line: it runs successfully now

## Phase 2 — Simulate the AWS sources
- Broke: Reading /Volumes/northpeak/... failed with NO_SUCH_CATALOG_EXCEPTION, because Free Edition doesn't allow creating a new catalog.
- Fixed: Recreated the schemas and volume in the built-in `workspace` catalog and pointed every path at /Volumes/workspace/landing/raw.
- Interview line: "I simulated three AWS sources (S3 CSV, DMS-style CDC JSON, Kinesis JSON) with deliberate data-quality issues, and wrote a source-to-target type mapping before building the pipeline, for example DECIMAL(18,2) for money, never FLOAT."