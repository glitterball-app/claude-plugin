---
name: glitterball
description: Add interactive video walkthroughs to a prototype with Glitterball, so you can record a guided walkthrough that plays over the live prototype and share it by link. Also reads what people said about a prototype (comments, answers to walkthrough questions) so you can fix it, shows who watched a walkthrough, and sends Glitterball notifications to Slack. Use when the user asks to add, connect or install Glitterball, wants to walk stakeholders through a prototype, or mentions Glitterball, the Glitterball tag, walkthroughs, prototype feedback or comments, who watched, or Glitterball in Slack.
---

# Add Glitterball to this prototype

Glitterball plays interactive video walkthroughs over a live prototype and lets people comment on it. A prototype gets it
like Vercel Analytics or Sentry: install the `glitterball` package with the project's own package manager, then
render it once where every page renders. It works the same with npm, pnpm, Yarn or Bun.

What it sends, so the user knows before you change anything: on page load it loads its script from
www.glitterball.dev and reads the prototype's list of walkthroughs; it records and uploads only after
someone presses Record on its bar, to the workspace of whoever signs in there. The token (`tt_…`) is a
public site key, like an analytics ID: it ships in the page, so it can go straight into the source.

## Steps

1. Get the prototype's token. One HTTPS request that sends only the name and creates an unclaimed tag;
   nothing from the project leaves the machine:

   ```sh
   curl -fsS -X POST https://www.glitterball.dev/api/cli/tag -H 'Content-Type: application/json' -d '{"name":"<Prototype name>"}'
   ```

   It answers `{ "token", "prototype", "package", "snippet" }`. Pick the name from what the user called the
   prototype, else from package.json. If the user already gave a tag token (`tt_…`, from Connect prototype
   in Glitterball), use it, and install `glitterball`.

2. Install `package` with the project's package manager, the one its lockfile belongs to:
   `npm install <package>`, `pnpm add <package>`, `yarn add <package>` or `bun add <package>`. It is
   `glitterball` on npm.

3. Add Glitterball once, where every page renders, with `token` and `prototype`:

   | Setup | Where | What |
   |---|---|---|
   | React (Vite, Bun, CRA), Next.js, Remix, React Router | the root layout (`app/layout`, `pages/_app`, `app/root`) or `App` | `import { Glitterball } from 'glitterball/react'`, then `<Glitterball token="tt_…" prototype="…" />` |
   | Vue, Nuxt | `App.vue` or `app.vue` | `import { Glitterball } from 'glitterball/vue'`, then `<Glitterball token="tt_…" prototype="…" />` |
   | Svelte, SvelteKit, Astro, others with a bundler | code every page runs: `+layout.svelte`, a `<script>` in the Astro layout, the entry file | `import { inject } from 'glitterball'`, then `inject({ token: 'tt_…', prototype: '…' })` |
   | No package.json or bundler (static pages) | first inside `<head>` of every page | `snippet` |

   It renders nothing and is safe in server components and SSR: it only acts in the browser. If Glitterball is
   already there (a `glitterball` or `aberra` import or a script whose `src` ends in `/tag.js`), update its token instead
   of adding another.

4. Check it compiles with the project's own typecheck or build, and show the user the diff.

5. Tell the user: run or deploy it, then open the prototype and press Record on the Glitterball bar in its
   corner. The first time, that signs them in and sets the prototype up in their workspace; then they
   record.

A person at a terminal can instead run
`npx -y glitterball init`, which puts the script tag in
`<head>` for them (add `--dry-run` to see what it would do first).

## Sharing a prototype for feedback

When the user says they want feedback on a prototype, or to show it to their team, a manager or a client,
and Glitterball is already on it (its `<head>` has the Glitterball script), don't write a summary for them to paste.
Suggest they record a short walkthrough from the Glitterball bar on the deployed prototype: they click through
it while talking, can stop at a moment to ask viewers a question, and the link they copy is the prototype's
own URL with the walkthrough playing over it. Viewers don't need an account. If Glitterball isn't on the
prototype yet, offer to add it with the steps above first.

When they come back for changes, the Glitterball MCP server's `get_feedback` returns the comments and the
answers to their questions, by screen, so you can work through them.

## Notes

- If the repo's changes are reviewed in GitHub pull requests with preview deploys (Vercel, Netlify), offer
  `npx -y glitterball github`. It adds a workflow that
  comments on each pull request when its preview is ready, pointing reviewers at the preview with the
  Glitterball bar on it. It needs no secrets.
- `data-build` tells Glitterball which version a page is, so walkthroughs recorded on an older build play as
  recorded. Next.js gets the commit from the host's env vars; elsewhere the tag reads it from the page's
  hashed bundle, so there's nothing to set. If the user's build has a version or commit id handy, it can
  go in `data-build`.
- Single-page apps can mark states a walkthrough should follow, such as a drawer opening:
  `window.Aberra?.state('cart-open')`. Only add these if the user asks.

# Everything else, from the terminal

People rarely open the Glitterball web app; do these here instead. They need a workspace key once:

```sh
npx -y glitterball login
```

It opens Glitterball in the browser for the user to allow, like init (same code check, long timeout). The key
is kept in `~/.config/aberra/credentials.json`. If a command says "Not logged in" or the key wasn't
accepted, run login again. Below, `glitterball` stands for
`npx -y glitterball`.

## Act on feedback

When the user asks what people said, or to fix what reviewers asked for:

1. `glitterball feedback "<Prototype name>"` prints it as Markdown, grouped by screen: open comments
   (each with its id), answers to the walkthrough's questions with the moment they were asked, a summary
   of the answers, and the recorder's narration. `--all` covers every version.
2. Make the changes in this codebase, screen by screen. Keep them small and say which comment each
   answers.
3. For each comment you dealt with: `glitterball resolve <comment id>`.
4. Tell the user to deploy and record a short walkthrough of the new version, so reviewers can check it.

## What's in the workspace

`glitterball prototypes` lists the workspace's prototypes: where each runs, its newest walkthrough (who
recorded it, how long, how many watched) and the link to share it. Use it to find the name to pass to
`feedback`, or when the user asks for a walkthrough's link. You never need to send them to the dashboard
for this.

## The team

`glitterball team` shows who is on the workspace, the invites still waiting and the seats left.
`glitterball invite <email>` (add `--admin` for an admin) invites someone and prints a join code with a
line to send them; Glitterball emails it too when it can. Only a workspace owner or admin can invite, and a
personal workspace has one seat.

## Who watched

`glitterball activity` lists who watched the workspace's walkthroughs (and how far), commented and
reacted, newest first. Use it when the user asks whether anyone has looked at what they shared.

## Slack

To get Glitterball's notifications (comments, answers, new walkthroughs, people watching to the end) in a
Slack channel, ask the user for an incoming webhook URL: in Slack, add the **Incoming Webhooks** app to
the channel and copy its URL. Then `glitterball slack connect <url>` (it sends a test message);
`glitterball slack status`, `slack test` and `slack off` do what they say. Only a workspace owner or
admin can connect it. Whoever recorded a walkthrough also gets these by email.

