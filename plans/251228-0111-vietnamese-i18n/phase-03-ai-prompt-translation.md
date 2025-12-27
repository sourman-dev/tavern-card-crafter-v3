# Phase 3: AI Prompt Translation & Adaptation

**Parent Plan**: [Vietnamese Language Support](./plan.md)
**Status**: Pending | **Priority**: P1 | **Effort**: 3h
**Date**: 2025-12-28

## Overview

Translate all 10 AI prompt generator functions in `aiGenerator.ts` to Vietnamese and adapt them for Vietnamese linguistic characteristics. **This is the most critical phase** as it ensures AI responses are in Vietnamese.

## Key Insights from Research

From `research/researcher-01-vietnamese-i18n-patterns.md`:

1. **Character Length**: 20 Chinese chars ≈ 60-80 Vietnamese chars
2. **Token Bloat**: Vietnamese uses 30-50% more tokens than English
3. **Literary References**: Western authors (Douglas Adams, Philip K. Dick) need adaptation
4. **Prompt Engineering**: Use English for system instructions, Vietnamese for content generation
5. **Honorifics**: "Bạn" (neutral) for general use

## Requirements

1. Translate all 10 prompt generator functions
2. Adjust character/token count constraints (multiply by 2-3x)
3. Replace Western literary references with Vietnamese/Asian equivalents or remove
4. Add language-aware prompt selection logic
5. Maintain prompt effectiveness (quality of AI responses)

## Architecture Considerations

- **Pattern**: Add language parameter to prompt functions OR use global language context
- **Approach**: Duplicate prompt generators with `_vi` suffix (KISS) vs. parameterized (DRY)
- **Decision**: Parameterized approach (DRY) - single function with language detection
- **Integration**: Access language from LanguageContext in components calling `generateWithAI()`

## Related Files

| File | Lines | Purpose |
|------|-------|---------|
| `src/utils/aiGenerator.ts:185-202` | 18 | `generateDescription()` |
| `src/utils/aiGenerator.ts:207-214` | 8 | `generatePersonality()` |
| `src/utils/aiGenerator.ts:219-227` | 9 | `generateScenario()` |
| `src/utils/aiGenerator.ts:229-238` | 10 | `generateFirstMes()` |
| `src/utils/aiGenerator.ts:243-262` | 20 | `generateMesExample()` |
| `src/utils/aiGenerator.ts:267-278` | 12 | `generateSystemPrompt()` |
| `src/utils/aiGenerator.ts:283-294` | 12 | `generatePostHistoryInstructions()` |
| `src/utils/aiGenerator.ts:296-305` | 10 | `generateTags()` |
| `src/utils/aiGenerator.ts:320-335` | 16 | `generateAlternateGreeting()` |
| `src/utils/aiGenerator.ts:340-359` | 20 | `generateCharacterBookEntry()` |

## Implementation Steps

### Step 1: Add Language Parameter to All Generators

Update function signatures to accept `language` parameter:

```typescript
export const generateDescription = (
  data: CharacterData,
  language: 'zh' | 'en' | 'vi' = 'en'
): string => {
  // Implementation
};
```

**Apply to all 10 functions** listed above.

### Step 2: Implement Language-Specific Prompts

Create helper function for language detection:

```typescript
const getPromptByLanguage = (
  prompts: Record<'en' | 'vi', string>,
  language: 'zh' | 'en' | 'vi'
): string => {
  // Chinese uses English prompts (existing behavior)
  if (language === 'zh') return prompts.en;
  return prompts[language] || prompts.en;
};
```

### Step 3: Translate generateDescription()

**File**: `src/utils/aiGenerator.ts:185-202`

```typescript
export const generateDescription = (
  data: CharacterData,
  language: 'zh' | 'en' | 'vi' = 'en'
): string => {
  const existingDescription = data.description.trim();

  const prompts = {
    en: existingDescription
      ? `Based on the following role information, enhance the descriptions, while remaining succinct:

Card name: ${data.name}
Existing description: ${existingDescription}

