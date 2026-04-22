# ResumeIQ — Complete Technical Analysis

---

## 1. PROJECT OVERVIEW

**Application Name:** ResumeIQ — AI Resume Analyzer & Cover Letter Generator

**Purpose:** SaaS web application that uses Google Gemini AI to:
- Upload and parse resume PDFs
- Extract structured resume data
- Perform semantic matching between resume and job descriptions
- Generate ATS (Applicant Tracking System) scores
- Provide AI-powered resume improvement suggestions
- Generate personalized cover letters
- Display company match scores and skill gap analysis

**Project Type:** 3rd-Year Computer Science Generative AI Project demonstrating Prompt Engineering, Semantic Analysis, Context-Aware Generation, and LLM-Based Reasoning

---

## 2. TECH STACK

### Backend
- **Runtime:** Node.js v18+
- **Framework:** Express.js 4.18.2
- **API Client:** @google/generative-ai 0.21.0
- **File Parsing:** pdf-parse 1.1.1
- **File Upload:** multer 1.4.5-lts.1 (memory storage, 5MB limit)
- **CORS:** cors 2.8.5
- **Configuration:** dotenv 16.4.5
- **Dev Tool:** nodemon 3.1.0 (auto-restart on changes)

### Frontend
- **HTML:** Semantic HTML5 (single file: index.html)
- **CSS:** Embedded styles only, no external CSS files
  - CSS Variables for design system
  - CSS Grid and Flexbox for layouts
  - CSS Animations and transitions
  - Mobile-responsive (320px - 4K+)
- **JavaScript:** Vanilla ES6+ (no framework, no bundler)
  - Fetch API for HTTP requests
  - DOM manipulation

### External APIs
- **Google Gemini API** (primary AI engine)
  - Model: gemini-1.5-flash (fallback: gemini-2.5-flash, gemini-3.1-flash-lite-preview)
  - Temperature: 0.1 (deterministic)
  - Top P: 0.95
  - Max Output Tokens: 8192
  - Response Format: JSON

---

## 3. ARCHITECTURE

### Directory Structure
```
project-root/
├── server.js                    # Express server entry point
├── package.json                 # Dependencies and scripts
├── .env                        # Environment variables (GEMINI_API_KEY, PORT, etc.)
├── index.html                  # Complete frontend (HTML + CSS + JS)
├── resumeRoutes.js             # API route definitions
├── resumeController.js         # Request handling & validation
├── geminiService.js            # ALL Gemini AI logic & prompts
├── pdfParser.js                # PDF extraction utility
└── README.md                   # Documentation
```

### Frontend-Backend Connection
- **Frontend:** Single-page HTML file served statically by Express
- **Communication:** Fetch API + JSON over HTTP
- **CORS:** Configured to allow development (localhost) and configurable origins
- **Base URL Resolution:** Client tries multiple API endpoints: API origin, localhost:5001, localhost:5000
- **Error Handling:** Global error handler returns JSON errors

### Request Flow

```
User uploads resume
  ↓
POST /api/upload-resume (with PDF or text)
  ↓
pdfParser.extractTextFromPDF() (if PDF) / direct text (if pasted)
  ↓
resumeController.uploadResume() validates & returns resumeText
  ↓
User provides job description
  ↓
POST /api/analyze
  ↓
geminiService.runFullAnalysis() (orchestrator)
  ├→ extractResumeData()
  ├→ semanticMatch()
  ├→ generateATSScore()
  ├→ improveResume()
  └→ generateSuggestions()
  ↓
Frontend renders results with company matches
  ↓
User can generate cover letter via POST /api/generate-cover-letter
```

---

## 4. FEATURES IMPLEMENTED

### Feature 1: Resume Upload & Parsing
- **PDF Upload:** Via multer (memory storage, 5MB max)
- **Text Paste:** Direct paste into textarea
- **Validation:** PDF must contain text (image PDFs rejected)
- **Extraction:** pdf-parse extracts text, normalizes newlines
- **Metadata Returned:**
  - File name
  - File size (KB)
  - Page count
  - Word count
  - Character count

