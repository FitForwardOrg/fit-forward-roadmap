# Milestone 1 User Stories - Jira/Linear Format

**Epic:** M1 - Working Prototype (CLI Tool)  
**Timeline:** Weeks 1-6  
**Total Story Points:** 60

---

## Week 1: Foundation

### STORY-001: Repository Setup and Python Environment

**Type:** Task  
**Priority:** Critical  
**Assignee:** Dev C  
**Story Points:** 5  
**Sprint:** Week 1

**Description:**
Set up the foundational repository structure and Python development environment for the Resume Fine-Tuner project.

**Acceptance Criteria:**

- [ ] GitHub repository created with README.md
- [ ] Python 3.11+ environment configured (Poetry or UV)
- [ ] `.gitignore` configured for Python projects
- [ ] Project structure created:

  ```
  resume-fine-tuner/
  ├── backend/
  │   ├── parsers/
  │   ├── llm/
  │   ├── generators/
  │   └── tests/
  ├── requirements.txt
  ├── .env.example
  └── README.md
  ```

- [ ] Virtual environment activates successfully
- [ ] Dependencies installable via `pip install -r requirements.txt`

**Technical Notes:**

- Use Poetry for dependency management (faster than pip)
- Include: `fastapi`, `uvicorn`, `openai`, `pymupdf`, `python-docx`, `pytest`
- Set Python version constraint: `>=3.11,<4.0`

**Definition of Done:**

- Repository cloned and runs on all 3 dev machines
- `python --version` returns 3.11+
- All dependencies install without errors

---

### STORY-002: PDF Text Extraction

**Type:** Feature  
**Priority:** Critical  
**Assignee:** Dev B  
**Story Points:** 3  
**Sprint:** Week 1  
**Depends On:** STORY-001

**User Story:**
> AS a developer  
> I WANT to extract text from PDF resumes  
> SO THAT I can feed it to the LLM for parsing

**Acceptance Criteria:**

- [ ] Function `extract_text_from_pdf(file_path: str) -> str` implemented
- [ ] Uses PyMuPDF (fitz) library
- [ ] Handles multi-page PDFs (concatenates text)
- [ ] Returns empty string for corrupted/invalid PDFs (no crash)
- [ ] Tested with 5+ different resume PDF formats
- [ ] Preserves line breaks and basic structure

**Test Cases:**

1. Single-page resume PDF → Returns full text
2. Multi-page resume PDF → Returns concatenated text
3. Corrupted PDF → Returns empty string, logs error
4. Password-protected PDF → Returns empty string, logs error
5. Scanned PDF (image-based) → Returns empty string (OCR not in scope)

**Technical Notes:**

```python
import fitz  # PyMuPDF

def extract_text_from_pdf(file_path: str) -> str:
    try:
        doc = fitz.open(file_path)
        text = ""
        for page in doc:
            text += page.get_text()
        doc.close()
        return text.strip()
    except Exception as e:
        print(f"Error extracting PDF: {e}")
        return ""
```

**Definition of Done:**

- Function works with 10+ real resume PDFs from team
- Unit test passes: `test_pdf_extraction.py`
- Code committed to `backend/parsers/pdf_parser.py`

---

### STORY-003: DOCX Text Extraction

**Type:** Feature  
**Priority:** Critical  
**Assignee:** Dev B  
**Story Points:** 2  
**Sprint:** Week 1  
**Depends On:** STORY-001

**User Story:**
> AS a developer  
> I WANT to extract text from DOCX resumes  
> SO THAT I can support both PDF and Word formats

**Acceptance Criteria:**

- [ ] Function `extract_text_from_docx(file_path: str) -> str` implemented
- [ ] Uses python-docx library
- [ ] Extracts text from paragraphs and tables
- [ ] Preserves basic formatting (headings, bullets as plain text)
- [ ] Handles corrupted DOCX gracefully
- [ ] Tested with 5+ different resume DOCX formats

