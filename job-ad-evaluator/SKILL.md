---
name: job-ad-evaluator
description: Vyhodnocení kvality pracovních inzerátů podle 6 klíčových kritérií s podporou automatického stahování z URL. 
---


## Overview

Professional job ad evaluation with simple HTTP scraping and 6-criteria scoring system.

**Version:** 3.0  
**Language:** Czech output, Czech/English input  
**Tools:** Bash (curl, sed)

---

## Workflow

### Step 1: Input Detection

Detect if user provided URL or text block:

- If input contains `http://` or `https://` → Process as URL
- Otherwise → Process as text directly

---

### Step 2: URL Processing

When URL is detected, execute simple HTTP request:

```bash
#!/bin/bash
URL="$1"

echo "📥 Načítám obsah z URL..."

# Simple HTTP request
CONTENT=$(curl -s -L --max-time 30 "$URL")
STATUS=$?

# Check if request succeeded
if [ $STATUS -ne 0 ]; then
    echo "❌ CHYBA: URL není dostupná (404 nebo timeout)"
    echo "→ Zkopíruj text inzerátu ručně a vlož ho sem."
    exit 1
fi

# Check if content is empty or very small
CONTENT_SIZE=${#CONTENT}
if [ $CONTENT_SIZE -lt 500 ]; then
    echo "❌ CHYBA: Stránka vrátila prázdný obsah"
    echo "→ Pravděpodobně používá JavaScript pro načítání obsahu"
    echo "→ Neumím číst JS stránky"
    echo "→ Zkopíruj text inzerátu ručně a vlož ho sem."
    exit 1
fi

# Clean HTML: Remove URLs, images, and HTML tags
CLEANED=$(echo "$CONTENT" | \
    sed 's|https\?://[^[:space:]]*||g' | \
    sed 's|<img[^>]*>||g' | \
    sed 's|<[^>]*>||g' | \
    sed 's|&nbsp;| |g' | \
    sed 's|&quot;|"|g' | \
    sed 's|&amp;|\&|g' | \
    sed 's|&lt;|<|g' | \
    sed 's|&gt;|>|g' | \
    tr -s '[:space:]' '\n' | \
    sed '/^$/d')

# Validate cleaned content
WORD_COUNT=$(echo "$CLEANED" | wc -w)

if [ $WORD_COUNT -lt 50 ]; then
    echo "⚠️ VAROVÁNÍ: Vyčištěný text má jen $WORD_COUNT slov"
    echo "→ Obsah může být neúplný"
fi

echo "✅ Načteno $WORD_COUNT slov"
echo ""
echo "📄 OBSAH K VYHODNOCENÍ:"
echo "=========================================="
echo "$CLEANED"
echo "=========================================="
echo ""
echo "Pokračuji s vyhodnocením..."
```

#### HTTP Request Outcomes

**Success scenario:**
- HTTP returns content (>500 characters)
- HTML is cleaned (URLs, images, tags removed)
- Display cleaned content to user
- Continue to evaluation

**Error scenarios:**

1. **404 or timeout** (`STATUS -ne 0`)
   - Message: "❌ URL není dostupná (404 nebo timeout)"
   - Action: Request manual copy-paste
   - Stop execution

2. **Empty response** (`CONTENT_SIZE < 500`)
   - Message: "❌ Neumím číst JS stránky"
   - Reason: Page uses JavaScript for content loading
   - Action: Request manual copy-paste
   - Stop execution

3. **Low word count** (`WORD_COUNT < 50`)
   - Message: "⚠️ Obsah může být neúplný"
   - Action: Show warning but continue

---

### Step 3: Evaluation

Evaluate job ad against 6 criteria. Each criterion receives:
- **Score:** 0, 50, or 100 points
- **Evaluation:** Explanation in Czech (max 120 words for criteria 1-5, max 160 words for criterion 6)

---

## Output Format

Return pure JSON without markdown formatting:

```json
{
  "Eval1_score": 0,
  "Eval1_eval": "Czech text explaining score, max 120 words",
  "Eval2_score": 50,
  "Eval2_eval": "Czech text explaining score, max 120 words",
  "Eval3_score": 100,
  "Eval3_eval": "Czech text explaining score, max 120 words",
  "Eval4_score": 50,
  "Eval4_eval": "Czech text explaining score, max 120 words",
  "Eval5_score": 100,
  "Eval5_eval": "Czech text explaining score, max 120 words",
  "Eval6_score": 50,
  "Eval6_eval": "Czech text explaining score, max 160 words"
}
```