### Feature 2: Resume Analysis Pipeline
**5-Stage AI Analysis (Sequential):**
1. **Resume Data Extraction** - Structured parsing of resume into components
   - Personal info (name, email, phone, location, links)
   - Summary
   - Experience (company, title, duration, achievements, impact)
   - Education (institution, degree, field, year, GPA)
   - Skills (technical, soft, languages, frameworks, tools)
   - Projects
   - Certifications
   - Seniority level, experience years, domain, content quality

2. **Semantic Matching** - Resume vs Job Description comparison
   - Matching skills with confidence levels
   - Missing skills with importance (Critical/Important/Nice-to-have)
   - Partial matches (skill gap analysis)
   - Alignment reasoning (4-5 sentences)
   - Culture fit indicators
   - Red flags
   - Overall match percentage

3. **ATS Score Generation** - AI-driven scoring system
   - Formula: 5 weighted components
     - Keyword Coverage (25 pts)
     - Semantic Relevance (25 pts)
     - Resume Structure & ATS Parsability (20 pts)
     - Experience Alignment (15 pts)
     - Content Quality (15 pts)
   - Grade (A+, A, B+, B, C+, C, D, F)
   - Verdict (Strong Match / Good Match / Average / Below Average / Poor Match)
   - Component breakdown with reasoning
   - ATS pass probability
   - Top 3 improvement actions

4. **Resume Improvements** - AI-suggested enhancements
   - Professional summary transformation (BEFORE → AFTER)
   - Bullet point improvements with STAR method
   - Skills section reorganization
   - Suggested new sections (if needed)

5. **AI Suggestions** - Actionable advice with reasoning
   - 8-12 specific suggestions
   - Categories: Skills Gap / Content Quality / Formatting / Missing Metrics / Weak Phrasing / Keyword Optimization
   - Priority levels: Critical / High / Medium / Low
   - Reasoning: WHY this matters
   - How-to-fix: Step-by-step instructions
   - Estimated ATS point gain (0-100)
   - Quick wins (3 items, <10 min)
   - Long-term goals (2-3 skills/experiences)

### Feature 3: AI Cover Letter Generation
- Input: Resume data, job description, semantic match data
- Output:
  - Email subject line
  - Salutation
  - 3-4 paragraph body
  - Professional closing
  - Candidate signature
- Writing strategy: Explanations of tone/hooks/structure
- Key points highlighted: 3-5 resume points woven into letter
- Personalization score: High/Medium/Low

### Feature 4: Company Match Scores (NEW - ADDED)
**6 Companies with Skill Requirements:**
1. Google (Tech Giant) - Python, Java, C++, System Design, Algorithms, Distributed Systems, Cloud Computing
2. Amazon (E-Commerce Leader) - Java, Python, AWS, Microservices, SQL, NoSQL, System Design, Leadership
3. Microsoft (Software Giant) - C#, .NET, Azure, JavaScript, TypeScript, React, SQL, Cloud Architecture
4. Meta (Social Tech) - JavaScript, React, Python, GraphQL, CSS, System Design, Analytics
5. Apple (Hardware+Software) - Swift, Objective-C, iOS Development, Metal, CoreData, Security, Performance
6. Flipkart (India E-Commerce) - Java, Python, Node.js, SQL, NoSQL, Microservices, Leadership

**Display:**
- Grid of 6 company cards
- Company logo/emoji with brand gradient
- Match percentage (0-100%)
- Animated progress bar
- Clickable for detail modal

**Match Calculation:**
```
Score = (Required Skills Matched / Total Required Skills) × 80%
      + (Optional Skills Matched / Total Optional Skills) × 20%
```

**Modal Detail View:**
- Your Skills (green, with ✓ prefix)
- Missing Skills (red, with ⊘ prefix)
- Optional Skills (yellow, with ⊙ prefix)
- Learning path (top 4 missing skills with suggestions)
- Match summary (percentage + core skill ratio)

**Skill Matching Logic:**
- Fuzzy text matching (substring matching, normalized variants)
- Handles: C++ → "cpp", C# → "csharp", .NET → "dotnet"
- Combines `matchData.matchingSkills` + `resumeData.skills.technical`

---

## 5. API DETAILS

### Endpoints

