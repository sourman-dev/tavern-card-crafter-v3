# Phase 5: Testing & Validation

**Parent Plan**: [Vietnamese Language Support](./plan.md)
**Status**: Pending | **Priority**: P1 | **Effort**: 30min
**Date**: 2025-12-28

## Overview

Comprehensive testing of Vietnamese language support across all features: UI elements, AI generation, token estimation, PNG export/import, and layout rendering.

## Key Insights from Research

Critical test scenarios based on `research/researcher-01-vietnamese-i18n-patterns.md`:

1. **Text Length**: Vietnamese 20-30% longer than English - test UI overflow
2. **Diacritics**: Test rendering at small sizes, check for clipping
3. **AI Responses**: Verify all prompts return Vietnamese, not English
4. **Token Accuracy**: Verify estimation within 10% of actual

## Requirements

1. Test all UI elements display Vietnamese correctly
2. Test all 10 AI generation functions return Vietnamese responses
3. Verify PNG export/import preserves Vietnamese text
4. Check layout doesn't break with longer Vietnamese strings
5. Verify token estimation accuracy
6. Test language switching persistence (localStorage)

## Architecture Considerations

- **Testing Strategy**: Manual QA (no automated tests in this phase)
- **AI Provider**: Test with at least one provider (OpenAI/Anthropic/Ollama)
- **Browser Testing**: Chrome/Safari (Electron uses Chromium)
- **Focus Areas**: UI overflow, diacritics rendering, AI quality

## Related Files

| Component | Test Focus |
|-----------|------------|
| `src/contexts/LanguageContext.tsx` | UI translation, localStorage |
| `src/utils/aiGenerator.ts` | AI prompts, token estimation |
| `src/components/CharacterForm/*` | Layout with Vietnamese text |
| `src/components/CharacterPreview.tsx` | JSON preview, token display |
| `src/components/Toolbar.tsx` | Language selector |
| PNG export/import | Vietnamese text preservation |

## Implementation Steps

### Step 1: UI Translation Testing

**Test Plan**:

1. **Language Selector**:
   - [ ] Switch to "Tiếng Việt"
   - [ ] Verify all labels/buttons update to Vietnamese
   - [ ] Check no English fallbacks visible

2. **All Sections**:
   - [ ] Basic Info: "Thông tin cơ bản"
   - [ ] Personality: "Tính cách"
   - [ ] Prompts: "Các lời nhắc"
   - [ ] Alternate Greetings: "Lời chào thay thế"
   - [ ] Character Book: "Sách nhân vật"
   - [ ] Tags: "Thẻ tag"
   - [ ] Metadata: "Siêu dữ liệu"

3. **Buttons**:
   - [ ] "Nhập thẻ" (Import Card)
   - [ ] "Cài đặt AI" (AI Settings)
   - [ ] "Sao chép" (Copy)
   - [ ] "Xuất JSON" (Export JSON)
   - [ ] "Xuất PNG" (Export PNG)

4. **Messages/Toasts**:
   - [ ] Trigger success: "Sao chép thành công"
   - [ ] Trigger error: "Nhập thất bại"

### Step 2: Layout Testing with Long Vietnamese Text

**Test Plan**:

Fill all fields with long Vietnamese text to test overflow:

```
Name: "Nguyễn Thị Hương Linh"
Description: "Một chiến binh dũng cảm với mái tóc dài đen nhánh, đôi mắt sắc sảo và vóc dáng thanh mảnh nhưng rất khỏe mạnh. Cô luôn mặc áo giáp truyền thống màu đỏ thẫm với họa tiết rồng vàng, tượng trưng cho sức mạnh và lòng dũng cảm."
```

**Check**:
- [ ] No text overflow in input fields
- [ ] No horizontal scroll in containers
- [ ] Diacritics not clipped (check á, ả, ã, ạ at top/bottom)
- [ ] Line height sufficient (≥1.5)
- [ ] Mobile responsive (narrow viewport)

### Step 3: Diacritics Rendering Testing

**Test Plan**:

Test all Vietnamese diacritics render correctly:

**Vowels with tones**:
```
a: à á ả ã ạ
ă: ắ ằ ẳ ẵ ặ
â: ấ ầ ẩ ẫ ậ
e: è é ẻ ẽ ẹ
ê: ế ề ể ễ ệ
i: ì í ỉ ĩ ị
o: ò ó ỏ õ ọ
ô: ố ồ ổ ỗ ộ
ơ: ớ ờ ở ỡ ợ
u: ù ú ủ ũ ụ
ư: ứ ừ ử ữ ự
y: ỳ ý ỷ ỹ ỵ
d: đ Đ
```

