# Research Report: Vietnamese i18n Patterns for React & AI

## 1. Vietnamese Language Characteristics

### Character Length & Layout
- **Vs. English**: Vietnamese text is generally **20-30% longer** than English equivalents.
- **Vs. Chinese**: Vietnamese uses a Latin-based alphabet (Chï QuÑc ngï). While Chinese is logographic (one character = one word/concept), Vietnamese is syllabic with spaces between syllables. A 20-character Chinese string will likely expand to 60-100 characters in Vietnamese.
- **Line Height**: Vietnamese requires **larger line-height** (1.5 or more) because diacritics appear both above and below vowels (e.g., `Ç`, `×`). Tight line heights will cause clipping.

### Diacritical Marks (Tone Marks)
- Vietnamese has 6 tones and complex vowel markers.
- **Normalization**: Always use Unicode Normalization Form C (**NFC**) to avoid search and rendering mismatches where one character (e.g., `¿`) might be represented as two (e.g., `e` + `Ì` + `Ì`).
- **Font Support**: Many standard fonts lack proper Vietnamese glyphs, leading to "tofu" or fallback font issues where diacritics look different from the base letters.

### Translation Challenges
- **Honorifics**: Highly dependent on the relationship between speaker and listener (e.g., "I/You" changes based on age, gender, and social status). For a general UI, "B¡n" (You) is the standard neutral choice.
- **Terminology**: Modern tech terms are often kept in English or translated into compound Sino-Vietnamese words which can be lengthy.

## 2. i18n Library Recommendations

### Library Choice: `react-i18next`
- **Recommendation**: Use `react-i18next`. The current codebase uses a custom `LanguageContext.tsx` with a simple key-value mapping. This is **insufficient** for Vietnamese due to:
  - Lack of ICU support.
  - No handling of pluralization.
  - Difficulty managing long translation strings.

### ICU Message Format
- **Simplification**: Vietnamese has extremely simple pluralization.
- **Rules**: Unicode CLDR specifies only one category for Vietnamese: `other`.
- **Example**: `{count, plural, other {B¡n có # món Ó.}}` works for 0, 1, or 100 items.

### Context vs. i18next
- The current `LanguageContext` is KISS but lacks the robustness for a professional i18n implementation. Transitioning to `i18next` with `JSON` resource files is recommended to keep `LanguageContext.tsx` clean.

## 3. AI Prompt Translation Considerations

### Character & Token Estimation
- **Token Bloat**: Vietnamese suffers from significant "token bloat" in LLMs (OpenAI/Anthropic).
  - English "One" = 1 token.
  - Vietnamese "MÙt" = up to 5 tokens in older tokenizers.
- **Cost/Context**: Expect 30-50% higher token usage for Vietnamese prompts vs. English, and significantly higher vs. Chinese for the same meaning.
- **Length Constraints**: If the UI limits inputs to "20 characters" based on Chinese/English logic, this **must be increased** to at least 60-80 for Vietnamese to allow for comparable semantic depth.

### Prompt Engineering Best Practices
- **Explicit Instruction**: When asking AI to generate Vietnamese, explicitly instruct it to use "natural, modern Vietnamese" to avoid overly formal or robotic translations.
- **Avoid Literal Translation**: Prompts should focus on "Meaning-to-Meaning" rather than "Word-to-Word" to handle the complex honorific system.
- **Token Awareness**: Use English for system instructions and only use Vietnamese for the specific content generation to save tokens.

## Unresolved Questions
1. Does the current `shadcn/ui` font stack (Inter/System) render Vietnamese diacritics correctly at small sizes?
2. Should we implement a "Formal/Informal" toggle for AI generation to handle Vietnamese social dynamics?
3. How will the PNG export handle the increased character length without breaking the layout?
