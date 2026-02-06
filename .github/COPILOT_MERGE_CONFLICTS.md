# GitHub Copilot Workspace Integration for Merge Conflicts

## Overview

This repository uses an automated sync workflow that merges changes from the upstream Apache Kafka repository. When merge conflicts are detected, the workflow automatically creates an issue that can be resolved using GitHub Copilot Workspace.

## How It Works

### Automatic Conflict Detection

The sync workflow runs daily (or can be manually triggered) and attempts to merge changes from the upstream Apache Kafka repository. When a merge conflict is detected:

1. **Conflict Analysis**: The workflow identifies which files have conflicts
2. **Issue Creation**: An issue is automatically created with:
   - Title: `🤖 Copilot Agent: Resolve merge conflict on <branch>`
   - Labels: `sync`, `conflict`, `copilot-workspace`
   - Detailed context including branch names, commit hashes, and conflicting files

### Using GitHub Copilot Workspace (Recommended)

When a merge conflict issue is created, you'll see an **"Open in Workspace"** button in the Development section on the right side of the issue page.

#### Steps to Resolve with Copilot:

1. **Open the Issue**: Navigate to the auto-created merge conflict issue
2. **Launch Workspace**: Click the "Open in Workspace" button in the Development section
3. **Review Plan**: Copilot will analyze the conflict and propose a resolution plan
4. **Review Changes**: Review the proposed changes to ensure BUX-specific patches are preserved
5. **Implement**: Click "Implement" to apply the changes
6. **Create PR**: Copilot will create a pull request with the resolved conflicts
7. **Review & Merge**: Review the PR, run tests, and merge when ready
8. **Close Issue**: Close the original conflict issue

### Manual Resolution (Alternative)

If you prefer to resolve conflicts manually or Copilot Workspace is unavailable:

1. Checkout the branch: `git checkout <branch>`
2. Add upstream remote: `git remote add upstream https://github.com/apache/kafka.git`
3. Fetch upstream: `git fetch upstream <branch>`
4. Merge upstream: `git merge upstream/<branch>`
5. Resolve conflicts in your editor
6. Stage changes: `git add .`
7. Commit: `git commit -m "Resolve merge conflicts with upstream"`
8. Push: `git push origin <branch>`

## Important Notes

### Preserving BUX-Specific Patches

When resolving merge conflicts, it's critical to:
- **Preserve all BUX-specific customizations** and patches
- **Review changes carefully** to ensure upstream changes don't override custom logic
- **Test thoroughly** after merging to ensure functionality is maintained

### Testing After Resolution

Before closing the issue or merging the PR:
1. Run the build: `./gradlew build`
2. Run relevant tests
3. Verify that BUX-specific features still work correctly

## Workflow Configuration

The sync workflow is located at `.github/workflows/sync-upstream.yml` and can be:
- **Automatically triggered**: Daily at 2 AM UTC
- **Manually triggered**: Via the Actions tab with custom branch selection

### Workflow Permissions

The workflow requires the following permissions:
- `contents: write` - To push merged changes
- `issues: write` - To create conflict notification issues

## Benefits of This Approach

1. **Automated Detection**: Conflicts are detected immediately during sync
2. **Rich Context**: Issues include all necessary information for resolution
3. **AI-Powered Resolution**: Copilot Workspace can intelligently resolve many conflicts
4. **Audit Trail**: All resolutions are tracked via issues and PRs
5. **Flexible**: Supports both automated (Copilot) and manual resolution

## Troubleshooting

### "Open in Workspace" Button Not Appearing

- Ensure you have access to GitHub Copilot Workspace
- Check that the issue has the `copilot-workspace` label
- Try refreshing the issue page

### Copilot Makes Incorrect Resolution

- Review the proposed changes carefully before implementing
- Use manual resolution if the conflict is too complex
- Edit the Copilot-proposed changes if only minor adjustments are needed

### Multiple Conflicts on Same Branch

- Each sync attempt creates a new issue if conflicts persist
- Resolve the most recent issue first
- Close duplicate issues after resolution

## Additional Resources

- [GitHub Copilot Workspace Documentation](https://docs.github.com/en/copilot/using-github-copilot/using-github-copilot-workspace)
- [Apache Kafka Repository](https://github.com/apache/kafka)
- [Resolving Merge Conflicts Guide](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/addressing-merge-conflicts)
