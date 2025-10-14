# DMZ Revert Workflow - Security Setup

## Overview

The DMZ revert workflow uses GitHub Environment protection rules to ensure only authorized users can revert commits. This security cannot be bypassed by modifying the workflow file.

## Why Environment Protection?

### Problems with Code-Based Authorization ❌
- Anyone with write access can modify the workflow file
- Can add themselves to authorized users list
- Can remove authorization checks entirely
- No audit trail of permission changes

### Benefits of Environment Protection ✅
- **Cannot be bypassed** - even repository admins must be explicitly authorized
- **GitHub-native security** - uses GitHub's permission system
- **Audit trail** - all approvals are logged
- **Optional manual approval** - can require human review before running
- **Protection rules** - set wait timers, required reviewers, etc.

## Setup Instructions

### Step 1: Create the Environment

1. Go to your repository on GitHub
2. Click **Settings** > **Environments**
3. Click **New environment**
4. Name it: `dmz-revert`
5. Click **Configure environment**

### Step 2: Configure Protection Rules

#### Option A: Restrict to Specific Users/Teams (Recommended)

1. Under **Environment protection rules**, enable:
   - ✅ **Required reviewers** (optional, for manual approval)
   - ✅ **Deployment branches** (restrict which branches can deploy)

2. Click **Add reviewers** under deployment protection rules
3. Add authorized users:
   - `sc-chathumikaalwis` (your username)
   - Add other authorized users or teams

4. **IMPORTANT**: Check "Allow administrators to bypass configured protection rules"
   - ⚠️ **Leave this UNCHECKED** for maximum security
   - This ensures even admins need authorization

#### Option B: Manual Approval Required (Most Secure)

1. Enable **Required reviewers**
2. Add 1-2 trusted reviewers
3. Every revert will require manual approval from one of these reviewers
4. Adds human oversight to every revert operation

### Step 3: Restrict Who Can Access the Environment

GitHub environments support:

1. **Selected Reviewers**
   - Only specific users/teams can approve workflow runs
   - Cannot be modified by workflow code

2. **Wait Timer** (optional)
   - Add a delay before workflow runs (e.g., 5 minutes)
   - Gives time to cancel accidental triggers

3. **Deployment Branches** (optional)
   - Restrict which branches can use this environment
   - Set to `dmz` branch only

### Step 4: Configure CODEOWNERS (Additional Layer)

Create `.github/CODEOWNERS` file:

```
# Workflow files require approval from authorized users
/.github/workflows/revert-dmz.yml @sc-chathumikaalwis
/.github/workflows/dmz-validation.yml @sc-chathumikaalwis

# DMZ revert setup requires approval
/.github/REVERT_DMZ_SETUP.md @sc-chathumikaalwis
```

This ensures any changes to the revert workflow require your approval.

### Step 5: Enable Branch Protection for dmz

1. Go to **Settings** > **Branches**
2. Add rule for `dmz` branch
3. Enable:
   - ✅ **Require pull request reviews before merging**
   - ✅ **Require status checks to pass before merging**
   - ✅ **Include administrators** (even admins must follow rules)
   - ✅ **Require approval from Code Owners**

## How It Works

### Normal Workflow (Authorized User)
```
1. User clicks "Run workflow"
2. GitHub checks environment protection rules
3. ✅ User is authorized → workflow runs immediately
4. Revert is executed
```

### With Manual Approval (Most Secure)
```
1. User clicks "Run workflow"
2. GitHub checks environment protection rules
3. ⏸️ Workflow waits for approval
4. Authorized reviewer approves/denies
5. ✅ Approved → workflow runs
   ❌ Denied → workflow cancelled
```

### Unauthorized User Attempt
```
1. User clicks "Run workflow"
2. GitHub checks environment protection rules
3. ❌ User not authorized → workflow fails immediately
4. Error: "User not authorized to deploy to environment"
5. Workflow never executes
```

## Security Benefits

| Attack Vector | Code-Based Auth | Environment Auth |
|--------------|----------------|------------------|
| Modify workflow file | ❌ Vulnerable | ✅ Protected |
| Add self to auth list | ❌ Vulnerable | ✅ Protected |
| Remove auth check | ❌ Vulnerable | ✅ Protected |
| Fork and bypass | ❌ Vulnerable | ✅ Protected |
| Impersonate username | ✅ Protected | ✅ Protected |
| Admin bypass | ❌ Vulnerable | ✅ Protected* |

\* Protected if "Allow administrators to bypass" is disabled

## Audit Trail

All authorization checks are logged:
- **Actions** tab shows who triggered workflow
- **Environment** tab shows approval history
- **Settings audit log** shows permission changes
- Cannot be deleted or modified by users

## Testing the Security

### Test 1: Authorized User
```bash
# As sc-chathumikaalwis:
1. Go to Actions > Revert DMZ Branch
2. Click "Run workflow"
3. Enter commit SHA and reason
4. Should run successfully ✅
```

### Test 2: Unauthorized User
```bash
# As any other user:
1. Go to Actions > Revert DMZ Branch
2. Click "Run workflow"
3. Enter commit SHA and reason
4. Should fail with: "not authorized to deploy to environment dmz-revert" ❌
```

## Managing Authorized Users

### Add User
1. Go to **Settings** > **Environments** > **dmz-revert**
2. Under protection rules, click **Add reviewers**
3. Search and select user/team
4. Click **Save protection rules**

### Remove User
1. Go to **Settings** > **Environments** > **dmz-revert**
2. Find user in reviewers list
3. Click **X** to remove
4. Click **Save protection rules**

### View Current Authorized Users
1. Go to **Settings** > **Environments** > **dmz-revert**
2. See list under "Required reviewers"

## Additional Security Recommendations

### 1. Enable 2FA for All Authorized Users
- Require two-factor authentication for the organization
- Prevents account compromise

### 2. Regular Access Reviews
- Quarterly review of who has access
- Remove users who no longer need access

### 3. Separate Environments for Different Operations
```yaml
# Revert operations
environment: dmz-revert

# Deploy operations  
environment: production-deploy

# Each with different authorized users
```

### 4. Use GitHub Teams Instead of Individual Users
```yaml
# Instead of listing individual users:
Authorized: @your-org/dmz-admins

# Benefits:
- Centralized team management
- Easier to add/remove members
- Better organization
```

## Troubleshooting

### Error: "not authorized to deploy to environment"
**Cause**: User not in authorized list
**Solution**: Add user to dmz-revert environment reviewers

### Error: "Environment not found"
**Cause**: Environment `dmz-revert` not created
**Solution**: Follow Step 1 to create environment

### Workflow runs without asking for approval
**Cause**: No required reviewers configured
**Solution**: Add required reviewers in environment settings

### Need to bypass for emergency
**Options**:
1. Have an authorized user run the workflow (recommended)
2. Temporarily add user to environment (audit trail maintained)
3. If absolutely critical and no authorized users available:
   - Admin can temporarily enable "Allow administrators to bypass"
   - Run workflow
   - Disable bypass immediately
   - Document in incident report

## References

- [GitHub Environments Documentation](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment)
- [Environment Protection Rules](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment#environment-protection-rules)
- [CODEOWNERS Documentation](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners)
- [Branch Protection Rules](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)

