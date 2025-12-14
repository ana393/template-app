# Automated Git Tag Generation

## Overview
The CI/CD pipeline now automatically creates and pushes Git tags after successful deployment to CloudHub 2.0. This provides clear release tracking and traceability for each environment.

## Tag Naming Strategy

### Development Branch
**Format**: `v{version}-dev`
**Example**: `v1.0.8-dev`
**Trigger**: Merged PR to `development` branch

### UAT Branch
**Format**: `v{version}-uat`
**Example**: `v1.0.8-uat`
**Trigger**: Merged PR to `uat` branch

### Production (Main) Branch
**Format**: `v{version}`
**Example**: `v1.0.8`
**Trigger**: Merged PR to `main` branch
**Note**: Production tags have no suffix for clean release versioning

## Important: PR-Only Workflow
The pipeline **only triggers on merged pull requests**, not on direct pushes. This ensures:
- Code review before deployment
- Controlled releases through PR approval process
- No accidental deployments from direct commits

## How It Works

### Pipeline Flow
1. **Version Bump**: Automatically increments version (e.g., 1.0.7 → 1.0.8)
2. **Version Commit**: Commits new version with `[skip ci]` prefix
3. **Build & Publish**: Publishes artifact to Anypoint Exchange
4. **Deploy**: Deploys application to CloudHub 2.0 environment
5. **Tag Creation**: Creates annotated Git tag (NEW)
6. **Tag Push**: Pushes tag to GitHub repository (NEW)

### Tag Creation Details

**Location**: `.github/workflows/build.yml:149-180`

**What Happens**:
```bash
# Reads version from version file
VERSION=$(cat version)

# Determines tag name based on branch
# main → v1.0.8
# development → v1.0.8-dev
# uat → v1.0.8-uat

# Creates annotated tag with deployment info
git tag -a "v1.0.8-dev" -m "Release v1.0.8-dev - Deployed to Sandbox"

# Pushes tag to remote repository
git push origin "v1.0.8-dev"
```

## Tag Information

Each tag is an **annotated tag** containing:
- **Tag Name**: Version with environment suffix
- **Message**: Release version and deployment target environment
- **Tagger**: github-actions[bot]
- **Date**: Timestamp of successful deployment

Example tag message:
```
Release v1.0.8-dev - Deployed to Sandbox
```

## Viewing Tags

### GitHub Web Interface
Navigate to: `https://github.com/{org}/{repo}/tags`

### Command Line
```bash
# List all tags
git tag -l

# List tags with specific pattern
git tag -l "v1.0.*"
git tag -l "v*-dev"

# Show tag details
git show v1.0.8-dev

# Fetch all tags from remote
git fetch --tags
```

## Use Cases

### 1. Release Tracking
Quickly identify which version is deployed to each environment:
```bash
git tag -l "v*-dev"    # Development releases
git tag -l "v*-uat"    # UAT releases
git tag -l "v*" --exclude="*-*"  # Production releases only
```

### 2. Rollback Reference
Use tags to identify the exact commit for rollback:
```bash
# Show commit for specific tag
git show v1.0.7-dev

# Checkout code at specific tag
git checkout v1.0.7-dev
```

### 3. Release Notes
Generate release notes between tags:
```bash
# Changes between two versions
git log v1.0.7-dev..v1.0.8-dev --oneline

# Changes in production releases
git log v1.0.7..v1.0.8 --oneline
```

### 4. CI/CD Debugging
Correlate deployment issues with exact code version:
- Tag timestamp = deployment time
- Tag commit = exact deployed code

## Error Handling

The pipeline handles tag creation errors gracefully:

**Duplicate Tag**:
```bash
git tag -a "$TAG_NAME" -m "..." || echo "Tag already exists, skipping"
```
- If tag exists: Warning logged, pipeline continues
- Pipeline doesn't fail due to duplicate tags

**Push Failure**:
```bash
git push origin "$TAG_NAME" || echo "Tag push failed or tag already exists remotely"
```
- Network issues: Warning logged, pipeline continues
- Remote tag exists: Warning logged, pipeline continues

## Permissions Required

### GitHub Secrets
The workflow uses `GH_PAT` (GitHub Personal Access Token) for authentication:
- **Required Scopes**: `repo` (full repository access)
- **Used For**:
  - Committing version changes
  - Pushing tags to remote repository

**Important**: The same `GH_PAT` token used for version commits also handles tag pushing.

## Testing the Workflow

### Via Pull Request (Recommended)
The workflow only triggers on merged PRs. To test:

1. **Create a feature branch**:
   ```bash
   git checkout -b feature/test-tag-automation
   # Make some changes
   git add .
   git commit -m "test: Verify tag automation"
   git push origin feature/test-tag-automation
   ```