#### 1. GET /api/health
**Purpose:** Check server & API key status
**Response:**
```json
{
  "status": "online",
  "message": "AI Resume Analyzer API is running",
  "version": "1.0.0",
  "gemini": "configured" | "⚠️ API key missing",
  "timestamp": "ISO datetime"
}
```

#### 2. POST /api/upload-resume
**Purpose:** Upload PDF or paste resume text
**Accepts:** 
- multipart/form-data with file field "resume" (PDF, max 5MB)
- application/json with "resumeText" field (min 100 chars after trim)

**Response:**
```json
{
  "success": true,
  "message": "Resume parsed successfully",
  "data": {
    "resumeText": "full extracted text",
    "metadata": {
      "fileName": "resume.pdf",
      "fileSize": "4.2 KB",
      "pages": 1,          // if PDF
      "wordCount": 250
    }
  }
}
```

**Validation:**
- PDF: Must contain text (min 50 chars), validates as real resume (keywords)
- Text: Min 200 chars, must contain resume indicators (experience, education, skills, etc.)

#### 3. POST /api/analyze
**Purpose:** Run full AI analysis pipeline
**Body:**
```json
{
  "resumeText": "full resume text",
  "jobDescription": "full job description text"
}
```

**Validation:**
- resumeText: min 100 chars after trim
- jobDescription: min 50 chars after trim

**Response:**
```json
{
  "success": true,
  "message": "Analysis complete",
  "data": {
    "resumeData": { /* 5. Core Logic section */ },
    "matchData": { /* semantic match output */ },
    "atsScore": { /* ATS scoring output */ },
    "improvements": { /* resume improvements */ },
    "suggestions": { /* actionable suggestions */ }
  },
  "meta": {
    "processingTime": "15.23s",
    "model": "gemini-1.5-flash",
    "timestamp": "ISO datetime"
  }
}
```

#### 4. POST /api/generate-cover-letter
**Purpose:** Generate personalized cover letter
**Body:**
```json
{
  "resumeText": "full resume text",
  "jobDescription": "full job description",
  "resumeData": { /* optional: use cache */ },
  "matchData": { /* optional: use cache */ }
}
```

**Response:**
```json
{
  "success": true,
  "message": "Cover letter generated successfully",
  "data": {
    "coverLetter": {
      "subject": "Email subject",
      "salutation": "Dear Hiring Manager",
      "body": "Letter body with \\n\\n paragraph breaks",
      "closing": "Best regards",
      "signature": "Candidate Name"
    },
    "writingStrategy": "Explanation of writing choices",
    "keyPointsHighlighted": ["Point 1", "Point 2", "Point 3"],
    "personalizationScore": "High"
  },
  "meta": { /* same as /analyze */ }
}
```

### Error Responses
- **400:** Validation errors (file too large, text too short, invalid content)
- **500:** API/processing errors (Gemini API failure, malformed JSON from AI)

---

## 6. CORE LOGIC

### Resume Analysis Pipeline (geminiService.js)

**Orchestrator:** `runFullAnalysis(resumeText, jobDescription)`
- Runs 5 stages sequentially
- 2-second sleep between stages (to avoid rate limiting on Gemini Free Tier)
- Passes context between stages to avoid recomputation
- Catches and re-throws errors for frontend handling

### Stage 1: Resume Data Extraction
**Function:** `extractResumeData(resumeText)`

**Prompt Engineering:**
- Role: "Expert HR analyst with 15+ years in tech talent acquisition"
- Task: DEEP SEMANTIC ANALYSIS of resume (not just keyword extraction)
- Output Schema: JSON with explicit structure (personal, summary, experience, education, skills, projects, certifications, derived metrics)
- Chain-of-Thought: "Infer things not explicitly stated" (e.g., soft skills from responsibilities)

**Logic:**
1. Parse resume text into structured components
2. Infer soft skills from responsibilities
3. Calculate totalExperienceYears by summing durations
4. Assess seniorityLevel (Entry/Mid/Senior/Lead/Executive)
5. Determine industryDomain (Web Dev, Data Science, DevOps, ML/AI, etc.)
6. Score contentQualityScore (0-100) based on clarity, quantification, completeness

### Stage 2: Semantic Matching
**Function:** `semanticMatch(resumeText, jobDescription)`

