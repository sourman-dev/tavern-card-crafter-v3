# Phase 4: Token Estimation Update

**Parent Plan**: [Vietnamese Language Support](./plan.md)
**Status**: Pending | **Priority**: P2 | **Effort**: 30min
**Date**: 2025-12-28

## Overview

Update the `estimateTokens()` function in `aiGenerator.ts` to accurately estimate token counts for Vietnamese text. This ensures the "Total Tokens" display in the JSON preview is accurate.

## Key Insights from Research

From `research/researcher-01-vietnamese-i18n-patterns.md`:

1. **Token Bloat**: Vietnamese uses 30-50% more tokens than English
2. **Recommended Multiplier**: 1.2-1.4 (vs current 1.5 for Chinese)
3. **Character Detection**: Vietnamese uses Latin alphabet with diacritics
4. **Current Estimation**: Only handles Chinese (1.5x) and English (1x)

## Requirements

1. Detect Vietnamese characters (Latin alphabet + Vietnamese diacritics)
2. Apply Vietnamese-specific token multiplier (1.3x - middle of 1.2-1.4 range)
3. Maintain backward compatibility with Chinese and English
4. Keep function simple (KISS principle)

## Architecture Considerations

- **Pattern**: Extend existing regex-based approach
- **Detection**: Vietnamese diacritics: `àáảãạăắằẳẵặâấầẩẫậèéẻẽẹêếềểễệìíỉĩịòóỏõọôốồổỗộơớờởỡợùúủũụưứừửữựỳýỷỹỵđ`
- **Multiplier**: 1.3x (research suggests 1.2-1.4, use middle value)
- **Precision**: Rough estimation acceptable (current function already rough)

## Related Files

| File | Lines | Purpose |
|------|-------|---------|
| `src/utils/aiGenerator.ts:21-28` | 8 | `estimateTokens()` function |

## Implementation Steps

### Step 1: Understand Current Implementation

**File**: `src/utils/aiGenerator.ts:21-28`

```typescript
export const estimateTokens = (text: string): number => {
  // Press 1 in Chinese characters 5 tokens are calculated, English words are calculated based on average 4 characters
  const chineseChars = (text.match(/[\u4e00-\u9fff]/g) || []).length;
  const englishWords = (text.match(/[a-zA-Z]+/g) || []).length;
  const otherChars = text.length - chineseChars - (text.match(/[a-zA-Z]/g) || []).length;

  return Math.ceil(chineseChars * 1.5 + englishWords + otherChars * 0.5);
};
```

**Current Logic**:
- Chinese chars: 1.5 tokens each
- English words: 1 token each
- Other chars: 0.5 tokens each

**Problem**: Vietnamese uses Latin alphabet (counted as English words) but has higher token cost.

### Step 2: Add Vietnamese Character Detection

Update function to detect Vietnamese diacritics:

```typescript
export const estimateTokens = (text: string): number => {
  // Character detection patterns
  const chineseChars = (text.match(/[\u4e00-\u9fff]/g) || []).length;

  // Vietnamese diacritics: à á ả ã ạ ă ắ ằ ẳ ẵ ặ â ấ ầ ẩ ẫ ậ è é ẻ ẽ ẹ ê ế ề ể ễ ệ ì í ỉ ĩ ị ò ó ỏ õ ọ ô ố ồ ổ ỗ ộ ơ ớ ờ ở ỡ ợ ù ú ủ ũ ụ ư ứ ừ ử ữ ự ỳ ý ỷ ỹ ỵ đ Đ
  const vietnameseDiacritics = /[àáảãạăắằẳẵặâấầẩẫậèéẻẽẹêếềểễệìíỉĩịòóỏõọôốồổỗộơớờởỡợùúủũụưứừửữựỳýỷỹỵđĐÀÁẢÃẠĂẮẰẲẴĂÂẤẦẨẪẬÈÉẺẼẸÊẾỀỂỄỆÌÍỈĨỊÒÓỎÕỌÔỐỒỔỖỘƠỚỜỞỠỢÙÚỦŨỤƯỨỪỬỮỰỲÝỶỸỴ]/g;
  const vietnameseChars = (text.match(vietnameseDiacritics) || []).length;

  // All Latin alphabet characters
  const allLatinChars = (text.match(/[a-zA-Z]/g) || []).length;

  // Vietnamese words = words containing Vietnamese diacritics + proportional share
  // Simplified: if text has Vietnamese chars, count all Latin words as Vietnamese
  const hasVietnamese = vietnameseChars > 0;
  const latinWords = (text.match(/[a-zA-Z\u00C0-\u1EF9]+/g) || []).length;

  let vietnameseWords = 0;
  let englishWords = 0;

  if (hasVietnamese) {
    // If Vietnamese detected, count all Latin words as Vietnamese
    vietnameseWords = latinWords;
  } else {
    // Pure English text
    englishWords = latinWords;
  }

  // Other characters (punctuation, spaces, etc.)
  const otherChars = text.length - chineseChars - allLatinChars - vietnameseChars;

  // Token estimation:
  // - Chinese: 1.5 tokens/char
  // - Vietnamese: 1.3 tokens/word (research: 1.2-1.4)
  // - English: 1.0 tokens/word
  // - Other: 0.5 tokens/char
  return Math.ceil(
    chineseChars * 1.5 +
    vietnameseWords * 1.3 +
    englishWords * 1.0 +
    otherChars * 0.5
  );
};
```

### Step 3: Optimize for Clarity (DRY)

Refactor for readability:

