---
title: "Vietnamese Language Support for Tavern Card Crafter"
description: "Add complete Vietnamese i18n support including UI labels, AI prompts, and token estimation"
status: pending
priority: P1
effort: 6h
branch: prime
tags: [i18n, vietnamese, ai-prompts, localization]
created: 2025-12-28
---

# Vietnamese Language Support Implementation Plan

## Overview

Add comprehensive Vietnamese language support to Tavern Card Crafter, including:
- UI translation (all labels, buttons, messages)
- AI prompt translation with cultural adaptation
- Token estimation adjustments for Vietnamese
- Character length constraint modifications

**Critical Requirement**: All AI prompts must be translated and adjusted for Vietnamese linguistic characteristics (character length differences, token multipliers, cultural references).

## Research Context

Based on research reports in `./research/`:
- Vietnamese text is 20-30% longer than English
- Vietnamese tokens consume 30-50% more than English
- Requires token multiplier: 1.2-1.4 (vs 1.5 for Chinese)
- Character constraints need 2-3x adjustment (20 Chinese chars ≈ 60-80 Vietnamese chars)
- Diacritics require font support and larger line-height (1.5+)
- Cultural adaptation needed for Western literary references

## Implementation Phases

### [Phase 1: Git Branch Setup](./phase-01-git-branch-setup.md)
**Status**: Pending | **Effort**: 15min
- Create `vietnamese` branch from `prime`
- Set up plan structure

### [Phase 2: LanguageContext Vietnamese Translation](./phase-02-languagecontext-vietnamese.md)
**Status**: Pending | **Effort**: 2h
- Add `vi` language type
- Translate 70+ UI strings to Vietnamese
- Update localStorage and type unions
- Verify font rendering for diacritics

### [Phase 3: AI Prompt Translation & Adaptation](./phase-03-ai-prompt-translation.md)
**Status**: Pending | **Effort**: 3h
- Translate 10 prompt generator functions
- Adjust character/token count constraints (2-3x)
- Adapt literary references to Vietnamese context
- Add language-aware prompt selection

### [Phase 4: Token Estimation Update](./phase-04-token-estimation.md)
**Status**: Pending | **Effort**: 30min
- Update `estimateTokens()` to detect Vietnamese
- Apply Vietnamese multiplier (1.2-1.4)
- Test accuracy

### [Phase 5: Testing & Validation](./phase-05-testing-validation.md)
**Status**: Pending | **Effort**: 30min
- Test all UI elements in Vietnamese
- Verify AI responses return Vietnamese
- Test PNG export/import with Vietnamese text
- Check layout with longer text

## File Structure

```
plans/251228-0111-vietnamese-i18n/
├── plan.md (this file)
├── research/
│   └── researcher-01-vietnamese-i18n-patterns.md
├── phase-01-git-branch-setup.md
├── phase-02-languagecontext-vietnamese.md
├── phase-03-ai-prompt-translation.md
├── phase-04-token-estimation.md
└── phase-05-testing-validation.md
```

## Success Criteria

1. ✅ All UI elements display Vietnamese when language set to `vi`
2. ✅ All AI prompts generate Vietnamese responses
3. ✅ Token estimation accurate for Vietnamese (within 10% of actual)
4. ✅ Character constraints adjusted for Vietnamese length
5. ✅ PNG export/import works with Vietnamese text
6. ✅ Layout does not break with longer Vietnamese strings

## Key Files Modified

| File | Changes | Lines |
|------|---------|-------|
| `src/contexts/LanguageContext.tsx` | Add `vi` translations, update types | ~195→290 |
| `src/utils/aiGenerator.ts` | Translate prompts, adjust constraints | ~360→420 |
| `src/utils/aiGenerator.ts:21-28` | Update `estimateTokens()` | ~8→20 |

## Dependencies

- None (all changes within existing codebase)

## Risks

1. **Vietnamese text overflow**: Longer text may break UI layout
   - Mitigation: Test responsive breakpoints, adjust Tailwind classes
2. **Diacritics rendering**: Font may not support Vietnamese glyphs
   - Mitigation: Verify shadcn/ui default fonts, add fallback fonts if needed
3. **AI output quality**: Translated prompts may produce lower quality responses
   - Mitigation: Test with multiple AI providers, refine prompts iteratively

## Security Considerations

- No security impact (localization only)
- No new dependencies or external APIs

## Next Steps

1. Execute Phase 1: Create `vietnamese` branch
2. Proceed sequentially through phases 2-5
3. Test thoroughly before merging to `prime`

## Validation Summary

**Validated:** 2025-12-28
**Questions Asked:** 6

### Confirmed Decisions

1. **Token Multiplier**: Use 1.3x for Vietnamese (middle of 1.2-1.4 research range)
2. **Length Constraints**: Switch from character count to word count (e.g., "100-150 từ")
   - Avoids confusion between Vietnamese chars vs Chinese chars
   - More intuitive for Vietnamese users
3. **Literary References**: Use Vietnamese novel genres instead of author names
   - Genres: "kiếm hiệp" (martial arts), "tiên hiệp" (xianxia), "đồng nhân" (fanfic), "ngôn tình" (romance)
   - Remove Western author names (Douglas Adams, Philip K. Dick)
   - Use genre-based style guidance
4. **Prompt Architecture**: Parameterized approach (single function with language parameter)
   - DRY principle
   - Maintainable and scalable
5. **AI Testing**: Test with OpenAI Completion API only
   - User's primary provider
   - Focus testing effort on production environment

### Action Items

- [x] Update Phase 3 to use word count instead of character count
- [x] Update Phase 3 to reference Vietnamese novel genres (kiếm hiệp, tiên hiệp, đồng nhân, ngôn tình)
- [x] Update Phase 4 to confirm 1.3x multiplier
- [x] Update Phase 5 to focus on OpenAI testing
- [ ] Research additional Vietnamese novel genres for comprehensive coverage
- [ ] Create examples of genre-specific prompts during implementation

### Notes

- User prefers Vietnamese novel genre references over Chinese web novel authors
- Word count approach ("từ") is clearer than character count for Vietnamese
- Focus on OpenAI compatibility simplifies testing scope

## References

- Research: `plans/251228-0111-vietnamese-i18n/research/researcher-01-vietnamese-i18n-patterns.md`
- Current i18n: `src/contexts/LanguageContext.tsx`
- AI prompts: `src/utils/aiGenerator.ts:185-359`
- Development rules: `.claude/workflows/development-rules.md`
