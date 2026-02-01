# Resume Fine-Tuner: Development Plan (Side Project Edition)

**Version:** 3.0  
**Date:** January 31, 2026  
**Team Capacity:** 10 hours/week (2.5 devs × 4 hours each)  
**Timeline:** 10 months to revenue validation

---

## Executive Summary

**Mission:** Build AI-powered resume tailoring SaaS that helps job seekers pass ATS screening and get recruiter callbacks.

**Core Constraint:** 10 hours/week → Ruthless scope prioritization, validate before building.

**Success Definition:** $1K MRR within 10 months (proving model works before scaling).

---

## Technology Stack Decision: Python

### Why Python (FastAPI) Over Go

**Decision: Python for MVP → Consider Go migration post-PMF**

#### Quantitative Justification

| Factor | Python (FastAPI) | Go (Gin) | Winner |
|--------|------------------|----------|--------|
| **Dev Speed** | 2-3x faster (dynamic typing, REPL) | Slower (compile, type annotations) | Python |
| **Time to MVP** | 6 months | 9-12 months (estimate) | Python |
| **AI Ecosystem** | Mature (OpenAI SDK, LangChain, prompt tools) | Immature (basic wrappers only) | Python |
| **Document Libraries** | Battle-tested (PyMuPDF, python-docx) | Limited (go-docx experimental) | Python |
| **Team Skill Overlap** | All 3 devs proficient | 1 backend dev proficient | Python |
| **Deployment Cost** | $20/month (Railway/Render) | $15/month (lower memory) | Tie |
| **Performance** | 4K req/sec (FastAPI async) | 40K req/sec (Go goroutines) | Go* |
| **Memory Usage** | 200-300MB per instance | 50-100MB per instance | Go* |

**Performance Note:** Go wins raw speed, BUT:
- Bottleneck is LLM API (5-15 seconds), not Python
- At 500 users: Python handles load trivially
- At 10K+ users: Migrate critical paths to Go (gradual)

#### Qualitative Justification

**Python Advantages:**
1. **Rapid Iteration:** REPL testing of LLM prompts (critical for quality)
2. **Error Recovery:** 10 hrs/week → Can't afford Go debugging time
3. **Library Richness:** PDF parsing, DOCX generation, async file handling production-ready
4. **Hiring/Onboarding:** Easier to bring in contractors if needed
5. **Prototyping Speed:** Can test ideas in Jupyter notebooks before implementation