2. **Create a Pull Request**:
   - Go to GitHub and create PR from `feature/test-tag-automation` → `development`
   - Get approval and merge the PR

3. **Workflow triggers automatically**:
   - Bumps version (e.g., 1.0.7 → 1.0.8)
   - Commits version bump with `[skip ci]`
   - Publishes to Exchange
   - Deploys to CloudHub 2.0
   - Creates tag `v1.0.8-dev`
   - Pushes tag to GitHub

4. **Verify the tag was created**:
   ```bash
   git checkout development
   git pull
   git fetch --tags
   git tag -l "v*-dev"
   git show v1.0.8-dev
   ```

**Note**: Direct pushes to `development`, `uat`, or `main` will NOT trigger the workflow. Only merged PRs trigger deployments.

## Testing Locally

You can manually create and push tags using the same naming convention:

```bash
# Get current version
VERSION=$(cat version)

# Create development tag
git tag -a "v${VERSION}-dev" -m "Release v${VERSION}-dev - Manual test deployment"

# Push tag
git push origin "v${VERSION}-dev"

# Verify tag was created
git tag -l "v${VERSION}*"

# Delete tag if needed (local and remote)
git tag -d "v${VERSION}-dev"
git push origin --delete "v${VERSION}-dev"
```

## Tag Management

### Deleting Tags

**Delete Local Tag**:
```bash
git tag -d v1.0.8-dev
```

**Delete Remote Tag**:
```bash
git push origin --delete v1.0.8-dev
```

**Delete Both**:
```bash
git tag -d v1.0.8-dev && git push origin --delete v1.0.8-dev
```

### Listing Tags by Environment

```bash
# Development tags only
git tag -l "v*-dev" --sort=-version:refname

# UAT tags only
git tag -l "v*-uat" --sort=-version:refname

# Production tags only (no suffix)
git tag -l "v*.*.*" --sort=-version:refname | grep -v "-"

# Latest tag for each environment
git tag -l "v*-dev" --sort=-version:refname | head -1
git tag -l "v*-uat" --sort=-version:refname | head -1
git tag -l "v*.*.*" --sort=-version:refname | grep -v "-" | head -1
```

## Example Pipeline Output

```
==========================================
GIT TAG CREATION
==========================================
Branch: development
Version: 1.0.8
Tag Name: v1.0.8-dev
==========================================
Tag v1.0.8-dev created and pushed successfully
```

## Benefits

1. **Traceability**: Every deployment has a corresponding Git tag
2. **Rollback Safety**: Easy identification of previous stable versions
3. **Release Management**: Clear separation between environment releases
4. **Audit Trail**: Timestamps and metadata for compliance
5. **Automation**: No manual tag creation required
6. **Consistency**: Standardized naming across all environments

## Integration with Semantic Versioning

Current implementation uses automatic patch version bumping. Tags follow semantic versioning:

**Format**: `v{MAJOR}.{MINOR}.{PATCH}[-{ENVIRONMENT}]`

Examples:
- `v1.0.8-dev` → Major: 1, Minor: 0, Patch: 8, Env: dev
- `v1.0.8-uat` → Major: 1, Minor: 0, Patch: 8, Env: uat
- `v1.0.8` → Major: 1, Minor: 0, Patch: 8, Env: production

**Future Enhancement**: Could support major/minor version bumps via commit message conventions or manual triggers.

## Troubleshooting

### Tag Not Created
**Symptom**: Pipeline succeeds but no tag appears

**Check**:
1. Verify deployment step succeeded
2. Check pipeline logs for tag creation errors
3. Ensure `GH_PAT` token has `repo` scope
4. Verify branch name matches expected pattern

### Duplicate Tag Warning
**Symptom**: Log shows "Tag already exists, skipping"

**Cause**: Re-running pipeline for same version

**Solution**:
- Expected behavior - pipeline continues normally
- Delete old tag if you need to recreate it
- Version should increment on next commit

### Permission Denied
**Symptom**: "Permission denied" during tag push

**Cause**: `GH_PAT` token lacks permissions

**Solution**:
1. Verify token has `repo` scope
2. Check token hasn't expired
3. Regenerate token if necessary
4. Update `GH_PAT` secret in repository settings

## Future Enhancements

Potential improvements to tag automation:

1. **Release Notes**: Auto-generate release notes in tag message
2. **GitHub Releases**: Create GitHub release objects from tags
3. **Tag Validation**: Verify tag doesn't exist before deployment
4. **Custom Messages**: Include commit messages or PR info in tag
5. **Tag Cleanup**: Automatic deletion of old dev/uat tags
6. **Semantic Versioning**: Support major/minor bumps via commit conventions
