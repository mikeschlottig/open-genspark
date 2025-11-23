# CLAUDE.md - AI Assistant Guide for Open Genspark (Super Agent)

## Project Overview

**Open Genspark** (Super Agent) is an AI-powered assistant application built with Next.js that leverages Google's Gemini AI for content generation, presentation creation, and Google Workspace integration.

### What It Does
- AI chat interface for content generation
- Automatic PowerPoint presentation creation
- Google Sheets/Docs integration and analysis
- Web browsing via Puppeteer automation

### Target Users
- Content creators needing quick presentations
- Business users working with Google Workspace
- Anyone needing AI-assisted document analysis

### Problem Solved
Provides a unified interface for AI-powered content creation and Google Workspace integration, eliminating the need for multiple tools.

## Tech Stack

### Core
- **Framework**: Next.js 15.3.5
- **Language**: TypeScript 5.x
- **React**: 19.0.0
- **Styling**: Tailwind CSS 4.1.11

### AI & Integrations
- **AI SDK**: Vercel AI SDK 4.3.8
- **AI Provider**: @ai-sdk/google 0.0.52 (Google Gemini)
- **Integrations**: @composio/core 0.1.35, @composio/vercel 0.1.35

### Key Libraries
- **Presentations**: pptxgenjs 3.12.0
- **Browser Automation**: Puppeteer 22.6.5, Playwright 1.54.1
- **Markdown**: react-markdown 10.1.0, remark-gfm 4.0.1
- **Animation**: Framer Motion 12.23.6
- **3D**: Three.js 0.178.0, @react-three/fiber 9.2.0
- **Validation**: Zod 3.22.4

## Project Structure

```
open-genspark/
├── app/                        # Next.js App Router
│   ├── api/                    # API Routes
│   │   ├── superagent/         # Main AI agent endpoint
│   │   ├── convert-to-ppt/     # PowerPoint generation
│   │   ├── generate-slides/    # Slide content generation
│   │   ├── google-sheets-agent/ # Google Sheets analysis
│   │   ├── connection/         # OAuth connection handlers
│   │   │   ├── google-sheet/   # Google Sheets OAuth
│   │   │   └── google-docs/    # Google Docs OAuth
│   │   └── connecting-email/   # Email integration
│   ├── components/             # Page components
│   │   ├── SuperAgent.tsx      # Main chat interface (715 lines)
│   │   ├── PPTCreator.tsx      # Presentation creator
│   │   ├── SlidePreview.tsx    # Slide preview component
│   │   ├── GoogleSheetsAgent.tsx
│   │   ├── SparkPages.tsx
│   │   └── Navigation.tsx
│   ├── signin/                 # Authentication page
│   ├── layout.tsx              # Root layout
│   └── page.tsx                # Home page
├── components/                 # Shared UI components
│   └── ui/                     # Reusable UI components
│       ├── button.tsx
│       ├── chat-message-list.tsx
│       ├── ai-prompt-box.tsx
│       ├── hyper-text.tsx
│       ├── meteors.tsx
│       └── ...
├── hooks/                      # Custom React hooks
│   ├── use-auto-scroll.ts
│   └── use-textarea-resize.tsx
├── lib/                        # Utilities
│   └── utils.ts
├── public/                     # Static assets
├── middleware.ts               # Auth middleware
├── package.json
├── tailwind.config.js
├── tsconfig.json
└── next.config.ts
```

## Architecture & Data Flow

### Request Flow
1. User sends message in `SuperAgent.tsx`
2. Request goes to `/api/superagent` route
3. Composio tools are initialized (slide generator, puppeteer)
4. Gemini AI processes with available tools
5. Response returned with optional slide data
6. Slides rendered using HTML templates or converted to PPTX

### Key Abstractions
- **Composio SDK**: Handles tool registration and execution
- **Vercel AI SDK**: Manages AI model interactions
- **Custom Tools**: Slide generation, Puppeteer browser automation

### Authentication Flow
1. User visits `/signin`
2. OAuth initiated via Composio for Google Sheets + Docs
3. Cookies set: `googlesheet_user_id`, `googledoc_user_id`
4. Middleware checks cookies for protected routes

## Environment Variables

**Required** (create `.env.local`):
```bash
GOOGLE_GENERATIVE_AI_API_KEY=   # From Google AI Studio
COMPOSIO_API_KEY=               # From Composio dashboard
NODE_ENV=development            # development/production
```

## Common Commands

```bash
# Install dependencies (REQUIRES --legacy-peer-deps)
npm install --legacy-peer-deps

# Development server
npm run dev

# Production build
npm run build

# Start production server
npm start

# Lint
npm run lint
```

## API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/superagent` | POST | Main AI chat endpoint |
| `/api/generate-slides` | POST | Generate slide content |
| `/api/convert-to-ppt` | POST | Convert slides to PPTX |
| `/api/google-sheets-agent` | POST | Google Sheets analysis |
| `/api/connection/google-sheet` | POST/GET | Google Sheets OAuth |
| `/api/connection/google-docs` | POST/GET | Google Docs OAuth |

