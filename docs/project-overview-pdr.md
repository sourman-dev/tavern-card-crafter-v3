# Tavern Card Crafter - Project Overview & PDR

## Executive Summary

**Project:** Tavern Card Crafter v3
**Version:** 0.0.0
**Type:** Electron Desktop Application + Web App
**Purpose:** Professional AI-powered character card creation tool for chatbot and roleplay applications
**Target Platform:** Character Card V3 Specification (backward compatible with V1/V2)

## Product Vision

Tavern Card Crafter simplifies AI character card creation by combining intelligent AI generation with comprehensive manual editing capabilities. Users can paste any character-related text, leverage AI to extract structured information, and fine-tune every aspect of their character card before exporting to JSON or PNG formats.

## Target Users

### Primary Personas

1. **Chatbot Creators**: Developers building conversational AI characters for platforms like SillyTavern, TavernAI
2. **Roleplay Enthusiasts**: Users creating detailed characters for roleplay scenarios and interactive fiction
3. **Content Creators**: Authors, game designers, worldbuilders needing structured character documentation
4. **AI Researchers**: Teams experimenting with character AI behavior and prompt engineering

### User Segments

- **Casual Users**: Simple character creation with AI assistance
- **Power Users**: Advanced editing with character books, lorebooks, multiple greetings
- **Developers**: JSON manipulation, API integration, local LLM deployment

## Core Features

### 1. AI-Powered Generation

**Capability**: Intelligent extraction and generation of character attributes from unstructured text

**Components**:
- Multi-provider support: OpenAI-compatible APIs, Ollama, LM Studio, OpenRouter
- Configurable models and endpoints
- Field-specific generation (description, personality, scenario, greetings, etc.)
- Context-aware prompts with literary style fusion (Joyce, Dick, Nin, Adams, Le Guin)

**File References**:
- `/src/utils/aiGenerator.ts`: Core generation logic, token estimation
- `/src/components/AISettings.tsx`: Provider configuration UI
- `/src/components/CharacterForm/AIAssistant.tsx`: AI extraction interface

### 2. Comprehensive Character Editing

**Capability**: Full-featured editor supporting Character Card V3 specification

**Sections**:
- Basic Info: Name, nickname, description, avatar
- Personality: Traits, behavior patterns, quirks
- Prompts: System prompts, post-history instructions
- Dialogue: First message, conversation examples, alternate greetings
- Character Book: Lorebook entries with keyword triggers
- Tags & Metadata: Classification, versioning, creator info

**File References**:
- `/src/pages/Index.tsx`: Main layout, three-panel UI (AI | Editor | Preview)
- `/src/components/CharacterForm/BasicInfoSection.tsx`: Basic attributes
- `/src/components/CharacterForm/PersonalitySection.tsx`: Character traits
- `/src/components/CharacterForm/PromptsSection.tsx`: System prompts
- `/src/components/CharacterForm/CharacterBook.tsx`: Lorebook management

### 3. Multi-Format Export

**Capability**: Export character cards as JSON or PNG with embedded steganography

**Formats**:
- **JSON**: Pure character data (V3 spec compliant)
- **PNG**: Avatar image with embedded character data in tEXt chunk
- **Import**: Both JSON and PNG card import with V1/V2 auto-conversion

**Technical Details**:
- PNG steganography uses tEXt chunk with keywords: `chara`, `ccv3`, `ccv2`, `Comment`
- Base64 encoding for embedded data
- UTF-8 text decoding with fallback handling
- Multi-method extraction: tEXt chunk → string search → base64 detection

**File References**:
- `/src/components/CharacterPreview.tsx`: Export logic (lines 63-180)
- `/src/pages/Index.tsx`: Import logic with PNG parsing (lines 177-381)

### 4. Cross-Platform Desktop App

**Capability**: Standalone Electron application with native features

**Features**:
- Native window management (1400x900 default, 1200x800 minimum)
- Application menu with shortcuts (Ctrl/Cmd+N/O/S)
- Security hardening: no nodeIntegration, contextIsolation enabled
- Hot reload in development mode
- Production packaging: DMG (macOS), Portable/AppImage (Linux), Installer (Windows)

