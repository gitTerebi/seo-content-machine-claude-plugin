---
name: scm
description: Use SEO Content Machine (SCM) through its MCP tools to plan pages in an SEO Workspace and have SCM research, write and review them as drafts, or to write AI pages from page templates for a list of keywords. Use when the user mentions SCM or SEO Content Machine, or asks to plan or write a batch of SEO pages with it.
---

# SEO Content Machine

SEO Content Machine (SCM) is a desktop app. Its MCP tools plan pages in an SEO Workspace and run Page Pipeline tasks inside the app, which research, write and review each page on the user's machine. Drafts land in the SEO Workspace, and the user reviews and publishes them in the app.

## Before the first call

The app must be open. If no SCM tools are available, or a tool call fails because SCM cannot be reached, ask the user to start SEO Content Machine and log in, then run `/mcp` and reconnect the `scm` server. Do not retry in a loop.

Call `task_list` first in a new session. It shows the tasks that already exist and how busy the machine is.

## How a task runs

`seo_workspace_create` and `page_pipeline_upsert` only make a draft task. The pattern is always:

1. Create it. The reply has the task id.
2. Start it with `task_run` and that id.
3. Check progress with `task_list` filtered by the task's `name`. It returns the status and run counts without the task's settings. Call `task_get` with `include_logs: true` only to read the log, once the run ends or when it looks stuck or failed, because `task_get` also returns every setting.
4. For pages written from an SEO Workspace, read the drafts with `workspace_query` on `documents`. For a Page Pipeline run on a keyword list, call `task_results`. Its rows point to the files the run wrote, and `run.output_folder` is the output folder. Read those files with your own file tools.

Runs take from seconds to many minutes. Poll `task_list` every 20 to 60 seconds rather than in a tight loop, and tell the user what is happening while you wait.

`task_run` resumes an unfinished run. `task_run` with `fromStart: true` starts over from the first input line. `task_cancel` stops a run.

## Which tool for which job

| Job | Tool |
| --- | --- |
| Plan pages: the brief, the page list and the site plan | `seo_workspace_create`, `workspace_write` |
| Read a Workspace's pages, site plan, documents or imported Search Console data | `workspace_query` |
| Write pages already planned in an SEO Workspace | `workspace_create_pages` |
| AI pages from page templates for a list of keywords | `page_pipeline_upsert` |
| Fix exact text in one draft | `workspace_documents_edit` |
| Follow, resume, stop or delete a run | `task_list`, `task_get`, `task_run`, `task_cancel`, `task_delete` |

Output folders accept `%project_folder%`, which is the task's own project folder.

## SEO Workspace pages

An SEO Workspace is a task of type "seo workspace". Find its id with `task_list` and `type: "seo workspace"`, or make one with `seo_workspace_create`. That id is the `task_id` the workspace tools take.

- `workspace_query` reads one collection, such as `pages`, `sitePlan`, `brief` or `documents`, with a Mongo filter. Pass `fields` to keep the reply short.
- `workspace_write` inserts or updates rows straight away, with no confirmation step. Show the user the list of pages and get a yes before you add them. To update exact pages, filter with `{"pageId": {"$in": [...]}}`. Page and Site plan changes are journaled, and the user can undo them in the app.
- `workspace_create_pages` hands pages to a Page Pipeline, which researches, writes and reviews one page each and saves it as a draft document. Pass `run: false` to set the model and effort with `page_pipeline_upsert` before `task_run`. Use effort "medium" or higher, because on low the writer stops researching.
- `workspace_documents_edit` replaces exact text in one document's body.

For local service pages across towns, follow the `local-seo` skill.

The user reviews and publishes pages in the app. Do not describe a page as published because its status changed.

## Care

- `task_delete` removes the task folder, including its results and log. Confirm with the user first.
- AI images in a Page Pipeline cost SCM AI credits (5k per image on the scm service). Mention the cost before turning them on.
- AI pages use the user's own AI service, roughly 75k to 300k tokens a page. Mention it before a large batch.
- Do not start many large tasks at once. `task_list` shows the machine's CPU, RAM and task load.
