**[Live Demo](https://YOUR_USERNAME.github.io/ai-resume-generator)** · **[Report a Bug](https://github.com/YOUR_USERNAME/ai-resume-generator/issues)**

---

## ✨ Features

- **Resume Generator** — Full resume with professional summary, experience bullets, education, skills, projects, and certifications
- **Cover Letter Generator** — Role-targeted cover letters with tailored opening, achievements, company-specific motivation, and closing
- **Three writing tones** — Professional, Creative, Executive
- **Formatted HTML preview** — See a clean, printable resume rendered in-browser
- **Plain text export** — ATS-safe text you can paste anywhere
- **Print / PDF** — One-click print-to-PDF for submission
- **Dynamic form** — Add/remove work experiences and education entries

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    CLIENT BROWSER                        │
│                                                         │
│  ┌──────────────┐    ┌─────────────────────────────┐   │
│  │  Form UI     │───▶│   Prompt Builder (JS)        │   │
│  │  (HTML/CSS)  │    │   - Collects form fields     │   │
│  │              │    │   - Structures data as JSON   │   │
│  │  • Personal  │    │   - Injects tone instruction  │   │
│  │  • Experience│    └────────────┬────────────────-─┘   │
│  │  • Education │                 │                      │
│  │  • Skills    │                 ▼                      │
│  │  • Projects  │    ┌─────────────────────────────┐   │
│  │  • Certs     │    │  fetch() → Anthropic API     │   │
│  └──────────────┘    │  POST /v1/messages           │   │
│                      │  Model: claude-sonnet-4-6    │   │
│                      └────────────┬─────────────────┘   │
│                                   │                      │
│                                   ▼                      │
│                      ┌─────────────────────────────┐   │
│                      │  Response Handler            │   │
│                      │  - Parse JSON (resume)       │   │
│                      │  - Render HTML preview       │   │
│                      │  - Generate plain text       │   │
│                      │  - Enable copy/print         │   │
│                      └─────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

### Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Vanilla HTML5, CSS3, JavaScript (ES2020) |
| AI Model | Anthropic Claude Sonnet 4.6 |
| API | Anthropic Messages API (`/v1/messages`) |
| Fonts | Google Fonts (DM Serif Display + Inter) |
| Hosting | GitHub Pages / Netlify / Vercel (static) |

No backend. No database. No build step. One HTML file.

---

## 🔄 Workflow

### Resume Generation Flow

```
User fills form
      │
      ▼
collectResumeData()
  Gathers: name, email, phone, location, linkedin, website,
           summary, target role, experiences[], education[],
           skills[], projects, certifications, tone
      │
      ▼
Prompt Construction
  System: "You are an expert resume writer..."
  Instructions: tone guide, ATS optimization rules
  Data: JSON-serialized form fields
  Output spec: strict JSON schema
      │
      ▼
POST https://api.anthropic.com/v1/messages
  model: claude-sonnet-4-6
  max_tokens: 1000
  messages: [{ role: "user", content: <prompt> }]
      │
      ▼
Response Parsing
  Strip markdown fences → JSON.parse()
  Validate structure
      │
      ├──▶ renderResume(json)
      │     - HTML formatted preview
      │     - Print-friendly layout
      │
      └──▶ Plain text output
            - ATS-compatible format
            - Copy to clipboard
```

### Cover Letter Generation Flow

```
User fills form
      │
      ▼
collectCoverLetterData()
  Gathers: applicant info, job title, company,
           hiring manager, job description snippets,
           key achievements, why this company, tone
      │
      ▼
Prompt Construction
  4-paragraph structure:
    1. Strong opening hook
    2. Quantified achievements matching JD
    3. Company-specific motivation
    4. Confident call to action
      │
      ▼
POST /v1/messages → Plain text response
      │
      ▼
Render as formatted text in preview pane
```

---

## 🤖 Prompt Engineering

### Resume Prompt Strategy

The resume prompt uses **structured JSON output** to allow deterministic rendering:

```
You are an expert resume writer. Generate a polished, ATS-optimized 
resume in JSON format based on the following candidate data. 
Tone: [professional|creative|executive description].

Candidate data: { ...JSON... }

Return ONLY a JSON object with this exact structure:
{
  "name": "...",
  "contact": { ... },
  "summary": "2-3 sentence summary",
  "experience": [{ "title", "company", "period", "bullets": [] }],
  "education": [{ "degree", "school", "year", "note" }],
  "skills": [],
  "projects": [{ "name", "description" }],
  "certifications": []
}

Rules:
- Start bullets with strong past-tense action verbs
- Quantify achievements wherever data allows
- Keep summary specific to target role: "..."
- Enrich vague descriptions professionally
```

**Why JSON output?** It allows the frontend to render a properly formatted, styled HTML resume — not just a text blob. The model is instructed to return pure JSON with no markdown fences.

### Cover Letter Prompt Strategy

The cover letter uses **free-form prose output** with a 4-paragraph structure directive:

```
Paragraph 1: Opening hook — genuine interest + lead with value
Paragraph 2: 2-3 specific, quantified achievements matching JD requirements  
Paragraph 3: Why THIS specific company (uses applicant's "why" input)
Paragraph 4: Clear, confident call to action
```

The prompt injects actual company name, role title, and hiring manager name to eliminate placeholder text like `[Company Name]`.

### Tone System

| Tone | Instruction |
|------|------------|
| Professional | formal, achievement-oriented, clear and concise business language |
| Creative | engaging, personality-driven, vivid action verbs, slightly more expressive |
| Executive | strategic, high-level impact, leadership focus, board-room ready |

---

## 📁 Project Structure

```
ai-resume-generator/
│
├── index.html          # Complete application (single file)
│   ├── <style>         # All CSS — design tokens, layout, components
│   ├── Resume Form     # Personal info, experience, education, skills, tone
│   ├── Cover Letter    # Applicant info, job details, achievements, why
│   ├── Output Section  # Preview (HTML) + Raw text tabs, copy, print
│   └── <script>        # Form logic, API calls, rendering, utilities
│
└── README.md           # This file
```

---

## 🚀 Deployment

### Option 1: GitHub Pages (Free, recommended)

1. Fork this repository
2. Go to **Settings → Pages**
3. Set source to **Deploy from branch → main → / (root)**
4. Your app is live at `https://YOUR_USERNAME.github.io/ai-resume-generator`

> **Note:** The Anthropic API key is handled by Anthropic's infrastructure when calling from claude.ai artifacts. For standalone deployment outside of claude.ai, you'll need to add authentication — see the section below.

### Option 2: Netlify (Free tier)

```bash
# Drag and drop the folder at netlify.com/drop
# Or use the CLI:
npm install -g netlify-cli
netlify deploy --prod --dir=.
```

### Option 3: Vercel (Free tier)

```bash
npm install -g vercel
vercel --prod
```

### Option 4: Local Development

```bash
# No build step needed — just serve the HTML file
npx serve .
# or
python3 -m http.server 3000
# Then open http://localhost:3000
```

---

## 🔑 API Key Configuration (for standalone deployment)

When deploying outside of Anthropic's claude.ai environment, you need to supply your own API key. **Do not hardcode it in the HTML file.**

### Safe approach — environment variable via serverless function

Create a `/api/generate.js` serverless function (Netlify/Vercel):

```javascript
// api/generate.js (Netlify Functions / Vercel Serverless)
export default async function handler(req, res) {
  const response = await fetch("https://api.anthropic.com/v1/messages", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "x-api-key": process.env.ANTHROPIC_API_KEY,  // set in dashboard
      "anthropic-version": "2023-06-01"
    },
    body: JSON.stringify(req.body)
  });
  const data = await response.json();
  res.json(data);
}
```

Then update the fetch URL in `index.html`:
```javascript
// Change this:
const response = await fetch("https://api.anthropic.com/v1/messages", { ... });
// To this:
const response = await fetch("/api/generate", { ... });
```

Set `ANTHROPIC_API_KEY` in your Netlify/Vercel environment variable dashboard.

---

## 🔒 Security & Privacy

- **No data storage** — form data is processed only in the browser and sent directly to the Anthropic API; nothing is stored on any server
- **No authentication required** — the app runs entirely client-side when used within claude.ai
- **API key security** — for standalone deployments, always use server-side proxy functions, never expose API keys in client-side code
- **No cookies or tracking** — the application does not use analytics, cookies, or any tracking

---

## 🎨 Design Decisions

### Visual Identity
- **DM Serif Display** — editorial display typeface for headings; creates a premium, editorial feel appropriate for career documents
- **Inter** — neutral, highly legible body typeface used across the UI and resume preview
- **Warm cream palette** (`#faf9f6`) — distinct from stark white, creates a warmer, more inviting document feel
- **Gold accent** (`#b5811a`) — evokes prestige and professionalism; used for labels and highlights

### UX Choices
- **Single HTML file** — zero dependencies, no build toolchain, deploys anywhere
- **Tab-based navigation** — resume and cover letter share one page to reduce friction
- **Live chip inputs for skills** — more engaging and tactile than a textarea for skill lists
- **Dual output tabs** — formatted preview satisfies the visual desire; plain text serves practical ATS needs
- **Print-to-PDF** — opens a minimal print window with only the resume content

---

## 🛠️ Customization

### Adding a new tone

In the HTML, add a new radio button to the tone grid:
```html
<div>
  <input type="radio" name="r-tone" id="tone-academic" class="tone-option" value="academic" />
  <label for="tone-academic"><span class="tone-icon">🎓</span>Academic</label>
</div>
```

Add the tone description in the JS:
```javascript
const toneGuide = {
  // ...existing tones...
  academic: 'scholarly, research-focused, emphasizes publications and methodologies'
}[data.tone];
```

### Modifying the resume schema

Edit the JSON schema in the prompt string inside `generateResume()` and update the `renderResume()` function to handle the new fields.

### Adding a new section (e.g. Languages)

1. Add form fields in the HTML form section
2. Collect the data in `collectResumeData()`
3. Add to the prompt schema
4. Render in `renderResume()`

---

## 📝 Example Output

### Sample Resume JSON (from Claude)

```json
{
  "name": "Jane Smith",
  "contact": {
    "email": "jane@email.com",
    "phone": "+1 (555) 000-0000",
    "location": "San Francisco, CA",
    "linkedin": "linkedin.com/in/janesmith"
  },
  "summary": "Results-driven Software Engineer with 5+ years building scalable React and Node.js applications. Led cross-functional teams of 4–8 engineers and shipped products used by 50k+ users. Passionate about developer experience and systems that scale.",
  "experience": [
    {
      "title": "Senior Software Engineer",
      "company": "Acme Corp",
      "period": "Jan 2021 – Present",
      "bullets": [
        "Architected microservices migration reducing API latency by 40% and serving 2M daily requests",
        "Mentored 3 junior engineers, all promoted within 18 months",
        "Shipped real-time collaboration feature adopted by 90% of enterprise customers"
      ]
    }
  ],
  "skills": ["React", "Node.js", "TypeScript", "AWS", "PostgreSQL", "System Design"]
}
```

---

## 🤝 Contributing

Pull requests are welcome. For major changes, please open an issue first.

1. Fork the repo
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit changes: `git commit -m 'Add some feature'`
4. Push to branch: `git push origin feature/your-feature`
5. Open a Pull Request

---

## 📄 License

MIT License — see [LICENSE](LICENSE) for details.

---

## 🙏 Acknowledgments

- [Anthropic](https://anthropic.com) — Claude API
- [DM Serif Display](https://fonts.google.com/specimen/DM+Serif+Display) — Google Fonts
- [Inter](https://rsms.me/inter/) — Rasmus Andersson
