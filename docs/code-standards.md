# Code Standards & Development Guidelines

## Overview

This document establishes coding standards, architectural patterns, and best practices for Tavern Card Crafter development.

## Project Structure

### Directory Organization

```
tavern-card-crafter-v3/
├── electron/                      # Electron main process
│   ├── main.cjs                   # Main process entry point
│   └── preload.js                 # Preload script (IPC bridge)
├── src/                           # React application source
│   ├── components/                # React components
│   │   ├── CharacterForm/         # Character editing sections
│   │   │   ├── AIAssistant.tsx    # AI generation interface
│   │   │   ├── BasicInfoSection.tsx
│   │   │   ├── PersonalitySection.tsx
│   │   │   ├── PromptsSection.tsx
│   │   │   ├── AlternateGreetings.tsx
│   │   │   ├── CharacterBook.tsx
│   │   │   ├── TagsSection.tsx
│   │   │   ├── MetadataSection.tsx
│   │   │   ├── CharaCardV2.js     # V2 format encoder
│   │   │   └── CharaCardV3.js     # V3 format encoder
│   │   ├── ui/                    # shadcn/ui components
│   │   ├── AISettings.tsx         # AI provider configuration
│   │   ├── CharacterPreview.tsx   # JSON preview & export
│   │   ├── LocalDeploymentPanel.tsx
│   │   └── Toolbar.tsx            # Theme/language switcher
│   ├── contexts/                  # React Context providers
│   │   ├── LanguageContext.tsx    # i18n state
│   │   └── ThemeContext.tsx       # Theme state
│   ├── hooks/                     # Custom React hooks
│   │   ├── use-mobile.tsx
│   │   └── use-toast.ts
│   ├── lib/                       # Utility libraries
│   │   └── utils.ts               # Tailwind cn() helper
│   ├── pages/                     # Page components
│   │   ├── Index.tsx              # Main application page
│   │   └── NotFound.tsx
│   ├── utils/                     # Business logic utilities
│   │   ├── aiGenerator.ts         # AI generation functions
│   │   └── buildApiUrl.ts         # API URL construction
│   ├── App.tsx                    # Root component
│   ├── main.tsx                   # React entry point
│   └── vite-env.d.ts
├── public/                        # Static assets
│   └── favicon.ico
├── docs/                          # Documentation
├── plans/                         # Project plans
│   └── reports/                   # Generated reports
├── .claude/                       # Claude Code configuration
│   ├── workflows/                 # Development workflows
│   └── skills/                    # Agent skills
├── vite.config.ts                 # Vite configuration
├── tsconfig.json                  # TypeScript config (base)
├── tsconfig.app.json              # TypeScript config (app)
├── tsconfig.node.json             # TypeScript config (Node)
├── tailwind.config.ts             # Tailwind configuration
├── postcss.config.js
├── eslint.config.js
├── electron-builder.json          # Electron Builder config
├── package.json                   # npm dependencies & scripts
└── README.md
```

### File Naming Conventions

**React Components**: PascalCase with `.tsx` extension
- Examples: `BasicInfoSection.tsx`, `CharacterPreview.tsx`

**Utilities**: camelCase with `.ts` extension
- Examples: `aiGenerator.ts`, `buildApiUrl.ts`

**Contexts**: PascalCase with `Context` suffix
- Examples: `LanguageContext.tsx`, `ThemeContext.tsx`

**Hooks**: camelCase with `use-` prefix
- Examples: `use-toast.ts`, `use-mobile.tsx`

**Electron**: CommonJS `.cjs` extension (required for main process)
- Examples: `main.cjs`

## TypeScript Standards

### Configuration

**Compiler Options** (tsconfig.json):
```json
{
  "noImplicitAny": false,          // Relaxed for rapid development
  "noUnusedParameters": false,      // Disabled for prototyping
  "noUnusedLocals": false,          // Disabled for prototyping
  "strictNullChecks": false,        // Relaxed type safety
  "skipLibCheck": true,             // Skip lib validation
  "allowJs": true                   // Allow JS files
}
```

**Rationale**: Relaxed TypeScript configuration prioritizes development velocity over strict type safety. Production builds should gradually enable stricter checks.

### Type Definitions