**Semantic Matching Rules:**
- "React.js" = "React" = "UI development" = "component-based frontend"
- "Team leadership" = "managed a team of 5 engineers"
- Implied skills: 3 years Node.js → npm, REST APIs, async programming

**Output:**
- matchingSkills: Array with confidence levels (High/Medium/Low), resume evidence, job requirement mapping
- missingSkills: Array with importance (Critical/Important/Nice-to-have), learnability score (1-10), recommendation
- partialMatches: Skills where user has partial experience, with gap analysis
- alignmentReasoning: 3-4 sentence paragraph about overall fit
- redFlags: Recruiter concerns (if any)
- overallMatchPercentage: 0-100 (60% skills + 25% experience + 15% domain alignment)

### Stage 3: ATS Score Generation
**Function:** `generateATSScore(resumeText, jobDescription, matchData)`

**Scoring Formula:**
```
Total = KeywordCoverage(25) + SemanticRelevance(25) + Structure(20)
        + ExperienceAlignment(15) + ContentQuality(15)
```

**Components:**
1. **KeywordCoverage (25 pts):** Semantic keyword overlap from JD
2. **SemanticRelevance (25 pts):** How contextually relevant is experience for this role?
3. **ResumStructure (20 pts):** Well-formatted for ATS? Proper sections? Correct length?
4. **ExperienceAlignment (15 pts):** Does background match role level & requirements?
5. **ContentQuality (15 pts):** Quantified achievements? Strong language? Impact-driven?

**Grade Mapping:**
- 90-100: A+
- 80-89: A
- 70-79: B+
- 60-69: B
- 50-59: C+
- 40-49: C
- 30-39: D
- <30: F

**Verdicts:**
- 70+: Strong Match
- 50-69: Good Match / Average
- 30-49: Below Average
- <30: Poor Match

**ATS Pass Probability:**
- High (>70%), Medium (40-70%), Low (<40%)

### Stage 4: Resume Improvements
**Function:** `improveResume(resumeData, jobDescription)`

**Transformation Strategy:**
- BEFORE → AFTER format for each component
- Apply STAR method: Situation, Task, Action, Result
- Add quantification: "Improved" → "Improved by 40%"
- Use strong action verbs: Built, Architected, Engineered, Spearheaded, Optimized
- Show impact: What changed because of this person?
- Align language with JD keywords (without being dishonest)

**Output:**
- improvedSummary: Professional summary transformation
- improvedBulletPoints: 5-10 STAR-formatted bullets with improvements
- improvedSkillsSection: Reorganized, prioritized skills aligned with JD
- suggestedNewSections: Sections to add (e.g., Key Achievements, Technical Projects)

### Stage 5: AI Suggestions
**Function:** `generateSuggestions(resumeData, matchData, atsScore)`

