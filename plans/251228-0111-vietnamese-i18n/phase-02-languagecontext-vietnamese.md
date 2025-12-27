# Phase 2: LanguageContext Vietnamese Translation

**Parent Plan**: [Vietnamese Language Support](./plan.md)
**Status**: Pending | **Priority**: P1 | **Effort**: 2h
**Date**: 2025-12-28

## Overview

Add Vietnamese (`vi`) language support to the `LanguageContext.tsx` by translating all 70+ UI strings and updating TypeScript types. This phase focuses on UI localization only (AI prompts handled in Phase 3).

## Key Insights from Research

From `research/researcher-01-vietnamese-i18n-patterns.md`:
- Vietnamese text is 20-30% longer than English
- Requires line-height ≥ 1.5 for diacritics (á, ả, ã, à, ạ, ă, ắ, etc.)
- Use "Bạn" (neutral "You") for general UI
- Modern tech terms often kept in English (e.g., "Token", "JSON")
- Unicode NFC normalization for consistency

## Requirements

1. Add `vi` to language type union (`'zh' | 'en' | 'vi'`)
2. Translate all 70+ translation keys to Vietnamese
3. Update localStorage handling for `vi`
4. Verify shadcn/ui fonts support Vietnamese diacritics
5. Maintain KISS principle (no migration to react-i18next)

## Architecture Considerations