**Component Props**: Always define interfaces for component props
```typescript
// Good
interface AIAssistantProps {
  aiSettings: AISettings | null;
  onInsertField: (field: string, value: string | string[]) => void;
}

const AIAssistant = ({ aiSettings, onInsertField }: AIAssistantProps) => {
  // ...
};

// Bad
const AIAssistant = (props: any) => {
  // ...
};
```

**State Types**: Use TypeScript inference where clear, explicit types for complex state
```typescript
// Good - clear inference
const [isGenerating, setIsGenerating] = useState(false);

// Good - explicit complex type
const [characterData, setCharacterData] = useState<CharacterCardV3>({
  spec: "chara_card_v3",
  spec_version: "3.0",
  // ...
});

// Bad - unnecessary explicit primitive
const [count, setCount] = useState<number>(0);
```

**Utility Functions**: Export interfaces for complex return types
```typescript
// aiGenerator.ts
export interface CharacterData {
  name: string;
  nickname?: string;
  description: string;
  personality?: string;
  // ...
}

export const generateWithAI = async (
  settings: AISettings,
  prompt: string
): Promise<string> => {
  // ...
};
```

### ESLint Rules

**Disabled Rules** (for specific cases):
```typescript
/* eslint-disable @typescript-eslint/no-explicit-any */
// Use sparingly, only for:
// 1. Third-party library types without declarations
// 2. Dynamic JSON parsing (AI responses, character card imports)
// 3. Event handlers with unknown structure
```

**Enforced Rules**:
- No unused variables in production code
- Consistent arrow function syntax
- No console.log in production (use console.error/warn)

## React Patterns

### Component Structure

**Functional Components**: Use arrow functions with TypeScript
```typescript
const ComponentName = ({ prop1, prop2 }: ComponentProps) => {
  // Hooks first
  const { toast } = useToast();
  const { t } = useLanguage();

  // State declarations
  const [state, setState] = useState(initialValue);

  // Effects
  useEffect(() => {
    // side effects
  }, [dependencies]);

  // Event handlers
  const handleEvent = () => {
    // handler logic
  };

  // Render
  return (
    <div>
      {/* JSX */}
    </div>
  );
};
```

### State Management

**Local State**: Use `useState` for component-specific state
```typescript
// Character form state (Index.tsx:84)
const [characterData, setCharacterData] = useState<CharacterCardV3>({
  spec: "chara_card_v3",
  spec_version: "3.0",
  data: {
    name: "",
    description: "",
    // ... full V3 structure
  }
});
```

**Context**: Use Context API for global state
```typescript
// LanguageContext.tsx
export const LanguageContext = createContext<LanguageContextType>(defaultValue);

export const useLanguage = () => {
  const context = useContext(LanguageContext);
  if (!context) {
    throw new Error('useLanguage must be used within LanguageProvider');
  }
  return context;
};
```

**Props Drilling**: Pass update functions down, not setState directly
```typescript
// Good - dedicated update function
const updateField = (field: string, value: any) => {
  setCharacterData(prev => ({
    ...prev,
    data: {
      ...prev.data,
      [field]: value,
      modification_date: new Date().toISOString().split('T')[0]
    }
  }));
};

<BasicInfoSection updateField={updateField} />

// Bad - passing setState
<BasicInfoSection setCharacterData={setCharacterData} />
```

### Hooks Usage

**Custom Hooks**: Extract reusable logic
```typescript
// hooks/use-toast.ts
export function useToast() {
  const [state, setState] = useState<ToastState>(initialState);

  const toast = useCallback((props: ToastProps) => {
    // toast logic
  }, []);

  return { toast, state };
}
```

**Effect Dependencies**: Always specify complete dependency arrays
```typescript
// Good
useEffect(() => {
  const savedSettings = localStorage.getItem('ai-settings');
  if (savedSettings) {
    setAISettings(JSON.parse(savedSettings));
  }
}, []); // Empty array = run once on mount

// Bad
useEffect(() => {
  // ...
}); // Missing dependency array
```

## Component Patterns

### Form Sections

**Pattern**: Controlled components with prop-based updates
```typescript
interface SectionProps {
  data: CharacterCardV3['data'];
  updateField: (field: string, value: any) => void;
  aiSettings?: AISettings | null;
}

const BasicInfoSection = ({ data, updateField, aiSettings }: SectionProps) => {
  return (
    <Card>
      <CardHeader>
        <CardTitle>Basic Information</CardTitle>
      </CardHeader>
      <CardContent>
        <Input
          value={data.name}
          onChange={(e) => updateField('name', e.target.value)}
        />
      </CardContent>
    </Card>
  );
};
```