**Test Cases:**

1. Simple DOCX (paragraphs only) → Returns full text
2. DOCX with tables → Returns table text inline
3. DOCX with bullets → Returns bullets as `- Item`
4. Corrupted DOCX → Returns empty string, logs error
5. Empty DOCX → Returns empty string

**Technical Notes:**

```python
from docx import Document

def extract_text_from_docx(file_path: str) -> str:
    try:
        doc = Document(file_path)
        text = []
        for para in doc.paragraphs:
            text.append(para.text)
        return "\n".join(text).strip()
    except Exception as e:
        print(f"Error extracting DOCX: {e}")
        return ""
```

**Definition of Done:**

- Function works with 10+ real resume DOCX files
- Unit test passes: `test_docx_extraction.py`
- Code committed to `backend/parsers/docx_parser.py`

---

## Week 2: Resume Parsing

### STORY-004: OpenAI API Client Wrapper

**Type:** Task  
**Priority:** Critical  
**Assignee:** Dev C  
**Story Points:** 4  
**Sprint:** Week 2  
**Depends On:** STORY-001

**Description:**
Create a reusable OpenAI API client wrapper with error handling, retry logic, and cost tracking.

**Acceptance Criteria:**

- [ ] Class `LLMClient` implemented with method `generate(prompt: str, model: str) -> str`
- [ ] API key loaded from `.env` file (variable: `OPENAI_API_KEY`)
- [ ] Retry logic: 3 attempts with exponential backoff on rate limit errors
- [ ] Logs API usage (tokens consumed, cost estimate)
- [ ] Supports both `gpt-4o` and `gpt-4o-mini` models
- [ ] Returns parsed JSON when prompt requests JSON output

**Technical Notes:**

```python
import openai
import os
import time

class LLMClient:
    def __init__(self):
        openai.api_key = os.getenv("OPENAI_API_KEY")
        
    def generate(self, prompt: str, model: str = "gpt-4o-mini") -> str:
        for attempt in range(3):
            try:
                response = openai.ChatCompletion.create(
                    model=model,
                    messages=[{"role": "user", "content": prompt}],
                    temperature=0.3
                )
                return response.choices[0].message.content
            except openai.error.RateLimitError:
                time.sleep(2 ** attempt)  # Exponential backoff
        raise Exception("OpenAI API failed after 3 retries")
```

**Definition of Done:**

- Client successfully calls OpenAI API
- Retry logic tested (mock rate limit error)
- Cost logging outputs: "Tokens: 500, Estimated cost: $0.015"
- Code committed to `backend/llm/client.py`

---

### STORY-005: Resume Parsing LLM Prompt

**Type:** Feature  
**Priority:** Critical  
**Assignee:** Dev B  
**Story Points:** 4  
**Sprint:** Week 2  
**Depends On:** STORY-004

**User Story:**
> AS a system  
> I WANT to parse resume text into structured JSON  
> SO THAT I can analyze it against job requirements

**Acceptance Criteria:**

- [ ] Function `parse_resume(resume_text: str) -> dict` implemented
- [ ] Uses LLM to extract: `skills`, `experience`, `education`
- [ ] Returns valid JSON (validated with `json.loads()`)
- [ ] Handles edge cases: career gaps, multiple jobs at same company, freelance work
- [ ] Tested with 5 real resumes, achieves 90%+ accuracy
- [ ] Prompt instructs LLM to return ONLY JSON (no markdown formatting)

**Prompt Template:**

```
You are a resume parser. Extract the following information as JSON:

{
  "skills": ["skill1", "skill2", ...],
  "experience": [
    {
      "company": "Company Name",
      "position": "Job Title",
      "start_date": "YYYY-MM",
      "end_date": "YYYY-MM or Present",
      "achievements": ["achievement 1", "achievement 2", ...]
    }
  ],
  "education": [
    {
      "institution": "University Name",
      "degree": "Degree Type",
      "field": "Field of Study",
      "graduation_year": "YYYY"
    }
  ]
}

Resume text:
{resume_text}

Return ONLY valid JSON, no markdown code blocks or explanations.
```