**Format requirements:**
- Pure JSON only (no ```json wrapper, no markdown, no emoji in values)
- Scores must be exactly 0, 50, or 100 (no other values)
- All evaluations in Czech language
- Word limits strictly enforced

---

## Scoring System

All criteria use the same 3-level scoring:

### Score: 0 - Insufficient Level
Critical issues present: missing information, vague wording, or misleading content that harms candidate experience.

### Score: 50 - Adequate Level
Basic standards met but content is generic, lacks differentiation, or missing compelling details.

### Score: 100 - Excellent Level
Specific, concrete, candidate-centric content with unique value proposition clearly communicated.

---

## Evaluation Criteria

### Criterion 1: Job Title

**What to evaluate:**  
Clarity, searchability, and attractiveness of position name.

**Score 0 indicators:**
- Uses unclear creative terms (ninja, guru, rockstar)
- Confusing English jargon or technical codes
- Poor searchability
- Doesn't clearly communicate what the job is

**Score 50 indicators:**
- Title is clear and understandable
- But too generic
- Lacks differentiating details
- Doesn't stand out from competition

**Score 100 indicators:**
- Clear and understandable title
- Contains interesting details (technology, shifts, salary, benefits)
- Effectively communicates position value
- Increases probability of attracting relevant candidates

**Evaluation focus:**
- Does title tell candidates WHAT the job actually is?
- Does it include hook details (tech stack, benefits, location)?
- Is it searchable and competitive in the market?

---

### Criterion 2: Company & Position Description

**What to evaluate:**  
Quality of company introduction, role explanation, and position requirements.

**Score 0 indicators:**
- Only generic phrases ("dynamic company", "market leader")
- No concrete facts about size, focus, or achievements
- Vague job description ("various admin tasks", "client care")
- Missing specific responsibilities, daily tasks, or expected results

**Score 50 indicators:**
- Basic company information (focus, size)
- List of main tasks and responsibilities
- Lacks details about specific projects, team, or expected results
- Generic and interchangeable description
- Says what everyone does in similar roles, not what's unique here
- Missing deeper insight into what actually interests candidates

**Score 100 indicators:**
- Specific, relevant company description
- Clear picture of why working here is good
- Detailed job responsibilities explained
- Technologies, tools, and specific projects mentioned
- Clearly explains what makes this position unique and better than elsewhere
- Candidate has clear understanding and received highly relevant information

**Evaluation focus:**
- Does candidate understand what the company does?
- Is role description specific or could fit any similar job?
- Are technologies, tools, and projects mentioned?
- Does it answer "Why this job, not others?"

---

### Criterion 3: Compensation & Benefits

**What to evaluate:**  
Concreteness and attractiveness of salary and benefits information.

**Score 0 indicators:**
- No salary information provided
- Vague phrases ("attractive compensation", "motivating salary")
- Benefits mentioned only generically ("standard benefits")
- Empty phrases ("pleasant work environment")
- No concrete information for candidate decision-making

**Score 50 indicators:**
- Missing specific monthly salary figure
- Bonuses and premiums mentioned without specific amounts
- List of common benefits (meal vouchers, extra vacation week, pension)
- Candidate has basic picture
- Nothing truly interesting or better than standard

**Score 100 indicators:**
- Specific monthly salary or salary range stated
- Bonus components explained (premiums, bonuses, allowances with amounts)
- Each benefit concrete with precise amounts (meal vouchers 150 CZK/day, MultiSport fully paid)
- Unique or above-standard benefits (medical care, company daycare, sports facilities, provided transportation)

**Evaluation focus:**
- Can candidate calculate monthly take-home pay?
- Are benefits specific with amounts and frequency?
- Is there something unique or better than market standard?

---

### Criterion 4: Time Flexibility

**What to evaluate:**  
Information about working hours, shifts, and flexibility options.

**Score 0 indicators:**
- No working hours information
- Vague terms ("flexible working hours", "shift work")
- Missing contract type information
- No schedule or home office information
- Candidate has no idea about work week structure

**Score 50 indicators:**
- Basic information present (e.g., "8.5 hours daily", "three-shift operation")
- Missing details about shift start/end times
- No information about flexibility in arrivals/departures
- Missing home office rules
- Candidate has rough picture
- Standard flexibility, nothing outstanding

**Score 100 indicators:**
- Detailed working hours description including exact shift breakdown
- Flexible arrival and departure options
- Home office rules specified (e.g., "3 days remote, 2 days office")
- Shift planning system explained
- Candidate has precise picture of work-life balance
- Company maximizes flexibility
- Above-standard options (shortened shifts, sick days, sabbatical)

**Evaluation focus:**
- Does candidate know their weekly schedule?
- Is flexibility real or just a buzzword?
- Are home office rules clear and specific?

---

### Criterion 5: Tone of Voice & Structure

**What to evaluate:**  
Writing style, text structure, and candidate-centric approach.

**Score 0 indicators:**
- Written from company perspective (excessive "we", "our company")
- Candidate described impersonally in 3rd person
- Full of clichés ("dynamic firm", "young team")
- Inappropriate jargon or excessive formality
- Contains typos or grammatical errors
- Poor structure with long unclear paragraphs
- Chaotic formatting
- Written in single gender without (m/f) notation

**Score 50 indicators:**
- Alternates company perspective ("we") and candidate perspective ("you")
- Company view still dominates
- Avoids biggest clichés
- Standard industry language, some generic parts
- Grammatically correct text
- Basic structure with simple formatting
- Feels neutral or impersonal

**Score 100 indicators:**
- Written primarily from candidate perspective
- Company presents itself through what it offers candidate
- Specific examples instead of clichés
- Appropriate technical terminology
- Every claim supported by specific details
- Perfectly crafted text
- Clear structure with effective formatting
- Professional yet friendly tone
- Candidate feels personally addressed and respected

**Evaluation focus:**
- Is it about "we need" or "you'll get"?
- Are claims backed by specifics or just buzzwords?
- Is text easy to scan and pleasant to read?
- Does it respect candidates (gender-neutral language)?

---

### Criterion 6: Candidate Feedback (Overall Impression)

**What to evaluate:**  
Total impression from candidate's perspective for THIS specific position.

**Evaluation instructions:**
1. Read entire job ad thoroughly
2. React ONLY to information present in ad (don't invent anything)
3. Imagine candidate persona exactly for this position
4. Think about what matters to candidates when changing jobs for such position
5. View ad through candidate's eyes
6. Focus on:
   - **Job description** - Is it specific enough?
   - **Value proposition** - Is it clear why the offer is interesting?
   - **Company offering** - Is it concrete and relevant?
   - Candidates want concrete and valuable information
   - HR clichés don't interest candidates
   - Vague wording discourages rather than attracts

**Score 0 indicators:**
- Ad didn't attract candidate
- Very generic content
- Doesn't differ from average market offering

**Score 50 indicators:**
- Interesting job ad
- Addresses most important candidate concerns
- Above-average ad

**Score 100 indicators:**
- Outstanding job ad
- Speaks directly to candidate
- Addresses exactly what matters
- Candidate wants to read through and apply
- Clear picture of why company and position are better than market alternatives

**Evaluation focus:**
- Would YOU apply based on this ad?
- Does it answer "Why should I choose YOU over others?"
- Is there enough information to make informed decision?

---

## Evaluation Rules

### Must Do:

- Evaluate each criterion independently (don't cross-reference other criteria)
- Quote specific examples from the job ad to justify scores
- Write all evaluations in Czech language
- Stay within word limits (120 words for criteria 1-5, 160 words for criterion 6)
- Use only valid scores: 0, 50, or 100
- Base evaluation ONLY on information actually present in the job ad

### Must NOT Do:

- Use markdown formatting in JSON output (no ```json blocks)
- Add emoji characters in JSON values
- Invent or assume information not present in the ad
- Mention other criteria when evaluating one specific criterion
- Exceed specified word limits
- Give scores other than 0, 50, or 100
- Evaluate if content is empty or failed to load