**Check**:
- [ ] All diacritics visible (no "tofu" �)
- [ ] No clipping at small font sizes
- [ ] No clipping in buttons/badges
- [ ] Correct vertical spacing

### Step 4: AI Generation Testing

**Test Plan**: Test all 10 AI prompt generators

**Setup**:
1. Configure AI settings (OpenAI/Anthropic/Ollama)
2. Set language to Vietnamese (`vi`)
3. Fill basic character info:
   ```
   Name: Linh
   Description: Một chiến binh dũng cảnh
   ```

**Test Each Generator**:

1. **Description**:
   - [ ] Click "Tạo bằng AI"
   - [ ] Verify response in Vietnamese
   - [ ] Check length ~100-150 words
   - [ ] Verify no English text

2. **Personality**:
   - [ ] Generate
   - [ ] Verify Vietnamese response
   - [ ] Check length ~80-120 words

3. **Scenario**:
   - [ ] Generate
   - [ ] Verify Vietnamese response
   - [ ] Check "historian's prose" style adapted

4. **First Message**:
   - [ ] Generate
   - [ ] Verify Vietnamese response
   - [ ] Check length ~150-200 words
   - [ ] Verify no Western author references in output

5. **Message Example**:
   - [ ] Generate
   - [ ] Verify Vietnamese response
   - [ ] Check `<START>` macro present
   - [ ] Verify format: `Linh: *hành động*`

6. **System Prompt**:
   - [ ] Generate
   - [ ] Verify Vietnamese response
   - [ ] Check brevity (~60-100 words)

7. **Post History Instructions**:
   - [ ] Generate
   - [ ] Verify Vietnamese response
   - [ ] Check extreme brevity (~30-50 words)

8. **Tags**:
   - [ ] Generate
   - [ ] Verify Vietnamese tags
   - [ ] Check 8-15 tags returned
   - [ ] Verify comma-separated

9. **Alternate Greeting**:
   - [ ] Generate (requires first_mes filled)
   - [ ] Verify Vietnamese response
   - [ ] Check "new chapter" structure

10. **Character Book Entry**:
    - [ ] Generate
    - [ ] Verify format: "Từ khóa: ..., Nội dung: ..."
    - [ ] Check 3-5 keywords
    - [ ] Check content ~80-120 words

### Step 5: Token Estimation Testing

**Test Plan**:

1. **Pure Vietnamese Text**:
   ```
   Input: "Xin chào thế giới"
   Expected: ~5 tokens (4 words * 1.3)
   Actual: [check JSON preview]
   ```

2. **Pure English Text**:
   ```
   Input: "Hello world"
   Expected: ~2 tokens (2 words * 1.0)
   Actual: [check JSON preview]
   ```

3. **Pure Chinese Text**:
   ```
   Input: "你好世界"
   Expected: ~6 tokens (4 chars * 1.5)
   Actual: [check JSON preview]
   ```

4. **Mixed Text**:
   ```
   Input: "Hello xin chào 你好"
   Expected: ~9 tokens
   Actual: [check JSON preview]
   ```

5. **Real Character Card**:
   - [ ] Fill complete card in Vietnamese
   - [ ] Note estimated tokens
   - [ ] Compare with actual tokens from AI provider
   - [ ] Verify accuracy within 10%

### Step 6: PNG Export/Import Testing

**Test Plan**:

1. **Export**:
   - [ ] Create character card in Vietnamese
   - [ ] Upload avatar image
   - [ ] Click "Xuất PNG"
   - [ ] Verify PNG downloads

2. **Verify PNG**:
   - [ ] Open PNG in image viewer
   - [ ] Verify image looks correct
   - [ ] Check file size reasonable

3. **Import**:
   - [ ] Clear current card (reload page)
   - [ ] Click "Nhập thẻ"
   - [ ] Select exported PNG
   - [ ] Verify all Vietnamese text preserved
   - [ ] Check diacritics not corrupted
   - [ ] Verify structure intact

### Step 7: localStorage Persistence Testing

**Test Plan**:

1. **Set Language**:
   - [ ] Select "Tiếng Việt"
   - [ ] Verify UI switches to Vietnamese

