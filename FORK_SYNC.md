# Fork Synchronization Guide

This document explains how to keep the BUX Kafka fork synchronized with the upstream Apache Kafka repository while preserving BUX-specific patches.

## Overview

The `buxapp/kafka` fork maintains custom patches for optimizing the internal CQRS & Event Sourcing framework. These patches must be preserved when synchronizing with upstream Apache Kafka updates.

## BUX-Specific Patches

### Branch: 3.9
The 3.9 branch contains the following BUX-specific commits (in chronological order):
1. `b55f2d3582` - Ported consumergroupfix from 3.2 branch
2. `894ae9a51f` - Fixed key variable usage and updated groupId handling
3. `3341c9dbd7` - Fixed broken code after porting, updated version to 3.9.0-consumergroupfix
4. `e1261e94d7` - Fixed test added in 3.8 branch

These commits are on top of the Apache Kafka 3.9 release and implement the "consumergroupfix" optimization.

### Branch: trunk
The trunk branch is kept in sync with Apache Kafka's trunk/main branch.

## Automatic Synchronization

A GitHub Actions workflow (`.github/workflows/sync-upstream.yml`) automatically syncs the fork:

- **Schedule**: Runs daily at 2 AM UTC
- **Branches**: Syncs `trunk` and `3.9` branches by default
- **Process**: 
  1. Fetches latest changes from `apache/kafka`
  2. Merges upstream changes into local branches
  3. Pushes updates to the fork
  4. Creates an issue if merge conflicts occur

### Manual Trigger

You can manually trigger the sync workflow:

1. Go to the Actions tab in GitHub
2. Select "Sync Fork with Apache Kafka" workflow
3. Click "Run workflow"
4. Optionally specify branches to sync (comma-separated)

## Manual Synchronization

If you need to manually sync the fork:

### Prerequisites

```bash
# Clone the repository
git clone https://github.com/buxapp/kafka.git
cd kafka

# Add upstream remote
git remote add upstream https://github.com/apache/kafka.git
git fetch upstream
```

### Syncing the trunk branch

```bash
# Checkout trunk
git checkout trunk
git fetch origin trunk
git fetch upstream trunk

# Merge upstream changes
git merge upstream/trunk -m "Sync trunk with apache/kafka upstream"

# Resolve any conflicts if they occur
# Then push
git push origin trunk
```

### Syncing the 3.9 branch (with BUX patches)

```bash
# Checkout 3.9
git checkout 3.9
git fetch origin 3.9
git fetch upstream 3.9

# Merge upstream changes
# This will merge apache/kafka 3.9 updates while preserving BUX patches on top
git merge upstream/3.9 -m "Sync 3.9 with apache/kafka upstream"

# If conflicts occur, resolve them carefully to preserve BUX patches
# The BUX patches should remain as the most recent commits

# Push
git push origin 3.9
```

### Handling Merge Conflicts

If merge conflicts occur during synchronization:

1. **Identify the conflict**: `git status` will show conflicting files
2. **Review the changes**: 
   ```bash
   git diff --ours --theirs <file>
   ```
3. **Resolve conflicts**: Edit files to keep both upstream changes and BUX patches
4. **Test the resolution**: Ensure BUX-specific optimizations still work
5. **Complete the merge**:
   ```bash
   git add <resolved-files>
   git commit
   git push origin <branch>
   ```

## Verifying BUX Patches

After synchronization, verify that BUX patches are still present:

### Check 3.9 branch

```bash
git checkout 3.9
git log --oneline -10

# Should show BUX commits at the top:
# - fixed test added in 3.8 branch
# - fix broken code after porting consumergroupfix code...
# - fix using key variable instead of builder.data.key()...
# - ported consumergroupfix from 3.2 branch...
```

### Check version

```bash
git show HEAD:gradle.properties | grep version
# Should show: version=3.9.0-consumergroupfix
```

## Testing After Sync

After synchronizing, run these tests to ensure everything works:

```bash
# Build the project
./gradlew clean build -x test

# Run unit tests
./gradlew test

# Run specific consumer group tests if available
./gradlew :core:test --tests "*ConsumerGroup*"
```

## Rollback Procedure

If a sync causes issues:

```bash
# Find the last known good commit
git log --oneline origin/<branch> -20

# Reset to the good commit
git reset --hard <good-commit-sha>

# Force push (use with caution)
git push origin <branch> --force-with-lease
```

## Best Practices

1. **Always test locally first**: Test the sync on a local branch before pushing
2. **Preserve BUX commits**: Ensure BUX-specific commits remain on top after merge
3. **Monitor CI/CD**: Check that builds pass after synchronization
4. **Update version carefully**: Maintain the `-consumergroupfix` suffix in version strings
5. **Document changes**: If new patches are added, document them in this document

## Support

For questions or issues with fork synchronization:
- Check the Actions tab for workflow logs
- Review any issues created by the sync workflow
- Contact the BUX Kafka maintainers

## References

- Upstream Apache Kafka: https://github.com/apache/kafka
- BUX Kafka Fork: https://github.com/buxapp/kafka
- Apache Kafka Documentation: https://kafka.apache.org/documentation/