### AI Integration Components

**Pattern**: Async operations with loading states and error handling
```typescript
const AIAssistant = ({ aiSettings, onInsertField }: AIAssistantProps) => {
  const [isGenerating, setIsGenerating] = useState(false);
  const abortControllerRef = useRef<AbortController | null>(null);

  const handleGenerate = async () => {
    if (!aiSettings) {
      toast({ title: "Error", description: "Configure AI settings first" });
      return;
    }

    setIsGenerating(true);
    abortControllerRef.current = new AbortController();

    try {
      const result = await generateWithAI(aiSettings, prompt);
      const parsed = JSON.parse(result);
      setParsedData(parsed);
    } catch (error) {
      toast({
        title: "Generation Failed",
        description: error.message,
        variant: "destructive"
      });
    } finally {
      setIsGenerating(false);
    }
  };

  // Cleanup on unmount
  useEffect(() => {
    return () => abortControllerRef.current?.abort();
  }, []);
};
```

## Data Handling

### Character Card Format

**V3 Structure**: Always maintain complete V3 structure in state
```typescript
// Index.tsx:84
const [characterData, setCharacterData] = useState<CharacterCardV3>({
  spec: "chara_card_v3",
  spec_version: "3.0",
  data: {
    name: "",
    nickname: "",
    description: "",
    personality: "",
    mes_example: "",
    scenario: "",
    first_mes: "",
    alternate_greetings: [],
    tags: [],
    creator: "",
    character_version: "",
    creator_notes: "",
    system_prompt: "",
    post_history_instructions: "",
    character_book: { entries: [] },
    group_only_greetings: [],
    creation_date: today,
    modification_date: today,
    extensions: {},
    assets: []
  }
});
```

**Import Compatibility**: Convert V1/V2 to V3 during import
```typescript
// Index.tsx:424-508
if (parsedData.spec === "chara_card_v3") {
  setCharacterData(parsedData);
} else if (parsedData.spec === "chara_card_v2") {
  // V2 → V3 conversion
  const v3Data: CharacterCardV3 = {
    spec: "chara_card_v3",
    spec_version: "3.0",
    data: {
      name: parsedData.data?.name || "",
      // ... map all V2 fields to V3
    }
  };
  setCharacterData(v3Data);
} else {
  // V1 → V3 conversion
  // ... similar mapping
}
```

### PNG Steganography

**Export Format**: Embed JSON in PNG tEXt chunk
```typescript
// CharacterPreview.tsx:97-180
const embedCharacterData = (uint8Array: Uint8Array, jsonData: string) => {
  const keyword = "chara"; // or "ccv3"
  const encodedData = btoa(jsonData); // Base64 encode

  // Create tEXt chunk:
  // [length: 4 bytes][type: "tEXt"][keyword + null + data][CRC: 4 bytes]
  const keywordBytes = new TextEncoder().encode(keyword);
  const dataBytes = new TextEncoder().encode(encodedData);
  // ... chunk assembly
};
```

**Import Parsing**: Multi-method extraction with fallbacks
```typescript
// Index.tsx:177-381
const extractPNGCharacterData = async (file: File): Promise<any> => {
  // Method 1: Find tEXt chunks
  for (let i = 8; i < uint8Array.length - 8; i++) {
    // Search for tEXt chunk (0x74455874)
    if (uint8Array[i+4] === 0x74 && /* ... */) {
      // Decode keyword, check for 'chara'/'ccv3'/'ccv2'
      // Try base64 decode → JSON parse
    }
  }

  // Method 2: String search for JSON patterns
  const patterns = [/\"spec\"\\s*:\\s*\"chara_card_v[123]\"/g, /* ... */];
  // Search full file text for JSON structures

  // Method 3: Base64 string search
  const base64Pattern = /[A-Za-z0-9+/]{100,}={0,2}/g;
  // Decode and check for character data markers
};
```

## AI Integration Standards

### Provider Configuration

**API URL Construction** (buildApiUrl.ts):
```typescript
export const buildApiUrl = (baseUrl: string, provider: string): string => {
  const trimmedUrl = baseUrl.trim().replace(/\/$/, '');

  // Provider-specific endpoint mapping
  const providerEndpoints: Record<string, string> = {
    'ollama': '/api/chat',
    'lmstudio': '/v1/chat/completions',
    'openai': '/v1/chat/completions',
    // ...
  };

  const endpoint = providerEndpoints[provider.toLowerCase()] || '/v1/chat/completions';
  return `${trimmedUrl}${endpoint}`;
};
```

