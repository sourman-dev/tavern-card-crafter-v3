# Phase 1: Git Branch Setup

**Parent Plan**: [Vietnamese Language Support](./plan.md)
**Status**: Pending | **Priority**: P1 | **Effort**: 15min
**Date**: 2025-12-28

## Overview

Create a new `vietnamese` branch from the current `prime` branch to isolate the Vietnamese i18n implementation. This ensures the main branch remains stable while development proceeds.

## Key Insights from Research

- No code changes in this phase
- Clean branch strategy follows git best practices
- Enables parallel development if needed

## Requirements

1. Create `vietnamese` branch from `prime`
2. Verify branch is clean (no uncommitted changes)
3. Push branch to remote for backup
4. Update session state to track active plan

## Architecture Considerations

- **Branching Strategy**: Feature branch workflow
- **Base Branch**: `prime` (current HEAD: 9a9e4c2)
- **Merge Strategy**: Will create PR to `prime` after Phase 5 completion

## Related Files

- `.git/` - Git repository
- Current branch: `prime` (ref: remotes/origin/prime)

## Implementation Steps

### Step 1: Verify Clean Working Directory

```bash
git status
```

**Expected**: No uncommitted changes. If changes exist, stash or commit first.

### Step 2: Create Vietnamese Branch

```bash
git checkout -b vietnamese
```

**Creates new branch**: `vietnamese` from current `prime` HEAD (9a9e4c2)

### Step 3: Verify Branch Creation

```bash
git branch
git log --oneline -n 3
```

**Verify**:
- `* vietnamese` is current branch
- HEAD matches `prime` (9a9e4c2 "Add images for AI results...")

### Step 4: Push Branch to Remote

```bash
git push -u origin vietnamese
```

**Creates remote tracking**: Sets upstream to `origin/vietnamese`

### Step 5: Update Session State

```bash
node .claude/scripts/set-active-plan.cjs plans/251228-0111-vietnamese-i18n
```

**Updates context**: Future subagents will receive plan context

## Todo List

- [ ] Verify working directory clean (`git status`)
- [ ] Create `vietnamese` branch from `prime`
- [ ] Verify branch HEAD matches `prime`
- [ ] Push branch to remote with upstream tracking
- [ ] Update session state with active plan path

## Success Criteria

1. ✅ Branch `vietnamese` exists locally
2. ✅ Branch `vietnamese` pushed to remote
3. ✅ Current branch is `vietnamese`
4. ✅ HEAD matches `prime` branch (9a9e4c2)
5. ✅ Session state updated with plan path

## Verification

```bash
# Verify branch
git branch -vv | grep vietnamese

# Expected output:
# * vietnamese 9a9e4c2 [origin/vietnamese] Add images for AI results...

# Verify remote
git remote show origin | grep vietnamese

# Expected output:
#   vietnamese pushes to vietnamese (up to date)
```

## Risk Assessment

**Low Risk**:
- Standard git operation
- No code changes
- Easy rollback (delete branch if needed)

**Mitigation**:
- Verify clean working directory before branching
- Push to remote immediately for backup

## Next Steps

→ Proceed to [Phase 2: LanguageContext Vietnamese Translation](./phase-02-languagecontext-vietnamese.md)

## Notes

- Keep branch up to date with `prime` via periodic rebases if needed
- After Phase 5, will create PR: `vietnamese` → `prime`