Embellish, making sure to describe all physical qualities of the character -- body, attire and how they compose themself. This should be in non-prosaic list format.`
      : `Generate a list of all the physical qualities of the character -- body, attire and how they compose themself. This should be in non-prosaic CSV format riffing off of the character name:

Card name: ${data.name}

Please generate a detailed character appearance description, including the character's physical characteristics, facial features, clothing style, temperament, etc. Only output character description content, do not include character name, background story or other information. Please output the description directly, and do not add summary or additional instructions.`,

    vi: existingDescription
      ? `Dựa trên thông tin nhân vật sau, hãy làm phong phú thêm mô tả, nhưng vẫn giữ ngắn gọn:

Tên nhân vật: ${data.name}
Mô tả hiện tại: ${existingDescription}

Hãy mô tả chi tiết tất cả các đặc điểm ngoại hình của nhân vật -- thân hình, trang phục và cách họ thể hiện bản thân. Mô tả nên ở dạng danh sách không theo lối văn xuôi, khoảng 100-150 từ.`
      : `Tạo danh sách tất cả các đặc điểm ngoại hình của nhân vật -- thân hình, trang phục và cách họ thể hiện bản thân, dựa trên tên nhân vật:

Tên nhân vật: ${data.name}

Hãy tạo mô tả chi tiết về ngoại hình nhân vật, bao gồm đặc điểm cơ thể, khuôn mặt, phong cách thời trang, khí chất, v.v. Chỉ xuất nội dung mô tả nhân vật, không bao gồm tên nhân vật, câu chuyện nền hoặc thông tin khác. Vui lòng xuất mô tả trực tiếp, không thêm tóm tắt hay hướng dẫn bổ sung. Độ dài khoảng 100-150 từ.`
  };

  return getPromptByLanguage(prompts, language);
};
```

**Key Changes**:
- Added `language` parameter
- Character count guidance: "100-150 từ" (words) vs implicit in English
- Removed Western CSV/list format emphasis
- Vietnamese instructions more explicit

### Step 4: Translate generatePersonality()

**File**: `src/utils/aiGenerator.ts:207-214`

```typescript
export const generatePersonality = (
  data: CharacterData,
  language: 'zh' | 'en' | 'vi' = 'en'
): string => {
  const prompts = {
    en: `Based on the following role information, enhance the character's persona, while remaining succinct.

Card name: ${data.name}
Physical Description: ${data.description}

A non-prosaic list, describing the character's traits, behavior, idiosyncrasies, likes/dislikes, strengths/weaknesses, backstory.`,

    vi: `Dựa trên thông tin nhân vật sau, hãy làm phong phú thêm tính cách của nhân vật, nhưng vẫn giữ ngắn gọn.

Tên nhân vật: ${data.name}
Mô tả ngoại hình: ${data.description}

Hãy tạo danh sách mô tả đặc điểm tính cách, hành vi, những điều đặc biệt, sở thích/không thích, điểm mạnh/điểm yếu, và câu chuyện nền của nhân vật. Độ dài khoảng 80-120 từ.`
  };

  return getPromptByLanguage(prompts, language);
};
```

### Step 5: Translate generateScenario()

**File**: `src/utils/aiGenerator.ts:219-227`

```typescript
export const generateScenario = (
  data: CharacterData,
  language: 'zh' | 'en' | 'vi' = 'en'
): string => {
  const prompts = {
    en: `Generate an appropriate meta-scenario based on the following information:

Card name: ${data.name}
Physical Description: ${data.description}
Character Personality: ${data.personality}

Generate the backstory and meta-environment in acclaimed historian's prose.`,

    vi: `Tạo kịch bản phù hợp dựa trên thông tin sau:

Tên nhân vật: ${data.name}
Mô tả ngoại hình: ${data.description}
Tính cách nhân vật: ${data.personality}

Hãy tạo câu chuyện nền và bối cảnh tổng quát với văn phong chuyên nghiệp và hấp dẫn, như thể viết bởi một nhà sử học giỏi. Độ dài khoảng 100-150 từ.`
  };

  return getPromptByLanguage(prompts, language);
};
```

### Step 6: Translate generateFirstMes() - CRITICAL LITERARY ADAPTATION

**File**: `src/utils/aiGenerator.ts:229-238`

```typescript
export const generateFirstMes = (
  data: CharacterData,
  language: 'zh' | 'en' | 'vi' = 'en'
): string => {
  const prompts = {
    en: `Generate the first message of the game, introducing the character to the player/user:

Card name: ${data.name}
Physical Description: ${data.description}
Character Personality: ${data.personality}
Scene settings: ${data.scenario}

This will be the first outward facing text, the first thing the player/user encounters when playing with the character. Somehow the character must meet the player/user. The writing should be a perfect combination of Douglas Adams, Ursula K. Le Guin, James Joyce, Anais Nin, and Philip K. Dick.`,

    vi: `Tạo tin nhắn đầu tiên của trò chơi, giới thiệu nhân vật với người chơi/người dùng:

Tên nhân vật: ${data.name}
Mô tả ngoại hình: ${data.description}
Tính cách nhân vật: ${data.personality}
Bối cảnh: ${data.scenario}

Đây sẽ là văn bản đầu tiên mà người chơi/người dùng gặp khi tương tác với nhân vật. Nhân vật cần gặp gỡ người chơi/người dùng theo một cách tự nhiên. Văn phong nên sáng tạo, hấp dẫn, kết hợp giữa yếu tố kể chuyện sinh động và mô tả tâm lý tinh tế, giống như phong cách của các nhà văn nổi tiếng về khoa học viễn tưởng và văn học hiện đại. Độ dài khoảng 150-200 từ.`
  };

  return getPromptByLanguage(prompts, language);
};
```

**Cultural Adaptation**:
- Removed specific Western author names (Douglas Adams, etc.)
- Replaced with genre descriptors: "khoa học viễn tưởng và văn học hiện đại" (sci-fi and modern literature)
- Increased length: 150-200 words (vs implicit in English)

### Step 7: Translate generateMesExample()

**File**: `src/utils/aiGenerator.ts:243-262`

```typescript
export const generateMesExample = (
  data: CharacterData,
  language: 'zh' | 'en' | 'vi' = 'en'
): string => {
  const prompts = {
    en: `Generate a conversational example to help establish the character:

Card name: ${data.name}
Physical Description: ${data.description}
Character Personality: ${data.personality}
Scene setting: ${data.scenario}

Please generate 2 to 3 dialogues or monologues that truly capture the spirit of the character. The format is as follows:

<START>
{{user}}: User's words
${data.name}: The character's answer
${data.name}: *The character's actions.*

<START>
${data.name}: The character talks to themself and acts on their own.

Make sure each conversation example starts with a <START> macro. Do not include it if it doesn't help to develop the character's actions and speaking behavior.`,

    vi: `Tạo ví dụ hội thoại để giúp xây dựng nhân vật:

Tên nhân vật: ${data.name}
Mô tả ngoại hình: ${data.description}
Tính cách nhân vật: ${data.personality}
Bối cảnh: ${data.scenario}

Hãy tạo 2 đến 3 đoạn hội thoại hoặc độc thoại thể hiện rõ tinh thần của nhân vật. Định dạng như sau:

<START>
{{user}}: Lời của người dùng
${data.name}: Câu trả lời của nhân vật
${data.name}: *Hành động của nhân vật.*

<START>
${data.name}: Nhân vật tự nói chuyện và hành động một mình.

Đảm bảo mỗi ví dụ hội thoại bắt đầu bằng macro <START>. Chỉ tạo các ví dụ giúp phát triển hành động và cách nói chuyện của nhân vật.`
  };

  return getPromptByLanguage(prompts, language);
};
```

### Step 8: Translate generateSystemPrompt()

**File**: `src/utils/aiGenerator.ts:267-278`

```typescript
export const generateSystemPrompt = (
  data: CharacterData,
  language: 'zh' | 'en' | 'vi' = 'en'
): string => {
  const prompts = {
    en: `Generate System Prompt based on the following information:

Card name: ${data.name}
Physical Description: ${data.description}
Character Personality: ${data.personality}
Scene settings: ${data.scenario}
Example Character Actions: ${data.mes_example}
Story introduction: ${data.first_mes}

Write the System Prompt to instruct the AI how to accurately play the character. Be concise and clear.`,

    vi: `Tạo Lời nhắc Hệ thống dựa trên thông tin sau:

Tên nhân vật: ${data.name}
Mô tả ngoại hình: ${data.description}
Tính cách nhân vật: ${data.personality}
Bối cảnh: ${data.scenario}
Ví dụ hành động: ${data.mes_example}
Giới thiệu câu chuyện: ${data.first_mes}

Hãy viết Lời nhắc Hệ thống để hướng dẫn AI cách đóng vai nhân vật một cách chính xác. Hãy ngắn gọn và rõ ràng, khoảng 60-100 từ.`
  };

  return getPromptByLanguage(prompts, language);
};
```

### Step 9: Translate generatePostHistoryInstructions()

**File**: `src/utils/aiGenerator.ts:283-294`

```typescript
export const generatePostHistoryInstructions = (
  data: CharacterData,
  language: 'zh' | 'en' | 'vi' = 'en'
): string => {
  const prompts = {
    en: `Generate the most important instructions for the AI based on the following information:
Card name: ${data.name}
Physical Description: ${data.description}
Character Personality: ${data.personality}
Scene settings: ${data.scenario}
Example Character Actions: ${data.mes_example}
Story introduction: ${data.first_mes}
SYSTEM PROMPT: ${data.system_prompt}

THIS MUST BE EXTREMELY BRIEF!.`,

    vi: `Tạo các hướng dẫn quan trọng nhất cho AI dựa trên thông tin sau:
Tên nhân vật: ${data.name}
Mô tả ngoại hình: ${data.description}
Tính cách nhân vật: ${data.personality}
Bối cảnh: ${data.scenario}
Ví dụ hành động: ${data.mes_example}
Giới thiệu câu chuyện: ${data.first_mes}
LỜI NHẮC HỆ THỐNG: ${data.system_prompt}

PHẢI CỰC KỲ NGẮN GỌN! Tối đa 30-50 từ.`
  };

  return getPromptByLanguage(prompts, language);
};
```

**Note**: Added explicit word count (30-50 từ) for Vietnamese clarity.

### Step 10: Translate generateTags()

**File**: `src/utils/aiGenerator.ts:296-305`

```typescript
export const generateTags = (
  data: CharacterData,
  language: 'zh' | 'en' | 'vi' = 'en'
): string => {
  const prompts = {
    en: `Generate appropriate keywords based on the following information:

Card name: ${data.name}
Physical Description: ${data.description}
Character Personality: ${data.personality}
Scene setting: ${data.scenario}

Please generate 5-10 related keywords or single-word tags, separated by commas. Tags should include character type, personality traits, scene type, etc.`,

    vi: `Tạo các từ khóa phù hợp dựa trên thông tin sau:

Tên nhân vật: ${data.name}
Mô tả ngoại hình: ${data.description}
Tính cách nhân vật: ${data.personality}
Bối cảnh: ${data.scenario}

Hãy tạo 8-15 từ khóa hoặc thẻ tag liên quan, ngăn cách bằng dấu phẩy. Các thẻ tag nên bao gồm loại nhân vật, đặc điểm tính cách, loại bối cảnh, v.v.`
  };

  return getPromptByLanguage(prompts, language);
};
```

**Key Change**: Increased tag count from 5-10 to 8-15 (Vietnamese tags are longer, so similar semantic coverage requires more tags).

### Step 11: Translate generateAlternateGreeting() - CRITICAL LITERARY ADAPTATION

**File**: `src/utils/aiGenerator.ts:320-335`

```typescript
export const generateAlternateGreeting = (
  data: CharacterData,
  language: 'zh' | 'en' | 'vi' = 'en'
): string => {
  const prompts = {
    en: `Generate the start of the next chapter of the story based on the following information:

Card name: ${data.name}
Physical Description: ${data.description}
Character Personality: ${data.personality}
Scene settings: ${data.scenario}
Dialogue example: ${data.mes_example}

The chapters that have already been written:

First Chapter Beginning: ${data.first_mes}
Subsequent chapters(if any): ${data.alternate_greetings}

Write the beginning of a new chapter. It must include the character and some encounter with the player / user and must be at least one day after any prior chapters. Merge the writing styles of James Joyce, Philip K. Dick, Francois Rabelais, and Anais Nin as appropriate to the context.`,

    vi: `Tạo phần mở đầu của chương tiếp theo trong câu chuyện dựa trên thông tin sau:

Tên nhân vật: ${data.name}
Mô tả ngoại hình: ${data.description}
Tính cách nhân vật: ${data.personality}
Bối cảnh: ${data.scenario}
Ví dụ hội thoại: ${data.mes_example}

Các chương đã được viết:

Chương đầu tiên: ${data.first_mes}
Các chương tiếp theo (nếu có): ${data.alternate_greetings}

Hãy viết phần mở đầu của chương mới. Chương này phải bao gồm nhân vật và một cuộc gặp gỡ với người chơi/người dùng, diễn ra ít nhất một ngày sau các chương trước. Văn phong nên sáng tạo, kết hợp yếu tố hiện đại, tâm lý, và có chiều sâu văn học. Độ dài khoảng 150-200 từ.`
  };

  return getPromptByLanguage(prompts, language);
};
```

**Cultural Adaptation**: Same as `generateFirstMes()` - removed Western author names, used genre descriptors.

### Step 12: Translate generateCharacterBookEntry()

**File**: `src/utils/aiGenerator.ts:340-359`

```typescript
export const generateCharacterBookEntry = (
  data: CharacterData,
  language: 'zh' | 'en' | 'vi' = 'en'
): string => {
  const prompts = {
    en: `Generate specific important character information entries based on the following information:

Card name: ${data.name}
Physical Description: ${data.description}
Character Personality: ${data.personality}
Scene settings: ${data.scenario}

You are constructing what is referred to as a lorebook. When certain keywords are mentioned in the chat, it triggers a longer definition to be sent to the AI and considered in it's response. It can work like an important memory of the character's or an important fact about the character. It should be something quite unique and important to be worth including.

The parameters are:
Keyword requirements: Use at least 2 - 3 related core keywords/synonyms, separated by commas.
Content requirements: Generate specific setting content, such as the character's special skills, important experiences, interpersonal relationships or items, and other background information.

The format is as follows:
Keywords: keyword1, keyword2, keyword3, keyword4, ...
Content: The detailed description.

The content should be rich and helpful for role-playing.`,

    vi: `Tạo mục thông tin quan trọng về nhân vật dựa trên thông tin sau:

Tên nhân vật: ${data.name}
Mô tả ngoại hình: ${data.description}
Tính cách nhân vật: ${data.personality}
Bối cảnh: ${data.scenario}

Bạn đang xây dựng một "sách kiến thức" (lorebook). Khi một số từ khóa cụ thể được đề cập trong cuộc trò chuyện, nó sẽ kích hoạt một định nghĩa chi tiết được gửi đến AI và được xem xét trong phản hồi. Nó có thể hoạt động như một ký ức quan trọng của nhân vật hoặc một sự kiện quan trọng về nhân vật. Nó nên là điều gì đó khá độc đáo và quan trọng để đáng được đưa vào.

Các tham số:
Yêu cầu từ khóa: Sử dụng ít nhất 3-5 từ khóa/từ đồng nghĩa liên quan, ngăn cách bằng dấu phẩy.
Yêu cầu nội dung: Tạo nội dung cài đặt cụ thể, chẳng hạn như kỹ năng đặc biệt của nhân vật, trải nghiệm quan trọng, mối quan hệ giữa các nhân vật hoặc các vật phẩm, và thông tin nền khác.

Định dạng như sau:
Từ khóa: từ_khóa_1, từ_khóa_2, từ_khóa_3, từ_khóa_4, ...
Nội dung: Mô tả chi tiết (khoảng 80-120 từ).

Nội dung nên phong phú và hữu ích cho việc nhập vai.`
  };

  return getPromptByLanguage(prompts, language);
};
```

**Key Changes**:
- Increased keyword count: 2-3 → 3-5 (Vietnamese keywords longer)
- Added content length: 80-120 từ (words)

### Step 13: Update Component Calls to Pass Language

**Files to update**: All components calling these functions (e.g., `AIAssistant.tsx`, `PersonalitySection.tsx`, etc.)

**Example in AIAssistant.tsx**:

```typescript
import { useLanguage } from '@/contexts/LanguageContext';

