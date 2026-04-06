<!--=# INTEL° App — Full Debrief

## What the App Is

INTEL° is a single-page browser-based company intelligence platform. The product is designed to let a user:

* Enter a company name, startup idea, or business problem
* Choose a type of analysis
* Send that request to OpenRouter
* Receive a structured AI-generated report
* Optionally upload files for consulting-style analysis
* Store prior searches locally in browser history

The entire product currently lives in one HTML file with:

* HTML layout
* CSS styling
* JavaScript logic
* API integration
* Local storage persistence

There is no backend server. Everything runs client-side in the browser.

---

# Product Modes

The app currently exposes 6 analysis modes:

1. Company Brief
2. Should I Start This?
3. How to Monetize
4. Make It Better
5. Competitive Intel
6. AI Consultant

Each mode changes the system prompt sent to the AI.

## 1. Company Brief

Purpose:

* Research an existing company
* Return overview, business model, competitors, news, and strategic assessment

Typical Input:

* "Saudi Business Machines"
* "Stripe"
* "OpenAI"

Expected Output Sections:

* OVERVIEW
* BUSINESS MODEL
* KEY FACTS
* RECENT NEWS
* COMPETITORS
* STRATEGIC ASSESSMENT

## 2. Should I Start This?

Purpose:

* Evaluate a startup idea
* Decide whether the user should pursue it

Expected Sections:

* VERDICT
* MARKET OPPORTUNITY
* STARTUP FEASIBILITY
* REVENUE POTENTIAL
* RISKS & KILLERS
* FIRST 30 DAYS

## 3. How to Monetize

Purpose:

* Generate monetization ideas for a product or company

Expected Sections:

* CORE REVENUE MODEL
* ADDITIONAL REVENUE STREAMS
* PRICING STRATEGY
* GROWTH LEVERS
* MONETIZATION TIMELINE
* BENCHMARKS

## 4. Make It Better

Purpose:

* Critique and improve a company or business

Expected Sections:

* CURRENT STATE
* QUICK WINS
* STRATEGIC IMPROVEMENTS
* GROWTH OPPORTUNITIES
* TECHNOLOGY & PRODUCT
* KEY METRICS

## 5. Competitive Intel

Purpose:

* Compare a company against competitors

Expected Sections:

* COMPETITIVE LANDSCAPE
* HEAD-TO-HEAD
* COMPETITIVE ADVANTAGES
* VULNERABILITIES
* MARKET TRENDS
* STRATEGIC RECOMMENDATIONS

## 6. AI Consultant

Purpose:

* Analyze uploaded files and act like a management consultant

This is the only mode that:

* Reveals the file uploader
* Reads user-uploaded documents
* Includes document contents in the AI request

Expected Sections:

* EXECUTIVE SUMMARY
* DIAGNOSIS
* WHERE YOU ARE LOSING MONEY
* OPERATIONAL EFFICIENCY
* REVENUE GROWTH OPPORTUNITIES
* ORGANIZATIONAL RECOMMENDATIONS
* 90-DAY ACTION PLAN
* EXPECTED IMPACT

---

# Current Technical Architecture

## Frontend Stack

The app uses:

* Plain HTML
* Plain CSS
* Plain JavaScript
* localStorage
* fetch()
* FileReader API

No frameworks are used:

* No React
* No Next.js
* No backend
* No database
* No authentication system

This makes deployment easy:

* GitHub Pages
* Netlify
* Vercel static hosting

---

# How the App Works Internally

## Flow

1. User enters query
2. User selects analysis type
3. Optional file upload
4. App builds a system prompt
5. App sends request to OpenRouter
6. AI returns structured markdown-like response
7. App parses "##" headers
8. App renders sections into cards
9. Query stored in browser localStorage

---

# OpenRouter Integration

The app uses:

```js
POST https://openrouter.ai/api/v1/chat/completions
```

Headers:

```js
Authorization: Bearer <api key>
Content-Type: application/json
HTTP-Referer: current URL
X-Title: "INTEL Company Intelligence"
```

The request body contains:

```js
{
  model: "google/gemini-2.5-flash",
  messages: [...],
  tools: [{ type: "openrouter:web_search" }]
}
```

---

# Why the App Is Failing Right Now

The main issue is NOT the key.

The failure is caused by your request path.

Current request:

```js
model: "google/gemini-2.5-flash"
```

plus:

```js
tools: [{ type: "openrouter:web_search" }]
```

That combination is billable.

