# Consumer Group Fix - BUX Kafka Fork Documentation

## Overview

The `buxapp/kafka` fork maintains custom patches implementing the "consumergroupfix" optimization required for BUX's internal CQRS & Event Sourcing framework. This document explains how to maintain these patches while keeping the fork synchronized with upstream Apache Kafka.

## What is ConsumerGroupFix?

ConsumerGroupFix is a set of optimizations applied to Apache Kafka to improve consumer group management for BUX's event sourcing infrastructure. These patches optimize performance and behavior critical to BUX's CQRS implementation.

## Current Implementation

### Branch: 3.9

The 3.9 branch contains 4 BUX-specific commits implementing consumergroupfix (in chronological order):

1. **b55f2d3582** - Ported consumergroupfix from 3.2 branch
   - Initial port of the optimization logic
   - Fixed tests and adapted to 3.9 codebase
   - Note: Some tests disabled due to summertime handling issues

2. **894ae9a51f** - Fixed key variable usage and updated groupId handling
   - Changed from `builder.data.key()` to key variable (newer approach)
   - Fixed broken test where groupId became important

3. **3341c9dbd7** - Fixed broken code after porting, updated version to 3.9.0-consumergroupfix
   - Resolved porting issues from 2.8 branch code
   - Updated version identifier to include `-consumergroupfix` suffix

4. **e1261e94d7** - Fixed test added in 3.8 branch
   - Resolved test failures from upstream 3.8 changes

**Version identifier**: `3.9.0-consumergroupfix` (in gradle.properties)

### Branch: trunk

The trunk branch is kept in sync with Apache Kafka's trunk/main branch and may receive consumergroupfix patches as needed for future development.

---

## Automated Synchronization

A GitHub Actions workflow automatically syncs the fork with upstream Apache Kafka while preserving BUX patches.

### How It Works

1. **Scheduled Run**: Executes daily at 2 AM UTC via cron
2. **Fetch Upstream**: Retrieves latest changes from `apache/kafka`
3. **Branch Sync**: For each configured branch (trunk, 3.9):
   - Checks out the branch
   - Fetches upstream changes
   - Merges upstream into local branch (preserving BUX commits on top)
   - Pushes updated branch to buxapp/kafka
4. **Conflict Handling**: If merge conflicts occur:
   - Aborts the merge
   - Creates a GitHub issue with details for manual resolution

### Manual Trigger

To manually trigger synchronization:

1. Navigate to: https://github.com/buxapp/kafka/actions/workflows/sync-upstream.yml
2. Click "Run workflow"
3. Optionally specify branches (default: `trunk,3.9`)
4. Click "Run workflow" button

---

## Manual Synchronization

If you need to manually sync the fork or add consumergroupfix to a new branch:

### Prerequisites

```bash
# Clone the repository
git clone https://github.com/buxapp/kafka.git
cd kafka

# Add upstream remote
git remote add upstream https://github.com/apache/kafka.git
git fetch upstream
```

### Syncing an Existing Branch (e.g., 3.9)

```bash
# Checkout the branch
git checkout 3.9
git fetch origin 3.9
git fetch upstream 3.9

# Merge upstream changes (preserves BUX patches on top)
git merge upstream/3.9 -m "Sync 3.9 with apache/kafka upstream"

# If conflicts occur, resolve them carefully to preserve BUX patches
# The BUX patches should remain as the most recent commits

# Push
git push origin 3.9
```

### Syncing trunk Branch

```bash
# Checkout trunk
git checkout trunk
git fetch origin trunk
git fetch upstream trunk

# Merge upstream changes
git merge upstream/trunk -m "Sync trunk with apache/kafka upstream"

# Resolve any conflicts if they occur, then push
git push origin trunk
```

---

## Applying ConsumerGroupFix to Future Branches

When a new Apache Kafka version is released (e.g., 4.0, 4.1), follow these steps to apply consumergroupfix:

### Step 1: Create Branch from Upstream

```bash
# Fetch the new upstream branch
git fetch upstream <new-version>

# Create a local branch from upstream
git checkout -b <new-version> upstream/<new-version>

# Push to create the branch in buxapp/kafka
git push origin <new-version>
```

