# Codebase Summary

## Repository Overview

**Project**: Tavern Card Crafter v3
**Language**: TypeScript (React 19.2 + Electron 38.4)
**Build Tool**: Vite 7.1.11
**Total Files**: 105 files (analyzed by Repomix)
**Total Tokens**: 97,578 tokens
**Total Characters**: 402,920 characters

## Directory Structure

```
tavern-card-crafter-v3/
├── electron/               # Electron desktop app (2 files)
│   ├── main.cjs           # Main process: window management, menus, security
│   └── preload.js         # Preload script: IPC bridge (contextBridge)
├── src/                   # React application source (85+ files)
│   ├── components/        # UI components
│   │   ├── CharacterForm/ # Character editing sections (10 files)
│   │   ├── ui/            # shadcn/ui components (40+ files)
│   │   ├── AISettings.tsx          # AI provider config (6,589 tokens, largest)
│   │   ├── CharacterPreview.tsx    # JSON/PNG export
│   │   ├── LocalDeploymentPanel.tsx
│   │   └── Toolbar.tsx             # Theme/language switcher
│   ├── contexts/          # React Context providers (2 files)
│   │   ├── LanguageContext.tsx
│   │   └── ThemeContext.tsx
│   ├── hooks/             # Custom hooks (2 files)
│   ├── lib/               # Utility libraries (1 file)
│   ├── pages/             # Page components (2 files)
│   │   ├── Index.tsx      # Main page (6,039 tokens, 2nd largest)
│   │   └── NotFound.tsx
│   └── utils/             # Business logic (2 files)
│       ├── aiGenerator.ts
│       └── buildApiUrl.ts
├── public/                # Static assets
├── docs/                  # Documentation (this directory)
├── plans/                 # Project plans & reports
├── .claude/               # Claude Code configuration
├── build-electron/        # Electron build output
├── dist/                  # Vite build output
└── Configuration files    # vite.config.ts, tsconfig.json, etc.
```

## Top 5 Files by Token Count

1. **src/components/AISettings.tsx** (6,589 tokens, 6.8%)
   - AI provider configuration UI
   - Model selection, API key management
   - Custom endpoint support

2. **src/pages/Index.tsx** (6,039 tokens, 6.2%)
   - Main application page
   - Three-panel layout: AI Assistant | Editor | Preview
   - Character data state management
   - Import/export logic (JSON, PNG)

3. **src/components/ui/sidebar.tsx** (5,936 tokens, 6.1%)
   - shadcn/ui sidebar component
   - Navigation structure

4. **src/components/CharacterForm/AIAssistant.tsx** (4,156 tokens, 4.3%)
   - AI character generation interface
   - Character type selection (anime, game, novel, etc.)
   - JSON parsing from AI responses

5. **src/components/LocalDeploymentPanel.tsx** (4,008 tokens, 4.1%)
   - Local AI deployment guidance
   - Ollama, LM Studio integration docs

## Core Components Analysis

### Main Application Flow

**Entry Point**: `src/main.tsx`
- Renders `<App />` into `#root`
- Wraps in `<React.StrictMode>`

**Root Component**: `src/App.tsx`
- Sets up React Router with `/` and `*` routes
- Wraps in `LanguageProvider` and `ThemeProvider`

**Main Page**: `src/pages/Index.tsx` (705 lines)
- **State Management**:
  - `characterData`: CharacterCardV3 structure (lines 84-112)
  - `characterImage`: Base64 image data for avatar
  - `aiSettings`: Persisted in localStorage
  - `activeTab`: "assistant" | "editor" | "preview"

- **Key Functions**:
  - `updateField()`: Update character data fields (lines 143-152)
  - `handleInsertField()`: Merge AI-generated data (lines 154-163)
  - `extractPNGCharacterData()`: PNG steganography parsing (lines 177-381)
  - `handleFileImport()`: Import V1/V2/V3 JSON or PNG (lines 383-524)

- **UI Layout** (lines 526-701):
  - Left sidebar: Tab navigation (AI, Editor, Preview)
  - Right panel: Content area switching based on activeTab
  - Three main views:
    1. AIAssistant: AI generation interface
    2. Editor: Form sections (Basic, Personality, Prompts, etc.)
    3. Preview: JSON preview with export buttons

### Character Form Components

**BasicInfoSection.tsx**
- Fields: name, nickname, description, avatar upload
- AI generation buttons for description enhancement

**PersonalitySection.tsx**
- Fields: personality, scenario, first_mes (first message)
- AI generation for personality and scenario

**PromptsSection.tsx**
- Fields: system_prompt, post_history_instructions, mes_example
- AI-powered prompt generation

**AlternateGreetings.tsx**
- Manage multiple greeting messages
- Add/remove/reorder greetings
- AI generation for alternate greetings

