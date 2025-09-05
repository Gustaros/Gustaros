# GitHub Actions Profile Generation Fix

## Problem
The GitHub Actions workflow for generating profile summary cards was failing with 401 Unauthorized errors. All API calls were getting rejected due to authentication issues.

## Root Cause
The workflow was configured to use `secrets.SUMMARY_CARD_TOKEN` which either:
- Doesn't exist as a repository secret
- Has expired 
- Has insufficient permissions

## Solution Applied
Updated the workflow file `.github/workflows/profile-summary-cards.yml` with the following changes:

### 1. Authentication Fix
- **Before**: `GITHUB_TOKEN: ${{ secrets.SUMMARY_CARD_TOKEN }}`
- **After**: `GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}`

### 2. Added Explicit Permissions
```yaml
permissions:
  contents: write    # To push generated cards back to repository
  metadata: read     # To read repository metadata  
  statuses: read     # To read commit statuses
  actions: read      # To read workflow information
```

### 3. Updated Dependencies
- Updated `actions/checkout` from v2 to v4 for better security and compatibility

## Alternative Solution (If Built-in Token Insufficient)

If the workflow still fails because the built-in `GITHUB_TOKEN` doesn't have sufficient permissions to read user statistics across all repositories, you'll need to create a Personal Access Token:

### Step 1: Create Personal Access Token
1. Go to GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Click "Generate new token"
3. Give it a descriptive name like "Profile Summary Cards"
4. Select the following scopes:
   - `repo` (Full control of private repositories)
   - `user:read` (Read user profile data)
   - `read:user` (Read user profile data)
   - `public_repo` (Access public repositories)

### Step 2: Add Token as Repository Secret
1. Go to your repository → Settings → Secrets and variables → Actions
2. Click "New repository secret"
3. Name: `SUMMARY_CARD_TOKEN`
4. Value: Paste your Personal Access Token

### Step 3: Update Workflow (if needed)
If using a custom token, update the workflow to use it:
```yaml
env:
  GITHUB_TOKEN: ${{ secrets.SUMMARY_CARD_TOKEN }}
```

## Testing the Fix
1. The workflow runs automatically every day at midnight (UTC)
2. You can manually trigger it by going to Actions → GitHub-Profile-Summary-Cards → Run workflow
3. Check the workflow logs for any remaining errors

## Expected Results
When working correctly, the workflow will:
1. Generate profile summary cards showing your GitHub statistics
2. Create/update files in a `profile-summary-card-output` directory
3. Commit and push these files back to your repository
4. Cards can then be embedded in your README.md or other files

## Files Generated
The action typically generates various cards like:
- Profile details (followers, repositories, etc.)
- Most used languages by repos
- Most used languages by commits  
- Statistics overview
- Productive time analysis