---

## Common Mistakes to Avoid

### During Scraping:

- ❌ Not checking if HTTP request succeeded before cleaning
- ❌ Attempting to evaluate empty or failed content
- ❌ Not informing user about JavaScript-rendered pages
- ❌ Missing user feedback about scraping status

### During Evaluation:

- ❌ Using markdown formatting in output
- ❌ Exceeding word limits in evaluations
- ❌ Giving invalid scores (not 0, 50, or 100)
- ❌ Making up information not in the ad
- ❌ Writing too generic explanations without specific examples from ad
- ❌ Cross-referencing other criteria
- ❌ Including emoji in JSON values

### Correct Approach:

**Check → Inform → Act**

1. After each step, verify the result
2. Inform user what happened (success/failure/warning)
3. If something fails, offer clear solution
4. Only proceed to evaluation when content is valid

---

## Technical Notes

### Bash Tools Used:

- `curl -s -L --max-time 30` - HTTP request with follow redirects and timeout
- `sed` - Text cleaning (remove URLs, HTML tags, decode entities)
- `wc -w` - Word count for validation
- `tr -s '[:space:]' '\n'` - Normalize whitespace

### Not Used (Forbidden):

- Apify, Make, Python scripts
- External APIs or services
- lynx, html2text (complex HTML parsers)
- File storage or persistence
- Create own files

### Error Codes:

- `STATUS -ne 0` - HTTP request failed (404, timeout, connection error)
- `CONTENT_SIZE < 500` - Response too small (likely JavaScript-rendered)
- `WORD_COUNT < 50` - Cleaned content insufficient (warning, but can continue)

