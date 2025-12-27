# System Architecture

## Architecture Overview

Tavern Card Crafter v3 is a hybrid Electron desktop application with embedded React web UI. It follows a **monolithic client-side architecture** with no backend services, prioritizing user privacy and offline functionality.

### Architecture Style
- **Type**: Desktop-first, single-page application (SPA)
- **Deployment**: Self-contained Electron executable
- **Data Flow**: Unidirectional (Flux-like pattern)
- **State Management**: Context API + Local State
- **Communication**: Direct API calls to external AI providers

## High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                     ELECTRON DESKTOP APP                        │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                   Main Process (Node.js)                   │  │
│  │  ┌─────────────┐  ┌──────────────┐  ┌─────────────────┐  │  │
│  │  │ Window Mgmt │  │ App Menu     │  │ Native Dialogs  │  │  │
│  │  │ main.cjs    │  │ Shortcuts    │  │ File System     │  │  │
│  │  └─────────────┘  └──────────────┘  └─────────────────┘  │  │
│  └───────────────────────────┬───────────────────────────────┘  │
│                              │ IPC (future)                      │
│  ┌───────────────────────────┴───────────────────────────────┐  │
│  │              Renderer Process (Chromium)                   │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │                  REACT WEB APP                       │  │  │
│  │  │                                                       │  │  │
│  │  │  ┌────────────────────────────────────────────────┐  │  │  │
│  │  │  │         Presentation Layer                     │  │  │  │
│  │  │  │  ┌──────────┐  ┌──────────┐  ┌──────────────┐  │  │  │  │
│  │  │  │  │ AI Panel │  │ Editor   │  │ Preview      │  │  │  │  │
│  │  │  │  │ (Tab 1)  │  │ (Tab 2)  │  │ (Tab 3)      │  │  │  │  │
│  │  │  │  └──────────┘  └──────────┘  └──────────────┘  │  │  │  │
│  │  │  └────────────────────────────────────────────────┘  │  │  │
│  │  │                         │                             │  │  │
│  │  │  ┌────────────────────────────────────────────────┐  │  │  │
│  │  │  │         State Management Layer                 │  │  │  │
│  │  │  │  ┌────────────┐  ┌─────────────┐  ┌─────────┐  │  │  │  │
│  │  │  │  │ Character  │  │ AI Settings │  │ Theme/  │  │  │  │  │
│  │  │  │  │ Data State │  │ (localStorage)│  │ i18n   │  │  │  │  │
│  │  │  │  └────────────┘  └─────────────┘  └─────────┘  │  │  │  │
│  │  │  └────────────────────────────────────────────────┘  │  │  │
│  │  │                         │                             │  │  │
│  │  │  ┌────────────────────────────────────────────────┐  │  │  │
│  │  │  │         Business Logic Layer                   │  │  │  │
│  │  │  │  ┌──────────────┐  ┌──────────────────────┐    │  │  │  │
│  │  │  │  │ aiGenerator  │  │ PNG Steganography    │    │  │  │  │
│  │  │  │  │ - Prompts    │  │ - Encode/Decode      │    │  │  │  │
│  │  │  │  │ - API Calls  │  │ - CRC32 Validation   │    │  │  │  │
│  │  │  │  └──────────────┘  └──────────────────────┘    │  │  │  │
│  │  │  └────────────────────────────────────────────────┘  │  │  │
│  │  │                         │                             │  │  │
│  │  │  ┌────────────────────────────────────────────────┐  │  │  │
│  │  │  │         Integration Layer                      │  │  │  │
│  │  │  │  ┌──────────┐  ┌──────────┐  ┌──────────────┐  │  │  │  │
│  │  │  │  │ Fetch API│  │ File API │  │ localStorage │  │  │  │  │
│  │  │  │  └──────────┘  └──────────┘  └──────────────┘  │  │  │  │
│  │  │  └────────────────────────────────────────────────┘  │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
└────────────────────────────┬────────────────────────────────────┘
                             │ HTTPS
         ┌───────────────────┴───────────────────┐
         │                                       │
    ┌────▼─────┐  ┌────────────┐  ┌────────────▼─────┐
    │ OpenAI   │  │ Anthropic  │  │ OpenRouter       │
    │ API      │  │ API        │  │ (Remote AI)      │
    └──────────┘  └────────────┘  └──────────────────┘
         │
    ┌────▼─────────────────┐
    │ Ollama / LM Studio   │
    │ (Local AI - HTTP)    │
    └──────────────────────┘