```typescript
export const estimateTokens = (text: string): number => {
  // Unicode ranges and patterns
  const CHINESE_REGEX = /[\u4e00-\u9fff]/g;
  const VIETNAMESE_DIACRITICS_REGEX = /[àáảãạăắằẳẵặâấầẩẫậèéẻẽẹêếềểễệìíỉĩịòóỏõọôốồổỗộơớờởỡợùúủũụưứừửữựỳýỷỹỵđĐÀÁẢÃẠĂẮẰẲẴĂÂẤẦẨẪẬÈÉẺẼẸÊẾỀỂỄỆÌÍỈĨỊÒÓỎÕỌÔỐỒỔỖỘƠỚỜỞỠỢÙÚỦŨỤƯỨỪỬỮỰỲÝỶỸỴ]/g;
  const LATIN_WORD_REGEX = /[a-zA-Z\u00C0-\u1EF9]+/g;

  // Character counting
  const chineseChars = (text.match(CHINESE_REGEX) || []).length;
  const vietnameseChars = (text.match(VIETNAMESE_DIACRITICS_REGEX) || []).length;
  const latinWords = (text.match(LATIN_WORD_REGEX) || []).length;

  // Language detection
  const hasVietnamese = vietnameseChars > 0;
  const vietnameseWords = hasVietnamese ? latinWords : 0;
  const englishWords = hasVietnamese ? 0 : latinWords;

  // Count remaining characters
  const allLatinChars = (text.match(/[a-zA-Z]/g) || []).length;
  const otherChars = text.length - chineseChars - allLatinChars - vietnameseChars;

  // Token multipliers based on research
  const CHINESE_MULTIPLIER = 1.5;
  const VIETNAMESE_MULTIPLIER = 1.3; // Research: 1.2-1.4, using middle
  const ENGLISH_MULTIPLIER = 1.0;
  const OTHER_MULTIPLIER = 0.5;

  return Math.ceil(
    chineseChars * CHINESE_MULTIPLIER +
    vietnameseWords * VIETNAMESE_MULTIPLIER +
    englishWords * ENGLISH_MULTIPLIER +
    otherChars * OTHER_MULTIPLIER
  );
};
```

### Step 4: Add JSDoc Documentation

```typescript
/**
 * Estimates token count for mixed-language text.
 *
 * Token multipliers:
 * - Chinese characters: 1.5 tokens/char
 * - Vietnamese words: 1.3 tokens/word (research: Vietnamese uses 30-50% more tokens than English)
 * - English words: 1.0 tokens/word
 * - Other characters: 0.5 tokens/char
 *
 * Detection:
 * - Chinese: Unicode range U+4E00-U+9FFF
 * - Vietnamese: Presence of diacritics (à, á, ả, etc.)
 * - English: Latin words without Vietnamese diacritics
 *
 * @param text - The text to estimate tokens for
 * @returns Estimated token count (rounded up)
 *
 * @example
 * estimateTokens("Hello world"); // ~2 tokens (English)
 * estimateTokens("Xin chào thế giới"); // ~5 tokens (Vietnamese: 4 words * 1.3)
 * estimateTokens("你好世界"); // ~6 tokens (Chinese: 4 chars * 1.5)
 */
export const estimateTokens = (text: string): number => {
  // ... implementation from Step 3
};
```

## Todo List

- [ ] Update `estimateTokens()` with Vietnamese detection
- [ ] Add Vietnamese diacritics regex pattern
- [ ] Apply 1.3x multiplier for Vietnamese words
- [ ] Add constants for multipliers (readability)
- [ ] Add JSDoc documentation
- [ ] Test with Vietnamese, English, Chinese, and mixed text
- [ ] Verify JSON preview displays updated token counts

## Success Criteria

1. ✅ Function detects Vietnamese text correctly
2. ✅ Vietnamese text estimated at 1.3x tokens/word
3. ✅ English and Chinese estimation unchanged (backward compatible)
4. ✅ Mixed language text handled correctly
5. ✅ Token count accuracy within 10% of actual (manual testing)

## Verification Tests

```typescript
// Test cases
console.log(estimateTokens("Hello world"));
// Expected: ~2 (2 words * 1.0)

console.log(estimateTokens("Xin chào thế giới"));
// Expected: ~5 (4 words * 1.3 = 5.2, rounded up to 6)

console.log(estimateTokens("你好世界"));
// Expected: ~6 (4 chars * 1.5)

console.log(estimateTokens("Hello xin chào 你好"));
// Expected: ~9 (Vietnamese detected: 4 Latin words * 1.3 + 2 Chinese chars * 1.5)

console.log(estimateTokens("Tên nhân vật: Linh. Mô tả: Một chiến binh dũng cảm."));
// Expected: ~13 (10 words * 1.3 + punctuation)
```

**Manual Verification**:
Test against actual AI provider token counts (OpenAI tiktoken or Anthropic counter) to verify accuracy.

## Risk Assessment

**Low Risk**:
- Non-critical feature (estimation only)
- Does not affect AI generation
- Backward compatible

**Mitigation**:
- Test with real-world Vietnamese text
- Compare against actual token counts from AI providers
- Adjust multiplier if needed (1.2 or 1.4 instead of 1.3)

## Security Considerations

- No security impact (calculation only)

## Next Steps

→ Proceed to [Phase 5: Testing & Validation](./phase-05-testing-validation.md)

## Notes

- **Multiplier Tuning**: May need adjustment after real-world testing
- **Accuracy**: Current function is rough estimation; Vietnamese estimation follows same pattern
- **Mixed Text**: If Vietnamese diacritics detected, all Latin words counted as Vietnamese (simplified approach)
- **Future Enhancement**: Could use more sophisticated word-by-word detection, but YAGNI for now