---

## Dependency Issues & Solutions

### CRITICAL: React 19 Peer Dependency Conflicts

**Issue**: React 19 is bleeding-edge and many dependencies haven't updated peer dependencies.

**Symptom**: `npm install` fails without `--legacy-peer-deps`

**Solution**:
```bash
# Always use this flag
npm install --legacy-peer-deps

# Or add to .npmrc
echo "legacy-peer-deps=true" >> .npmrc
```

**Long-term fix**: Wait for ecosystem to catch up or downgrade to React 18.

### Redundant Browser Automation Libraries

**Issue**: Both `puppeteer` and `playwright` are installed (redundant).

**Impact**:
- Increased bundle size (~150MB+ for both)
- Maintenance burden

**Solution**: Remove one. Recommend keeping Puppeteer since it's used in code:
```bash
npm uninstall playwright
```

### Early-Stage Dependencies (High Risk)

**Issue**: Several dependencies are v0.x with unstable APIs:

| Package | Version | Risk |
|---------|---------|------|
| @ai-sdk/google | 0.0.52 | Very early, breaking changes likely |
| @composio/core | 0.1.35 | Early, API may change |
| @composio/vercel | 0.1.35 | Early |
| browser-use-node | 0.1.11 | Experimental |

**Solution**:
- Pin exact versions in `package.json` (remove `^`)
- Monitor changelogs closely
- Consider vendoring critical code

### Tailwind CSS v4 Compatibility

**Issue**: Using Tailwind CSS 4.x which is still in development.

**Potential Issues**:
- Config format changed from v3
- Plugin compatibility issues

**Solution**: If issues arise, downgrade to Tailwind v3:
```bash
npm install tailwindcss@3 @tailwindcss/typography@0.5 autoprefixer postcss --legacy-peer-deps
```

### lucide-react Version Anomaly

**Issue**: Version `0.525.0` is unusually high for a 0.x release (likely auto-incremented).

**Risk**: Frequent updates may introduce breaking changes.

**Solution**: Pin to specific version and test before updating.

---

## Code Quality Issues

### Security Vulnerabilities

#### 1. XSS Vulnerability (CRITICAL)

**Location**: `/app/components/SuperAgent.tsx:165`

```typescript
dangerouslySetInnerHTML={{ __html: message.slideData[activeSlide].html! }}
```

**Issue**: HTML from AI responses rendered without sanitization.

**Solution**: Sanitize HTML before rendering:
```bash
npm install dompurify @types/dompurify --legacy-peer-deps
```
```typescript
import DOMPurify from 'dompurify';
// Then use:
dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(html) }}
```

#### 2. Hardcoded Integration ID

**Location**: `/app/api/connection/google-sheet/route.ts:33`

```typescript
'ac_FoalPoSZrq4Q' // Hardcoded Composio app ID
```

**Solution**: Move to environment variable:
```typescript
process.env.COMPOSIO_GOOGLE_SHEETS_APP_ID
```

#### 3. Non-HttpOnly Cookies

**Location**: `/app/api/connection/google-sheet/route.ts:56`

Cookies are readable by JavaScript, increasing XSS risk.

**Solution**: Set `httpOnly: true` for security-sensitive cookies.

#### 4. No Rate Limiting

All API routes lack rate limiting, vulnerable to abuse.

**Solution**: Add rate limiting middleware or use Vercel's built-in protection.

#### 5. No Input Validation

API routes don't validate input thoroughly.

**Solution**: Use Zod schemas for all API inputs:
```typescript
const inputSchema = z.object({
  prompt: z.string().min(1).max(10000),
  userId: z.string(),
});
```

### Anti-Patterns

#### 1. Excessive `any` Types

Multiple files use `any` extensively, defeating TypeScript benefits.

**Locations**: API routes, component props

**Solution**: Define proper interfaces for all data structures.

#### 2. Console.log in Production

Debug statements throughout codebase.

**Solution**: Remove or wrap in development checks:
```typescript
if (process.env.NODE_ENV === 'development') console.log(...);
```

#### 3. Duplicated Code

Color schemes duplicated in:
- `/app/api/superagent/route.ts`
- `/app/api/convert-to-ppt/route.ts`

**Solution**: Create shared constants file:
```typescript
// lib/constants.ts
export const colorSchemes = { ... };
```

#### 4. Large Component Files

`SuperAgent.tsx` is 715 lines - too large.

**Solution**: Break into smaller components:
- MessageBubble (separate file)
- WelcomeScreen (separate file)
- SpreadsheetSidebar (separate file)
- DocumentSidebar (separate file)

### Performance Issues

#### 1. No API Response Caching

Every request hits Gemini API.

**Solution**: Implement caching for repeated queries.

#### 2. Heavy Dependencies Loaded Client-Side

Three.js, Framer Motion loaded for all pages.

**Solution**: Dynamic imports for heavy components:
```typescript
const HeavyComponent = dynamic(() => import('./HeavyComponent'), { ssr: false });
```

---

## Testing Analysis

