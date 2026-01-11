# GitHub Setup Instructions

## Make the Proper Fork Your Main Branch

The new Defcoin 2.0 with proper lineage has been pushed to:
`claude/defcoin-2.0-main-35Buw`

### Steps to Set as Default Branch

1. Go to: **https://github.com/packetloss404/defcoin/settings**

2. Under "Default branch", click the switch/swap icon

3. Select `claude/defcoin-2.0-main-35Buw`

4. Click "Update" and confirm

5. Rename the branch to `main`:
   - Go to https://github.com/packetloss404/defcoin/branches
   - Click the pencil icon next to `claude/defcoin-2.0-main-35Buw`
   - Rename to `main`

### Alternative: Using GitHub CLI

```bash
# Set as default
gh repo edit --default-branch claude/defcoin-2.0-main-35Buw

# Rename to main
git push origin claude/defcoin-2.0-main-35Buw:main
git push origin --delete claude/defcoin-2.0-main-35Buw
```

### Clean Up Old Branches (Optional)

After setting up the new main, you can delete old branches:
- `claude/security-code-review-35Buw` (security audit docs)
- `claude/proper-fork-35Buw` (duplicate of this)
- Any other leftover branches

### What This Branch Contains

```
Git History:
  * 9c06daa6f Complete Defcoin 2.0 branding
  * e1ada3f35 Restore Defcoin consensus parameters
  * cc9b0e6c4 Merge Litecoin v0.21.2 into Defcoin 2.0
  |
  ├── Litecoin 0.21.2 (14,000+ commits)
  |
  * 1f56b0542 Defcoin v1.0.1 (mspicer/Defcoin)
  * ... original Defcoin history back to 2014

Remotes configured:
  origin   → packetloss404/defcoin (your repo)
  upstream → mspicer/Defcoin (original)
  litecoin → litecoin-project/litecoin
```

### Verify After Setup

```bash
git fetch origin
git checkout main
git log --oneline -5
```

Should show the proper Defcoin 2.0 commits with merge from Litecoin.