**Test Cases:**

1. Standard resume → Returns complete JSON
2. Resume with career gap (2020-2022) → Extracts all jobs, gap ignored
3. Resume with freelance work → Extracts as "Freelance" company
4. Resume with incomplete dates → Uses "Present" or "Unknown"
5. Resume with typos → LLM corrects basic typos

**Definition of Done:**

- JSON output validates with schema
- 90%+ accuracy on 5 test resumes (manual verification)
- Function returns dict, not string
- Code committed to `backend/parsers/resume_parser.py`

---

### STORY-006: Resume Parsing Quality Validation

**Type:** Task  
**Priority:** High  
**Assignee:** Dev B  
**Story Points:** 2  
**Sprint:** Week 2  
**Depends On:** STORY-005

**Description:**
Test resume parsing with 10+ real resumes and document accuracy metrics.

**Acceptance Criteria:**

- [ ] Collect 10 real resumes (team members + anonymized samples)
- [ ] Run `parse_resume()` on all 10 resumes
- [ ] Manually verify:
  - Skills extraction: >90% of skills captured
  - Experience extraction: >90% of jobs captured with correct dates
  - Education extraction: 100% of degrees captured
- [ ] Document failures in `tests/parsing_failures.md`
- [ ] Tune prompt if accuracy <90%

**Metrics to Track:**

- Skills accuracy: (extracted skills / total skills) per resume
- Experience accuracy: (extracted jobs / total jobs) per resume
- Education accuracy: (extracted degrees / total degrees) per resume
- JSON validity: 100% of outputs must be valid JSON

**Definition of Done:**

- Spreadsheet created: `Resume_Parsing_Accuracy.csv`
- Average accuracy >90% across all categories
- Prompt tuned if needed (max 2 iterations)
- Results documented in `docs/week2_parsing_results.md`

---

## Week 3: JD Parsing + Gap Analysis

### STORY-007: JD Parsing LLM Prompt

**Type:** Feature  
**Priority:** Critical  
**Assignee:** Dev C  
**Story Points:** 3  
**Sprint:** Week 3  
**Depends On:** STORY-004

**User Story:**
> AS a system  
> I WANT to parse job description into requirements  
> SO THAT I can compare it with the resume

**Acceptance Criteria:**

- [ ] Function `parse_jd(jd_text: str) -> dict` implemented
- [ ] Extracts: `required_skills`, `nice_to_have_skills`, `role_level`, `responsibilities`
- [ ] Returns valid JSON
- [ ] Handles short JDs (100 words) and long JDs (1000+ words)
- [ ] Tested with 10 real JDs from LinkedIn/Indeed

**Prompt Template:**

```
You are a job description analyzer. Extract these details as JSON:

{
  "required_skills": ["skill1", "skill2", ...],
  "nice_to_have_skills": ["skill3", "skill4", ...],
  "role_level": "Junior|Mid-Level|Senior|Lead|Principal",
  "responsibilities": ["responsibility 1", "responsibility 2", ...],
  "company_values": ["value 1", "value 2", ...]
}

Job Description:
{jd_text}

Return ONLY valid JSON.
```

**Test Cases:**

1. Standard JD → Returns complete JSON
2. JD with no "nice-to-have" → Returns empty array
3. Short JD (100 words) → Extracts core requirements
4. Long JD (1000+ words) → Extracts key requirements (not everything)
5. JD with ambiguous level → LLM infers from responsibilities

**Definition of Done:**

- JSON output validates with schema
- 90%+ accuracy on 10 test JDs
- Code committed to `backend/parsers/jd_parser.py`

---

### STORY-008: Gap Analyzer Implementation

