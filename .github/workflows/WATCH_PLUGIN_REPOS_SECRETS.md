# Required GitHub Secrets for Watch Plugin Repos Workflow

The `watch-plugin-repos.yml` workflow requires the following GitHub Secrets to be configured in your repository settings:

## Required Secrets

### 1. `PLUGIN_REPO_OWNER`
**Description**: The GitHub username or organization that owns the mattermost-plugin-* repositories you want to watch.

**Example**: `manybugsdev` or `mattermost`

**How to set**: 
- Go to your repository Settings → Secrets and variables → Actions
- Click "New repository secret"
- Name: `PLUGIN_REPO_OWNER`
- Value: Your GitHub username or organization name

---

### 2. `AWS_ACCESS_KEY_ID`
**Description**: AWS access key ID for deploying the Lambda function.

**How to get**:
1. Log in to AWS Console
2. Go to IAM → Users → Your User → Security credentials
3. Create access key if you don't have one
4. Copy the Access Key ID

**How to set**:
- Go to your repository Settings → Secrets and variables → Actions
- Click "New repository secret"
- Name: `AWS_ACCESS_KEY_ID`
- Value: Your AWS access key ID

---

### 3. `AWS_SECRET_ACCESS_KEY`
**Description**: AWS secret access key for deploying the Lambda function.

**How to get**:
1. When you create the access key in AWS Console (see above), you'll also get the Secret Access Key
2. Copy it immediately as it won't be shown again

**How to set**:
- Go to your repository Settings → Secrets and variables → Actions
- Click "New repository secret"
- Name: `AWS_SECRET_ACCESS_KEY`
- Value: Your AWS secret access key

---

## Optional Secrets

### 4. `PLUGIN_AUTHOR_TYPE`
**Description**: Specifies whether your plugins should be marked as official, partner, or community plugins.

**Possible values**: 
- `official` - For plugins maintained by Mattermost
- `partner` - For plugins maintained by Mattermost partners
- (not set or any other value) - For community plugins (default)

**How to set**:
- Go to your repository Settings → Secrets and variables → Actions
- Click "New repository secret"
- Name: `PLUGIN_AUTHOR_TYPE`
- Value: `official`, `partner`, or leave unset for community

---

### 5. `AWS_DEFAULT_REGION`
**Description**: AWS region where your Lambda function should be deployed.

**Default value**: `us-east-1` (if not set)

**How to set**:
- Go to your repository Settings → Secrets and variables → Actions
- Click "New repository secret"
- Name: `AWS_DEFAULT_REGION`
- Value: Your preferred AWS region (e.g., `us-west-2`, `eu-west-1`, etc.)

---

### 6. `SLS_STAGE`
**Description**: The Serverless stage/environment to deploy to.

**Default value**: `dev` (if not set)

**Common values**: `dev`, `staging`, `production`

**How to set**:
- Go to your repository Settings → Secrets and variables → Actions
- Click "New repository secret"
- Name: `SLS_STAGE`
- Value: Your deployment stage (e.g., `dev`, `staging`, or `production`)

---

## AWS IAM Permissions Required

The AWS user/role associated with your access keys needs the following permissions:
- Lambda function create/update permissions
- API Gateway permissions
- CloudFormation stack create/update permissions
- S3 bucket access (for Serverless deployment)
- CloudFront distribution management (as defined in serverless.yml)

It's recommended to use the AWS Serverless Application Repository policy or create a custom policy with the minimum required permissions.

---

## Testing the Workflow

After setting up all required secrets, you can test the workflow by:

1. Going to Actions tab in your repository
2. Selecting "Watch Plugin Repositories" workflow
3. Clicking "Run workflow" button (workflow_dispatch trigger)

This will manually trigger the workflow without waiting for the daily cron schedule.

---

## How It Works

The workflow:
1. Runs daily at 00:00 UTC (configurable via cron schedule)
2. Fetches all repositories matching `mattermost-plugin-*` pattern owned by `PLUGIN_REPO_OWNER`
3. Checks the latest release for each repository
4. Compares with existing entries in `plugins.json`
5. If a new version is found:
   - Adds it to `plugins.json` using the generator command
   - Commits the changes
   - Deploys to AWS Lambda using `make deploy-lambda`

---

## Troubleshooting

### Workflow fails with "No mattermost-plugin-* repositories found"
- Check that `PLUGIN_REPO_OWNER` is set correctly
- Verify that the user/org has public repositories matching the pattern
- Ensure `GITHUB_TOKEN` has read access to the repositories

### Deployment fails
- Verify AWS credentials are correct and have sufficient permissions
- Check that the AWS region is correct
- Ensure Serverless configuration is valid

### Plugin addition fails
- Ensure the plugin release is properly built and uploaded to the expected location
- Check that the plugin follows the expected structure and naming conventions
- Review the generator command requirements in the main README