```

## Component Architecture

### Layer 1: Electron Main Process

**Responsibilities**:
- Native window management and lifecycle
- Application menu and keyboard shortcuts
- File system access (future: file dialogs)
- Security isolation between main and renderer

**Key Files**:
- `electron/main.cjs`: Main process entry point
- `electron/preload.js`: IPC bridge (currently unused)

**Communication**:
- No IPC currently implemented
- Future: contextBridge for secure IPC

**Security Model**:
```javascript
webPreferences: {
  nodeIntegration: false,        // No Node.js in renderer
  contextIsolation: true,         // Separate JS contexts
  enableRemoteModule: false,      // No @electron/remote
  webSecurity: true               // Enforce CORS, CSP
}
```

### Layer 2: Renderer Process (React App)

#### 2.1 Presentation Layer

**Main Page Component**: `pages/Index.tsx`
- **Pattern**: Container component with three child views
- **Routing**: Not used (single-page UI with tab switching)
- **Layout**: Sidebar navigation + content panels

**View Components**:
1. **AIAssistant** (`components/CharacterForm/AIAssistant.tsx`)
   - Input: User text, character type
   - Output: Parsed character data
   - Interaction: Generate button → AI call → Parse → Insert fields

2. **Editor** (Multiple form sections)
   - BasicInfoSection: Name, avatar, description
   - PersonalitySection: Personality, scenario, first message
   - PromptsSection: System prompt, post-history instructions
   - AlternateGreetings: Multiple greeting management
   - CharacterBook: Lorebook entries
   - TagsSection: Tag management
   - MetadataSection: Creator info, dates

3. **Preview** (`components/CharacterPreview.tsx`)
   - Input: Character data state
   - Output: Syntax-highlighted JSON, export buttons
   - Actions: Copy, Download JSON, Export PNG

**UI Component Library**: shadcn/ui (40+ components)
- Built on Radix UI primitives
- Styled with Tailwind CSS
- Customizable via `src/components/ui/` files

#### 2.2 State Management Layer

**State Architecture**:
```
Global State (Context API)
├── LanguageContext: i18n strings, language switcher
└── ThemeContext: dark/light/system theme

Component State (useState)
└── Index.tsx
    ├── characterData: CharacterCardV3 (primary state)
    ├── characterImage: Base64 avatar
    ├── aiSettings: AI provider config (synced to localStorage)
    └── activeTab: Current view ("assistant" | "editor" | "preview")
```

**State Update Pattern**:
```typescript
// Unidirectional flow
User Action → Event Handler → updateField() → setState → Re-render

// Example: Edit character name
<Input onChange={(e) => updateField('name', e.target.value)} />
                          ↓
updateField(field, value) {
  setCharacterData(prev => ({
    ...prev,
    data: {
      ...prev.data,
      [field]: value,
      modification_date: today
    }
  }));
}
```

**Persistence Strategy**:
- **Character Data**: Not persisted (user must export)
- **AI Settings**: localStorage (key: `ai-settings`)
- **Theme**: localStorage via next-themes
- **Language**: localStorage via LanguageContext

#### 2.3 Business Logic Layer

**AI Generation** (`utils/aiGenerator.ts`)

**Flow Diagram**:
```
┌─────────────────────────────────────────────────────────────┐
│                    AI Generation Flow                       │
└─────────────────────────────────────────────────────────────┘
                             │
                    User clicks "Generate"
                             │
                             ▼
              ┌──────────────────────────┐
              │  Validate AI Settings    │
              │  - Check provider        │
              │  - Validate API key      │
              │    (skip for local)      │
              └──────────┬───────────────┘
                         │
                         ▼
              ┌──────────────────────────┐
              │  Build API Request       │
              │  - Construct URL         │
              │  - Format prompt         │
              │  - Set headers           │
              └──────────┬───────────────┘
                         │
                         ▼
              ┌──────────────────────────┐
              │  Fetch API Call          │
              │  - POST /chat/completions│
              │  - Timeout: 120s         │
              │  - AbortController       │
              └──────────┬───────────────┘
                         │
             ┌───────────┴──────────────┐
             │                          │
    ┌────────▼─────────┐    ┌───────────▼──────────┐
    │  Success (200)   │    │  Error (4xx/5xx)     │
    │                  │    │  - Parse error body  │
    │  Parse response: │    │  - Provider-specific │
    │  1. data.choices │    │    error messages    │
    │  2. data.response│    │  - Network errors    │
    │  3. data.text    │    └──────────────────────┘
    │  4. data.content │               │
    └────────┬─────────┘               │
             │                          │
             ▼                          ▼
    ┌──────────────────┐    ┌────────────────────┐
    │ Return content   │    │ Throw Error        │
    └──────────────────┘    └────────────────────┘