**Request Format**: OpenAI-compatible chat completions
```typescript
// aiGenerator.ts:76-81
const requestBody = {
  model: settings.model,
  messages: [{ role: 'user', content: prompt }],
  max_tokens: 1000,
  temperature: 0.7
};
```

### Error Handling

**Tiered Error Messages**: Provider-specific, actionable feedback
```typescript
// aiGenerator.ts:94-121
if (!response.ok) {
  let errorMessage = `API request failed: ${response.status}`;

  // Parse error body
  const errorData = JSON.parse(errorText);
  if (errorData.error?.message) {
    errorMessage += ` - ${errorData.error.message}`;
  }

  // Local service specific errors
  if (localServices.includes(provider)) {
    if (response.status === 400 || errorText.includes('model')) {
      errorMessage = `Model "${settings.model}" not present or loaded. Check available models.`;
    }
  }

  throw new Error(errorMessage);
}
```

**Network Error Handling**:
```typescript
// aiGenerator.ts:164-175
if (error.name === 'TimeoutError') {
  throw new Error('Request timeout, check network or API status');
}
if (error.message.includes('Failed to fetch')) {
  if (provider === 'ollama') {
    throw new Error('Cannot connect to Ollama. Ensure: ollama serve');
  }
  // ... provider-specific guidance
}
```

## Styling Standards

### Tailwind CSS Usage

**Component Classes**: Use `cn()` utility for conditional classes
```typescript
import { cn } from "@/lib/utils";

<button
  className={cn(
    "w-full flex items-center gap-3 px-3 py-3 rounded-lg transition-all",
    activeTab === "assistant"
      ? "bg-gradient-to-r from-purple-600 to-blue-600 text-white shadow-md"
      : "text-gray-700 dark:text-gray-300 hover:bg-gray-100"
  )}
>
```

**Color Palette**: Use Tailwind theme colors
- Primary: `purple-600`, `blue-600`
- Gradients: `from-purple-600 to-blue-600`
- Dark mode: `dark:from-purple-900 dark:via-blue-900`

### shadcn/ui Components

**Import Pattern**: Import from `@/components/ui/*`
```typescript
import { Button } from "@/components/ui/button";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Input } from "@/components/ui/input";
```

**Customization**: Modify in `src/components/ui/` files, not inline

## Electron Integration

### Main Process (electron/main.cjs)

**Window Configuration**:
```javascript
const mainWindow = new BrowserWindow({
  width: 1400, height: 900,
  minWidth: 1200, minHeight: 800,
  webPreferences: {
    nodeIntegration: false,        // Security: no Node in renderer
    contextIsolation: true,         // Security: isolate contexts
    enableRemoteModule: false,      // Security: no remote module
    webSecurity: true               // Security: enforce web security
  }
});
```

**Development vs Production**:
```javascript
const isDev = process.env.NODE_ENV === 'development';

if (isDev) {
  mainWindow.loadURL('http://localhost:6090'); // Dev server
  mainWindow.webContents.openDevTools();
} else {
  mainWindow.loadFile(path.join(__dirname, '../dist/index.html'));
}
```

### Security Best Practices

**Context Isolation**: Use preload script for IPC
```javascript
// electron/preload.js
const { contextBridge } = require('electron');

contextBridge.exposeInMainWorld('electronAPI', {
  // Expose safe APIs only
});
```

**No nodeIntegration**: Never enable `nodeIntegration: true`
**CSP Headers**: Content Security Policy in production
**External Links**: Open in system browser, not new windows

## Build & Deployment

### Development Scripts

```bash
# Web development
npm run dev                  # Vite dev server (port 8080)

# Electron development
npm run electron-dev         # Concurrent: Vite + Electron
                             # Waits for localhost:6090 (ISSUE: should be 8080)

# Production
npm run build                # Build web app → dist/
npm run electron             # Run Electron with built app
npm run electron-pack        # Build + run production
npm run electron-build       # Create installers
```

### Build Configuration