2. **Reload Page**:
   - [ ] Refresh browser
   - [ ] Verify language still Vietnamese
   - [ ] Check localStorage: `localStorage.getItem('language')` = `'vi'`

3. **Switch Languages**:
   - [ ] Switch to English
   - [ ] Reload
   - [ ] Verify English persists
   - [ ] Switch back to Vietnamese
   - [ ] Verify Vietnamese persists

### Step 8: Cross-Browser Testing (Optional)

If time permits:
- [ ] Chrome/Chromium
- [ ] Safari
- [ ] Firefox
- [ ] Electron app

### Step 9: Mobile Responsive Testing

**Test Plan**:

1. **Resize Browser**:
   - [ ] Set viewport to 375px width (iPhone)
   - [ ] Verify all Vietnamese text visible
   - [ ] Check no horizontal scroll
   - [ ] Verify buttons accessible

2. **Test Touch Targets**:
   - [ ] Buttons ≥44px tall
   - [ ] Input fields easily tappable

## Todo List

- [ ] Test all UI elements display Vietnamese
- [ ] Test layout with long Vietnamese text (no overflow)
- [ ] Test all Vietnamese diacritics render correctly
- [ ] Test all 10 AI generators return Vietnamese responses
- [ ] Test token estimation accuracy (within 10%)
- [ ] Test PNG export preserves Vietnamese text
- [ ] Test PNG import restores Vietnamese text
- [ ] Test localStorage persists language selection
- [ ] Test mobile responsive layout
- [ ] Document any issues found

## Success Criteria

1. ✅ All 70+ UI strings display Vietnamese
2. ✅ No layout overflow with Vietnamese text
3. ✅ All diacritics render correctly (no clipping)
4. ✅ All 10 AI generators return Vietnamese responses
5. ✅ Token estimation within 10% of actual
6. ✅ PNG export/import preserves Vietnamese text
7. ✅ Language selection persists across sessions
8. ✅ Mobile layout functional

## Bug Tracking

Document any issues found:

| Issue | Severity | Component | Status |
|-------|----------|-----------|--------|
| Example: Text overflow in tags section | Medium | TagsSection.tsx | - |

## Risk Assessment

**Medium Risk**:
- AI responses may be lower quality in Vietnamese
- Diacritics may not render on some systems

**Mitigation**:
- Test on multiple systems
- Refine prompts if quality issues
- Add font fallbacks if rendering issues

## Security Considerations

- Verify PNG import doesn't execute malicious code (existing security, not new)
- No new security risks

## Next Steps

1. **If all tests pass**:
   - Create PR: `vietnamese` → `prime`
   - Update README with Vietnamese language support

2. **If issues found**:
   - Document issues
   - Fix critical bugs
   - Re-test
   - Then create PR

## Notes

- **AI Quality**: Most critical test - Vietnamese responses must be natural
- **Iterative**: May need multiple rounds of prompt refinement
- **Documentation**: Update README to mention Vietnamese support
- **Future**: Consider react-i18next migration if adding more languages

## Test Report Template

```markdown
# Vietnamese i18n Testing Report
Date: YYYY-MM-DD
Tester: [Name]

## UI Translation: ✅ PASS / ❌ FAIL
- [Notes]

## Layout (Long Text): ✅ PASS / ❌ FAIL
- [Notes]

## Diacritics Rendering: ✅ PASS / ❌ FAIL
- [Notes]

## AI Generation (10/10): ✅ PASS / ❌ FAIL
- Description: ✅
- Personality: ✅
- Scenario: ✅
- First Message: ✅
- Message Example: ✅
- System Prompt: ✅
- Post History: ✅
- Tags: ✅
- Alternate Greeting: ✅
- Character Book: ✅

## Token Estimation: ✅ PASS / ❌ FAIL
- Vietnamese: Estimated X, Actual Y (±Z%)
- English: Estimated X, Actual Y (±Z%)
- Chinese: Estimated X, Actual Y (±Z%)

## PNG Export/Import: ✅ PASS / ❌ FAIL
- [Notes]

## localStorage: ✅ PASS / ❌ FAIL
- [Notes]

## Mobile Responsive: ✅ PASS / ❌ FAIL
- [Notes]

## Issues Found:
1. [Issue 1]
2. [Issue 2]

## Overall: ✅ READY FOR PR / ❌ NEEDS FIXES
```
