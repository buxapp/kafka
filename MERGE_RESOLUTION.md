# Merge Conflict Resolution Complete

This PR resolves the merge conflict between buxapp/kafka trunk and apache/kafka trunk.

## What Was Done

1. **Merged upstream/trunk** (commit `36eb8184b0`) into the trunk branch
2. **Resolved 3,382 file conflicts** by accepting upstream versions (using `git checkout --theirs`)
3. **Preserved BUX-specific files**:
   - `.github/workflows/sync-upstream.yml` - Automated sync workflow
   - `CONSUMERGROUPFIX.md` - Documentation for BUX patches

## Verification

✅ BUX-specific files preserved:
- `.github/workflows/sync-upstream.yml` exists and is unchanged
- `CONSUMERGROUPFIX.md` exists and is unchanged

✅ Upstream changes incorporated:
- NOTICE-binary updated with 2026 copyright
- All upstream commits from apache/kafka trunk are included
- New files from upstream are added (GitHub Actions workflows, new scripts, etc.)

## Next Steps

When this PR is merged into trunk:
1. The trunk branch will be updated with the resolved merge
2. The sync-upstream workflow will be able to continue syncing with apache/kafka
3. No manual intervention will be needed for future syncs (unless conflicts occur)

## Notes

- The trunk branch does NOT have consumergroupfix patches (as documented in CONSUMERGROUPFIX.md)
- The 3.9 branch contains the consumergroupfix patches
- This merge keeps trunk in sync with upstream apache/kafka as intended
