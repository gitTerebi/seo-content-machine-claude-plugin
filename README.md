# SEO Content Machine plugin for Claude Code

Lets Claude Code plan SEO pages with [SEO Content Machine](https://seocontentmachine.com) (SCM), the desktop SEO workspace, and have SCM write them:

- plan pages in an SEO Workspace: the brief, the page list and the site plan of hubs and the pages under them
- read the Workspace's pages, site plan and the Search Console data the user imported in the app
- hand a batch of pages to a Page Pipeline, which researches, writes and reviews each page and saves it as a draft
- follow the run, read the drafts and their review scores, and resume or stop a run

The user reviews and publishes the drafts in the SCM app.

The plugin adds SCM's MCP server with those tools only, and two skills: `scm`, which shows Claude how to use the tools, and `local-seo`, which plans and writes local service pages for every service in every town a business covers.

## Requirements

- SEO Content Machine running on the same machine, on Windows, Mac or Linux. SCM is a paid app with a [14-day free trial](https://seocontentmachine.com/members/signup/trial), no card needed.
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

The plugin works with SCM's default settings. It has two optional settings, which you can change in `/config`:

- **SCM API key**: only if you set one in SCM under Settings > App > Api secret key.
- **SCM API address**: only if SCM's API is not at `http://localhost:8008`.

You can open SCM before or after Claude Code. The SCM tools appear once SCM is running.

## Use

Open SCM, then ask Claude Code for the job, for example:

- "Add these 10 page ideas to my SEO Workspace and put them under the right hubs"
- "Write the planned pages in my SEO Workspace, 5 at a time"
- "Which of my Workspace pages get Search Console impressions but have no draft yet?"
- "Use the local-seo skill to plan service pages for my plumbing business in Austin and its suburbs"

## What the plugin runs and sends

The MCP server is one file, [mcp/index.mjs](mcp/index.mjs), with no dependencies. Claude Code runs it with Node. It reads messages from Claude Code on stdin and posts each one to SCM's API at the SCM API address, `http://localhost:8008/mcp` by default, with the API key as a Bearer header when you set one. It sends nothing anywhere else.

While SCM is closed, the server checks that address every 5 seconds and tells Claude Code to list the tools again once SCM answers.

The plugin starts the server with `SCM_TOOLSET=pages`, so it lists and accepts only the SEO Workspace, Page Pipeline and task tools. It refuses a call to any other SCM tool.

The work itself runs inside SCM on your machine. A Page Pipeline researches each page's topic on the web, writes the page with the AI service the user set up in SCM, and saves the draft in the SEO Workspace.

The same server is on npm as [seo-content-machine-mcp](https://www.npmjs.com/package/seo-content-machine-mcp) for other MCP clients.

## License

MIT