```

**Prompt Engineering Strategy**:
- **Character Type Context**: Adjust tone based on type (anime, game, novel, etc.)
- **Literary Fusion**: Blend writing styles (Joyce, Dick, Nin, Adams, Le Guin)
- **Progressive Detail**: Build on existing fields (name → description → personality → scenario)
- **Instruction Clarity**: Explicit format requirements in prompts

**PNG Steganography** (`pages/Index.tsx`, `components/CharacterPreview.tsx`)

**Encoding Process**:
```
┌─────────────────────────────────────────────────────────┐
│                PNG Encoding (Export)                    │
└─────────────────────────────────────────────────────────┘
                         │
    User uploads avatar + clicks "Export PNG"
                         │
                         ▼
          ┌──────────────────────────┐
          │  Load Avatar Image       │
          │  - Draw to canvas        │
          │  - Convert to Blob       │
          └──────────┬───────────────┘
                     │
                     ▼
          ┌──────────────────────────┐
          │  Prepare Character Data  │
          │  - JSON.stringify(V3)    │
          │  - Base64 encode         │
          └──────────┬───────────────┘
                     │
                     ▼
          ┌──────────────────────────┐
          │  Parse PNG Structure     │
          │  - Read as ArrayBuffer   │
          │  - Locate IEND chunk     │
          └──────────┬───────────────┘
                     │
                     ▼
          ┌──────────────────────────┐
          │  Create tEXt Chunk       │
          │  ┌────────────────────┐  │
          │  │ [4] Length         │  │
          │  │ [4] Type: "tEXt"   │  │
          │  │ [n] Keyword+"chara"│  │
          │  │ [1] Null byte      │  │
          │  │ [n] Base64 data    │  │
          │  │ [4] CRC32 checksum │  │
          │  └────────────────────┘  │
          └──────────┬───────────────┘
                     │
                     ▼
          ┌──────────────────────────┐
          │  Insert Before IEND      │
          │  - Copy PNG header       │
          │  - Copy existing chunks  │
          │  - Insert tEXt chunk     │
          │  - Append IEND           │
          └──────────┬───────────────┘
                     │
                     ▼
          ┌──────────────────────────┐
          │  Download PNG File       │
          │  - Create Blob URL       │
          │  - Trigger download      │
          └──────────────────────────┘
```

**Decoding Process (Import)**:
```
┌─────────────────────────────────────────────────────────┐
│                PNG Decoding (Import)                    │
└─────────────────────────────────────────────────────────┘
                         │
           User selects PNG file
                         │
                         ▼
          ┌──────────────────────────┐
          │  Read File as ArrayBuffer│
          └──────────┬───────────────┘
                     │
                     ▼
          ┌──────────────────────────┐
          │  METHOD 1: tEXt Chunks   │
          │  - Search for 0x74455874 │
          │  - Extract keyword+data  │
          │  - Try base64 decode     │
          │  - Try direct JSON parse │
          └──────────┬───────────────┘
                     │
                     ▼
             ┌───────┴────────┐
             │ Found?         │
             └───┬────────┬───┘
               Yes│      │No
                 │        │
                 │        ▼
                 │   ┌──────────────────────────┐
                 │   │  METHOD 2: String Search │
                 │   │  - Decode to UTF-8 text  │
                 │   │  - Regex for JSON start  │
                 │   │  - Match braces          │
                 │   │  - Parse JSON            │
                 │   └──────────┬───────────────┘
                 │              │
                 │              ▼
                 │      ┌───────┴────────┐
                 │      │ Found?         │
                 │      └───┬────────┬───┘
                 │        Yes│      │No
                 │          │        │
                 │          │        ▼
                 │          │   ┌──────────────────────────┐
                 │          │   │  METHOD 3: Base64 Search │
                 │          │   │  - Find base64 patterns  │
                 │          │   │  - Decode each match     │
                 │          │   │  - Check for char data   │
                 │          │   │  - Parse JSON            │
                 │          │   └──────────┬───────────────┘
                 │          │              │
                 ▼          ▼              ▼
          ┌──────────────────────────────────────┐
          │  V1/V2/V3 Format Detection           │
          │  - Check spec field                  │
          │  - Auto-convert to V3 if needed      │
          └──────────┬───────────────────────────┘
                     │
                     ▼
          ┌──────────────────────────┐
          │  Update Character State  │
          │  - Set characterData     │
          │  - Set characterImage    │
          └──────────────────────────┘
