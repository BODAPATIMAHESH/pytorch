# PyTorch PR → Buildbot Setup - Step by Step Guide

## ✅ Status: Workflow Files Added

Both workflow files have been successfully added to the local repositories:
- ✓ `pytorch/.github/workflows/relay-to-ibm-ghe.yml`
- ✓ `pytorch-power-backend/.github/workflows/buildbot-trigger.yml`

## Step-by-Step Setup Instructions

### Step 1: Create IBM GHE Personal Access Token

1. Open your browser and go to: https://github.ibm.com/settings/tokens
2. Click "Generate new token" (classic)
3. Give it a name: `PyTorch Buildbot Relay`
4. Select scopes:
   - ✓ `repo` (Full control of private repositories)
5. Click "Generate token"
6. **IMPORTANT**: Copy the token immediately (you won't see it again!)
7. Save it somewhere secure (you'll need it in Step 2)

### Step 2: Add Secret to Public GitHub Repository

1. Go to: https://github.com/BODAPATIMAHESH/pytorch/settings/secrets/actions
2. Click "New repository secret"
3. Name: `IBM_GHE_TOKEN`
4. Value: Paste the token from Step 1
5. Click "Add secret"

### Step 3: Commit and Push to Public GitHub (pytorch)

```bash
cd /Users/maheshbodapati/onnxruntime-extensions/pytorch

# Check status
git status

# Add the workflow file
git add .github/workflows/relay-to-ibm-ghe.yml

# Commit
git commit -m "Add GitHub Actions workflow to relay PRs to IBM GHE buildbot"

# Push to GitHub
git push origin main
```

### Step 4: Commit and Push to IBM GHE (pytorch-power-backend)

```bash
cd /Users/maheshbodapati/onnxruntime-extensions/pytorch-power-backend

# Check status
git status

# Add the workflow file
git add .github/workflows/buildbot-trigger.yml

# Commit
git commit -m "Add GitHub Actions workflow to trigger buildbot on PyTorch PRs"

# Push to IBM GHE
git push origin main
```

### Step 5: Verify Buildbot API Endpoint

Before testing, let's verify the buildbot API is accessible:

```bash
# Check if buildbot is accessible
curl http://ipds.pperf.tadn.ibm.com:8080/api/v2/forceschedulers

# Look for builder ID 30 in the response
# The force scheduler name should be something like "force-Custom_PyTorch"
```

If the API endpoint or builder ID is different, update the workflow file:
```bash
cd /Users/maheshbodapati/onnxruntime-extensions/pytorch-power-backend
nano .github/workflows/buildbot-trigger.yml
# Update the builderid and force scheduler name
```

### Step 6: Test the Setup

#### Option A: Create a Test PR (Full Flow Test)

1. In the pytorch repository, create a new branch:
```bash
cd /Users/maheshbodapati/onnxruntime-extensions/pytorch
git checkout -b test-buildbot-relay
echo "# Test PR for buildbot relay" >> TEST_BUILDBOT.md
git add TEST_BUILDBOT.md
git commit -m "Test: Verify buildbot relay workflow"
git push origin test-buildbot-relay
```

2. Go to https://github.com/BODAPATIMAHESH/pytorch/pulls
3. Click "New pull request"
4. Select `test-buildbot-relay` branch
5. Create the PR

6. Watch the magic happen:
   - **Public GitHub**: Check Actions tab → "Relay PR to IBM GHE" should run
   - **IBM GHE**: Go to https://github.ibm.com/MAHESH-BODAPATI/pytorch-power-backend/actions
   - **Buildbot**: Check http://ipds.pperf.tadn.ibm.com:8080/#/builders/30

#### Option B: Manual Trigger (Quick Test)

1. Go to: https://github.ibm.com/MAHESH-BODAPATI/pytorch-power-backend/actions
2. Select "Trigger Buildbot on PyTorch PR" workflow
3. Click "Run workflow"
4. Enter:
   - PR number: `1` (or any test number)
   - PR head SHA: `abc123` (or any test SHA)
5. Click "Run workflow"
6. Check buildbot: http://ipds.pperf.tadn.ibm.com:8080/#/builders/30

### Step 7: Monitor and Verify

After creating a PR, monitor these locations:

1. **Public GitHub Actions**:
   - URL: https://github.com/BODAPATIMAHESH/pytorch/actions
   - Look for: "Relay PR to IBM GHE" workflow
   - Should complete in ~10 seconds

2. **IBM GHE Actions**:
   - URL: https://github.ibm.com/MAHESH-BODAPATI/pytorch-power-backend/actions
   - Look for: "Trigger Buildbot on PyTorch PR" workflow
   - Should complete in ~1-2 minutes

3. **Buildbot**:
   - URL: http://ipds.pperf.tadn.ibm.com:8080/#/builders/30
   - Look for: New build with PR information
   - Build reason should show: "PyTorch PR #X by <author>"

## Troubleshooting

### Issue: "Relay PR to IBM GHE" fails with 401 Unauthorized

**Solution**: 
- Check that `IBM_GHE_TOKEN` secret is set correctly
- Verify token has `repo` scope
- Token might have expired - generate a new one

### Issue: "Trigger Buildbot" workflow doesn't start

**Solution**:
- Check IBM GHE repository name is correct: `MAHESH-BODAPATI/pytorch-power-backend`
- Verify the repository dispatch event type matches: `pytorch_pr_trigger`
- Check IBM GHE Actions are enabled for the repository

### Issue: Buildbot build doesn't trigger

**Solution**:
- Verify buildbot URL is accessible from IBM GHE runners
- Check builder ID (30) is correct: `curl http://ipds.pperf.tadn.ibm.com:8080/api/v2/builders`
- Verify force scheduler name: `curl http://ipds.pperf.tadn.ibm.com:8080/api/v2/forceschedulers`
- Update workflow if names don't match

### Issue: PR changes not being fetched

**Solution**:
- Ensure pytorch-power-backend can access public pytorch repo
- Check git fetch commands in the workflow
- Verify PR number is being passed correctly

## What Happens When You Create a PR

```
1. Developer creates PR in github.com/BODAPATIMAHESH/pytorch
   ↓
2. GitHub Actions triggers "Relay PR to IBM GHE"
   ↓
3. Workflow sends repository_dispatch to IBM GHE
   ↓
4. IBM GHE receives dispatch event
   ↓
5. "Trigger Buildbot on PyTorch PR" workflow starts
   ↓
6. Workflow fetches PR changes from public repo
   ↓
7. Merges PR with pytorch-power-backend code
   ↓
8. Sends POST request to buildbot API
   ↓
9. Buildbot builder #30 starts
   ↓
10. Build runs with PR information in properties
```

## Next Steps After Setup

1. **Test with a real PR**: Create a meaningful PR to test the full flow
2. **Monitor builds**: Watch a few builds to ensure everything works
3. **Adjust timing**: If needed, modify polling intervals or timeouts
4. **Add notifications**: Consider adding Slack/email notifications for build results
5. **Document for team**: Share this guide with your team

## Quick Reference Commands

```bash
# Check public GitHub workflow status
cd /Users/maheshbodapati/onnxruntime-extensions/pytorch
git log --oneline -1 .github/workflows/relay-to-ibm-ghe.yml

# Check IBM GHE workflow status  
cd /Users/maheshbodapati/onnxruntime-extensions/pytorch-power-backend
git log --oneline -1 .github/workflows/buildbot-trigger.yml

# Test buildbot API
curl http://ipds.pperf.tadn.ibm.com:8080/api/v2/builders
curl http://ipds.pperf.tadn.ibm.com:8080/api/v2/forceschedulers

# View recent builds
curl http://ipds.pperf.tadn.ibm.com:8080/api/v2/builders/30/builds?limit=5
```

## Support

If you encounter issues:
1. Check the workflow logs in GitHub Actions
2. Check buildbot logs: `tail -f buildbot-master/twistd.log`
3. Verify network connectivity between IBM GHE and buildbot
4. Review this guide's troubleshooting section

---
**Setup Date**: 2026-06-03
**Repositories**:
- Public: github.com/BODAPATIMAHESH/pytorch
- IBM GHE: github.ibm.com/MAHESH-BODAPATI/pytorch-power-backend
- Buildbot: http://ipds.pperf.tadn.ibm.com:8080/#/builders/30