**File References**:
- `/electron/main.cjs`: Electron main process
- `/electron-builder.json`: Build configuration

## Technical Architecture

### Technology Stack

**Frontend**:
- React 19.2.0 with TypeScript 5.9.3
- Vite 7.1.11 (build tool)
- Tailwind CSS 3.4.18 + shadcn/ui components
- TanStack Query 5.90.5 (data fetching)
- React Hook Form 7.65.0 + Zod 4.1.12 (validation)
- React Router DOM 7.9.4

**Desktop**:
- Electron 38.4.0
- electron-builder 26.0.12

**Development**:
- ESLint 9.38.0
- TypeScript ESLint 8.46.2
- Vite Plugin React SWC 4.1.0

### Architecture Patterns

**State Management**:
- Context API: `LanguageContext` (i18n), `ThemeContext` (dark mode)
- Local component state for form data
- localStorage for AI settings persistence

**Component Structure**:
- Container/Presenter separation
- Form sections as isolated components
- Shared UI components from shadcn/ui

**Data Flow**:
1. User input → AI generation → Parsed data
2. Manual editing → Character state updates
3. Character state → JSON preview → Export

## Product Requirements

### Functional Requirements

**FR-001**: AI Character Generation
- **Priority**: P0
- **Description**: Generate structured character data from unstructured text input
- **Acceptance Criteria**:
  - Support 5+ character types (general, anime, game, novel, historical)
  - Parse AI response to JSON with error handling
  - Handle empty/malformed responses gracefully
  - Generate all V3 spec fields (name, description, personality, scenario, etc.)

**FR-002**: Character Card Import/Export
- **Priority**: P0
- **Description**: Import/export character cards in JSON and PNG formats
- **Acceptance Criteria**:
  - Import V1, V2, V3 JSON formats with auto-conversion
  - Extract character data from PNG tEXt chunks
  - Export JSON with proper formatting
  - Embed character data into PNG avatar images
  - Preserve UTF-8 encoding for international characters

**FR-003**: Manual Character Editing
- **Priority**: P0
- **Description**: Comprehensive editing UI for all Character Card V3 fields
- **Acceptance Criteria**:
  - Edit basic info, personality, scenario, prompts
  - Manage alternate greetings (add/remove/reorder)
  - Create character book entries with keywords
  - Tag management with autocomplete
  - Real-time JSON preview with syntax highlighting

**FR-004**: AI Provider Configuration
- **Priority**: P1
- **Description**: Configure multiple AI providers and models
- **Acceptance Criteria**:
  - Support OpenAI, Anthropic, OpenRouter, Ollama, LM Studio
  - Model selection from predefined list or custom input
  - API key management with secure storage
  - Custom endpoint URLs with intelligent path completion

**FR-005**: Desktop Application
- **Priority**: P1
- **Description**: Cross-platform Electron desktop application
- **Acceptance Criteria**:
  - Launch as standalone app on macOS, Windows, Linux
  - Native menu with keyboard shortcuts
  - Window state persistence
  - Production builds: DMG, AppImage, Portable, Installer

### Non-Functional Requirements

**NFR-001**: Performance
- **Metric**: AI generation response time < 30s (P95)
- **Metric**: UI interaction response < 100ms
- **Metric**: PNG export < 5s for images up to 2MB

**NFR-002**: Compatibility
- **Standard**: Character Card V3 Specification
- **Backward Compatibility**: V1/V2 import with lossless conversion
- **Browser Support**: Web version works in Chrome 90+, Firefox 88+, Safari 14+

**NFR-003**: Usability
- **Metric**: New user creates first character within 5 minutes
- **Requirement**: All AI settings persist across sessions
- **Requirement**: Inline help text for all major features

**NFR-004**: Security
- **Requirement**: API keys stored in localStorage (client-side only)
- **Requirement**: No remote data transmission except to configured AI providers
- **Requirement**: Electron security best practices (contextIsolation, no nodeIntegration)

