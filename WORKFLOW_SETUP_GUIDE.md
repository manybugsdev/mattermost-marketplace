# Watch Plugin Repos Workflow - Setup Guide

This guide will help you set up the automated workflow that watches your `mattermost-plugin-*` repositories and automatically updates the marketplace when new releases are detected.

## What Was Created

1. **`.github/workflows/watch-plugin-repos.yml`** - The main GitHub Actions workflow that:
   - Runs daily at 00:00 UTC (and can be manually triggered)
   - Discovers all your `mattermost-plugin-*` repositories
   - Checks for new releases
   - Automatically adds them to `plugins.json`
   - Commits changes and deploys to AWS Lambda

2. **`.github/workflows/WATCH_PLUGIN_REPOS_SECRETS.md`** - Comprehensive documentation of all required and optional GitHub Secrets

## Quick Setup Steps

### Step 1: Configure Required GitHub Secrets

Go to your repository **Settings → Secrets and variables → Actions** and add:

#### Required Secrets:

1. **`PLUGIN_REPO_OWNER`**
   - Value: Your GitHub username or organization name
   - Example: `manybugsdev`

2. **`AWS_ACCESS_KEY_ID`**
   - Value: Your AWS access key ID
   - Get from: AWS Console → IAM → Users → Security credentials

3. **`AWS_SECRET_ACCESS_KEY`**
   - Value: Your AWS secret access key
   - Get from: AWS Console (shown when creating access key)

#### Optional Secrets (recommended):

4. **`PLUGIN_AUTHOR_TYPE`**
   - Value: `official`, `partner`, or leave unset for `community`
   - Determines how your plugins are categorized

5. **`SLS_STAGE`**
   - Value: `dev`, `staging`, or `production`
   - Default: `dev`

6. **`AWS_DEFAULT_REGION`**
   - Value: AWS region (e.g., `us-east-1`, `us-west-2`)
   - Default: `us-east-1`

### Step 2: AWS Permissions Setup

Your AWS credentials need permissions for:
- Lambda function management
- API Gateway
- CloudFormation
- S3 (for Serverless deployments)
- CloudFront

**Recommended**: Use AWS Serverless Application Repository policy or create a custom policy with minimum required permissions.

### Step 3: Test the Workflow

1. Go to **Actions** tab in your GitHub repository
2. Select **"Watch Plugin Repositories"** workflow
3. Click **"Run workflow"** button
4. Select the branch (usually `main` or `master`)
5. Click **"Run workflow"**

This will manually trigger the workflow so you can verify it works correctly.

### Step 4: Monitor the Workflow

After the workflow runs:
1. Check the **Actions** tab for workflow run results
2. Review any errors in the logs
3. Verify that new plugins were added to `plugins.json` (if any were found)
4. Confirm the Lambda deployment succeeded (if changes were made)

## How It Works

### Discovery Phase
1. Workflow fetches all repositories owned by `PLUGIN_REPO_OWNER`
2. Filters for repositories starting with `mattermost-plugin-`
3. Handles pagination for accounts with >100 repositories
4. Works with both user and organization accounts

### Validation Phase
1. Validates repository names (alphanumeric, hyphens, underscores only)
2. Validates version tags (semantic versioning format)
3. Checks for new releases via GitHub API

### Update Phase
1. Compares releases with existing entries in `plugins.json`
2. Uses the `generator` command to add new plugins
3. Commits changes with descriptive message
4. Deploys to AWS Lambda using `make deploy-lambda`

## Security Features

✅ Input validation to prevent command injection  
✅ Explicit workflow permissions (contents: write)  
✅ GitHub Actions pinned to commit hashes  
✅ Specific dependency versions for reproducibility  
✅ Support for semantic versioning with pre-release metadata  
✅ Passes CodeQL security analysis

## Troubleshooting

### No repositories found
- Verify `PLUGIN_REPO_OWNER` is correct
- Check that repositories are public or workflow has access
- Ensure repository names start with `mattermost-plugin-`

### AWS deployment fails
- Verify AWS credentials are correct
- Check IAM permissions
- Confirm AWS region is correct
- Review Serverless configuration in `serverless.yml`

### Plugin addition fails
- Ensure plugin release is properly built
- Check that release artifacts are available
- Review generator command requirements in main README
- Verify the plugin follows expected structure

### Workflow doesn't run automatically
- Check that the workflow file is in the default branch
- Verify cron schedule is correct (default: daily at 00:00 UTC)
- GitHub Actions must be enabled for the repository

## Customization

### Change Schedule
Edit the cron expression in `watch-plugin-repos.yml`:
```yaml
schedule:
  - cron: '0 0 * * *'  # Daily at 00:00 UTC
```

Examples:
- `'0 */6 * * *'` - Every 6 hours
- `'0 12 * * *'` - Daily at 12:00 UTC
- `'0 0 * * 1'` - Weekly on Monday

### Filter Repositories
Modify the repository filter in the workflow:
```bash
jq -r '.[] | select(.name | startswith("mattermost-plugin-")) | .name'
```

Change `"mattermost-plugin-"` to match your naming convention.

### Skip Deployment
If you only want to update `plugins.json` without deploying, comment out or remove the deployment step in the workflow.

## Support

For detailed information about each GitHub Secret, see:
- `.github/workflows/WATCH_PLUGIN_REPOS_SECRETS.md`

For general marketplace information, see:
- `README.md`

For issues or questions, please open a GitHub issue in this repository.