**Vite** (vite.config.ts):
```typescript
export default defineConfig({
  base: "./",                       // CRITICAL for Electron file:// protocol
  server: {
    host: "::",                     // IPv6 support
    port: 8080,                     // ISSUE: package.json expects 6090
  },
  build: {
    outDir: "dist",
    assetsDir: "assets"
  }
});
```

**Electron Builder** (electron-builder.json):
```json
{
  "appId": "com.tavern.cardcrafter",
  "productName": "Tavern Card Crafter",
  "files": [
    "dist/**/*",
    "electron/**/*",
    "package.json"
  ],
  "directories": {
    "output": "build-electron"
  },
  "mac": { "target": "dmg" },
  "linux": { "target": ["AppImage", "deb"] },
  "win": { "target": "nsis" }
}
```

## Testing Standards

### Manual Testing Checklist

**AI Generation**:
- [ ] Test all 5 character types (general, anime, game, novel, historical)
- [ ] Test with empty/malformed AI responses
- [ ] Test with network timeout
- [ ] Test with invalid API key
- [ ] Test local providers (Ollama, LM Studio)

**Import/Export**:
- [ ] Import V1 JSON → converts to V3
- [ ] Import V2 JSON → converts to V3
- [ ] Import V3 JSON → preserves structure
- [ ] Import PNG with embedded data
- [ ] Export JSON with all fields
- [ ] Export PNG with embedded data
- [ ] Re-import exported PNG successfully

**Electron**:
- [ ] Launch on macOS, Windows, Linux
- [ ] Menu shortcuts work (Cmd/Ctrl+N/O/S)
- [ ] Window size persistence
- [ ] Dev mode hot reload
- [ ] Production build loads correctly

## Documentation Standards

### JSDoc Comments

**Required for**:
- All exported functions
- Complex algorithms (PNG parsing, AI generation)
- Public component props interfaces

**Format**:
```typescript
/**
 * Generates a response from an AI service based on settings and prompt.
 *
 * This function checks if the AI service requires an API key, constructs the API URL,
 * prepares the request body, and handles the response including error management.
 *
 * @param settings - AI service configuration (provider, model, API key, URL)
 * @param prompt - Input prompt for AI generation
 * @returns Promise resolving to generated response content
 * @throws Error if API key missing, URL not configured, or request fails
 */
export const generateWithAI = async (
  settings: AISettings,
  prompt: string
): Promise<string> => {
  // ...
};
```

### Code Comments

**When to Comment**:
- Complex algorithms (PNG chunk parsing)
- Non-obvious business logic (V1/V2 conversion rules)
- Workarounds for browser/Electron quirks
- Security considerations

**When NOT to Comment**:
- Self-explanatory code
- Type information (use TypeScript)
- Obvious variable names

## Git Workflow

### Commit Messages

**Format**: `<type>: <subject>`

**Types**:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style (formatting, no logic change)
- `refactor`: Code restructuring (no feature/fix)
- `test`: Test additions/changes
- `chore`: Build, dependencies, tooling

**Examples**:
```
feat: Add support for Character Card V3 specification
fix: Resolve port mismatch between Vite and Electron dev mode
docs: Update README with PNG export instructions
refactor: Extract AI generation logic to aiGenerator.ts
```

### Branch Strategy

**Main Branch**: `prime` (production-ready)
**Feature Branches**: `feature/<description>`
**Bugfix Branches**: `fix/<issue-description>`

## Common Pitfalls & Solutions

### Issue: Port Mismatch in Development

**Problem**: `package.json:17` uses port 6090, `vite.config.ts:12` uses 8080
**Impact**: Electron dev mode fails to connect
**Solution**:
```json
// package.json - Update electron-dev script
"electron-dev": "concurrently \"npm run dev\" \"wait-on http://localhost:8080 && cross-env NODE_ENV=development electron .\""
```

### Issue: TypeScript Strict Mode Errors

**Problem**: Legacy code has loose typing
**Strategy**: Gradual migration
1. Keep strict mode disabled globally
2. Enable `strictNullChecks` per-file with `// @ts-check`
3. Refactor incrementally

### Issue: PNG Import Fails

**Problem**: Multi-encoding issues (UTF-8, base64)
**Solution**: Three-tier extraction strategy (Index.tsx:177-381)
1. Try tEXt chunk parsing
2. Fall back to string search
3. Fall back to base64 detection

---

**Document Version**: 1.0
**Last Updated**: 2025-12-28
**Maintainer**: Tavern Card Crafter Team