```

#### 2.4 Integration Layer

**External API Integration**:

**Provider Architecture**:
```
┌────────────────────────────────────────────────────────┐
│              AI Provider Abstraction                   │
└────────────────────────────────────────────────────────┘
                         │
          ┌──────────────┴──────────────┐
          │                             │
    ┌─────▼─────┐               ┌───────▼──────┐
    │  Remote   │               │    Local     │
    │ Providers │               │  Providers   │
    └─────┬─────┘               └───────┬──────┘
          │                             │
    ┌─────┴──────────────┐         ┌────┴────────────┐
    │                    │         │                 │
┌───▼───┐  ┌──────────┐ │    ┌────▼────┐  ┌────────▼─┐
│OpenAI │  │Anthropic │ │    │ Ollama  │  │LM Studio │
│       │  │          │ │    │         │  │          │
│Cloud  │  │  Cloud   │ │    │localhost│  │localhost │
│       │  │          │ │    │:11434   │  │:1234     │
└───────┘  └──────────┘ │    └─────────┘  └──────────┘
           ┌──────────┐ │
           │OpenRouter│ │
           │          │ │
           │Cloud/Agg │ │
           └──────────┘ │
                        │
          Requires API Key
          Uses HTTPS
                                 No API Key
                                 Uses HTTP
```

**API Request Flow**:
```typescript
// 1. Build URL (buildApiUrl.ts)
const apiUrl = buildApiUrl(settings.apiUrl, settings.provider);
// Example: "https://api.openai.com/v1/chat/completions"
// Example: "http://localhost:11434/api/chat"

// 2. Construct Headers
const headers = {
  'Content-Type': 'application/json',
  'Authorization': `Bearer ${settings.apiKey}` // Skip for local
};

// 3. Build Request Body (OpenAI-compatible)
const requestBody = {
  model: settings.model,
  messages: [{ role: 'user', content: prompt }],
  max_tokens: 1000,
  temperature: 0.7
};

// 4. Fetch with Timeout
const response = await fetch(apiUrl, {
  method: 'POST',
  headers,
  body: JSON.stringify(requestBody),
  signal: AbortSignal.timeout(120000)
});

// 5. Parse Response
const data = await response.json();
const content = data.choices[0]?.message?.content || data.response;
```

**Error Handling Strategy**:
```
Error Type                     Handler
────────────────────────────────────────────────────────
Network Error                  → Check provider status, suggest fixes
Timeout (120s)                 → Suggest retry, check network
401 Unauthorized               → Invalid API key message
404 Not Found                  → Invalid endpoint/model message
400 Bad Request (local)        → Model not loaded, list available models
429 Rate Limit                 → Quota exceeded, wait or upgrade
500 Server Error               → Provider issue, try later
Empty Response                 → Quota/overload warning
AbortController Canceled       → User cancellation, cleanup
```

**File System Integration**:
- **File Picker**: HTML `<input type="file">` (no Electron dialog yet)
- **File Reading**: FileReader API (ArrayBuffer, DataURL, Text)
- **File Writing**: Blob download via anchor tag
- **Persistence**: None (user-initiated export only)

**localStorage Schema**:
```typescript
// Key: "ai-settings"
{
  provider: "openai" | "anthropic" | "openrouter" | "ollama" | "lmstudio",
  model: string,
  apiKey: string,
  apiUrl: string
}

// Key: "theme" (managed by next-themes)
"light" | "dark" | "system"

// Key: "language" (managed by LanguageContext)
"en" | "zh"
```

## Data Flow Architecture

### Character Creation Flow

```
┌─────────────────────────────────────────────────────────────┐
│                  Character Creation Flow                    │
└─────────────────────────────────────────────────────────────┘

1. AI-Assisted Path:
   User Input (text)
        │
        ▼
   AI Generation (aiGenerator.ts)
        │
        ▼
   Parse JSON Response
        │
        ▼
   onInsertField() → Merge with characterData
        │
        ▼
   Switch to Editor Tab → Manual refinement
        │
        ▼
   updateField() → Update characterData state
        │
        ▼
   Switch to Preview Tab
        │
        ▼
   Export (JSON or PNG)

2. Manual Path:
   Switch to Editor Tab
        │
        ▼
   Fill form sections
        │
        ▼
   updateField() for each change
        │
        ▼
   Switch to Preview Tab
        │
        ▼
   Export (JSON or PNG)