### Step 2: Identify ConsumerGroupFix Commits

```bash
# View the consumergroupfix commits from the previous version
git checkout 3.9
git log --oneline -10

# Note the commit messages and changes related to consumergroupfix
# Typically these are the top 4 commits with "consumergroupfix" in their messages
```

### Step 3: Cherry-Pick ConsumerGroupFix Commits

```bash
# Checkout the new branch
git checkout <new-version>

# Cherry-pick the consumergroupfix commits from the previous branch
# Start with the oldest commit first (bottom of the list from Step 2)
git cherry-pick b55f2d3582  # Ported consumergroupfix from previous branch
git cherry-pick 894ae9a51f  # Fixed key variable usage
git cherry-pick 3341c9dbd7  # Fixed broken code after porting
git cherry-pick e1261e94d7  # Fixed test

# Note: You may need to resolve conflicts during cherry-picking
# Use git status and git diff to review conflicts
```

### Step 4: Update Version String

```bash
# Edit gradle.properties to update the version
vim gradle.properties

# Change the version line to include -consumergroupfix suffix
# For example: version=4.0.0-consumergroupfix
```

### Step 5: Test the Changes

```bash
# Build the project
./gradlew clean build -x test

# Run unit tests
./gradlew test

# Run consumer group specific tests
./gradlew :core:test --tests "*ConsumerGroup*"
```

### Step 6: Commit and Push

```bash
# Stage the version change
git add gradle.properties

# Commit
git commit -m "Updated version to <new-version>-consumergroupfix"

# Push the branch
git push origin <new-version>
```

### Step 7: Update Automation

Edit `.github/workflows/sync-upstream.yml` to add the new branch to the default sync list:

```yaml
# In the workflow file, update the default branches
default: 'trunk,3.9,<new-version>'
```

Commit and push this change to keep the new branch synchronized automatically.

### Step 8: Document the New Branch

Update this document (CONSUMERGROUPFIX.md) to include information about the new branch and its specific commits.

---

## Handling Merge Conflicts

Conflicts can occur during synchronization when upstream changes overlap with BUX patches.

### Automated Conflict Notification with GitHub Copilot Workspace

When the automated sync workflow encounters a merge conflict, it now:

1. **Aborts the merge** to preserve repository state
2. **Creates a GitHub issue** with title: `Merge Conflict: Resolve using GitHub Copilot Workspace on <branch>`
3. **Provides context** including branch names and commit hashes
4. **Enables "Open in Workspace" button** for AI-assisted resolution

#### Using GitHub Copilot Workspace (Recommended)

When a merge conflict issue is created, you'll see an **"Open in Workspace"** button in the Development section on the right side of the issue page.

**Steps:**
1. Click the "Open in Workspace" button on the issue
2. Review GitHub Copilot's proposed resolution plan
3. **Verify BUX-specific patches are preserved** (critical!)
4. Click "Implement" to apply the changes
5. Review and merge the generated pull request
6. Close the original conflict issue

**Important**: Always carefully review Copilot's proposed changes to ensure consumergroupfix patches are not removed or modified incorrectly.

### Manual Conflict Resolution Process

1. **Identify conflicts**:
   ```bash
   git status
   # Shows files with conflicts marked as "both modified"
   ```

2. **Review the changes**:
   ```bash
   git diff --ours --theirs <file>
   # Shows differences between local and upstream versions
   ```

3. **Resolve conflicts**:
   - Open conflicting files in your editor
   - Look for conflict markers: `<<<<<<<`, `=======`, `>>>>>>>`
   - Keep both upstream improvements AND BUX patches
   - Remove conflict markers
   - Test the merged code

4. **Complete the merge**:
   ```bash
   git add <resolved-files>
   git commit
   git push origin <branch>
   ```

5. **Verify BUX patches are preserved**:
   ```bash
   git log --oneline -10
   # Ensure consumergroupfix commits are still at the top
   
   grep "^version=" gradle.properties
   # Should show: version=X.X.X-consumergroupfix
   ```

---

## Verification After Sync

Always verify that consumergroupfix patches are preserved after synchronization:

### Check Commit History

```bash
git checkout <branch>
git log --oneline -10

# Should show BUX commits at the top:
# - fixed test added in X.X branch
# - fix broken code after porting consumergroupfix code...
# - fix using key variable...
# - ported consumergroupfix from previous branch...
```

### Check Version String

```bash
git show HEAD:gradle.properties | grep version
# Should show: version=X.X.X-consumergroupfix
```

### Run Tests

```bash
# Build the project
./gradlew clean build -x test

# Run unit tests
./gradlew test

# Run consumer group specific tests
./gradlew :core:test --tests "*ConsumerGroup*"
```

---

## Rollback Procedure

If a sync causes issues or breaks the consumergroupfix functionality:

```bash
# Find the last known good commit
git log --oneline origin/<branch> -20

# Note the commit hash before the problematic merge

# Reset to the good commit
git reset --hard <good-commit-sha>

# Force push (use with caution - coordinate with team)
git push origin <branch> --force-with-lease
```

**Important**: Always communicate with the team before force-pushing, as this rewrites history.

---

## Quick Command Reference

### Automated Sync (Recommended)
- The fork syncs automatically daily at 2 AM UTC
- Manual trigger: https://github.com/buxapp/kafka/actions/workflows/sync-upstream.yml

### Manual Sync Commands

**Sync any branch:**
```bash
git fetch origin <branch>
git checkout <branch>
git fetch upstream <branch>
git merge upstream/<branch> -m "Sync <branch> with apache/kafka"
git push origin <branch>
```

**Verify patches:**
```bash
git log --oneline -5
grep "^version=" gradle.properties
```

**Emergency rollback:**
```bash
git log --oneline origin/<branch> -10
git reset --hard <commit-sha>
git push origin <branch> --force-with-lease
```

---

## Best Practices

1. **Always test locally first**: Test sync on a local branch before pushing
2. **Preserve BUX commits**: Ensure consumergroupfix commits remain on top after merge
3. **Monitor CI/CD**: Check that builds pass after synchronization
4. **Update version carefully**: Always maintain the `-consumergroupfix` suffix
5. **Document changes**: Update this document when adding patches to new branches
6. **Coordinate force-pushes**: Always inform the team before using `--force`

---

## Troubleshooting

### Issue: Merge conflicts during automatic sync
**Solution**: 
- **Option 1 (Recommended)**: The workflow creates a GitHub issue with `copilot-workspace` label. Click "Open in Workspace" button to use AI-assisted resolution.
- **Option 2**: Resolve manually following the manual conflict resolution process above.

### Issue: Tests fail after sync
**Solution**: 
1. Check if upstream changes affected consumergroupfix logic
2. Review the failing tests
3. Update the patches if necessary
4. Commit fixes as new consumergroupfix patches

### Issue: Version string missing `-consumergroupfix` suffix
**Solution**:
```bash
vim gradle.properties
# Add -consumergroupfix suffix to version
git add gradle.properties
git commit -m "Restore consumergroupfix version suffix"
git push origin <branch>
```

### Issue: ConsumerGroupFix commits missing after sync
**Solution**:
1. Check `git log` to see if commits were lost
2. If lost, restore from backup or previous commit
3. Cherry-pick the missing commits from another branch
4. Push the restored branch

---

## Support and References

**For questions or issues:**
- Check the Actions tab for workflow logs: https://github.com/buxapp/kafka/actions
- Review issues created by the sync workflow
- Contact BUX Kafka maintainers

**References:**
- Upstream Apache Kafka: https://github.com/apache/kafka
- BUX Kafka Fork: https://github.com/buxapp/kafka
- Apache Kafka Documentation: https://kafka.apache.org/documentation/
- Workflow file: `.github/workflows/sync-upstream.yml`

---

## Summary

This fork maintains critical optimizations for BUX's CQRS & Event Sourcing infrastructure. The automated workflow ensures continuous synchronization with upstream Apache Kafka while preserving these optimizations. When applying patches to future branches, follow the step-by-step guide in the "Applying ConsumerGroupFix to Future Branches" section to ensure consistency and maintain the optimization benefits.
