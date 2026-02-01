# Resume Fine-Tuner: UI Design Specification

**Version:** 1.0  
**Date:** January 29, 2026  
**Purpose:** User scenario documentation for AI-driven UI generation

---

## Product Overview

Single-page application (SPA) enabling job seekers to upload resumes, paste job descriptions, answer clarifying questions, and receive AI-tailored resumes optimized for ATS systems.

**Core User Journey:** Upload → Analyze → Clarify → Tailor → Download

**Target User:** Mid-senior level professionals (3-10 years experience) actively job hunting, applying to 5-20 positions per week.

---

## Design Principles

1. **Speed:** Every action completes in <3s (excluding AI processing)
2. **Clarity:** Single clear CTA per screen
3. **Progress:** Always show "where am I in the flow"
4. **Trust:** Show AI reasoning, allow editing
5. **Mobile-first:** 70% of users access from mobile

---

## Application Structure

### SPA Sections

```
/
├── /auth
│   ├── /login
│   └── /register
├── /dashboard
├── /session/new
├── /session/:id
│   ├── /upload
│   ├── /job-description
│   ├── /questions
│   ├── /results
│   └── /download
└── /settings
```

---

## User Scenarios

### Scenario 1: First-Time User Registration

**User Story:**
> "As a job seeker, I want to quickly create an account so I can start tailoring my resume without friction."

**User Flow:**
1. **Landing Page**
   - Hero: "Tailor Your Resume to Any Job in 5 Minutes"
   - CTA: "Start Free" button (prominent, above fold)
   - Social proof: "Join 10,000+ job seekers" + testimonial carousel
   - Feature highlights: 3 cards (ATS Optimization, AI Tailoring, Instant Download)

