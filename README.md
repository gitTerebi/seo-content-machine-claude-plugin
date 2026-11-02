# SEO Content Machine plugin for Claude Code

Lets Claude Code plan SEO pages with [SEO Content Machine](https://seocontentmachine.com/?utm_source=claude-plugin&utm_medium=plugin&utm_campaign=claude-directory&utm_content=readme) (SCM), the desktop SEO workspace, and have SCM write them:

- plan pages in an SEO Workspace: the brief, the page list and the site plan of hubs and the pages under them
- read the Workspace's pages, site plan and the Search Console data the user imported in the app
- hand a batch of pages to a Page Pipeline, which researches, writes and reviews each page and saves it as a draft
- follow the run, read the drafts and their review scores, and resume or stop a run

The user reviews and publishes the drafts in the SCM app.

The plugin adds SCM's MCP server and two skills: `scm`, which shows Claude how to use the tools, and `local-seo`, which plans and writes local service pages for every service in every town a business covers.

## How SCM writes each page

Each page in a Page Pipeline gets its own page agent, run on the AI service the user set up in SCM. The agent follows a page skill: a SKILL.md file for that page type, which says what the page must cover and the form it takes. SCM ships page skills for 10 page types (Article, Topic Definition, Topic How To, Topic Guide, Topic Troubleshoot, Topic Comparison, Topic Listicle, Topic Review, Local SEO Service and Local SEO Hub), plus a refresh skill for pages already live. Users can add their own page skills as folders in SCM's skills folder.

The page agent works through three sub-agents, each defined in its own agent file:

1. **Research.** A page-researcher builds a SERP model of the reader's search: one entry for every first-page result in rank order, then a comparison of what they cover and miss. It reads each fact at the source that owns it and writes it to `facts.md`, and it lists the facts no source publishes as gaps. Facts about the business come only from the Workspace brief and the page's notes.
2. **Writing.** A page-writer writes the page from the writing brief, the SERP model and the facts, in the form the page skill gives.
3. **Review.** A page-reviewer searches what ranks, opens the pages behind the page's facts and links, and marks each fail as blocking or minor. A blocking fail is a claim no opened page states, a claim about the business the brief does not back, or skipped research. It scores the page out of 10 on Help, Specifics, Honest, Expertise and Focus. The writer makes the fixes and the same reviewer rescores, for up to 4 rounds, until no blocking fail is left and every score is 8 or more.

Before a page is kept, SCM checks its HTML against the page skill's lint rules, such as the length of the H1. When a fix needs a fact only the business knows, the agent saves the question in the page's notes in the SEO Workspace for the user to answer, instead of guessing.

The page agent and its sub-agents run in SCM's own agent engine, not in Claude Code. They work on any AI service in SCM that can call tools: a Claude Code or Codex subscription, an API key for OpenAI, Claude, Gemini, DeepSeek, Groq, OpenRouter, Z.ai or Azure, or a local model in LM Studio. Claude Code hands SCM the pages and reads the drafts back, so a batch of 50 pages does not fill its context.

Each page keeps its own folder with its SERP model, facts, research log and review. A stopped run resumes where each page left off, and the run log records every page's review rounds, scores and token use.

## Requirements

- SEO Content Machine running on the same machine, on Windows, Mac or Linux. SCM is a paid app with a [14-day free trial](https://seocontentmachine.com/members/signup/trial?utm_source=claude-plugin&utm_medium=plugin&utm_campaign=claude-directory&utm_content=readme_trial), no card needed.
- SCM's Api secret key (Settings > App) left blank, which is the default.
- Node.js 18 or newer.

## Install

In Claude Code, type one line:

```
/plugin install seo-content-machine --marketplace gitTerebi/seo-content-machine-claude-plugin
```

Claude Code shows the source and asks once before it adds the marketplace. On Claude Code older than 2.1.275, use the two-step form:

```
/plugin marketplace add gitTerebi/seo-content-machine-claude-plugin
/plugin install seo-content-machine@seo-content-machine
```

From a terminal, the same install is `claude plugin install seo-content-machine --marketplace gitTerebi/seo-content-machine-claude-plugin`.

The plugin works with SCM's default settings. It has one optional setting, which you can change in `/config`:

- **SCM API address**: only if SCM's API is not at `http://localhost:8008`.

You can open SCM before or after Claude Code. The SCM tools appear once SCM is running.

## Use

Open SCM, then ask Claude Code for the job, for example:

- "Add these 10 page ideas to my SEO Workspace and put them under the right hubs"
- "Write the planned pages in my SEO Workspace, 5 at a time"
- "Which of my Workspace pages get Search Console impressions but have no draft yet?"
- "Use the local-seo skill to plan service pages for my plumbing business in Austin and its suburbs"

## What the plugin runs and sends

The MCP server is one file, [mcp/index.mjs](mcp/index.mjs), with no dependencies. Claude Code runs it with Node. It reads messages from Claude Code on stdin and posts each one to SCM's API at the SCM API address, `http://localhost:8008/mcp` by default. It sends nothing anywhere else.

While SCM is closed, the server checks that address every 5 seconds and tells Claude Code to list the tools again once SCM answers.

The work itself runs inside SCM on your machine. A Page Pipeline researches each page's topic on the web, writes the page with the AI service the user set up in SCM, and saves the draft in the SEO Workspace.

## License

MIT