**Suggestion Strategy:**
- 8-12 specific, prioritized suggestions (NOT generic advice)
- Data-driven: Based on gap analysis + ATS scoring
- Each suggestion includes:
  - Title (short action)
  - Category (Skills Gap / Content Quality / Formatting / etc.)
  - Priority (Critical / High / Medium / Low)
  - Current Issue (what's wrong)
  - Reasoning (WHY it matters)
  - How-to-Fix (step-by-step)
  - Estimated ATS point gain
  - Optional: Before/After example

**Additional Outputs:**
- quickWins: 3 items fixable in <10 min for max impact
- longTermGoals: 2-3 skills to develop over 3-6 months
- overallPriorityAdvice: Strategic guidance on what to focus first

### AI JSON Processing

**Challenges Handled:**
1. Markdown fence extraction: Remove ```json and ``` markers
2. Aggressive cleaning: Strip extra whitespace, normalize quotes
3. Nested structure extraction: If AI adds extra text, find first { and last }
4. Error recovery: Fallback parsing with detailed error messages
5. Validation: Check for empty responses, malformed JSON

**Code Path:**
```javascript
callGemini(prompt, expectJson=true)
  → Get response text
  → If not JSON, return as-is
  → Remove markdown fences
  → Try direct JSON.parse()
  → If fails, extract {...} portion
  → Try parse extracted JSON
  → If fails, throw detailed error
```

### Rate Limiting Mitigation
- 2000ms sleep between Gemini API calls (5 stages × 2s = ~10s minimum)
- Prevents hitting Free Tier rate limits (15 requests/min)
- Currently handles ~6-8 full analyses per minute

### Retry Logic
**NOT implemented** in current version. Failures throw immediately.

---

## 7. ERROR HANDLING

### File Validation (pdfParser.js)
```javascript
extractTextFromPDF()
  → Parse PDF
  → Check if text >= 50 chars
  → Normalize newlines
  → Throw if empty or image-based

validateResumeContent()
  → Check for resume indicators (15+ keywords)
  → Require at least 2 indicators present
  → Require text length >= 200 chars
  → Throw descriptive error if fails
```

### Request Validation (resumeController.js)
```javascript
uploadResume()
  → Check if file OR text provided
  → Validate text length (min 100 for final, 200 for validation)
  → Call extractTextFromPDF() if PDF
  → Call validateResumeContent()
  → Return structured response

analyzeResume()
  → Check resumeText >= 100 trimmed chars
  → Check jobDescription >= 50 trimmed chars
  → Catch geminiService errors
  → Return 500 with error message
```

### API-Level Errors (server.js)
```javascript
CORS Error Handler
  → Check origin against allowedOrigins
  → Allow localhost: / dev
  → Reject invalid origins with 403

Multer Error Handler
  → Catch MulterError for file size
  → Return 413 if file > 5MB
  → Return 400 for type mismatches

Global Error Handler
  → Catch unhandled errors
  → Log error with stack
  → Return 500 with details (dev mode only)
  → Return 500 with generic message (production)
```

### Gemini API Errors
```javascript
callGemini()
  → Check GEMINI_API_KEY set
  → Resolve compatible model from /models endpoint
  → Call generateContent()
  → Parse JSON response
  → If parse fails, try fault-tolerant extraction
  → Throw detailed error with debug info
```

**Error Messages:**
- "No resume provided. Please upload a PDF or paste resume text."
- "File too large. Maximum file size is 5MB."
- "Only PDF files are accepted."
- "PDF appears to be empty or image-based."
- "Gemini API key is invalid or missing."
- "Analysis returned empty data. Please try again."
- "No valid JSON structure found in Gemini response"

### Missing Implementation
- **NO retry logic** for failed Gemini calls
- **NO exponential backoff** for rate limiting
- **NO caching** of analysis results
- **NO database** (stateless, all in-memory)

---

## 8. UI/UX DETAILS

### Design System (CSS Variables)
```css
Colors:
  --accent: #d6ff4b (primary yellow-green)
  --accent-2: #00d1b2 (teal)
  --accent-3: #6ee7a8 (green)
  --accent-warn: #ffb84d (orange)
  --accent-danger: #ff5c7a (red)
  --text-primary: #f7f2e7
  --text-secondary: #b8ad9a
  --text-muted: #746a5d
  --bg-base: #0b0a09 (dark theme)
  --bg-surface: #151311
  --bg-card: #1d1a16

Typography:
  --font-display: 'Space Grotesk' (headings)
  --font-body: 'Manrope' (body text)

Spacing:
  --radius-sm: 6px
  --radius-md: 8px
  --radius-lg: 8px
  --radius-xl: 8px

Transitions:
  --transition: 0.3s cubic-bezier(0.4, 0, 0.2, 1)

Shadows:
  --shadow-card: 0 24px 70px rgba(0,0,0,0.42)
  --shadow-glow: 0 0 0 1px rgba(214,255,75,0.05)
```

### Layout Components
- **Navigation:** Fixed top bar with logo, status indicator
- **Hero:** 132px top padding, centered title, feature pills
- **Main Card:** Tab-based UI for 4-step workflow
- **Panels:** Fade-in animations between steps
- **Step Tabs:** Numbered indicators, state tracking (active/completed/disabled)
- **Grid Layouts:** CSS Grid with `repeat(auto-fit, minmax())` for responsiveness

### Step-Based Workflow
1. **Upload Resume** - Upload PDF / Paste text
2. **Job Description** - Input target job description
3. **AI Analysis** - Results dashboard (loading → results)
4. **Cover Letter** - Generate and display

### Results Dashboard Components
- **ATS Card:** Circular score ring (SVG), component breakdown, semantic match bar
- **Skills Cards:** Matching skills (green), missing skills (red), alignment reasoning
- **AI Suggestions:** Categorized, prioritized suggestions with before/after examples
- **Resume Improvements:** Tab-based view (Summary / Bullets / Skills)
- **Cover Letter:** Generated text, subject line, writing strategy

### Company Matches Section (NEW)
- **Grid:** `repeat(auto-fit, minmax(160px, 1fr))`
- **Cards:** 18px padding, hover translate(-3px), gradient background
- **Progress Bars:** Animated width, 1.2s easing
- **Modal:** Backdrop blur 4px, scale-and-fade animation (0.4s), z-index 9997
- **Responsive Breakpoints:**
  - Desktop (1024+): 6 columns
  - Tablet (680px): 2 columns
  - Mobile (400px): 1 column

### Animations
- **Card Hover:** translateY(-3px), border glow, background shift
- **Progress Bars:** Ease-in-out over 1.2-1.5 seconds with initial delay
- **Modal Entry:** Scale(0.95) + fade-in over 0.4s
- **Loading Orb:** Pulse Y-axis infinite, box-shadow breathing
- **Loading Steps:** Pulse opacity 0.6-1.0 when active
- **Tabs:** Color fade on active/completed

### Responsive Design
- **Mobile First:** Base mobile, then @media queries
- **Breakpoints:**
  - 768px: Panel padding reduced, hero padding adjusted
  - 680px: Grid 2 columns (company matches, skills cards)
  - 400px: Grid 1 column (mobile)
- **Scrollbar:** Custom webkit scrollbar with accent color
- **Viewport:** `<meta name="viewport" content="width=device-width, initial-scale=1.0">`

### Accessibility Features
- **Semantic HTML:** Proper heading hierarchy, nav, main, footer
- **Screen Reader Text:** .sr-only class for hidden labels
- **Color Contrast:** Text meets WCAG AA standards
- **Keyboard Navigation:** Tabs, buttons, forms all keyboard-accessible
- **ARIA:** Not explicitly used (semantic HTML sufficient)

### Frontend State Management
```javascript
state = {
  resumeText: '',              // Current resume content
  jobDescription: '',          // Current job description
  uploadedFile: null,          // Uploaded PDF file object
  analysisData: null,          // Full analysis response
  coverLetterData: null,       // Generated cover letter
  currentStep: 1               // Active step (1-4)
}
```

### Event Handling
- **File Upload:** Drag-drop + click, file validation
- **Text Input:** Textarea paste, trim before validation
- **Step Navigation:** Tab click, button navigation (`goToStep()`)
- **Company Cards:** Click → `openCompanyModal(companyName)`
- **Modal:** Esc key, click overlay, button close
- **Copy to Clipboard:** Button click → `copyText()` / `copyElement()`
- **Downloads:** Button click → generate file blob, download via link

---

## 9. LIMITATIONS

### Architecture Limitations
- **No Database:** All data is in-memory, lost on server restart
- **No User Accounts:** No authentication, no saved analyses
- **No File Storage:** Resumes processed but not saved to disk
- **Single Frontend File:** 2000+ lines in index.html (not modular)
- **No Build Step:** Plain HTML/CSS/JS, no minification or bundling

### Performance Limitations
- **Sequential API Calls:** 5 stages run sequentially (10+ seconds minimum)
- **No Caching:** Every analysis re-runs full pipeline
- **Rate Limited:** Free Tier Gemini: 15 req/min (2s sleep between stages)
- **Large Responses:** Max 8192 output tokens per call
- **JSON Parsing:** Fragile fallback extraction if AI response malformed

### Feature Limitations
- **Skill Matching:** Keyword-based only, no semantic AI matching for companies
- **6 Companies Hardcoded:** Would need code change to add more
- **No AI Cover Letter Cache:** Regenerated every time user reopens
- **No Resume Improvement Tracking:** Users can't save/compare versions
- **No Learning Resources:** Suggestions mention skills but no links to courses
- **No Video Resume:** Text-only, no video analysis
- **No Portfolio Analysis:** Only resume and job description

### API Limitations
- **No API Authentication:** No tokens, anyone can call endpoints
- **No Rate Limiting:** No per-IP or per-user limits enforced
- **No Request Logging:** Console logs only, no persistent logs
- **No Webhooks:** All synchronous, no async processing
- **No Pagination:** No list endpoints (only single responses)
- **CORS Hardcoded:** Must edit code to allow new domains

### Frontend Limitations
- **No Error Retry UI:** Failed requests show error but no manual retry button
- **No Progress Indicator:** Dark loading overlay doesn't show actual progress
- **No Analytics:** No tracking of user interactions or success rates
- **No Offline Support:** No service worker or offline functionality
- **No PWA Features:** Not installable, no manifest.json
- **No i18n:** English only, no internationalization

### Data Quality Limitations
- **Resume Parsing:** OCR not supported (image PDFs fail)
- **Context Window:** Each Gemini call independent (no multi-turn conversation)
- **Training Data Cutoff:** Gemini knows tech stack only up to training date
- **Salary Data:** Not calculated or provided
- **Job Market Trends:** No trending skills or hot technologies

### Security Limitations
- **No Input Sanitization:** Raw text passed to Gemini (potential prompt injection)
- **No Rate Limiting:** Could be abused with bulk requests
- **API Key Visible:** .env file must not be committed (but could be leaked)
- **CORS Not Strict:** Allows wildcard or configurable origins
- **No HTTPS Enforcement:** Works over HTTP (bad for production)

### Model Limitations
- **No Fine-Tuning:** Using base Gemini models, not specialized
- **Temperature 0.1:** Very deterministic (less creative, but consistent)
- **Token Limit:** 8192 max output (can truncate long analyses)
- **No Vision:** Can't analyze resume images or diagrams

---

## 10. HARDCODED VALUES & CONFIGURATION

### Hardcoded Limits
- **PDF File Size:** 5MB max
- **Request Timeout:** Not set (infinite)
- **Resume Min Length:** 100 chars (analysis), 200 chars (validation)
- **Job Description Min:** 50 chars
- **Max Output Tokens:** 8192 (Gemini)
- **Temperature:** 0.1
- **Top P:** 0.95
- **Sleep Between Calls:** 2000ms (rate limit mitigation)
- **Modal Max-Width:** 420px

### Hardcoded Models
- **Primary:** process.env.GEMINI_MODEL (if set)
- **Fallbacks:** 
  1. gemini-3.1-flash-lite-preview
  2. gemini-2.5-flash
  3. gemini-1.5-flash

### Hardcoded Companies (Company Match Feature)
```javascript
COMPANY_SKILL_REQUIREMENTS = {
  'Google': { required: [...], optional: [...] },
  'Amazon': { required: [...], optional: [...] },
  'Microsoft': { required: [...], optional: [...] },
  'Meta': { required: [...], optional: [...] },
  'Apple': { required: [...], optional: [...] },
  'Flipkart': { required: [...], optional: [...] }
}
```

### Environment Variables
- **GEMINI_API_KEY** - Required, no default
- **PORT** - Default: 5000
- **NODE_ENV** - Default: development
- **FRONTEND_URL** - Comma-separated allowed origins

---

## SUMMARY TABLE

| Aspect | Details |
|--------|---------|
| **Backend** | Express.js, Node.js |
| **Frontend** | Single HTML file, vanilla JS |
| **Database** | None (stateless) |
| **AI Engine** | Google Gemini API |
| **File Upload** | PDF parsing via pdf-parse |
| **Styling** | Embedded CSS, dark theme, responsive |
| **Responsiveness** | Mobile-first, 320px to 4K+ |
| **Features** | Resume analysis, ATS score, suggestions, cover letter, company matches |
| **Performance** | 10-20s per analysis (5 sequential Gemini calls) |
| **Scaling** | Rate limited (Free Tier), no caching |
| **Security** | Basic (no auth, no rate limiting, potential prompt injection risks) |
| **Deployment** | Single Node.js server, static frontend |
| **Testing** | No test suite |
| **Monitoring** | Console logs only |