**When to Migrate to Go:**
- After $10K MRR (proven model)
- When self-hosting LLMs (Go's memory efficiency matters)
- When processing >1K concurrent sessions (unlikely in year 1)

**Migration Path:**
- Keep Python for LLM orchestration (rich ecosystem)
- Rewrite document processing in Go (CPU-bound)
- Use Go for API gateway (handle 10x traffic)

---

## Team Composition & Allocation

**Total Capacity:** 10 hours/week = 43 hours/month

| Developer | Skills | Hours/Week | Allocation |
|-----------|--------|------------|------------|
| **Dev A** | Frontend (React, TypeScript) | 4 hrs | 40% FE, 30% Integration, 30% Testing |
| **Dev B** | Full-stack (Python, React) | 4 hrs | 50% Backend, 30% FE, 20% DevOps |
| **Dev C** | Backend (Python, APIs, DB) | 2 hrs | 80% Backend, 20% Architecture |

**Work Distribution Pattern:**
- **Weeks 1-6:** Dev B+C focus backend, Dev A spikes UI prototypes
- **Weeks 7-14:** Dev A leads frontend, Dev B integrates, Dev C backend APIs
- **Weeks 15+:** All hands on product polish, testing, optimization

---

## Revised Milestone Timeline (10 Months)

```
M1: Working Prototype (Weeks 1-6)    ████████░░░░░░░░░░░░░░░░░░░░░░░░ 60 hrs
M2: Beta Web App (Weeks 7-14)       ░░░░░░░░████████░░░░░░░░░░░░░░░░ 80 hrs  
M3: Public Launch (Weeks 15-26)     ░░░░░░░░░░░░░░░░████████████░░░░ 120 hrs
M4: Revenue Validation (Weeks 27-40) ░░░░░░░░░░░░░░░░░░░░░░░░░░██████ 140 hrs
                                     |---6 months---|--4 months--|
```

**Total:** 40 weeks (10 months), 400 development hours

---

## Milestone 1: Working Prototype (Weeks 1-6)

**Capacity:** 10 hrs/week × 6 weeks = **60 hours**

### Objective

Build Python CLI tool that proves core value: Upload resume + JD → Get tailored resume that passes ATS.

**Success Criteria:**
- ✅ One team member uses tool to tailor real resume
- ✅ Tailored resume scores 80+ on Jobscan (ATS test)
- ✅ Apply to 5 jobs, get 1+ recruiter callback
- ✅ Core AI pipeline working (no UI needed)

### Week-by-Week Breakdown

#### **Week 1: Foundation (10 hours)**

**Sprint Goal:** Project setup + document parsing working

**Tasks:**
- [ ] **Dev C (5 hrs):** Repository setup, Python environment (Poetry/UV), FastAPI skeleton
- [ ] **Dev B (3 hrs):** PDF text extraction (PyMuPDF library), handle 10+ resume formats
- [ ] **Dev B (2 hrs):** DOCX text extraction (python-docx), test with sample resumes

**Deliverable:** `python parser.py resume.pdf` → Prints raw text

**Story for Week 1:**
> AS a developer, I WANT to extract text from PDF/DOCX resumes SO THAT I can feed it to LLM for parsing.

---

#### **Week 2: Resume Parsing (10 hours)**

**Sprint Goal:** LLM extracts structured data from resume

**Tasks:**
- [ ] **Dev C (4 hrs):** OpenAI API client wrapper, environment config for API keys
- [ ] **Dev B (4 hrs):** Resume parsing prompt (extract skills, experience, education as JSON)
- [ ] **Dev B (2 hrs):** Test with 5 real resumes, validate JSON output quality

**Deliverable:** `python parse_resume.py resume.pdf` → Returns JSON (skills, experience, education)

**Story for Week 2:**
> AS a system, I WANT to parse resume into structured JSON SO THAT I can analyze it against job requirements.

---

#### **Week 3: JD Parsing + Gap Analysis (10 hours)**

**Sprint Goal:** Extract JD requirements, compare with resume

**Tasks:**
- [ ] **Dev C (3 hrs):** JD parsing prompt (required skills, nice-to-haves, role level)
- [ ] **Dev B (4 hrs):** Gap analyzer (keyword overlap, missing skills, experience match)
- [ ] **Dev A (3 hrs):** Manual testing with 10 real JDs from LinkedIn/Indeed

**Deliverable:** `python analyze.py resume.json jd.txt` → Prints gap analysis (80% match, missing: Kubernetes, Python)

**Story for Week 3:**
> AS a job seeker, I WANT to see how my resume matches the JD SO THAT I know what to emphasize.

---

#### **Week 4: Resume Tailoring (10 hours)**

**Sprint Goal:** LLM rewrites resume to match JD

**Tasks:**
- [ ] **Dev C (5 hrs):** Tailoring prompt (rewrite bullets, inject keywords naturally, maintain truthfulness)
- [ ] **Dev B (3 hrs):** Integrate gap analysis into tailoring context
- [ ] **Dev B (2 hrs):** Test tailoring quality (compare before/after Jobscan scores)

**Deliverable:** `python tailor.py resume.json jd.json` → Returns tailored resume JSON

**Story for Week 4:**
> AS a job seeker, I WANT my resume tailored to the JD SO THAT I pass ATS screening.

---

#### **Week 5: DOCX Generation (10 hours)**

**Sprint Goal:** Output professional DOCX resume

**Tasks:**
- [ ] **Dev B (5 hrs):** DOCX generator (python-docx), ATS-friendly template (no tables, simple fonts)
- [ ] **Dev C (3 hrs):** Formatting (headings, bullets, spacing, margins)
- [ ] **Dev A (2 hrs):** Test DOCX opens in Word/Google Docs, validate ATS compatibility

**Deliverable:** `python generate_docx.py tailored.json` → Creates `resume_tailored.docx`

**Story for Week 5:**
> AS a job seeker, I WANT to download my tailored resume as DOCX SO THAT I can apply to jobs immediately.

---

#### **Week 6: Integration + Real-World Test (10 hours)**

**Sprint Goal:** End-to-end CLI works, team member applies to real jobs

**Tasks:**
- [ ] **Dev B (4 hrs):** CLI wrapper (`python main.py --resume resume.pdf --jd jd.txt --output tailored.docx`)
- [ ] **Dev C (2 hrs):** Error handling, logging, cost tracking (OpenAI API usage)
- [ ] **All devs (4 hrs):** Real-world test (tailor 3 resumes, apply to jobs, track results)

**Deliverable:** Working CLI tool, at least 1 team member applies to 5 jobs with tailored resumes

**Story for Week 6:**
> AS a team, I WANT to validate our tool works SO THAT we prove the core value proposition before building UI.

---

### Milestone 1 Exit Criteria

**Must Have:**
- [ ] CLI tool tailors resume in <30 seconds (excluding LLM call)
- [ ] Jobscan ATS score increases by 10+ points vs original
- [ ] One team member gets recruiter callback (proves PMF signal)
- [ ] LLM cost per session <$0.50

**Go/No-Go Decision:** If no recruiter callback after applying to 10 jobs → Pivot or kill project.

---

## Milestone 2: Beta Web App (Weeks 7-14)

**Capacity:** 10 hrs/week × 8 weeks = **80 hours**

### Objective

Build minimal web UI so 10 friends can use the tool without technical knowledge.

**Success Criteria:**
- ✅ Web app deployed (Railway/Render + Vercel)
- ✅ 10 friends invited, 7+ complete tailoring successfully
- ✅ Average session time <15 minutes
- ✅ Q&A prototype (basic version, 2-3 questions hardcoded)

### Week-by-Week Breakdown

#### **Week 7-8: Backend API (20 hours)**

**Sprint Goal:** FastAPI REST API for core operations

**Tasks:**
- [ ] **Dev C (8 hrs):** FastAPI setup, CORS, `/upload` endpoint (file upload), `/parse-resume` endpoint
- [ ] **Dev B (8 hrs):** `/parse-jd` endpoint, `/tailor` endpoint, async processing (Celery/Background tasks)
- [ ] **Dev A (4 hrs):** API documentation (auto-generated with FastAPI), Postman collection for testing

**Deliverable:** API endpoints working, testable via Postman

**Stories:**
- User uploads resume → API returns parsed JSON
- User submits JD → API returns tailored resume JSON

---

#### **Week 9-10: Basic Auth + Database (20 hours)**

**Sprint Goal:** User accounts, session persistence

**Tasks:**
- [ ] **Dev C (8 hrs):** SQLite database (users, sessions, resumes), SQLAlchemy ORM
- [ ] **Dev B (6 hrs):** JWT authentication (register, login, refresh tokens)
- [ ] **Dev A (6 hrs):** Auth endpoints testing, session storage (store resume + JD + tailored output)

**Deliverable:** Users can register, login, create sessions

**Stories:**
- User registers with email/password
- User logs in, receives JWT token
- User's tailoring sessions saved to database

---

#### **Week 11-12: React Frontend (20 hours)**

**Sprint Goal:** Minimal UI for upload → tailor → download flow

**Tasks:**
- [ ] **Dev A (12 hrs):** React setup (Vite), auth pages (login/register), file upload component
- [ ] **Dev A (4 hrs):** JD input textarea, tailoring results display (JSON → formatted preview)
- [ ] **Dev B (4 hrs):** API integration (Axios/Fetch), loading states, error handling

**Deliverable:** Web UI works end-to-end (local testing)

**Stories:**
- User uploads resume, sees parsed data
- User pastes JD, sees gap analysis preview
- User downloads tailored DOCX

---

#### **Week 13: Q&A Prototype (10 hours)**

**Sprint Goal:** Hardcoded 2-3 clarification questions (basic version)

**Tasks:**
- [ ] **Dev B (5 hrs):** Hardcode 3 generic questions (e.g., "Years of experience with [skill]?", "Led team? Yes/No", "Production or learning?")
- [ ] **Dev A (3 hrs):** Q&A UI component (radio buttons, text input)
- [ ] **Dev C (2 hrs):** Integrate Q&A answers into tailoring prompt context

**Deliverable:** After gap analysis, user answers 2-3 questions before tailoring

**Story:**
> AS a user, I WANT to answer clarifying questions SO THAT the AI tailors my resume more accurately.

**Note:** This is NOT dynamic Q&A (that's M3). Just proving the UX flow.

---

#### **Week 14: Deployment + Beta Testing (10 hours)**

**Sprint Goal:** Deploy to production, invite 10 friends

**Tasks:**
- [ ] **Dev B (4 hrs):** Deploy backend to Railway/Render, environment variables setup
- [ ] **Dev A (3 hrs):** Deploy frontend to Vercel, connect to production API
- [ ] **All devs (3 hrs):** Beta testing with 10 friends, collect feedback (Google Form)

**Deliverable:** Live web app, 10 users complete at least 1 tailoring session

**Success Metrics:**
- 7/10 users complete flow without help
- Average session time: 10-20 minutes
- User feedback: "Would you use this again?" >80% yes

---

### Milestone 2 Exit Criteria

**Must Have:**
- [ ] Web app accessible via public URL
- [ ] 7/10 beta users successfully tailor resume
- [ ] No critical bugs (P0: crashes, data loss)
- [ ] Q&A prototype validates UX flow

**Key Learnings to Capture:**
- Which step takes longest? (optimize in M3)
- What confuses users? (UX improvements)
- Do users trust AI-generated content? (show reasoning in M3)

---

## Milestone 3: Public Launch (Weeks 15-26)

**Capacity:** 10 hrs/week × 12 weeks = **120 hours**

### Objective

Launch publicly, get 100 users, validate freemium conversion, implement full Q&A feature.

**Success Criteria:**
- ✅ 100 registered users (organic + Product Hunt)
- ✅ 10+ paid conversions ($15/month plan)
- ✅ Dynamic Q&A generation (LLM-generated questions based on gaps)
- ✅ Session completion rate >70%
- ✅ CAC <$30 (mostly organic)

### Epic Breakdown

#### **Epic 1: Dynamic Q&A Feature (Weeks 15-18, 40 hours)**

**Goal:** Replace hardcoded questions with LLM-generated contextual questions

**Tasks:**
- [ ] **Dev C (12 hrs):** Q&A generation prompt (analyze resume vs JD gaps, generate 3-5 targeted questions)
- [ ] **Dev B (10 hrs):** Q&A orchestrator (question types: selection, text, multiselect), validation logic
- [ ] **Dev A (10 hrs):** Q&A UI refinement (show context, progress indicator, edit previous answers)
- [ ] **All (8 hrs):** Test Q&A quality with 20 real resume+JD pairs, tune prompt

**Deliverable:** Dynamic Q&A working, questions contextually relevant

**Story:**
> AS a user, I WANT to answer questions specific to my resume gaps SO THAT tailoring is more accurate than generic tool.

---

#### **Epic 2: Freemium + Payments (Weeks 19-20, 20 hours)**

**Goal:** Stripe integration, usage limits, upgrade flow

**Tasks:**
- [ ] **Dev C (8 hrs):** Stripe integration (subscriptions API), webhook handling (payment success/failure)
- [ ] **Dev B (6 hrs):** Usage tracking (3 sessions/month free tier), upgrade gate UI
- [ ] **Dev A (6 hrs):** Pricing page, checkout flow (Stripe Checkout), billing management

**Deliverable:** Users can sign up free, upgrade to Pro ($15/month)

**Story:**
> AS a user, I WANT to upgrade to Pro SO THAT I can tailor unlimited resumes.

---

#### **Epic 3: UX Polish (Weeks 21-22, 20 hours)**

**Goal:** Professional UI, mobile responsive, error handling

**Tasks:**
- [ ] **Dev A (10 hrs):** Responsive design (mobile breakpoints), loading states, empty states
- [ ] **Dev A (5 hrs):** Match score visualization (donut chart, breakdown by category)
- [ ] **Dev B (5 hrs):** Error handling (user-friendly messages, retry buttons, fallback flows)

**Deliverable:** Product feels polished, mobile-friendly

**Story:**
> AS a user, I WANT a professional interface SO THAT I trust the product quality.

---

#### **Epic 4: Landing Page + Marketing (Weeks 23-24, 20 hours)**

**Goal:** Public landing page, SEO, Product Hunt launch

**Tasks:**
- [ ] **Dev A (10 hrs):** Landing page (hero, features, pricing, testimonials, CTA)
- [ ] **Dev B (5 hrs):** SEO basics (meta tags, sitemap, robots.txt), analytics (PostHog/Plausible)
- [ ] **All (5 hrs):** Product Hunt launch prep (screenshots, demo video, outreach to supporters)

**Deliverable:** Landing page live, Product Hunt launch day

**Story:**
> AS a visitor, I WANT to understand the product value SO THAT I decide to sign up.

---

#### **Epic 5: Analytics + Optimization (Weeks 25-26, 20 hours)**

**Goal:** Track funnel, optimize conversion, fix drop-offs

**Tasks:**
- [ ] **Dev B (8 hrs):** Event tracking (user_registered, session_created, resume_downloaded, user_upgraded)
- [ ] **Dev C (6 hrs):** LLM cost optimization (caching identical resumes, use GPT-4o-mini for parsing)
- [ ] **All (6 hrs):** Analyze funnel data, A/B test pricing page ($10 vs $15), optimize slow steps

**Deliverable:** Clear understanding of where users drop off, cost per session <$2

**Story:**
> AS a team, I WANT to track user behavior SO THAT I optimize conversion and reduce costs.

---

### Milestone 3 Exit Criteria

**Must Have:**
- [ ] 100 registered users
- [ ] 10+ paid users ($150+ MRR)
- [ ] Dynamic Q&A increases match score by 5+ points vs M2 hardcoded version
- [ ] Session completion rate >70% (tracked in analytics)
- [ ] Product Hunt launch (top 10 of the day = success)

**Go/No-Go Decision:** If <5 paid conversions → Pricing too high? Feature not valuable? Interview users before continuing.

---

## Milestone 4: Revenue Validation (Weeks 27-40)

**Capacity:** 10 hrs/week × 14 weeks = **140 hours**

### Objective

Prove business model works: Reach $1K MRR (70 paid users), optimize costs, prepare for scale.

**Success Criteria:**
- ✅ $1K MRR (70 paid users @ $15/month)
- ✅ CAC <$25 (organic SEO driving 60%+ signups)
- ✅ LTV >$120 (8+ month retention)
- ✅ Churn <8% monthly
- ✅ 10 SEO articles live, ranking for long-tail keywords

### Epic Breakdown

#### **Epic 1: SEO Content Machine (Weeks 27-32, 60 hours)**

**Goal:** Publish 10 SEO articles targeting job-specific resume keywords

**Tasks:**
- [ ] **Dev A (20 hrs):** SEO research (target keywords: "software engineer resume", "product manager resume ATS", etc.)
- [ ] **Dev B (20 hrs):** Write 10 articles (1,500+ words each, include examples, tool CTA at end)
- [ ] **Dev A (10 hrs):** On-page SEO (internal linking, meta descriptions, images, schema markup)
- [ ] **All (10 hrs):** Publish articles, submit to search console, track rankings

**Deliverable:** 10 articles live, indexed by Google, driving 50+ organic visitors/month by week 32

**Content Calendar:**
1. "How to Tailor Your Resume to a Job Description (2026 Guide)"
2. "Software Engineer Resume Keywords That Pass ATS"
3. "Product Manager Resume: ATS Optimization Guide"
4. "Data Scientist Resume: How to Highlight Projects"
5. "Senior vs Mid-Level Resume: What Recruiters Look For"
6. "Resume Action Verbs for Leadership Roles (150+ Examples)"
7. "How to Beat Applicant Tracking Systems (ATS) in 2026"
8. "Resume vs CV: Differences and When to Use Each"
9. "Career Change Resume: Highlighting Transferable Skills"
10. "Remote Job Resume: Keywords and Format Tips"

**Story:**
> AS a job seeker, I WANT resume advice SO THAT I discover your tool organically.

---

#### **Epic 2: Referral Program (Weeks 33-34, 20 hours)**

**Goal:** Built-in viral loop (refer friends, get free month)

**Tasks:**
- [ ] **Dev B (10 hrs):** Referral code generation, tracking (referrer_id in database)
- [ ] **Dev A (6 hrs):** Referral UI (share link, show referral count, reward status)
- [ ] **Dev C (4 hrs):** Reward logic (give 1 month Pro free for each successful referral)

**Deliverable:** Users can share referral links, get rewarded

**Story:**
> AS a user, I WANT to refer friends SO THAT I get free Pro access.

---

#### **Epic 3: Performance Optimization (Weeks 35-36, 20 hours)**

**Goal:** Handle 100 concurrent users, reduce latency

**Tasks:**
- [ ] **Dev C (10 hrs):** Database optimization (indexes on user_id, session_id, created_at)
- [ ] **Dev B (6 hrs):** API response caching (Redis for repeated resume parses)
- [ ] **Dev C (4 hrs):** LLM caching (cache identical JD parses, reduce OpenAI calls by 30%)

**Deliverable:** P95 latency <500ms (excluding LLM), cost per session <$1.50

**Story:**
> AS a user, I WANT fast responses SO THAT I don't abandon the session.

---

#### **Epic 4: User Retention Features (Weeks 37-38, 20 hours)**

**Goal:** Reduce churn, increase session frequency

**Tasks:**
- [ ] **Dev A (10 hrs):** Session history page (view past tailored resumes, re-download)
- [ ] **Dev B (6 hrs):** Email notifications (resume ready, monthly usage summary, tips)
- [ ] **Dev A (4 hrs):** In-app tips (e.g., "Tailor for multiple jobs to increase callback rate")

**Deliverable:** Users return 2+ times per month (up from 1.2x in M3)

**Story:**
> AS a user, I WANT to access my past resumes SO THAT I reuse them for similar jobs.

---

#### **Epic 5: Business Metrics Dashboard (Weeks 39-40, 20 hours)**

**Goal:** Internal dashboard to track MRR, CAC, LTV, churn

**Tasks:**
- [ ] **Dev B (12 hrs):** Admin dashboard (Retool or custom React page), key metrics visualization
- [ ] **Dev C (5 hrs):** Data exports (user list, revenue report, cohort analysis)
- [ ] **All (3 hrs):** Weekly review of metrics, identify churn reasons (interview churned users)

**Deliverable:** Real-time business metrics, data-driven decisions

**Story:**
> AS a founder, I WANT to see business health SO THAT I know if we're on track.

---

### Milestone 4 Exit Criteria

**Must Have:**
- [ ] $1K MRR (70 paid users)
- [ ] CAC <$25 (proven organic channel)
- [ ] Churn <8% (healthy retention)
- [ ] SEO driving 40%+ signups
- [ ] Unit economics validated (LTV:CAC >4:1)

**Success Definition:** Proven model → Ready to invest in marketing (paid ads, partnerships, content team).

---

## Risk Management

### Technical Risks

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|-----------|
| **LLM API downtime** | Medium | High | Retry logic (exponential backoff), fallback to GPT-3.5, queue system |
| **LLM cost overrun** | High | High | Caching (30% reduction), rate limiting (5 sessions/hour free tier), cheaper models for parsing |
| **Poor tailoring quality** | Medium | Critical | Prompt testing with 50+ real cases, A/B test prompts, user feedback loop |
| **Slow development pace** | High | Medium | Ruthless scope cuts, weekly check-ins, defer non-critical features |
| **Team burnout (10 hrs/week)** | Medium | High | 1-week breaks every 2 months, celebrate small wins, flexible schedules |

### Product Risks

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|-----------|
| **No recruiter callbacks (M1)** | Medium | Critical | Test with 10 jobs per dev (30 total), if <3 callbacks → pivot or kill |
| **Users don't trust AI output** | Medium | High | Show reasoning ("Why we changed X"), allow editing, display match score |
| **Freemium not converting (<5%)** | High | High | A/B test pricing ($10 vs $15), offer annual plan (2 months free), trial period (7 days Pro free) |
| **High churn (>10%)** | Medium | High | Exit surveys, cohort analysis, email re-engagement campaign |

### Timeline Risks

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|-----------|
| **M1 takes 8+ weeks** | Medium | Medium | Cut DOCX generation (output plain text first), defer error handling |
| **Dev unavailable** | Medium | High | Document everything, async communication (GitHub issues), pair programming recordings |
| **Scope creep in M3** | High | Medium | Strict feature freeze after M3 week 20, defer everything to M4 |

---

## Success Metrics by Milestone

| Metric | M1 (Week 6) | M2 (Week 14) | M3 (Week 26) | M4 (Week 40) |
|--------|-------------|--------------|--------------|--------------|
| **Users** | 3 (team) | 10 (friends) | 100 | 500 |
| **Paid Users** | 0 | 0 | 10 | 70 |
| **MRR** | $0 | $0 | $150 | $1,000 |
| **Session Completion** | 100% | 70% | 70% | 75% |
| **Match Score Improvement** | +12 points | +15 points | +18 points | +20 points |
| **LLM Cost/Session** | $0.50 | $0.40 | $0.30 | $0.20 |
| **CAC** | N/A | $0 (organic) | $30 | $25 |
| **Churn** | N/A | N/A | N/A | <8% |

---

## Weekly Standup Format (10-minute async)

**Post in Slack/Discord every Monday:**

1. **Last week accomplishments** (3 bullet points)
2. **This week goals** (3 bullet points)
3. **Blockers** (if any)
4. **Hours spent** (actual vs planned)

**Example:**
```
Dev B - Week 7 Update

✅ Last Week:
- Completed FastAPI setup, /upload endpoint working
- Integrated PyMuPDF, tested with 10 resumes
- Deployed dev environment to Railway

🎯 This Week:
- Build /parse-jd endpoint (4 hrs)
- Add async background tasks for LLM calls (2 hrs)
- Write API integration tests (2 hrs)

🚧 Blockers:
- OpenAI API rate limits in dev (solution: use separate keys)

⏱️ Hours: 8 actual (planned: 8)
```

---

## Milestone Review Template

**After each milestone, fill out:**

### What Went Well
- [Specific wins, metrics achieved]

### What Went Wrong
- [Problems encountered, metrics missed]

### Key Learnings
- [Insights to apply to next milestone]

### Scope Adjustments
- [Features to cut/add for next milestone]

### Go/No-Go Decision
- [ ] Continue to next milestone
- [ ] Pivot (describe)
- [ ] Kill project (justify)

---

## Appendix: Tech Stack Details

### Backend
- **Language:** Python 3.11+
- **Framework:** FastAPI (async, auto docs, validation)
- **Database:** PostgreSQL 15+ (Supabase free tier or Railway)
- **ORM:** SQLAlchemy 2.0 (async support)
- **Task Queue:** Celery + Redis (for background LLM calls)
- **Auth:** JWT (python-jose library)
- **File Storage:** Local filesystem (migrate to S3 in M4 if needed)

### AI/LLM
- **Provider:** OpenAI API
- **Models:** 
  - GPT-4o-mini for parsing (cheaper, faster)
  - GPT-4o for tailoring (higher quality)
- **Libraries:** openai SDK, langchain (optional, for prompt management)

### Frontend
- **Framework:** React 18+ with TypeScript
- **Build Tool:** Vite
- **State Management:** TanStack Query (React Query)
- **Styling:** Tailwind CSS
- **Routing:** React Router v6

### DevOps
- **Backend Hosting:** Railway or Render (free tier → $7/month)
- **Frontend Hosting:** Vercel (free tier)
- **Database:** Railway Postgres or Supabase (free tier)
- **Monitoring:** Sentry (errors), PostHog (analytics)
- **CI/CD:** GitHub Actions (deploy on push to main)

### Development Tools
- **Package Manager:** Poetry (Python) or UV (faster)
- **Code Quality:** Black (formatting), Ruff (linting), mypy (type checking)
- **Testing:** pytest, pytest-asyncio
- **API Testing:** Postman or HTTPie

---

## Quick Start for Developers

### Initial Setup (Week 1, Day 1)

```bash
# Clone repo
git clone https://github.com/yourusername/resume-fine-tuner.git
cd resume-fine-tuner

# Backend setup
cd backend
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt

# Environment variables
cp .env.example .env
# Add: OPENAI_API_KEY=sk-...

# Run migrations (in M2)
alembic upgrade head

# Start backend (M2+)
uvicorn main:app --reload

# Frontend setup (M2, Week 11)
cd ../frontend
npm install
npm run dev
```

### Running Tests

```bash
# Backend tests
cd backend
pytest

# Frontend tests (if added)
cd frontend
npm test
```

---

## Document Status

**Status:** ✅ Ready for Development Kickoff  
**Next Steps:** 
1. Dev team reviews plan (async, 2 days)
2. Kick-off meeting (30 min, align on M1 goals)
3. Week 1 Sprint starts Monday

**Contact:** [Your name/email for questions]

---

**Remember:** 10 hrs/week is TIGHT. Ruthlessly cut scope, validate early, celebrate small wins. Ship fast, learn fast, iterate fast. 🚀