**Type:** Feature  
**Priority:** Critical  
**Assignee:** Dev B  
**Story Points:** 4  
**Sprint:** Week 3  
**Depends On:** STORY-005, STORY-007

**User Story:**
> AS a job seeker  
> I WANT to see how my resume matches the JD  
> SO THAT I know what to emphasize

**Acceptance Criteria:**

- [ ] Function `analyze_gap(resume: dict, jd: dict) -> dict` implemented
- [ ] Calculates:
  - `match_score`: 0-100 (percentage of required skills present)
  - `missing_skills`: List of required skills not in resume
  - `matched_skills`: List of required skills present in resume
  - `experience_match`: "Under-qualified" | "Qualified" | "Over-qualified"
- [ ] Returns structured dict for downstream use
- [ ] Tested with 5 resume+JD pairs

**Algorithm:**

```python
def analyze_gap(resume: dict, jd: dict) -> dict:
    resume_skills = set([s.lower() for s in resume["skills"]])
    required_skills = set([s.lower() for s in jd["required_skills"]])
    
    matched = resume_skills.intersection(required_skills)
    missing = required_skills - resume_skills
    
    match_score = int((len(matched) / len(required_skills)) * 100) if required_skills else 0
    
    return {
        "match_score": match_score,
        "matched_skills": list(matched),
        "missing_skills": list(missing),
        "experience_match": infer_experience_level(resume, jd)
    }
```

**Test Cases:**

1. 100% skill match → `match_score: 100`
2. 50% skill match → `match_score: 50`
3. 0% skill match → `match_score: 0`
4. Resume has extra skills → Not penalized (only missing skills matter)
5. Junior resume vs Senior JD → `experience_match: "Under-qualified"`

**Definition of Done:**

- Function returns expected output for all test cases
- Code committed to `backend/parsers/gap_analyzer.py`

---

### STORY-009: Gap Analysis Manual Testing

**Type:** Task  
**Priority:** High  
**Assignee:** Dev A  
**Story Points:** 3  
**Sprint:** Week 3  
**Depends On:** STORY-008

**Description:**
Manually test gap analyzer with 10 real resume+JD pairs from LinkedIn/Indeed and validate match scores.

**Acceptance Criteria:**

- [ ] Collect 10 real JDs (Software Engineer, Product Manager, Data Scientist, etc.)
- [ ] Run gap analyzer on team resumes vs collected JDs
- [ ] Manually verify match scores are reasonable (±10% of expected)
- [ ] Document cases where match score seems incorrect
- [ ] Tune algorithm if accuracy issues found

**Test Matrix:**

| Resume | JD | Expected Match | Actual Match | Pass/Fail |
|--------|-----|---------------|--------------|-----------|
| DevB-SWE | Google SWE | 80-90% | 85% | ✅ |
| DevC-Backend | AWS Backend | 70-80% | 72% | ✅ |
| ... | ... | ... | ... | ... |

**Definition of Done:**

- 10 resume+JD pairs tested
- Match scores within ±10% of manual assessment
- Test results documented in `docs/week3_gap_analysis.md`

---

## Week 4: Resume Tailoring

### STORY-010: Resume Tailoring LLM Prompt

**Type:** Feature  
**Priority:** Critical  
**Assignee:** Dev C  
**Story Points:** 5  
**Sprint:** Week 4  
**Depends On:** STORY-008

**User Story:**
> AS a job seeker  
> I WANT my resume tailored to the JD  
> SO THAT I pass ATS screening

**Acceptance Criteria:**