Even if:

* the account has a valid key
* the model appears cheap
* OpenRouter previously gave free trial credits

The request can still return:

```text
402 Payment Required
Out of credits
```

because:

* Web search costs credits
* Gemini routes may cost credits
* A negative balance blocks all requests
* Some free models still fail when the account is negative

---

# Main Weaknesses in the Current App

## 1. No Real Free Mode

Right now every search tries to use:

* A paid-capable model
* Optional web search

The app should instead support:

### Free Mode

* Free model only
* No web search
* Guaranteed no billing

### Research Mode

* Paid model
* Web search enabled
* Better answers

Without this split, users immediately hit billing errors.

---

## 2. No Model Selection

The user cannot choose:

* Free model
* Cheap model
* Fast model
* Premium model

You should add a selector like:

* Gemini Flash (cheap)
* Claude Sonnet (premium)
* DeepSeek Free
* Llama Free

---

## 3. No Billing-State Awareness

The app currently only says:

"Out of credits"

It should instead:

* Detect 402 errors
* Show a specific explanation
* Offer automatic fallback to free mode

Example:

"Research mode requires OpenRouter credits. Switching to Free Mode automatically..."

---

## 4. File Upload Is Too Weak

Current limitations:

* Reads only text-based files
* No PDFs
* No spreadsheets
* No Word docs
* No parsing for CSV or financial models

Users likely want to upload:

* PDF investor decks
* Excel P&L files
* Financial statements
* Annual reports

To make consultant mode truly useful, you need:

* PDF parsing
* XLSX parsing
* CSV parsing
* File size limits
* Better document chunking

---

## 5. No Streaming

The user waits silently until the entire answer returns.

Better:

* Stream the response token-by-token
* Render sections as they arrive
* Feels faster and more premium

---

## 6. No Persistence Beyond Browser

History is stored only in localStorage.

That means:

* Clearing browser cache erases history
* Switching devices loses everything
* No user accounts

---

# Current Strengths

Despite the problems, the app has several strong points:

## Strong Visual Design

The interface already looks premium:

* Dark theme
* Neon-accent business aesthetic
* Clear cards
* Clean typography
* Strong visual hierarchy

It already looks more polished than many startup MVPs.

## Strong Prompt Design

The system prompts are well-structured.

Why this matters:

* The AI usually returns organized sections
* Results feel more consulting-like
* Easier to render into cards

## Good Single-File Simplicity

Everything being in one file makes it:

* Easy to debug
* Easy to deploy
* Easy to share
* Easy to iterate rapidly

---

# Recommended Immediate Fixes

## Priority 1 — Make It Work Reliably

Change:

```js
var useSearch = false;
```

And change model:

```js
model: "deepseek/deepseek-chat-v3-0324:free"
```

This gives you a true free fallback mode.

---

## Priority 2 — Add Automatic Fallback

Pseudo logic:

```js
try paid model + web search
if 402 error:
    retry using free model
    disable search
```

This will make the app appear resilient.

---

## Priority 3 — Add Mode Toggle

UI toggle:

* Free Mode
* Research Mode

Free Mode:

```js
model = free model
useSearch = false
```

Research Mode:

```js
model = premium model
useSearch = true
```

---

# Best Future Version of This Product

A more complete version of INTEL° would look like this:

## Left Side

* Search
* Mode selector
* Uploaded files
* History

## Main Panel

* Streaming report
* Expandable sections
* Charts
* Competitor table
* Financial summary

## Right Side

* Sources used
* Citations
* Links
* Suggested next actions

Additional features:

* Export to PDF
* Save projects
* Share reports
* Team workspace
* Multiple company comparison
* Charts and graphs
* Auto-generated SWOT analysis
* Financial modeling

---

# Realistic Positioning

Right now the app is:

* More advanced than a toy project
* Strong enough for a portfolio piece
* Close to an MVP
* Not yet reliable enough for public users

The single biggest blocker is that billing behavior and fallback logic are unfinished.

Once you add:

* free mode
* better error handling
* fallback model routing

the app will become dramatically more usable.

---

# Bottom Line

The app is fundamentally good.

The UI, prompts, and structure are already strong.

The reason it feels broken is not because the concept is bad — it is because every request currently depends on a paid OpenRouter path.

The fastest path to making it usable is:

1. Add a true free model
2. Disable search in free mode
3. Retry automatically when 402 happens
4. Show the user which mode is active

Once you do that, the app will work consistently even without credits.
-->