- **Pattern**: Keep custom Context approach (YAGNI - don't introduce react-i18next)
- **Type Safety**: Update all type unions to include `vi`
- **Storage**: localStorage key remains `language`, value adds `vi`
- **Fallback**: If key missing, return key (same as current behavior)

## Related Files

| File | Lines | Purpose |
|------|-------|---------|
| `src/contexts/LanguageContext.tsx:5-6` | 2 | Type definition `'zh' \| 'en'` → `'zh' \| 'en' \| 'vi'` |
| `src/contexts/LanguageContext.tsx:10-195` | 185 | Translation objects (add `vi` section) |
| `src/contexts/LanguageContext.tsx:211` | 1 | useState initial type |
| `src/contexts/LanguageContext.tsx:214,223` | 2 | localStorage type assertions |

## Implementation Steps

### Step 1: Update Type Definitions

**File**: `src/contexts/LanguageContext.tsx:5-6`

```typescript
// OLD:
interface LanguageContextType {
  language: 'zh' | 'en';
  setLanguage: (lang: 'zh' | 'en') => void;
  t: (key: string) => string;
}

// NEW:
interface LanguageContextType {
  language: 'zh' | 'en' | 'vi';
  setLanguage: (lang: 'zh' | 'en' | 'vi') => void;
  t: (key: string) => string;
}
```

### Step 2: Add Vietnamese Translation Object

**File**: `src/contexts/LanguageContext.tsx:195` (after `en` object)

Add complete `vi` translation object with 70+ keys:

```typescript
const translations = {
  zh: { /* existing */ },
  en: { /* existing */ },
  vi: {
    // Page title
    pageTitle: 'Trình tạo thẻ nhân vật SillyTavern V3',
    pageDescription: 'Tạo thẻ nhân vật chuyên nghiệp định dạng SillyTavern V3, hỗ trợ nhập/xuất V1/V2/V3',

    // Buttons
    importCard: 'Nhập thẻ',
    aiSettings: 'Cài đặt AI',
    copy: 'Sao chép',
    exportJson: 'Xuất JSON',
    exportPng: 'Xuất PNG',
    save: 'Lưu',
    cancel: 'Hủy',

    // Form titles
    characterInfo: 'Chỉnh sửa thông tin nhân vật',
    basicInfo: 'Thông tin cơ bản',
    personality: 'Tính cách',
    prompts: 'Các lời nhắc',
    alternateGreetings: 'Lời chào thay thế',
    characterBook: 'Sách nhân vật',
    tags: 'Thẻ tag',
    metadata: 'Siêu dữ liệu',

    // Field labels
    name: 'Tên thẻ',
    nickname: 'Tên nhân vật',
    description: 'Mô tả nhân vật',
    personalityDescription: 'Mô tả tính cách',
    scenario: 'Kịch bản',
    first_mes: 'Tin nhắn đầu tiên',
    mes_example: 'Ví dụ tin nhắn',
    creatorNotes: 'Ghi chú của tác giả',
    systemPrompt: 'Lời nhắc hệ thống',
    postHistoryInstructions: 'Hướng dẫn sau lịch sử',
    creator: 'Tác giả',
    characterVersion: 'Phiên bản nhân vật',

    // Preview
    jsonPreview: 'Xem trước JSON',
    totalChars: 'Tổng số ký tự',
    totalTokens: 'Tổng số Token',
    chars: 'ký tự',
    tokens: 'Token',

    // Messages
    importSuccess: 'Nhập thành công',
    importSuccessDesc: 'Dữ liệu thẻ nhân vật đã được nhập thành công',
    importError: 'Nhập thất bại',
    importErrorDesc: 'Lỗi định dạng tệp hoặc dữ liệu bị hỏng',
    copySuccess: 'Sao chép thành công',
    copySuccessDesc: 'JSON thẻ nhân vật đã được sao chép vào clipboard',
    uploadImageHint: 'Vui lòng tải lên ảnh đại diện nhân vật trước',
    pngExportHint: 'Xuất PNG yêu cầu triển khai phức tạp hơn, vui lòng sử dụng xuất JSON',

    // AI Generation
    aiGenerate: 'Tạo bằng AI',
    generating: 'Đang tạo...',
    generateSuccess: 'Tạo thành công',
    generateError: 'Tạo thất bại',
    configError: 'Lỗi cấu hình',
    configApiKey: 'Vui lòng cấu hình khóa API trong cài đặt AI trước',
    incompleteInfo: 'Thông tin chưa đầy đủ',
    fillNameDesc: 'Vui lòng điền tên và mô tả nhân vật trước',
    unknownError: 'Lỗi không xác định',

    // Alternate Greetings
    addNewGreeting: 'Thêm lời chào mới',
    addAlternateGreetingPlaceholder: 'Thêm lời chào thay thế...',
    aiGenerateGreeting: 'Tạo lời chào bằng AI',
    alternateGreetingGenerated: 'Lời chào thay thế đã được tạo',

    // Tags
    enterTag: 'Nhập thẻ tag...',
    aiGenerateTags: 'Tạo thẻ tag bằng AI',
    tagsGenerated: 'Thẻ tag đã được tạo',

    // Character Book
    addNewEntry: 'Thêm mục mới',
    addEntry: 'Thêm mục',
    aiGenerateEntry: 'Tạo mục bằng AI',
    entryGenerated: 'Mục sách nhân vật đã được tạo',
    entry: 'Mục',
    keywords: 'Từ khóa',
    content: 'Nội dung',
    insertionOrder: 'Thứ tự chèn',
    enabled: 'Đã bật',

    // Theme toggle
    lightMode: 'Chế độ sáng',
    darkMode: 'Chế độ tối',
  }
};
```

### Step 3: Update useState and localStorage

**File**: `src/contexts/LanguageContext.tsx:211,214,223`

```typescript
// Line 211: Update type
const [language, setLanguageState] = useState<'zh' | 'en' | 'vi'>('en');

// Line 214: Update type assertion
const savedLanguage = localStorage.getItem('language') as 'zh' | 'en' | 'vi';

// Line 223: Update function signature
const setLanguage = (lang: 'zh' | 'en' | 'vi') => {
  setLanguageState(lang);
  localStorage.setItem('language', lang);
};
```

### Step 4: Verify Font Support

**Check shadcn/ui font stack** (typically in `globals.css` or `tailwind.config.ts`):

Default shadcn/ui fonts (Inter, system-ui) support Vietnamese diacritics ✅
- If issues, add fallback: `"Segoe UI", "Roboto", sans-serif`

**Test diacritics rendering**:
- á à ả ã ạ ă ắ ằ ẳ ẵ ặ â ấ ầ ẩ ẫ ậ
- é è ẻ ẽ ẹ ê ế ề ể ễ ệ
- í ì ỉ ĩ ị
- ó ò ỏ õ ọ ô ố ồ ổ ỗ ộ ơ ớ ờ ở ỡ ợ
- ú ù ủ ũ ụ ư ứ ừ ử ữ ự
- ý ỳ ỷ ỹ ỵ

### Step 5: Add Language Selector UI Support

Verify `Toolbar.tsx` or language selector component can handle `vi`:

```typescript
// Example selector (verify existing implementation)
<select value={language} onChange={(e) => setLanguage(e.target.value as 'zh' | 'en' | 'vi')}>
  <option value="zh">中文</option>
  <option value="en">English</option>
  <option value="vi">Tiếng Việt</option>
</select>
```

## Todo List

- [ ] Update `LanguageContextType` interface to include `vi`
- [ ] Add complete `vi` translation object (70+ keys)
- [ ] Update `useState` type to include `vi`
- [ ] Update localStorage type assertions
- [ ] Verify font support for Vietnamese diacritics
- [ ] Test all UI elements display Vietnamese correctly
- [ ] Verify layout doesn't break with longer Vietnamese strings
- [ ] Update language selector to include Vietnamese option

## Success Criteria

1. ✅ TypeScript compiles without errors
2. ✅ All 70+ translation keys present in `vi` object
3. ✅ UI switches to Vietnamese when `vi` selected
4. ✅ Vietnamese diacritics render correctly
5. ✅ No layout overflow/breaking with longer Vietnamese text
6. ✅ localStorage persists `vi` selection across sessions

## Verification Tests

```typescript
// Test translation retrieval
const { t, language, setLanguage } = useLanguage();
setLanguage('vi');
console.log(t('pageTitle'));
// Expected: "Trình tạo thẻ nhân vật SillyTavern V3"

// Test diacritics
console.log(t('characterInfo'));
// Expected: "Chỉnh sửa thông tin nhân vật"

// Test missing key fallback
console.log(t('nonexistentKey'));
// Expected: "nonexistentKey"
```

## Risk Assessment

**Medium Risk**:
- Vietnamese text 20-30% longer → potential UI overflow
- Diacritics may clip if line-height too small

**Mitigation**:
1. Test all form sections with Vietnamese text
2. Adjust Tailwind classes if overflow detected (add `break-words`, increase padding)
3. Verify line-height ≥ 1.5 in global styles
4. Test on mobile viewports (most constrained)

## Security Considerations

- No security impact (localization only)
- No external data sources

## Next Steps

→ Proceed to [Phase 3: AI Prompt Translation & Adaptation](./phase-03-ai-prompt-translation.md)

## Notes

- **Translation Quality**: Translations provided are professional Vietnamese
- **Cultural Context**: Used neutral "Bạn" for "You", suitable for general app
- **Tech Terms**: Kept "Token", "JSON", "AI" in English (common practice)
- **Line Length**: Some labels significantly longer (e.g., "Character Information Editor" → "Chỉnh sửa thông tin nhân vật")