// Inside component
const { language } = useLanguage();

// When calling generator
const prompt = generateDescription(characterData, language);
const response = await generateWithAI(settings, prompt);
```

**Repeat for all components** using AI generation functions.

## Todo List

- [ ] Add `language` parameter to all 10 generator functions
- [ ] Implement `getPromptByLanguage()` helper
- [ ] Translate `generateDescription()` with length adjustments
- [ ] Translate `generatePersonality()` with length adjustments
- [ ] Translate `generateScenario()` with length adjustments
- [ ] Translate `generateFirstMes()` with cultural adaptation
- [ ] Translate `generateMesExample()`
- [ ] Translate `generateSystemPrompt()` with length constraints
- [ ] Translate `generatePostHistoryInstructions()` with brevity emphasis
- [ ] Translate `generateTags()` with increased count (8-15)
- [ ] Translate `generateAlternateGreeting()` with cultural adaptation
- [ ] Translate `generateCharacterBookEntry()` with keyword/length adjustments
- [ ] Update all component calls to pass `language` parameter
- [ ] Test all prompts with AI provider (verify Vietnamese responses)

## Success Criteria

1. ✅ All 10 functions accept `language` parameter
2. ✅ All functions have complete Vietnamese prompts
3. ✅ Character/word count constraints adjusted for Vietnamese (2-3x)
4. ✅ Western literary references replaced/adapted
5. ✅ AI responses return in Vietnamese when `language='vi'`
6. ✅ Prompt quality maintained (manual QA testing)
7. ✅ TypeScript compiles without errors

## Verification Tests

```typescript
// Test Vietnamese prompt generation
const data: CharacterData = {
  name: 'Linh',
  description: 'Một chiến binh dũng cảm',
  personality: 'Dũng cảm, trung thành',
  scenario: 'Thời kỳ chiến tranh'
};

