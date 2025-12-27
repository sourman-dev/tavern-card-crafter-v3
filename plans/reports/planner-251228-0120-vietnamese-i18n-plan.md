# Vietnamese i18n Implementation Plan - Planning Report

**Date**: 2025-12-28 01:20
**Planner**: planner agent
**Plan Directory**: `plans/251228-0111-vietnamese-i18n/`

## Summary

Created comprehensive implementation plan for Vietnamese language support in Tavern Card Crafter. Plan addresses critical requirement: all AI prompts must be translated and culturally adapted for Vietnamese.

## Plan Structure

```
plans/251228-0111-vietnamese-i18n/
├── plan.md (master plan with YAML frontmatter)
├── phase-01-git-branch-setup.md (15min)
├── phase-02-languagecontext-vietnamese.md (2h)
├── phase-03-ai-prompt-translation.md (3h) ⚠️ CRITICAL
├── phase-04-token-estimation.md (30min)
└── phase-05-testing-validation.md (30min)
```

**Total Effort**: 6 hours

## Key Deliverables

### Phase 1: Git Branch Setup (15min)
- Create `vietnamese` branch from `prime`
- Push to remote
- Update session state

### Phase 2: LanguageContext Vietnamese (2h)
- Add `vi` to type unions (`'zh' | 'en' | 'vi'`)
- Translate 70+ UI strings to Vietnamese
- Verify font support for diacritics
- Test layout with longer Vietnamese text (20-30% longer than English)

**Sample Translations**:
```typescript
vi: {
  pageTitle: 'Trình tạo thẻ nhân vật SillyTavern V3',
  aiGenerate: 'Tạo bằng AI',
  characterInfo: 'Chỉnh sửa thông tin nhân vật',
  // ... 67 more keys
}
```

### Phase 3: AI Prompt Translation (3h) ⚠️ CRITICAL
**Most important phase** - ensures AI responses in Vietnamese.

**10 Prompt Generators Updated**:
1. `generateDescription()` - Physical appearance
2. `generatePersonality()` - Character traits
3. `generateScenario()` - Backstory/environment
4. `generateFirstMes()` - First encounter
5. `generateMesExample()` - Dialogue examples
6. `generateSystemPrompt()` - AI instructions
7. `generatePostHistoryInstructions()` - Brief directives
8. `generateTags()` - Keywords (8-15 tags vs 5-10)
9. `generateAlternateGreeting()` - New chapter
10. `generateCharacterBookEntry()` - Lorebook entries

**Key Adaptations**:
- **Character count adjustments**: 20 Chinese chars → 60-80 Vietnamese chars
- **Literary references**: Removed Western authors (Douglas Adams, Philip K. Dick) → genre descriptors
- **Word count guidance**: Added explicit limits (e.g., "100-150 từ")
- **Cultural sensitivity**: "historian's prose" → "văn phong chuyên nghiệp và hấp dẫn"

**Example Transformation**:
```typescript
// ENGLISH (original)
"The writing should be a perfect combination of Douglas Adams,
Ursula K. Le Guin, James Joyce, Anais Nin, and Philip K. Dick."

// VIETNAMESE (adapted)
"Văn phong nên sáng tạo, hấp dẫn, kết hợp giữa yếu tố kể chuyện
sinh động và mô tả tâm lý tinh tế, giống như phong cách của các
nhà văn nổi tiếng về khoa học viễn tưởng và văn học hiện đại.
Độ dài khoảng 150-200 từ."
```

### Phase 4: Token Estimation (30min)
Update `estimateTokens()` function:

**Current**:
- Chinese: 1.5x tokens/char
- English: 1.0x tokens/word
- Other: 0.5x tokens/char

**Updated**:
- Vietnamese: 1.3x tokens/word (research: 1.2-1.4 range)
- Detection: Vietnamese diacritics regex
- Backward compatible with Chinese/English

### Phase 5: Testing & Validation (30min)
Comprehensive manual QA:
- ✅ All 70+ UI strings in Vietnamese
- ✅ No layout overflow with longer text
- ✅ Diacritics render correctly (à, á, ả, ã, ạ, etc.)
- ✅ All 10 AI generators return Vietnamese responses
- ✅ Token estimation within 10% accuracy
- ✅ PNG export/import preserves Vietnamese
- ✅ localStorage persists language selection

## Research Integration

Plan based on research from `researcher-01-vietnamese-i18n-patterns.md`:

| Finding | Implementation |
|---------|----------------|
| Vietnamese text 20-30% longer | Test layout overflow, adjust Tailwind |
| Token bloat 30-50% | Apply 1.3x multiplier in `estimateTokens()` |
| Character constraints (20 chars) | Multiply by 2-3x (→ 60-80 chars) |
| Western literary references | Replace with Vietnamese/genre descriptors |
| Diacritics rendering | Verify line-height ≥ 1.5, test fonts |

## Files Modified

| File | Changes | Est. Lines |
|------|---------|------------|
| `src/contexts/LanguageContext.tsx` | Add `vi` translations, update types | 195 → 290 |
| `src/utils/aiGenerator.ts` | Translate 10 prompts, add language param | 360 → 420 |
| `src/utils/aiGenerator.ts:21-28` | Update `estimateTokens()` | 8 → 35 |
| Components calling AI functions | Add `language` parameter | ~10 files |

## Success Criteria

1. ✅ All UI displays Vietnamese when `language='vi'`
2. ✅ All AI prompts generate Vietnamese responses
3. ✅ Token estimation accurate (±10%)
4. ✅ Character constraints adjusted for Vietnamese length
5. ✅ PNG export/import works with Vietnamese
6. ✅ Layout functional with longer Vietnamese text

## Risks & Mitigations

| Risk | Severity | Mitigation |
|------|----------|------------|
| Vietnamese text overflow | Medium | Test responsive breakpoints, adjust Tailwind |
| Diacritics rendering issues | Low | Verify shadcn/ui fonts, add fallbacks |
| AI output quality degradation | Medium | Test multiple providers, iterate prompts |
| Token estimation inaccuracy | Low | Compare with actual counts, adjust multiplier |

## Unresolved Questions

1. Does shadcn/ui default font (Inter) render Vietnamese diacritics correctly at all sizes?
   - **Mitigation**: Test in Phase 2, add font fallback if needed

2. Will Vietnamese prompts produce comparable quality to English prompts?
   - **Mitigation**: Test in Phase 5, iterate prompt wording

3. How will PNG export handle increased text length without layout breaking?
   - **Mitigation**: Test in Phase 5, adjust layout if needed

## Next Steps

1. Review plan with user
2. Execute Phase 1: Create `vietnamese` branch
3. Proceed sequentially through phases 2-5
4. Create PR: `vietnamese` → `prime` after successful testing

## Notes

- **KISS/DRY Adherence**: Keep custom LanguageContext (no react-i18next), use parameterized prompts (not duplicated _vi functions)
- **Cultural Sensitivity**: Removed Western-centric literary references, adapted to Vietnamese context
- **Token Efficiency**: Plan files under 300 lines each, extensive file:line references
- **Quality Focus**: Phase 3 (AI prompts) is critical - most time allocated here

---

**Plan Location**: `/Users/uspro/Projects/mianix-v2/tavern-card-crafter-v3/plans/251228-0111-vietnamese-i18n/plan.md`

**Ready for Implementation**: Yes ✅