3. Import Path:
   Select JSON/PNG file
        │
        ▼
   Parse file (extractPNGCharacterData for PNG)
        │
        ▼
   Detect format (V1/V2/V3)
        │
        ▼
   Auto-convert to V3 if needed
        │
        ▼
   setCharacterData() → Update state
        │
        ▼
   Switch to Editor Tab → Review/edit
```

### State Synchronization

**Pattern**: Lift state up, pass callbacks down
```
Index.tsx (State Owner)
├── characterData: CharacterCardV3
├── updateField: (field, value) => void
└── onInsertField: (field, value) => void
    │
    ├─→ AIAssistant (onInsertField callback)
    │   └── Calls onInsertField when AI generates data
    │
    ├─→ Editor Sections (updateField callback)
    │   └── Call updateField on user input
    │
    └─→ CharacterPreview (characterData prop)
        └── Read-only display, triggers export
```

## Security Architecture

### Threat Model

**Threats**:
1. **Malicious PNG Import**: Crafted PNG with XSS payload in character data
2. **API Key Leakage**: localStorage accessible to malicious browser extensions
3. **MITM Attacks**: Unencrypted local AI providers (HTTP)
4. **Code Injection**: Unsanitized AI responses rendered as HTML
5. **Electron-Specific**: Renderer process escape to main process

**Mitigations**:
1. ✅ Context isolation prevents renderer → main process access
2. ⚠️ No input sanitization for AI-generated content
3. ⚠️ localStorage not encrypted (client-side only mitigation)
4. ⚠️ Local AI uses HTTP (localhost only)
5. ✅ No `dangerouslySetInnerHTML` in React components

### Security Layers

```
┌───────────────────────────────────────────────────────┐
│           Security Architecture Layers                │
└───────────────────────────────────────────────────────┘

Layer 1: Electron Security
─────────────────────────────
- nodeIntegration: false      ← No Node.js in renderer
- contextIsolation: true      ← Separate contexts
- webSecurity: true           ← Enforce CORS, CSP
- allowRunningInsecureContent: false

Layer 2: Network Security
─────────────────────────────
- HTTPS for remote APIs       ← Encrypted in transit
- HTTP for localhost only     ← Local AI providers
- No CORS issues (Electron)   ← Not browser-based
- Timeout: 120s               ← Prevent hanging

Layer 3: Data Security
─────────────────────────────
- localStorage only           ← No server transmission
- No encryption at rest       ⚠️ Potential improvement
- User-initiated export       ← No automatic sync
- No telemetry/analytics      ← Privacy-first

Layer 4: Input Validation
─────────────────────────────
- TypeScript type checking    ← Compile-time safety
- Zod schema validation       ⚠️ Not fully utilized
- JSON.parse error handling   ← Graceful failures
- File type checking          ← .json, .png only
```

## Scalability & Performance

### Performance Characteristics

**Bottlenecks**:
1. **AI Generation**: 5-30s latency (external API dependency)
2. **PNG Parsing**: O(n) file scan, slow for large PNGs (>10MB)
3. **JSON Preview**: Re-render on every character data change
4. **Bundle Size**: 40+ shadcn/ui components not tree-shaken

**Optimization Strategies**:
```
Current State → Optimization → Expected Gain
────────────────────────────────────────────────────────
PNG Parsing (main thread)
  → Web Worker parsing
  → 50% perceived performance (non-blocking UI)

AI Generation (blocking UI)
  → Loading states + AbortController
  → ✅ Already implemented

JSON Preview (eager re-render)
  → React.memo + useMemo for syntax highlighting
  → 70% fewer re-renders

Bundle Size (all components loaded)
  → Code splitting by route
  → 40% smaller initial bundle

Character Book (large lists)
  → Virtual scrolling (react-window)
  → 90% faster rendering for 100+ entries
```

### Scalability Limits

**Current Constraints**:
- **Character Size**: No limit, but 100KB+ JSON may slow preview
- **Lorebook Entries**: No pagination, UI degrades at 50+ entries
- **Concurrent AI Calls**: One at a time (AbortController prevents duplicates)
- **File Size**: PNG import struggles with >20MB files

**Future Scalability**:
- Add pagination for character book entries
- Implement virtual scrolling for large lists
- Stream AI responses for faster perceived performance
- Optimize PNG parsing with Web Workers

## Deployment Architecture

### Build Process

```
┌───────────────────────────────────────────────────────────┐
│                    Build Pipeline                         │
└───────────────────────────────────────────────────────────┘