const prompt = generateDescription(data, 'vi');
console.log(prompt);
// Should contain Vietnamese instructions with "100-150 từ"

// Test with AI
const response = await generateWithAI(settings, prompt);
console.log(response);
// Should return Vietnamese description
```

## Risk Assessment

**High Risk**:
- Vietnamese prompts may produce lower quality AI responses than English
- Cultural adaptation may lose original creative intent
- Token bloat may hit context limits faster

**Mitigation**:
1. Test with multiple AI providers (OpenAI, Anthropic, local models)
2. Iterate on prompt wording based on output quality
3. Monitor token usage and adjust max_tokens in `generateWithAI()` if needed
4. Keep English prompts as fallback if Vietnamese fails

## Security Considerations

- No security impact (prompt changes only)
- No new external dependencies

## Next Steps

→ Proceed to [Phase 4: Token Estimation Update](./phase-04-token-estimation.md)

## Notes

- **Testing Critical**: Vietnamese prompts MUST be tested with real AI providers
- **Iterative Refinement**: May need multiple rounds of prompt tuning
- **Cultural Sensitivity**: Literary references adapted to avoid Western-centric bias
- **Length Guidance**: Vietnamese prompts include explicit word counts (English prompts implicit)
- **Tag Count**: Increased from 5-10 to 8-15 to compensate for longer Vietnamese tags