**CharacterBook.tsx**
- Lorebook entry management
- Keywords (trigger words) + content
- insertion_order, enabled toggles
- AI generation for book entries

**TagsSection.tsx**
- Tag management (add/remove)
- AI tag generation from character data

**MetadataSection.tsx**
- Fields: creator, character_version, creator_notes
- creation_date, modification_date (auto-managed)
- source field

### AI Integration

**aiGenerator.ts** (360 lines)
- **Core Function**: `generateWithAI(settings, prompt)` (lines 40-180)
  - OpenAI-compatible API calls
  - Provider detection (local vs remote)
  - API key validation (skip for Ollama, LM Studio)
  - Error handling with provider-specific messages
  - Response parsing with multiple fallbacks

- **Prompt Generators**:
  - `generateDescription()`: Physical appearance (lines 185-202)
  - `generatePersonality()`: Traits, behavior (lines 207-214)
  - `generateScenario()`: Backstory, meta-environment (lines 219-227)
  - `generateFirstMes()`: Opening message (lines 229-238)
  - `generateMesExample()`: Dialogue examples (lines 243-262)
  - `generateSystemPrompt()`: AI instructions (lines 267-278)
  - `generatePostHistoryInstructions()`: Brief AI guidance (lines 283-294)
  - `generateTags()`: Keyword generation (lines 296-305)
  - `generateAlternateGreeting()`: Additional greetings (lines 320-335)
  - `generateCharacterBookEntry()`: Lorebook entries (lines 340-359)

- **Utility**: `estimateTokens(text)` (lines 21-28)
  - Chinese chars: 1.5 tokens each
  - English words: 1 token each
  - Other chars: 0.5 tokens each

**buildApiUrl.ts**
- Intelligent endpoint construction
- Provider-specific path mapping:
  - Ollama: `/api/chat`
  - LM Studio: `/v1/chat/completions`
  - OpenAI: `/v1/chat/completions`
  - Others: `/v1/chat/completions` (default)

**AISettings.tsx** (464 lines)
- Provider selection: OpenAI, Anthropic, OpenRouter, Ollama, LM Studio
- Model configuration:
  - Predefined models per provider
  - Custom model input
- API key management (localStorage persistence)
- Custom endpoint URLs
- Local provider detection (no API key required)

### Export/Import Logic

**CharacterPreview.tsx** (lines 63-180)
- **JSON Export**: Standard V3 format download
- **PNG Export**: Embed character data in PNG tEXt chunk
  - Read avatar image via canvas
  - Convert to PNG ArrayBuffer
  - Locate IEND chunk
  - Insert tEXt chunk before IEND:
    - Chunk length (4 bytes)
    - Chunk type: "tEXt" (4 bytes)
    - Keyword: "chara" + null byte
    - Data: Base64-encoded JSON
    - CRC32 checksum (4 bytes)
  - Write IEND chunk
  - Download as PNG file

**Index.tsx PNG Import** (lines 177-381)
- **Method 1**: Parse PNG tEXt chunks
  - Search for tEXt chunk signature (0x74455874)
  - Extract keyword and data
  - Check for keywords: "chara", "ccv3", "ccv2", "Comment"
  - Try base64 decode → JSON parse
  - Try direct JSON parse

- **Method 2**: String search for JSON patterns
  - Decode full PNG as UTF-8 text
  - Search for patterns:
    - `"spec":"chara_card_v[123]"`
    - `"name":"`
    - `{"name":`
    - `{"char_name":`
  - Extract JSON by matching braces
  - Parse JSON

- **Method 3**: Base64 string search
  - Find long base64 strings (100+ chars)
  - Decode to UTF-8
  - Check for character data markers
  - Parse JSON

**V1/V2 Conversion** (lines 424-508)
- Auto-detect spec version
- Map V1/V2 fields to V3 structure
- Handle type mismatches (string vs array)
- Set default values for new V3 fields

## Electron Desktop Integration

**electron/main.cjs** (171 lines)
- **Window Configuration**:
  - Size: 1400x900 (default), 1200x800 (minimum)
  - Security: contextIsolation, no nodeIntegration
  - Title: "Tavern Card Crafter - AI character card creation tool"

- **Development Mode** (lines 29-35):
  - Load `http://localhost:6090` (ISSUE: vite uses 8080)
  - Open DevTools automatically
  - Hot reload via electron-reload

- **Production Mode** (lines 34):
  - Load `dist/index.html` from file:// protocol

- **Application Menu** (lines 91-170):
  - File: New, Import (Ctrl+O), Export (Ctrl+S), Exit
  - Edit: Undo, Redo, Cut, Copy, Paste
  - View: Reload, DevTools (F12), Zoom, Fullscreen (F11)
  - Help: About dialog

- **Security**:
  - Prevent new windows (open external URLs in browser)
  - No remote module access
  - Web security enforced

