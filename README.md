# AI Article Agent

Every day, this agent automatically:
1. Pulls fresh news links per topic from Google News RSS (free, no API key)
2. Extracts the real article text from each link
3. Has a free LLM (Groq) write an original ~500-word article synthesizing the sources
4. Saves each article as Markdown under `output/YYYY-MM-DD/topic-name.md`
5. Commits the new files back to this repo — so you just check the `output/` folder each morning

Total cost: **$0**. GitHub Actions' free tier covers this easily (a few minutes/day), and Groq's free tier covers typical daily usage for a handful of topics.

## One-time setup (10 minutes)

### 1. Get a free Groq API key
- Go to https://console.groq.com/keys and sign up (free)
- Create an API key

### 2. Create a GitHub repo
- Push this folder to a new **private** GitHub repo (recommended, since it's your content)

### 3. Add your API key as a secret
- In your repo: **Settings → Secrets and variables → Actions → New repository secret**
- Name: `GROQ_API_KEY`
- Value: paste your key

### 4. Edit `topics.yaml`
Add/remove topics, tweak the search query per topic, and set word count. This is the only file you'll need to touch day-to-day.

### 5. Done
The workflow runs automatically every day at 07:00 UTC (edit the `cron` line in `.github/workflows/daily.yml` to change the time — use https://crontab.guru to build a schedule).

You can also trigger it manually anytime: go to the **Actions** tab → **Daily Article Generation** → **Run workflow**.

## Running it locally (optional, to test before relying on the schedule)

```bash
pip install -r requirements.txt
export GROQ_API_KEY=your_key_here
python src/main.py
```

Articles land in `output/<today's date>/`.

## Notes & things worth knowing

- **Sourcing quality**: Google News RSS is free and reliable but occasionally returns a paywalled or blocked article — the scraper just skips those and moves on, so some days a topic might get fewer sources than usual.
- **Originality**: the prompt in `src/writer.py` explicitly instructs the model to paraphrase rather than copy source text, and to cite sources at the bottom of each article rather than quote them at length. Worth spot-checking the output occasionally, especially if you plan to publish rather than just read it yourself.
- **Swapping the LLM**: if you'd rather use Google Gemini's free tier or something else, only `src/writer.py` needs to change — everything else is provider-agnostic.
- **Scaling up**: more topics or more sources per topic = more Groq API calls. Free tier limits are generous for a handful of daily topics; check https://console.groq.com/settings/limits if you add a lot more.
