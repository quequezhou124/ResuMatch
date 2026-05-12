# ResuMatch

ResuMatch is a web app that helps job seekers turn a single profile into tailored resumes, cover letters, and form-ready JSON. Upload a resume and a job description, then review structured experience, matched job evidence, and copy-ready outputs in a private, human-first workflow.

## Getting Started

1. Install deps:

```bash
npm install
```

2. Configure environment:

```
DATABASE_URL=postgresql://USER:PASSWORD@HOST:PORT/DB?sslmode=verify-full
DB_URL_PSQL=postgresql://USER:PASSWORD@HOST:PORT/DB?sslmode=verify-full
OPENAI_API_KEY=your_key_here
PROFILE_PARSER_STRATEGY=llm
OPENAI_BASE_URL=https://newapi.houdao.com
OPENAI_MODEL=gpt-4.1-mini
OPENAI_API_KEY is required when PROFILE_PARSER_STRATEGY=llm.
If PROFILE_PARSER_STRATEGY is unset, it defaults to llm in development and heuristic in production.
DB_URL_PSQL is only used for psql migrations because psql rejects uselibpqcompat.
```

3. Apply schema:

```bash
npm run db:migrate
```

4. Start the dev server:

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) with your browser to see the result.

## Profile ingestion

- Upload a PDF resume on `/app/profile` to extract text and merge into the canonical profile.
- Manual edits (education/work/projects/skills) merge into the same profile without overwriting other sections.
- Each update creates a version snapshot in `profile_versions`.

## JD sentence matching

- Sentence extraction: the app converts the rendered job brief into canonical `JobSentence` rows (section + exact sentence text), including responsibilities and skills lists.
- Similarity: each resume evidence bullet is compared against each `JobSentence` using token-frequency cosine similarity in `src/lib/job-matcher.ts`.
- Threshold and top-k: matches are kept when boosted similarity is `>= 0.45` (default), and each evidence item keeps top `K=2` JD sentences.
- Fallback + phrase boost: long JD sentences can still match on strong token overlap, and shared 2-3 word phrases increase similarity.
- UI mapping: the left panel highlights matched JD sentences; the right panel shows exact matched JD sentences under each evidence item.
- Tuning:
  - Lower threshold (`0.35-0.45`) increases recall.
  - Higher threshold (`0.50-0.65`) increases precision.
  - Increase `topK` if you want more than two JD sentence links per evidence block.

## Job-tailored profile rendering

- The job detail "Resume bullets" panel now reuses the same structured profile sections (Education, Skills, Experience, Projects, Highlights) as the profile view.
- Skills and highlights are reordered with optional `jobContext` keywords; non-matching items are kept and order is stable for ties.
- Copy behavior:
  - click chips or values to copy individual content
  - each section has a copy control
  - panel-level copy generates a clean structured text block from the same view model

### Manual test checklist

- Upload a resume PDF → contact/skills populate.
- Upload a PDF with `PROFILE_PARSER_STRATEGY=llm` → education/work/projects parsed.
- Force LLM failure (unset OPENAI_API_KEY) → warning banner "needs review" appears.
- Click upload without selecting a file → error banner, no crash.
- Upload a file >10MB → error banner, no crash.
- Add a Project manually → existing education/work remains.
- Upload a second PDF → profile merges, nothing is deleted.
- Delete an entry via UI → only that entry is removed.
- `/api/profile/versions` shows recent snapshots.
- Click any copyable box (skills lines, file name, entry copy button) → clipboard updates and toast appears.
- Click the copy icon inside any input/textarea on /app/profile → value copied.

## Learn More

To learn more about Next.js, take a look at the following resources:

- [Next.js Documentation](https://nextjs.org/docs) - learn about Next.js features and API.
- [Learn Next.js](https://nextjs.org/learn) - an interactive Next.js tutorial.

You can check out [the Next.js GitHub repository](https://github.com/vercel/next.js).

## Deploy on Vercel

The easiest way to deploy your Next.js app is to use the [Vercel Platform](https://vercel.com/new?utm_medium=default-template&filter=next.js&utm_source=create-next-app&utm_campaign=create-next-app-readme) from the creators of Next.js.

Check out our [Next.js deployment documentation](https://nextjs.org/docs/app/building-your-application/deploying) for more details.
# ResuMatch