2. **Registration Modal** (overlay, doesn't navigate away)
   - Email input (autofocus)
   - Password input (show/hide toggle, strength indicator)
   - "Continue with Google" button (OAuth)
   - "Already have account? Log in" link
   - On submit → JWT stored, redirect to /dashboard

3. **Dashboard (First Visit)**
   - Empty state illustration
   - Headline: "Let's tailor your first resume"
   - CTA: "Create New Session" button (large, centered)
   - Navigation: Top bar with logo, "Dashboard", "Settings", avatar menu

**UI Components:**
- Modal: Semi-transparent backdrop, centered card, close X icon
- Input fields: Label above, border highlight on focus, inline error messages
- Button states: Default, hover (lift shadow), active (press down), disabled (50% opacity)
- Empty state: SVG illustration + headline + description + CTA

**Success Metrics:**
- Registration completion: >80%
- Time to first session: <60 seconds
- Bounce rate: <30%

---

### Scenario 2: Resume Upload & Analysis

**User Story:**
> "As a user, I want to upload my resume and see what the AI extracted so I can verify accuracy before proceeding."

**User Flow:**
1. **Upload Screen** (`/session/new`)
   - Progress bar: Step 1 of 4 (Upload Resume)
   - Drag-and-drop zone (dashed border, "Drop your resume here" text)
   - OR "Browse Files" button
   - Supported formats: PDF, DOCX (shown below upload zone)
   - Max file size: 10MB (shown below)
   - On file selected → Upload animation (spinner + "Analyzing your resume...")

2. **Analysis Results** (same screen, transitions after upload)
   - Success checkmark animation
   - Extracted data preview (collapsible accordions):
     - **Skills** (tags/chips, e.g., "Python", "React", "Leadership")
     - **Experience** (timeline view: Company → Role → Dates → 2-3 bullet achievements)
     - **Education** (list: Degree, Institution, Year)
   - Inline edit icons (pencil) next to each section
   - CTA: "Looks good, Next" button (bottom right)
   - Secondary action: "Re-upload" link (bottom left)

3. **Edit Mode** (if user clicks pencil icon)
   - Section expands to editable form
   - Skills: Tag input (add/remove chips)
   - Experience: Textareas for each bullet point
   - Save/Cancel buttons
   - Auto-save after 2s of inactivity (toast: "Changes saved")

**UI Components:**
- Drag-and-drop: Dashed border, hover state (solid border, blue tint), active state (green tint)
- Progress bar: 4 segments, current segment filled with primary color, future segments gray
- Accordion: Chevron icon (down = collapsed, up = expanded), smooth expand animation
- Tags/chips: Pill-shaped, light background, remove X icon on hover
- Timeline: Vertical line connecting experience items, circle indicators

**Success Metrics:**
- Upload success rate: >95%
- Users editing extracted data: <20% (indicates parsing accuracy)
- Time on screen: 30-60 seconds (too short = not reviewing, too long = confused)

---

### Scenario 3: Job Description Input

**User Story:**
> "As a user, I want to paste a job description and see what the AI identified as key requirements."

**User Flow:**
1. **JD Input Screen** (`/session/:id/job-description`)
   - Progress bar: Step 2 of 4 (Add Job Description)
   - Large textarea (placeholder: "Paste the full job description here...")
   - Character count: "0 / 10,000 characters" (bottom right of textarea)
   - Tips callout (info icon): "Include the entire posting for best results"
   - CTA: "Analyze Job" button (disabled until text entered)
   - On submit → Processing animation ("Extracting key requirements...")

2. **JD Analysis Results** (same screen, transitions after analysis)
   - Success checkmark animation
   - Extracted requirements (grouped cards):
     - **Required Skills** (tags, e.g., "5+ years React", "AWS", "Team leadership")
     - **Nice-to-Have Skills** (tags, lighter color)
     - **Role Level** (badge: "Senior", "Mid-Level", "Lead")
     - **Company Values** (list: "Innovation", "Collaboration", "Impact")
   - Match preview: "Your resume matches 68% of requirements" (progress bar)
   - CTA: "Continue to Questions" button

**UI Components:**
- Textarea: Monospace font, line numbers, syntax highlighting for key phrases (bold)
- Character count: Gray text, turns red if exceeds limit
- Info callout: Light blue background, left border, info icon
- Card groups: White background, shadow, rounded corners, 16px padding
- Progress bar (match score): Gradient fill (red → yellow → green based on percentage)

**Success Metrics:**
- JD paste rate: >90% (vs manual typing)
- Match score distribution: Avg 65-75% (too high = not useful, too low = discouraging)
- Time on screen: 45-90 seconds

---

### Scenario 4: Q&A Clarification

**User Story:**
> "As a user, I want to answer a few quick questions so the AI can tailor my resume more accurately."

**User Flow:**
1. **Q&A Screen** (`/session/:id/questions`)
   - Progress bar: Step 3 of 4 (Answer Questions)
   - Question counter: "Question 1 of 3" (top)
   - Question card:
     - Question text (large, bold): "What's your Kubernetes experience?"
     - Context explanation (smaller, gray): "The JD emphasizes Kubernetes, but your resume only mentions Docker."
     - Answer options (radio buttons or dropdown):
       - "Never used"
       - "Learning/Side projects"
       - "Production environment"
       - "Led K8s migration"
     - OR text input for open-ended questions
   - Navigation: "Back" button (disabled on Q1), "Next" button
   - On last question → "Submit Answers" button

2. **Submission Animation**
   - Full-screen overlay
   - Animated progress indicator: "Tailoring your resume..." (steps shown):
     1. "Analyzing gaps..." (2s)
     2. "Rewriting experience bullets..." (5s)
     3. "Optimizing keywords..." (3s)
     4. "Generating cover letter..." (4s)
     5. "Calculating match score..." (2s)
   - Can't dismiss (prevents accidental navigation)

**UI Components:**
- Question card: Centered, max-width 600px, shadow, rounded corners
- Radio buttons: Custom styled (circle, checkmark on selected, primary color)
- Context box: Light yellow background, left border, italic text
- Progress steps: Checkmark for completed, spinner for current, gray for pending
- Loading overlay: Dark semi-transparent backdrop, white card with animations

**Success Metrics:**
- Question completion rate: >90%
- Avg time per question: 15-20 seconds
- Users clicking "Back": <10% (indicates clear questions)

---

### Scenario 5: Results Review & Editing

**User Story:**
> "As a user, I want to see my tailored resume, understand what changed, and make quick edits before downloading."

**User Flow:**
1. **Results Screen** (`/session/:id/results`)
   - Progress bar: Step 4 of 4 (Review & Download)
   - Split view (desktop) / tabs (mobile):
     - **Left/Tab 1: Tailored Resume**
       - Preview in resume-style formatting
       - Sections: Summary, Experience (with revised bullets), Skills, Education
       - Changed content highlighted (yellow background, fade after 5s)
       - Inline edit icons
     - **Right/Tab 2: Match Score**
       - Large score: "85%" (animated count-up on load)
       - Donut chart with breakdown:
         - Keyword Match: 90%
         - Experience Level: 85%
         - Required Skills: 80%
         - ATS Compatibility: 95%
       - Strengths list (green checkmarks):
         - "Strong Kubernetes experience highlighted"
         - "Leadership keywords added"
       - Gaps list (yellow warning icons):
         - "Rust experience missing (nice-to-have)"

2. **Edit Resume Section**
   - Click any section heading → Edit mode
   - Bullet point editing: Click bullet → Textarea appears, Save/Cancel buttons
   - "View Changes" button → Modal with diff view (old vs new, red/green highlighting)
   - Auto-save with toast notifications

3. **Cover Letter Tab** (additional tab)
   - Generated cover letter preview (formatted)
   - Edit button → Full-screen editor
   - "Copy to clipboard" button

**UI Components:**
- Split view: 60% resume, 40% score (desktop), stacked tabs (mobile)
- Donut chart: Animated fill, tooltips on hover, color-coded segments
- Diff view: Side-by-side comparison, line-level highlighting, expand/collapse
- Toast notifications: Bottom-right, auto-dismiss after 3s, success/warning/error colors

**Success Metrics:**
- Users editing results: 30-40% (indicates engagement, not dissatisfaction)
- Time on results screen: 2-5 minutes (reviewing)
- Match score satisfaction: >80% of users see score >70%

---

### Scenario 6: Download & Export

**User Story:**
> "As a user, I want to download my tailored resume in multiple formats so I can apply immediately."

**User Flow:**
1. **Download Section** (bottom of results screen)
   - Download buttons (side-by-side):
     - "Download DOCX" (primary button)
     - "Download PDF" (secondary button)
     - "Copy Text" (tertiary button)
   - On click → File downloads immediately (browser download prompt)
   - Success toast: "Resume downloaded! Ready to apply 🚀"

2. **Post-Download CTA**
   - Modal appears after first download:
     - Headline: "Want to tailor for another job?"
     - CTA: "Start New Session" button
     - Secondary: "Back to Dashboard" link
   - Dismissible (X icon)

**UI Components:**
- Button group: Horizontal layout, same height, icons + text
- File download: Browser native behavior (no custom download UI)
- Success modal: Centered, confetti animation (subtle), 400px width

**Success Metrics:**
- Download rate: >95% (primary KPI)
- Format preference: 70% DOCX, 25% PDF, 5% text
- Multi-session users: >40% create 2nd session within 7 days

---

### Scenario 7: Dashboard - Session History

**User Story:**
> "As a returning user, I want to see all my past tailored resumes so I can reuse or update them."

**User Flow:**
1. **Dashboard** (`/dashboard`)
   - Top bar: "Your Sessions" headline + "Create New Session" button (top right)
   - Session cards (grid layout, 3 columns desktop / 1 column mobile):
     - Card content:
       - Job title (extracted from JD)
       - Company name (extracted from JD)
       - Match score badge: "85%" (colored by score)
       - Date: "Tailored 3 days ago"
       - Status: "Downloaded" or "Draft" badge
       - Action buttons: "View", "Download", "Delete" (icon buttons)
   - Empty state (if no sessions): Illustration + "No sessions yet" + "Create New Session" button

2. **View Session** (click "View" button)
   - Navigate to `/session/:id/results` (same as Scenario 5)
   - "Edit & Re-tailor" button (re-runs AI with edits)

3. **Delete Session** (click delete icon)
   - Confirmation modal: "Delete this session? This can't be undone."
   - Buttons: "Cancel" (default) / "Delete" (destructive, red)
   - On delete → Card fades out, toast: "Session deleted"

**UI Components:**
- Card grid: CSS Grid, gap 24px, auto-fill columns (min 300px)
- Badge: Small, rounded, colored background (green >80%, yellow 60-80%, red <60%)
- Icon buttons: Ghost style (no background), hover shows background, tooltips on hover
- Confirmation modal: Centered, 400px width, red accent for destructive action

**Success Metrics:**
- Session reuse rate: >25% (users view past sessions)
- Multi-session users: >60% create 3+ sessions
- Delete rate: <10% (indicates content value)

---

### Scenario 8: Settings & Account

**User Story:**
> "As a user, I want to manage my account, view usage, and upgrade to paid plans."

**User Flow:**
1. **Settings Screen** (`/settings`)
   - Tab navigation: "Profile", "Usage", "Billing", "Security"
   
2. **Profile Tab**
   - Avatar upload (circle crop, drag-and-drop or file browse)
   - Name input
   - Email (read-only, "Change email" link → modal with verification flow)
   - "Save Changes" button

3. **Usage Tab** (Freemium gate)
   - Current plan badge: "Free Plan" or "Pro Plan"
   - Usage meters:
     - "3 / 3 sessions this month" (progress bar)
     - Resets: "in 12 days"
   - CTA: "Upgrade to Pro" button → Pricing modal
   - Recent activity log (list):
     - "Tailored resume for Senior Engineer at Google - 2 days ago"
     - "Tailored resume for Product Manager at Meta - 5 days ago"

4. **Billing Tab** (Pro users only)
   - Current plan: "Pro - $15/month"
   - Payment method: "Visa ending in 4242" (edit button)
   - Next billing date: "February 15, 2026"
   - Invoice history (table): Date, Amount, Status, "Download" link
   - "Cancel Subscription" link (bottom, small, red)

5. **Security Tab**
   - Change password form (current password, new password, confirm)
   - Two-factor authentication toggle (if available)
   - "Delete Account" button (bottom, red, confirmation required)

**UI Components:**
- Tab navigation: Underline active tab, hover state
- Avatar upload: Circle frame, hover overlay with "Change photo" text
- Usage meter: Progress bar with label, color changes when limit reached (red)
- Pricing modal: 2 plans side-by-side (Free vs Pro), feature comparison table
- Table: Sortable columns, hover row highlight, pagination if >10 items

**Success Metrics:**
- Free → Pro conversion: >15% within 30 days
- Upgrade CTA click rate: >40% of free users reaching limit
- Account deletion rate: <2%

---

## Component Library

### Navigation

**Top Bar (Desktop):**
```
┌────────────────────────────────────────────────────┐
│ [Logo]  Dashboard  Sessions        [Avatar Menu ▼] │
└────────────────────────────────────────────────────┘
```

**Mobile Navigation:**
```
Bottom tab bar:
[ Home ] [ Sessions ] [ Settings ] [ Account ]
Icons + labels, active state highlighted
```

### Buttons

**Primary:** Solid background (primary color), white text, shadow on hover, press animation
**Secondary:** Outlined, background on hover
**Tertiary:** Text only, underline on hover
**Destructive:** Red background, used for delete actions

**Sizes:**
- Small: 32px height, 12px padding
- Medium: 40px height, 16px padding
- Large: 48px height, 24px padding

### Forms

**Input fields:**
- Label above (14px, medium weight)
- Border: 1px solid gray, focus state (2px primary color)
- Error state: Red border, error message below in red
- Success state: Green border, checkmark icon inside

**Validation:**
- Real-time for email format, password strength
- On blur for required fields
- Submit button disabled until valid

### Feedback

**Toast notifications:**
- Position: Bottom-right
- Types: Success (green), Error (red), Warning (yellow), Info (blue)
- Icon + message + dismiss X
- Auto-dismiss: 3s (success), 5s (error), manual (warning)

**Loading states:**
- Skeleton screens (pulsing gray blocks) for content loading
- Spinners for actions (<2s expected)
- Progress bars for multi-step processes

**Empty states:**
- Illustration (SVG, brand colors)
- Headline (24px, bold)
- Description (16px, gray)
- Primary action button

### Modals

**Structure:**
- Backdrop: Semi-transparent dark (80% opacity)
- Card: Centered, max-width 600px, white background, shadow
- Header: Close X icon (top-right)
- Body: Scrollable if content exceeds viewport
- Footer: Action buttons (right-aligned)

**Types:**
- Confirmation: Headline + description + Cancel/Confirm buttons
- Form: Headline + form fields + Cancel/Submit buttons
- Info: Headline + content + Close button

---

## Responsive Breakpoints

**Mobile:** < 768px
- Single column layout
- Bottom navigation bar
- Stacked cards
- Full-width inputs
- Hamburger menu for secondary actions

**Tablet:** 768px - 1024px
- 2-column grid where applicable
- Side navigation (collapsible)
- Adapted card sizes

**Desktop:** > 1024px
- Full layout with sidebars
- Multi-column grids
- Hover states enabled
- Keyboard shortcuts visible

---

## Accessibility Requirements

**WCAG AA Compliance:**
- Color contrast: 4.5:1 for text, 3:1 for UI elements
- Focus indicators: 2px outline on all interactive elements
- Screen reader labels: `aria-label` on all icons, `role` attributes
- Keyboard navigation: Tab order logical, Enter/Space activate buttons, Esc closes modals
- Alt text: All images and icons have descriptive alt text

**Additional:**
- Font size: Minimum 14px, increase with zoom
- Touch targets: Minimum 44x44px (mobile)
- Error messages: Specific, actionable (not just "Invalid input")

---

## Design Tokens

**Colors:**
- Primary: #2563EB (Blue)
- Secondary: #10B981 (Green)
- Error: #EF4444 (Red)
- Warning: #F59E0B (Yellow)
- Gray scale: #F9FAFB (bg) → #111827 (text)

**Typography:**
- Font family: Inter (sans-serif)
- Headings: Bold, tight line-height (1.2)
- Body: Regular, comfortable line-height (1.5)
- Sizes: 12px (small) / 14px (body) / 16px (large) / 20px (h3) / 24px (h2) / 32px (h1)

**Spacing:**
- Base unit: 4px
- Scale: 4px, 8px, 12px, 16px, 24px, 32px, 48px, 64px

**Shadows:**
- Small: 0 1px 3px rgba(0,0,0,0.1)
- Medium: 0 4px 6px rgba(0,0,0,0.1)
- Large: 0 10px 15px rgba(0,0,0,0.1)

**Border radius:**
- Small: 4px (inputs, tags)
- Medium: 8px (cards, buttons)
- Large: 16px (modals)
- Full: 9999px (pills, avatars)

---

## Animations

**Micro-interactions:**
- Button press: Scale 0.98 + shadow reduce (150ms)
- Card hover: Lift shadow (200ms ease-out)
- Input focus: Border color transition (150ms)

**Page transitions:**
- Fade + slide up (300ms ease-out)
- Modal appear: Scale 0.95 → 1 + fade in (200ms)

**Loading:**
- Skeleton pulse: Opacity 0.4 → 0.6 (1s loop)
- Spinner rotation: 360deg (800ms linear loop)
- Progress bar fill: Width transition (500ms ease-out)

**Success animations:**
- Checkmark draw (SVG path animation, 400ms)
- Confetti burst (particles fade out, 2s)
- Toast slide-in: Translate Y 100% → 0 (300ms)

---

## Error States & Messages

**Resume upload failures:**
- "File too large (max 10MB)" → Suggest compressing PDF
- "Unsupported format" → List supported: PDF, DOCX
- "Couldn't extract text" → Offer manual paste option

**JD analysis failures:**
- "Job description too short" → Suggest minimum 100 characters
- "Couldn't identify requirements" → Ask to paste full posting, not summary

**AI processing failures:**
- "AI service unavailable" → Show retry button + estimate wait time
- "Processing timed out" → Offer to notify via email when ready

**Payment failures:**
- "Card declined" → Update payment method link
- "Subscription cancelled" → Reactivate button

---

## Performance Targets

**Load times:**
- Initial page load: <2s (LCP)
- Route transitions: <300ms
- API responses: <500ms (excluding AI)
- AI processing: <15s (with progress indicator)

**Asset optimization:**
- Images: WebP format, lazy loading, responsive sizes
- JavaScript: Code splitting, tree shaking, <200KB initial bundle
- CSS: Critical CSS inlined, unused styles purged

---

## Analytics Events

**Track user actions:**
- `user_registered` (source: organic, google, linkedin)
- `session_created`
- `resume_uploaded` (format: pdf, docx)
- `jd_analyzed` (word_count, match_score)
- `questions_completed` (time_taken)
- `resume_tailored` (match_score_improvement)
- `resume_downloaded` (format: docx, pdf, text)
- `user_upgraded` (plan: pro, team)
- `session_revisited`

**Funnel analysis:**
- Registration → Upload → JD → Questions → Tailor → Download
- Identify drop-off points

**A/B test candidates:**
- Hero CTA text ("Start Free" vs "Tailor Resume Now")
- Pricing ($10 vs $15 vs $20)
- Match score threshold (show if >60% vs >70%)
- Q&A question count (3 vs 5 questions)

---

## Future Enhancements (Post-MVP)

**Phase 2:**
- Browser extension (tailor resume directly from job site)
- LinkedIn integration (import profile)
- Resume version comparison
- Team collaboration (share sessions)

**Phase 3:**
- Interview prep (AI-generated questions based on JD)
- Application tracking (track applied jobs, follow-ups)
- Email templates (networking, follow-up)
- Salary negotiation tool

---

**Document Status:** Ready for UI Generation  
**Usage:** Feed this document to LLM for design mockups, Figma generation, or frontend implementation guidance