- [ ] Function `tailor_resume(resume: dict, jd: dict, gap: dict) -> dict` implemented
- [ ] LLM rewrites experience bullets to emphasize matched skills
- [ ] Injects missing keywords naturally (no keyword stuffing)
- [ ] Maintains truthfulness (doesn't invent experience)
- [ ] Returns tailored resume JSON with same structure as original
- [ ] Tested with 3 resume+JD pairs

**Prompt Template:**

```
You are an expert resume writer. Tailor this resume for the job description.

Original Resume:
{resume_json}

Job Description Requirements:
{jd_json}

Gap Analysis:
- Matched skills: {matched_skills}
- Missing skills: {missing_skills}

Instructions:
1. Rewrite experience bullets to emphasize matched skills
2. Inject missing keywords naturally (where truthful)
3. Highlight leadership, impact, metrics
4. Maintain honesty - don't invent experience
5. Keep same JSON structure as input

Output tailored resume as JSON with same structure.
```

**Quality Checks:**

- Keyword density: <2% increase vs original (no stuffing)
- Truthfulness: All claims verifiable from original resume
- Readability: Bullets remain clear and concise

**Definition of Done:**

- Tailored resume JSON validates
- Manual review: 3/3 tailored resumes are truthful and well-written
- Code committed to `backend/llm/resume_tailor.py`

---

### STORY-011: Gap Analysis Integration into Tailoring

**Type:** Task  
**Priority:** High  
**Assignee:** Dev B  
**Story Points:** 3  
**Sprint:** Week 4  
**Depends On:** STORY-010

**Description:**
Integrate gap analysis results into the tailoring prompt context to provide LLM with strategic guidance.

**Acceptance Criteria:**

- [ ] Tailoring prompt includes gap analysis summary
- [ ] LLM prioritizes emphasizing matched skills
- [ ] LLM subtly addresses missing skills (if applicable)
- [ ] Tested: Tailored resume improves match score by 5+ points

**Implementation:**

```python
def tailor_resume(resume: dict, jd: dict, gap: dict) -> dict:
    context = f"""
    Gap Analysis Summary:
    - Current match score: {gap['match_score']}%
    - Matched skills to emphasize: {', '.join(gap['matched_skills'])}
    - Missing skills to address (if applicable): {', '.join(gap['missing_skills'][:3])}
    """
    
    prompt = TAILORING_PROMPT.format(
        resume_json=json.dumps(resume),
        jd_json=json.dumps(jd),
        context=context
    )
    
    return llm_client.generate(prompt)
```

**Definition of Done:**

- Tailored resumes show 5-10 point match score improvement
- No false claims introduced
- Code committed and integrated

---

### STORY-012: Tailoring Quality Testing with Jobscan

**Type:** Task  
**Priority:** High  
**Assignee:** Dev B  
**Story Points:** 2  
**Sprint:** Week 4  
**Depends On:** STORY-011

**Description:**
Test tailored resumes using Jobscan ATS checker and validate score improvements.

**Acceptance Criteria:**

- [ ] Create Jobscan account (free trial if available)
- [ ] Upload 3 original resumes + JDs to Jobscan → Record scores
- [ ] Tailor resumes using our tool
- [ ] Upload 3 tailored resumes + same JDs to Jobscan → Record scores
- [ ] Validate: Tailored score > Original score by 10+ points
- [ ] Document results in `docs/week4_jobscan_results.md`

**Test Matrix:**

| Resume | JD | Original Score | Tailored Score | Improvement |
|--------|-----|---------------|----------------|-------------|
| DevB | Google SWE | 68% | 82% | +14% ✅ |
| DevC | AWS Backend | 72% | 85% | +13% ✅ |
| ... | ... | ... | ... | ... |

**Definition of Done:**

- All 3 tailored resumes score 10+ points higher
- Screenshots saved in `docs/jobscan_screenshots/`
- If <10 point improvement → Tune tailoring prompt

---

## Week 5: DOCX Generation

### STORY-013: DOCX Resume Generator

**Type:** Feature  
**Priority:** Critical  
**Assignee:** Dev B  
**Story Points:** 5  
**Sprint:** Week 5  
**Depends On:** STORY-011

**User Story:**
> AS a job seeker  
> I WANT to download my tailored resume as DOCX  
> SO THAT I can apply to jobs immediately

**Acceptance Criteria:**

- [ ] Function `generate_docx(resume: dict, output_path: str)` implemented
- [ ] Uses python-docx library
- [ ] Template is ATS-friendly:
  - No tables (ATS often can't parse)
  - Simple fonts: Arial or Calibri, 11pt
  - Clear section headings (bold, larger font)
  - Bullet points for achievements
- [ ] Sections: Summary, Experience, Skills, Education
- [ ] Generated DOCX opens in Microsoft Word without errors
- [ ] Tested with 3 resumes

**Technical Implementation:**

```python
from docx import Document
from docx.shared import Pt, Inches

def generate_docx(resume: dict, output_path: str):
    doc = Document()
    
    # Name and contact (centered)
    name = doc.add_heading(resume['name'], level=1)
    name.alignment = WD_ALIGN_PARAGRAPH.CENTER
    
    # Summary section
    doc.add_heading('Professional Summary', level=2)
    doc.add_paragraph(resume.get('summary', ''))
    
    # Experience section
    doc.add_heading('Professional Experience', level=2)
    for exp in resume['experience']:
        doc.add_heading(f"{exp['position']} - {exp['company']}", level=3)
        doc.add_paragraph(f"{exp['start_date']} - {exp['end_date']}")
        for achievement in exp['achievements']:
            doc.add_paragraph(achievement, style='List Bullet')
    
    # Skills section
    doc.add_heading('Skills', level=2)
    doc.add_paragraph(', '.join(resume['skills']))
    
    # Education section
    doc.add_heading('Education', level=2)
    for edu in resume['education']:
        doc.add_paragraph(f"{edu['degree']}, {edu['institution']} ({edu['graduation_year']})")
    
    doc.save(output_path)
```

**Definition of Done:**

- DOCX opens in Word/Google Docs
- Format is clean and professional
- Code committed to `backend/generators/docx_generator.py`

---

### STORY-014: DOCX Formatting and Styling

**Type:** Task  
**Priority:** High  
**Assignee:** Dev C  
**Story Points:** 3  
**Sprint:** Week 5  
**Depends On:** STORY-013

**Description:**
Refine DOCX formatting for ATS compatibility and visual appeal.

**Acceptance Criteria:**

- [ ] Margins: 1 inch all sides
- [ ] Font: Arial 11pt (body), 14pt (headings)
- [ ] Line spacing: 1.15
- [ ] Section spacing: 12pt before headings
- [ ] Bullet points: Standard disc style
- [ ] No graphics, logos, or images (ATS can't parse)
- [ ] Tested: Upload to 3 different ATS systems (simulate with Jobscan)

**ATS Compatibility Checklist:**

- [x] No tables
- [x] No text boxes
- [x] No headers/footers
- [x] No images
- [x] No columns
- [x] Simple bullet points
- [x] Standard fonts

**Definition of Done:**

- Generated DOCX passes ATS compatibility check (Jobscan)
- Visual review: Professional appearance

---

### STORY-015: DOCX Validation Testing

**Type:** Task  
**Priority:** Medium  
**Assignee:** Dev A  
**Story Points:** 2  
**Sprint:** Week 5  
**Depends On:** STORY-014

**Description:**
Test generated DOCX files in multiple applications to ensure compatibility.

**Acceptance Criteria:**

- [ ] Test DOCX opens in:
  - Microsoft Word (Windows)
  - Microsoft Word (Mac)
  - Google Docs
  - LibreOffice Writer
- [ ] Formatting preserved across all applications
- [ ] No corruption errors
- [ ] Test with 3 different resumes

**Test Matrix:**

| Application | Resume 1 | Resume 2 | Resume 3 | Issues |
|-------------|----------|----------|----------|--------|
| Word Win | ✅ | ✅ | ✅ | None |
| Word Mac | ✅ | ✅ | ✅ | None |
| Google Docs | ✅ | ✅ | ✅ | Slight spacing |
| LibreOffice | ✅ | ✅ | ✅ | None |

**Definition of Done:**

- All applications open DOCX without errors
- Major formatting preserved (minor differences acceptable)
- Issues documented in `docs/docx_compatibility.md`

---

## Week 6: Integration + Real-World Test

### STORY-016: CLI Wrapper Implementation

**Type:** Feature  
**Priority:** Critical  
**Assignee:** Dev B  
**Story Points:** 4  
**Sprint:** Week 6  
**Depends On:** STORY-014

**User Story:**
> AS a team member  
> I WANT to run the entire pipeline from command line  
> SO THAT I can tailor my resume without writing code

**Acceptance Criteria:**

- [ ] CLI script `main.py` implemented
- [ ] Command: `python main.py --resume resume.pdf --jd jd.txt --output tailored.docx`
- [ ] Progress indicators: "Parsing resume...", "Analyzing gaps...", "Tailoring...", "Done!"
- [ ] Displays match score after completion
- [ ] Error handling: Clear error messages for invalid inputs
- [ ] Help text: `python main.py --help`

**CLI Usage:**

```bash
$ python main.py --resume DevB_Resume.pdf --jd google_swe.txt --output DevB_Tailored.docx

[1/4] Extracting text from resume...
[2/4] Parsing resume and job description...
[3/4] Analyzing gaps... Match score: 68%
[4/4] Tailoring resume...
✅ Tailored resume saved to DevB_Tailored.docx
📊 New match score: 82% (+14%)
💰 Cost: $0.35
```

**Definition of Done:**

- CLI runs end-to-end without manual intervention
- Clear progress feedback
- Code committed to `backend/main.py`

---

### STORY-017: Error Handling and Logging

**Type:** Task  
**Priority:** High  
**Assignee:** Dev C  
**Story Points:** 2  
**Sprint:** Week 6  
**Depends On:** STORY-016

**Description:**
Add comprehensive error handling and logging to the CLI tool.

**Acceptance Criteria:**

- [ ] Log file created: `resume_finetuner.log`
- [ ] Logs include: timestamp, action, result, API cost
- [ ] Graceful error handling:
  - Invalid file path → Clear error message
  - Corrupted PDF/DOCX → Suggest alternative format
  - OpenAI API error → Display retry message
  - Invalid JD (too short) → Suggest adding more details
- [ ] User-friendly error messages (no stack traces)

**Error Handling Examples:**

```python
# File not found
if not os.path.exists(resume_path):
    print(f"❌ Error: Resume file not found at {resume_path}")
    sys.exit(1)

# API error
try:
    tailored = tailor_resume(resume, jd, gap)
except Exception as e:
    print(f"❌ Error calling OpenAI API: {str(e)}")
    print("💡 Tip: Check your OPENAI_API_KEY in .env file")
    sys.exit(1)
```

**Definition of Done:**

- All error cases handled gracefully
- Log file contains useful debugging info
- No unhandled exceptions

---

### STORY-018: Real-World Validation Test

**Type:** Test  
**Priority:** Critical  
**Assignee:** All Devs  
**Story Points:** 4  
**Sprint:** Week 6  
**Depends On:** STORY-017

**Description:**
Each team member uses the CLI tool to tailor their resume and apply to 5+ real jobs. Track results to validate product value.

**Acceptance Criteria:**

- [ ] Each dev tailors their resume for 3 different JDs
- [ ] Each dev applies to 5+ jobs with tailored resume
- [ ] Track applications in spreadsheet:
  - Date applied
  - Company/role
  - Original match score
  - Tailored match score
  - Recruiter response? (Yes/No)
  - Days until response
- [ ] Goal: At least 1 recruiter callback across team (proves PMF signal)
- [ ] Document results in `docs/week6_real_world_results.md`

**Application Tracking Spreadsheet:**

| Dev | Company | Role | Original Score | Tailored Score | Applied Date | Recruiter Response? | Days to Response |
|-----|---------|------|---------------|----------------|--------------|--------------------|--------------------|
| Dev B | Google | SWE | 68% | 82% | 2026-02-10 | ✅ Yes | 4 days |
| Dev C | AWS | Backend | 72% | 85% | 2026-02-11 | ❌ No | - |
| ... | ... | ... | ... | ... | ... | ... | ... |

**Success Metrics:**

- 15 total applications (5 per dev)
- Average match score improvement: >10 points
- Recruiter callbacks: ≥1 (minimum for PMF validation)

**Definition of Done:**

- All 15 applications submitted
- Results tracked for 2 weeks
- At least 1 recruiter callback received
- Retrospective meeting held to discuss learnings

---

## Milestone 1 Exit Criteria

### EPIC-M1-EXIT: Milestone 1 Review and Go/No-Go Decision

**Type:** Milestone  
**Priority:** Critical  
**Assignee:** All Devs  
**Sprint:** Week 6 (End)

**Exit Criteria Checklist:**

- [ ] CLI tool works end-to-end (all features functional)
- [ ] Jobscan ATS score increases by 10+ points vs original
- [ ] At least 1 team member gets recruiter callback
- [ ] LLM cost per session <$0.50
- [ ] All code committed to GitHub with tests
- [ ] Documentation complete: README, API docs, testing results

**Go/No-Go Decision Matrix:**

| Criteria | Target | Actual | Status |
|----------|--------|--------|--------|
| CLI functionality | 100% working | ___ % | ⬜ |
| ATS score improvement | +10 points | +___ points | ⬜ |
| Recruiter callbacks | ≥1 | ___ | ⬜ |
| LLM cost | <$0.50 | $___ | ⬜ |

**Decision:**

- ✅ **GO:** Continue to M2 (Beta Web App) if ≥3 criteria met
- 🔄 **PIVOT:** Adjust approach if 2 criteria met (e.g., cost too high → optimize prompts)
- ❌ **NO-GO:** Kill project if <2 criteria met (no PMF signal)

**Retrospective Questions:**

1. What worked well in M1?
2. What slowed us down?
3. What surprised us?
4. What should we change for M2?

**Next Steps if GO:**

- Schedule M2 kick-off meeting
- Assign Week 7-8 tasks
- Update roadmap based on M1 learnings

---

## Summary: M1 User Stories

| Week | Stories | Story Points | Key Deliverable |
|------|---------|--------------|-----------------|
| **Week 1** | STORY-001 to STORY-003 | 10 | Document parsers working |
| **Week 2** | STORY-004 to STORY-006 | 10 | Resume parsing with LLM |
| **Week 3** | STORY-007 to STORY-009 | 10 | JD parsing + gap analysis |
| **Week 4** | STORY-010 to STORY-012 | 10 | Resume tailoring working |
| **Week 5** | STORY-013 to STORY-015 | 10 | DOCX generation polished |
| **Week 6** | STORY-016 to STORY-018 | 10 | CLI + real-world validation |
| **Total** | **18 stories** | **60 points** | **Validated CLI prototype** |

---

## Jira/Linear Import Format

### CSV Import Template

```csv
Summary,Description,Type,Priority,Assignee,Story Points,Sprint,Depends On
"Repository Setup and Python Environment","Set up foundational repository...","Task","Critical","Dev C",5,"Week 1",""
"PDF Text Extraction","Extract text from PDF resumes...","Feature","Critical","Dev B",3,"Week 1","STORY-001"
...
```

### JSON Import Template (Linear)

```json
[
  {
    "title": "Repository Setup and Python Environment",
    "description": "Set up foundational repository...",
    "priority": 1,
    "estimate": 5,
    "assigneeId": "dev-c",
    "projectId": "milestone-1",
    "labelIds": ["backend", "infrastructure"]
  },
  ...
]
```

---

**Document Status:** ✅ Ready for Import into Jira/Linear  
**Next Action:** Import stories into project management tool and start Week 1 sprint
