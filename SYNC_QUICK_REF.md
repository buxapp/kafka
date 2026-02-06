# Quick Sync Reference

## Automated Sync (Recommended)

The fork automatically syncs with Apache Kafka daily via GitHub Actions.

**Manual Trigger:**
1. Go to: https://github.com/buxapp/kafka/actions/workflows/sync-upstream.yml
2. Click "Run workflow"
3. Select branches to sync (default: trunk,3.9)
4. Click "Run workflow"

## Manual Sync Commands

### Sync trunk branch
```bash
git fetch origin trunk
git checkout trunk
git fetch upstream trunk  # or use: https://github.com/apache/kafka.git
git merge upstream/trunk -m "Sync trunk with apache/kafka"
git push origin trunk
```

### Sync 3.9 branch (preserves BUX patches)
```bash
git fetch origin 3.9
git checkout 3.9
git fetch upstream 3.9
git merge upstream/3.9 -m "Sync 3.9 with apache/kafka"
git push origin 3.9
```

## Verify BUX Patches After Sync

```bash
# Check 3.9 branch
git checkout 3.9
git log --oneline -5
# Should show BUX patches at top:
# - fixed test added in 3.8 branch
# - fix broken code after porting consumergroupfix...
# - fix using key variable...
# - ported consumergroupfix from 3.2 branch...

# Check version
grep "^version=" gradle.properties
# Should show: version=3.9.0-consumergroupfix
```

## Conflict Resolution

If merge conflicts occur:

1. **View conflicts:** `git status`
2. **Resolve files:** Edit to preserve both upstream and BUX changes
3. **Test changes:** Build and test
4. **Complete merge:**
   ```bash
   git add .
   git commit
   git push origin <branch>
   ```

## Emergency Rollback

```bash
# Find last good commit
git log --oneline origin/<branch> -10

# Reset to that commit
git reset --hard <commit-sha>

# Force push (be careful!)
git push origin <branch> --force-with-lease
```

## More Information

See [FORK_SYNC.md](./FORK_SYNC.md) for detailed documentation.