**NFR-005**: Maintainability
- **Requirement**: TypeScript strict mode disabled for rapid development
- **Requirement**: Component-based architecture with clear separation of concerns
- **Requirement**: Comprehensive JSDoc comments for complex functions

## Technical Constraints

### Known Issues

**ISSUE-001**: Port Mismatch
- **Location**: `package.json:17` vs `vite.config.ts:12`
- **Details**: package.json electron-dev script expects port 6090, vite.config.ts uses 8080
- **Impact**: Electron dev mode fails to connect
- **Resolution**: Align both configs to same port (recommend 8080)

### Dependencies

**Critical**:
- Node.js 16+ (for Electron 38.4.0)
- npm 7+ (for workspace support)

**Optional**:
- Ollama service for local LLM (when using local AI)
- LM Studio for local LLM (alternative)

## Development Roadmap

### Phase 1: Core Stability (Current)
- Fix port mismatch issue
- Add comprehensive error handling for AI failures
- Improve PNG parsing robustness
- Add input validation for all fields

### Phase 2: Enhanced Features (Q1 2025)
- Multi-character project management
- Character versioning and diff viewer
- Template library for common character archetypes
- Advanced lorebook editor with visual graph

### Phase 3: Collaboration (Q2 2025)
- Cloud sync for character library
- Collaborative editing with conflict resolution
- Character marketplace integration
- Community template sharing

### Phase 4: Advanced AI (Q3 2025)
- Fine-tuned models for character generation
- Character testing sandbox
- Dialogue simulation preview
- Personality consistency checker

## Success Metrics

### User Engagement
- **DAU/MAU**: 30%+ engagement ratio
- **Character Creation Rate**: 2+ characters per active user per week
- **Export Rate**: 80%+ of created characters exported

### Quality Metrics
- **AI Generation Success Rate**: >95% successful completions
- **Character Completeness**: >70% of exports have all V3 fields filled
- **User Retention**: 60%+ return after first character creation

### Technical Metrics
- **Build Success Rate**: 100% CI/CD pass rate
- **Error Rate**: <1% application crashes
- **API Latency**: P95 < 30s for AI generation

## Appendix

### Character Card V3 Specification Fields

```typescript
interface CharacterCardV3 {
  spec: "chara_card_v3";
  spec_version: "3.0";
  data: {
    name: string;
    nickname?: string;
    description: string;
    personality: string;
    mes_example: string;
    scenario: string;
    first_mes: string;
    alternate_greetings: string[];
    tags: string[];
    creator: string;
    character_version: string;
    creator_notes: string;
    creator_notes_multilingual?: { [key: string]: string };
    system_prompt: string;
    post_history_instructions: string;
    character_book?: {
      entries: CharacterBookEntry[];
    };
    group_only_greetings: string[];
    creation_date?: string;
    modification_date?: string;
    source?: string;
    extensions: Record<string, any>;
    assets: Asset[];
  };
}
```

### AI Provider Configuration

**Supported Providers**:
- OpenAI (api.openai.com)
- Anthropic (api.anthropic.com)
- OpenRouter (openrouter.ai)
- Ollama (localhost:11434) - local, no API key
- LM Studio (localhost:1234) - local, no API key

**API Format**: OpenAI-compatible chat completions
**Required Headers**: `Content-Type: application/json`, `Authorization: Bearer {key}`
**Request Body**: `{ model, messages, max_tokens, temperature }`

### Build Commands Reference

```bash
# Web Development
npm run dev                 # Vite dev server (port 8080)
npm run build               # Production build → dist/
npm run preview             # Preview production build

# Electron Development
npm run electron-dev        # Web + Electron dev mode
npm run electron            # Electron production mode (no rebuild)

# Electron Production
npm run electron-pack       # Build + run production
npm run electron-build      # Build installers (DMG/AppImage/etc)
npm run dist                # Build without publishing
```

### Localization Status

**Current**: Partial English translation from original Chinese repo
**Supported Languages**: English (primary), Chinese (legacy)
**i18n Implementation**: `LanguageContext` with translation keys

---

**Document Version**: 1.0
**Last Updated**: 2025-12-28
**Maintainer**: Tavern Card Crafter Team