**electron/preload.js**
- Empty preload script (IPC bridge placeholder)
- contextBridge available for future IPC implementation

## Context & State Management

**LanguageContext.tsx**
- Provides `useLanguage()` hook
- Translation function `t(key)`
- Language switching (English, Chinese)

**ThemeContext.tsx**
- Provides `useTheme()` hook
- Theme switching: light, dark, system
- Persists preference in localStorage

**Global State Pattern**:
- Character data: Local state in Index.tsx
- AI settings: localStorage + Context (via AISettings component)
- Theme: Context API + next-themes
- Language: Context API

## Build Configuration

**vite.config.ts**
- Base path: `"./"` (critical for Electron file:// protocol)
- Server: port 8080, host "::" (IPv6)
- Plugins: React SWC, lovable-tagger (dev only)
- Path alias: `@` → `./src`
- Build output: `dist/` directory

**tsconfig.json**
- Base config referencing tsconfig.app.json and tsconfig.node.json
- Path alias: `@/*` → `./src/*`
- Relaxed type checking:
  - noImplicitAny: false
  - noUnusedParameters: false
  - strictNullChecks: false
  - allowJs: true

**electron-builder.json**
- appId: `com.tavern.cardcrafter`
- productName: "Tavern Card Crafter"
- Files: dist/**, electron/**, package.json
- Output: build-electron/
- Targets:
  - macOS: DMG
  - Linux: AppImage, deb
  - Windows: NSIS installer

**package.json Scripts**
- `dev`: Vite dev server
- `build`: Production build
- `electron-dev`: Concurrent dev (Vite + Electron)
- `electron`: Production Electron (no rebuild)
- `electron-pack`: Build + run production
- `electron-build`: Create installers
- `dist`: Build installers without publish

## Data Models

### CharacterCardV3 Interface

```typescript
interface CharacterCardV3 {
  spec: "chara_card_v3";
  spec_version: "3.0";
  data: {
    // Basic Info
    name: string;
    nickname?: string;
    description: string;

    // Personality & Scenario
    personality: string;
    scenario: string;

    // Dialogue
    first_mes: string;
    mes_example: string;
    alternate_greetings: string[];
    group_only_greetings: string[];

    // Prompts
    system_prompt: string;
    post_history_instructions: string;

    // Metadata
    tags: string[];
    creator: string;
    character_version: string;
    creator_notes: string;
    creator_notes_multilingual?: { [key: string]: string };
    source?: string;
    creation_date?: string;
    modification_date?: string;

    // Lorebook
    character_book?: {
      entries: CharacterBookEntry[];
    };

    // Extensions & Assets
    extensions: Record<string, any>;
    assets: Asset[];
  };
}

interface CharacterBookEntry {
  keys: string[];              // Trigger keywords
  content: string;             // Entry content
  insertion_order: number;     // Priority
  enabled: boolean;            // Active toggle
  use_regex?: boolean;         // V3 addition
  constant?: boolean;          // V3 required
  // Optional fields...
}

interface Asset {
  type: string;
  uri: string;
  name: string;
  ext: string;
}
```

### AISettings Interface

```typescript
interface AISettings {
  provider: string;      // "openai" | "anthropic" | "openrouter" | "ollama" | "lmstudio"
  model: string;         // Model identifier
  apiKey: string;        // API key (optional for local)
  apiUrl: string;        // API endpoint URL
}
```

## Dependencies Analysis

### Core Dependencies
- **react** (19.2.0): Latest React with new JSX transform
- **react-dom** (19.2.0): React DOM renderer
- **typescript** (5.9.3): TypeScript compiler
- **vite** (7.1.11): Build tool and dev server
- **electron** (38.4.0): Desktop app framework

### UI Framework
- **@radix-ui/react-*** (multiple): Headless UI primitives (50+ components)
- **tailwindcss** (3.4.18): Utility-first CSS
- **tailwind-merge** (3.3.1): Merge Tailwind classes
- **tailwindcss-animate** (1.0.7): Animation utilities
- **lucide-react** (0.546.0): Icon library

### Form & Validation
- **react-hook-form** (7.65.0): Form state management
- **@hookform/resolvers** (5.2.2): Validation resolvers
- **zod** (4.1.12): Schema validation

### State & Data
- **@tanstack/react-query** (5.90.5): Async state management
- **react-router-dom** (7.9.4): Client-side routing

### UI Components
- **sonner** (2.0.7): Toast notifications
- **cmdk** (1.1.1): Command palette
- **vaul** (1.1.2): Drawer component
- **embla-carousel-react** (8.6.0): Carousel
- **recharts** (3.3.0): Chart library
- **next-themes** (0.4.6): Theme management

### Electron-Specific
- **electron-builder** (26.0.12): Packaging and distribution
- **electron-reload** (2.0.0-alpha.1): Hot reload in dev
- **concurrently** (9.2.1): Run multiple npm scripts
- **wait-on** (9.0.1): Wait for dev server before launching Electron
- **cross-env** (10.1.0): Cross-platform env variables

## Key Algorithms & Techniques

### PNG Steganography
- **tEXt Chunk Structure**: [length][type]["chara"\0][base64 data][CRC]
- **CRC32 Calculation**: Polynomial 0xEDB88320, used for chunk integrity
- **Multi-Method Extraction**: Graceful degradation (tEXt → string search → base64)
- **UTF-8 Handling**: TextDecoder with fatal:false for encoding issues

### Token Estimation
- **Heuristic-Based**: Not using tokenizer library
- **Language-Aware**: Different multipliers for Chinese, English, other chars
- **Formula**: `ceil(chinese * 1.5 + english_words + other * 0.5)`

### AI Response Parsing
- **JSON Extraction**: Multiple fallback patterns
- **Error Recovery**: Handle incomplete/malformed responses
- **Timeout Handling**: 120s timeout with AbortController

### V1/V2 Conversion
- **Field Mapping**: Direct mapping with type coercion
- **Array Normalization**: Convert single strings to arrays
- **Default Values**: Fill missing V3 fields with sensible defaults

## Security Considerations

### Electron Security
- ✅ contextIsolation enabled
- ✅ nodeIntegration disabled
- ✅ webSecurity enabled
- ✅ External URLs open in system browser
- ⚠️ No Content Security Policy headers

### API Key Storage
- ⚠️ Stored in localStorage (client-side only)
- ⚠️ No encryption at rest
- ✅ Not transmitted except to configured providers
- ⚠️ Visible in DevTools

### Input Validation
- ⚠️ Limited input sanitization
- ⚠️ No XSS protection in AI-generated content
- ⚠️ JSON parsing without schema validation

## Known Issues & Technical Debt

### ISSUE-001: Port Mismatch
- **Location**: `package.json:17` vs `vite.config.ts:12`
- **Details**: electron-dev script waits for port 6090, Vite uses 8080
- **Impact**: Electron dev mode connection failure
- **Fix**: Update package.json script to `wait-on http://localhost:8080`

### Technical Debt
1. **TypeScript Strictness**: Relaxed type checking for rapid prototyping
2. **Error Handling**: Inconsistent error boundaries
3. **Testing**: No unit tests, integration tests, or E2E tests
4. **Accessibility**: Missing ARIA labels, keyboard navigation gaps
5. **i18n**: Incomplete English translation from Chinese
6. **Documentation**: Limited inline comments, no API docs

## Development Patterns

### Component Hierarchy
```
App (Router + Providers)
└── Index (Main Page)
    ├── Toolbar (Theme/Language)
    ├── AISettings (Modal)
    ├── Sidebar Navigation
    └── Content Panels
        ├── AIAssistant
        ├── Editor
        │   ├── BasicInfoSection
        │   ├── PersonalitySection
        │   ├── PromptsSection
        │   ├── AlternateGreetings
        │   ├── CharacterBook
        │   ├── TagsSection
        │   └── MetadataSection
        └── CharacterPreview
```

### State Flow
```
User Input → Local State (useState)
          → updateField() → characterData state
          → JSON Preview → Export

AI Generation → aiGenerator.ts → generateWithAI()
              → Parsed JSON → onInsertField()
              → Merge with characterData
```

### File Organization
- **Components**: Grouped by feature (CharacterForm/*) or UI library (ui/*)
- **Utilities**: Separated by concern (aiGenerator, buildApiUrl)
- **Contexts**: Global state providers
- **Pages**: Route components
- **Hooks**: Reusable React logic

## Performance Characteristics

### Bundle Size
- **shadcn/ui**: 40+ components, tree-shakeable
- **Radix UI**: Headless primitives, small bundle impact
- **TanStack Query**: ~20KB gzipped

### Optimization Opportunities
1. Code splitting by route
2. Lazy load AI generation UI
3. Virtual scrolling for large character books
4. Web Worker for PNG parsing
5. Service Worker for offline support

## Extension Points

### Adding New AI Providers
1. Update `buildApiUrl.ts` with provider endpoint mapping
2. Add provider to AISettings predefined list
3. Handle provider-specific error messages in `aiGenerator.ts`

### Custom Export Formats
1. Create new encoder in `components/CharacterForm/`
2. Add export button in CharacterPreview
3. Implement format conversion logic

### Additional Character Fields
1. Update CharacterCardV3 interface
2. Add form section component
3. Update AI prompt generators
4. Handle V1/V2 conversion if applicable

---

**Generated**: 2025-12-28
**Repomix Version**: 1.5.0
**Analysis Method**: Automated + Manual Review
**Maintainer**: Tavern Card Crafter Team
