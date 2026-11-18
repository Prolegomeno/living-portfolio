# Living Portfolio

An n8n automation that keeps a WordPress portfolio up to date by itself.

You tag a workflow with `Portfolio`. A few hours later there is a page about it on your site, with diagrams, written in your voice. Change the workflow and the page rewrites itself. Remove the tag and the page goes back to draft.

There is no human step in the normal flow.

## How it works

1. **Sync** lists every workflow tagged `Portfolio` and compares it with a table of what is already published. Every project is new, unchanged, changed or removed.
2. A project can call sub-workflows. Tag those `Portfolio Part` and they are read as part of the project. The page then shows the whole machine as one diagram.
3. **Process Project** reads the workflows, keeps only a short list of safe facts (the fact sheet), and sends them through the safety check.
4. **Safety Check** has two layers. Fixed patterns remove keys, tokens, passwords and email addresses. Then a reviewing model with no tools lists personal or confidential text that patterns cannot see, and that text is removed.
5. A writing model describes the project in your voice, using samples of your own writing. Its answer is checked and every piece of text is escaped before it goes into the page.
6. The page is created or updated in WordPress, under a parent page called "Automations".
7. **Unpublish** sends the page back to draft when a project loses its tag.
8. **Notify Error** emails you when something fails.

Sync runs every 6 hours. You can also start it by hand.

## Security rules

- The fact sheet is built from an allowlist. Everything not named is dropped: credentials, pinned data, static data, webhook identifiers, shared settings and identifiers.
- Sticky notes and prompts are untrusted free text. They always go through the safety check.
- Nothing reaches the writing model or the website without passing the safety check. Callers use the cleaned copy, never the original.
- Fail closed. If the safety check or the writer fails, the project is not published, its status is `blocked`, and it is tried again when the workflow changes.
- The diagram component only ever receives a small workflow built from the cleaned fact sheet, never the real workflow.
- The safety check is a best effort, not a guarantee. Do not tag a workflow that holds secrets you cannot afford to lose.

## What you need

- A self-hosted n8n version 2 or later, with Data Tables.
- A WordPress site and an application password for a dedicated user. An Editor role works. It has to be allowed to save scripts, because the diagrams need one.
- An OpenAI credential in n8n.
- Some samples of your own writing, so the model can learn your voice.

## Setup

1. Import the five files from `workflows/`.
2. Create two Data Tables.
   - `Living Portfolio Publicated Workflows` with the text columns `workflowId`, `wordpressId`, `lastUpdatedAt`, `status` and `name`.
   - `Living Portfolio Writing Samples` with one text column `samples`. Add a few rows of your own writing.
3. Create the credentials: n8n (to read your workflows), WordPress, OpenAI, and one to send email.
4. In WordPress, create a page called "Automations". Use `docs/automations-page.html` as a starting point. Note its page number.
5. Open each workflow and fix what imports cannot carry over:
   - In Sync, choose the sub-workflow again in `Create Project Page`, `Update Project Page` and `Unpublish Project Page`.
   - In Process Project, choose the sub-workflow again in `Check Safety`, and pick the two Data Tables.
   - In Process Project, change the site address in `Set Page Excerpt`, and the number `PARENT_PAGE_ID` in `Choose Page Action`.
   - In the two model prompts, change the author name.
   - In Notify Error, change the email address.
6. Tag `Process Project`, `Unpublish` and `Utility - Safety Check` with `Portfolio Part`. Do not also tag them `Portfolio`.
7. In every workflow, set `Utility - Notify Error` as the Error Workflow, and publish it.
8. In `Choose Page Action`, `PAGE_STATUS` starts as `publish`. Set it to `draft` while you are testing.
9. Tag one workflow `Portfolio` and run Sync by hand.

## Linking a project to its code

To add a "Get the code" link to a project page, put one plain line in a sticky note of the project's main workflow:

```
GitHub: https://github.com/your-name/repository-name
```

Only that exact shape is accepted: `https://github.com/` followed by a name and a repository. Anything else is ignored. A project without the line simply has no link, and the tag is still the only thing a workflow needs.

## Good to know

- The diagrams use the public n8n-demo component, loaded from jsDelivr.
- Every save of a workflow counts as a change, so a long editing session means more rewrites and more model calls.
- The tone is set in the writer prompt in `Process Project`. Change it to taste.

## Licence

MIT. See `LICENSE`.