Development Mode:
  npm run electron-dev
       │
       ├─→ vite dev (localhost:8080)
       │    ├─ Hot Module Replacement
       │    ├─ Source maps enabled
       │    └─ Fast refresh
       │
       └─→ electron . (development mode)
            ├─ Load http://localhost:6090 (ISSUE: port mismatch)
            ├─ Open DevTools
            └─ electron-reload enabled

Production Build:
  npm run electron-build
       │
       ├─→ vite build
       │    ├─ TypeScript compilation
       │    ├─ React SWC optimization
       │    ├─ Tailwind CSS purge
       │    ├─ Tree shaking
       │    └─ Minification → dist/
       │
       └─→ electron-builder
            ├─ Package dist/ + electron/
            ├─ Code signing (optional)
            ├─ Platform-specific builds:
            │   ├─ macOS: DMG (Apple Silicon + Intel)
            │   ├─ Windows: NSIS Installer, Portable
            │   └─ Linux: AppImage, deb
            └─ Output → build-electron/
```

### Deployment Targets

```
Platform     Format          Size (approx)   Distribution
─────────────────────────────────────────────────────────────
macOS        DMG             ~150MB          Direct download
             Universal       (Electron +     Self-update (future)
                             Chromium +
                             React bundle)

Windows      NSIS Installer  ~120MB          Direct download
             Portable .exe   ~140MB          USB/portable

Linux        AppImage        ~130MB          Direct download
             .deb            ~120MB          apt repository (future)
```

### Update Mechanism

**Current**: Manual download (no auto-update)

**Future**: electron-updater integration
```
Check for updates
     │
     ▼
Compare versions (semver)
     │
     ▼
Download delta update
     │
     ▼
Verify signature
     │
     ▼
Install on restart
```

## Technology Decisions

### Why Electron?
✅ Cross-platform desktop with single codebase
✅ Native file system access (future features)
✅ Offline-first architecture (privacy)
✅ No browser compatibility issues
⚠️ Large bundle size (~120-150MB)
⚠️ Memory overhead (Chromium instance)

### Why React 19?
✅ Mature ecosystem, extensive libraries
✅ Component-based architecture
✅ TypeScript integration
✅ Concurrent features (future optimization)
⚠️ Not as lightweight as Svelte/Solid

### Why Vite?
✅ Fast dev server with HMR
✅ Optimized production builds
✅ Native ESM support
✅ React SWC for faster transpilation
⚠️ Electron integration requires configuration

### Why shadcn/ui over Material-UI?
✅ Headless primitives (Radix UI) for accessibility
✅ Copy-paste customization (no complex theming)
✅ Smaller bundle (tree-shakeable)
✅ Tailwind CSS integration
⚠️ Manual component updates (not npm package)

### Why localStorage over IndexedDB?
✅ Simple API for small data (AI settings)
✅ Synchronous reads (no async complexity)
✅ Adequate for config persistence
⚠️ 5-10MB limit (not an issue for current use case)

## Extension Architecture

### Plugin System (Future)

**Proposed Architecture**:
```
┌─────────────────────────────────────────────────────┐
│                  Plugin System                      │
└─────────────────────────────────────────────────────┘

Core App
  │
  ├─→ Plugin Loader (main process)
  │    ├─ Scan plugins/ directory
  │    ├─ Validate plugin.json manifest
  │    ├─ Sandboxed execution (Web Workers)
  │    └─ IPC communication with main
  │
  ├─→ Plugin API (exposed to plugins)
  │    ├─ aiGenerator.register(customGenerator)
  │    ├─ exporters.register(customExporter)
  │    ├─ ui.registerPanel(customPanel)
  │    └─ events.on('characterUpdate', callback)
  │
  └─→ Plugin UI (React components)
       ├─ Plugin settings panel
       ├─ Custom export formats
       └─ Additional AI providers
```

### Custom Export Formats

**Current**: JSON, PNG
**Extensible via**:
```typescript
// Future API
interface ExportPlugin {
  name: string;
  fileExtension: string;
  mimeType: string;
  export: (characterData: CharacterCardV3) => Blob;
}

// Example: PDF export
const pdfExporter: ExportPlugin = {
  name: "PDF Character Sheet",
  fileExtension: ".pdf",
  mimeType: "application/pdf",
  export: (data) => generatePDF(data)
};
```

---

**Document Version**: 1.0
**Last Updated**: 2025-12-28
**Maintainer**: Tavern Card Crafter Team