### Current State: NO TESTS

The project has zero test files. This is a significant risk.

### Missing Test Coverage

1. **Unit Tests Needed**:
   - URL validation functions (`validateSheetUrl`, `extractSheetId`)
   - Slide HTML generation
   - Color scheme utilities

2. **Integration Tests Needed**:
   - API route handlers
   - OAuth flow
   - Slide generation pipeline

3. **E2E Tests Needed**:
   - Full user journey from signin to presentation download
   - Google Sheets/Docs connection flow

### Recommended Testing Stack

```bash
npm install -D jest @testing-library/react @testing-library/jest-dom @types/jest ts-jest playwright --legacy-peer-deps
```

---

## Architecture Notes for AI Assistants

### Key Files to Understand

1. **`/app/api/superagent/route.ts`** - Core AI logic, tool registration, prompt handling
2. **`/app/components/SuperAgent.tsx`** - Main UI, state management, user interactions
3. **`/middleware.ts`** - Authentication gate

### Patterns Used

1. **Composio Custom Tools**: Tools registered via `composio.tools.createCustomTool()`
2. **Vercel AI SDK**: `generateText()` and `generateObject()` for AI calls
3. **Dynamic Sidebars**: State-driven UI for Google Docs/Sheets panels

### State Management

- **Local state only** - No global state management (Redux, Zustand)
- State in `SuperAgent.tsx`:
  - `messages` - Chat history
  - `currentSlides` - Generated slides
  - `sheetUrl/docUrl` - Connected documents
  - `isSheetConnected/isDocConnected` - Connection status

### Special Commands

The AI uses `[SLIDES]` magic word to trigger slide generation:
1. AI responds with slide outline
2. Includes `[SLIDES]` at end
3. Frontend detects this and calls `/api/generate-slides`

### Composio Tool Flow

1. Tools registered on each request (not optimal)
2. Multiple toolkits fetched: GOOGLESHEETS, GOOGLEDOCS, COMPOSIO_SEARCH
3. Custom tools: GENERATE_PRESENTATION_SLIDES, PUPPETEER_BROWSER

---

## Prioritized Improvements

### Critical (Security/Breaking)

| Priority | Issue | Solution | Complexity |
|----------|-------|----------|------------|
| 1 | XSS vulnerability | Add DOMPurify sanitization | Low |
| 2 | No input validation | Add Zod schemas to all API routes | Medium |
| 3 | Hardcoded secrets | Move to environment variables | Low |
| 4 | No rate limiting | Add rate limiting middleware | Medium |

### High (Stability/Quality)

| Priority | Issue | Solution | Complexity |
|----------|-------|----------|------------|
| 5 | No tests | Add Jest + RTL + Playwright | High |
| 6 | Remove redundant deps | Uninstall playwright | Low |
| 7 | Pin dependency versions | Remove ^ from package.json | Low |
| 8 | Add .env.example | Document required env vars | Low |
| 9 | Fix TypeScript any types | Define proper interfaces | Medium |

### Medium (Maintainability)

| Priority | Issue | Solution | Complexity |
|----------|-------|----------|------------|
| 10 | Large component files | Split SuperAgent.tsx | Medium |
| 11 | Duplicated code | Create shared constants | Low |
| 12 | Console.log statements | Remove or wrap in dev check | Low |
| 13 | HttpOnly cookies | Update cookie settings | Low |
| 14 | Tool initialization | Cache tool setup, don't re-create per request | Medium |

### Low (Optimization)

| Priority | Issue | Solution | Complexity |
|----------|-------|----------|------------|
| 15 | No API caching | Add response caching | Medium |
| 16 | Heavy bundle size | Dynamic imports for heavy libs | Medium |
| 17 | Global state | Add Zustand for shared state | Medium |
| 18 | Error boundaries | Add React error boundaries | Low |

---

## Quick Reference

### Adding New Tools

```typescript
// In /app/api/superagent/route.ts
const myTool = await composio.tools.createCustomTool({
  slug: 'MY_TOOL',
  name: 'My Tool',
  description: 'What it does',
  inputParams: z.object({
    param1: z.string().describe('Description'),
  }),
  execute: async (input) => {
    // Implementation
    return { data: result, error: null, successful: true };
  }
});
```

### Adding New Slide Styles

Add to `colorSchemes` object in both:
- `/app/api/superagent/route.ts`
- `/app/api/convert-to-ppt/route.ts`

### Debugging

1. Check browser console for client errors
2. Check terminal for API route errors
3. Verify cookies are set in DevTools > Application > Cookies

---

## Known Limitations

1. **Single user sessions** - No multi-user support
2. **No persistence** - Chat history lost on refresh
3. **OAuth coupling** - Must connect BOTH Sheets and Docs
4. **Browser automation** - Puppeteer may not work in serverless environments

---

## Contributing Guidelines

1. Always use `npm install --legacy-peer-deps`
2. Test locally before committing
3. Don't commit console.log statements
4. Add types - avoid `any`
5. Document environment variables
6. Add tests for new features (when test framework is set up)
