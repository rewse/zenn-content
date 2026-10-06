# Repository Rules

This repository is linked to Zenn through GitHub integration: pushing to `main` deploys every article and book whose frontmatter has `published: true`. Articles are written in Japanese for beginner-to-intermediate engineers looking for practical, reproducible solutions, mostly on infrastructure, DevOps, web development, developer tooling, and troubleshooting.

## Zenn's Principles

Refine article text against the values Zenn is built on ([about Zenn](https://zenn.dev/about)). Zenn exists so that engineers can share what they learned casually, help each other, and gain from it: a better career, fair recognition, and income through reader badges and book sales. Its goal is a world where every engineer gets the most out of publishing, and where engineers who steadily share useful knowledge and contribute to the community can earn a fair return with confidence.

An article is therefore a contribution to the community, however small, and it is written for real readers in a public space. Keep the tone respectful and constructive toward readers and toward the authors of anything the article discusses or criticizes, and make sharing knowledge the article's purpose rather than promotion.

## Content Quality

Zenn's [community guidelines](https://zenn.dev/guideline) set the bar. The title must match what the article delivers, without exaggeration. The opening states what the article covers and what the reader will learn, and names the intended reader so the level of explanation has a reference point. Describe the environment, versions, and reproduction conditions so readers can reproduce the result, and add the author's own experience or analysis on top of official information, since that is what makes an article worth reading. Publish only finished articles.

Zenn does not accept posts whose main purpose is promoting a product or recruiting, clickbait titles, use of others' work beyond fair quotation, or AI-generated content published without verifying its accuracy. Verify every technical claim, command, and version in an article before publishing it.

The default article structure is overview and background, prerequisites, step-by-step reproducible body, and a summary with next steps. Deviate when the content calls for it.

Choose `type` by subject ([Zenn's guide](https://zenn.dev/tech-or-idea)):

- `idea`: careers, management, and other non-technical topics, plus abstract thinking about technology and roundups such as release summaries, book recommendations, or "I built this service" posts.
- `tech`: concrete content about programming, software, hardware, or infrastructure, such as troubleshooting a tool, setting up a CI deployment, or comparing framework performance.

## Repository Layout

- `articles/<slug>.md`: one file per article. The slug is the article's unique ID on Zenn, so keep it unchanged when updating an article; renaming the file publishes a new article. Use a descriptive kebab-case slug, such as `synology-nas-no-space-left-btrfs-metadata`.
- `books/<slug>/`: `config.yaml`, a `cover.png` or `cover.jpg` (500 × 700 px recommended), and one `.md` file per chapter.
- `images/<slug>/`: images for the article or book with the same slug.

## Frontmatter

```yaml
# articles/<slug>.md
---
title: "Specific, searchable title"
emoji: "😸"                 # exactly one emoji related to the content
type: "tech"                # "tech" or "idea"
topics: ["docker", "aws"]   # up to 5
published: true             # false keeps it as a draft
published_at: 2050-06-12 09:03  # optional scheduled publication
---
```

```yaml
# books/<slug>/config.yaml
title: "Book title"
summary: "Book description"
topics: ["markdown", "zenn"]  # up to 5
published: true
price: 0                      # 0 (free) or 200-5000 yen in steps of 100
toc_depth: 2                  # 0-3
chapters:                     # chapter file names in order, up to 100
  - chapter1
  - chapter2
```

```yaml
# books/<slug>/<chapter>.md
---
title: "Chapter title"
free: true  # optional: free preview in a paid book
---
```

Topics use lowercase letters only, with no hyphens.

## Writing Markdown

Follow the [Zenn Markdown guide](https://zenn.dev/zenn/articles/markdown-guide). Zenn-specific syntax that differs from GitHub-flavored Markdown:

- Code fence with a file name: ` ```js:path/to/file.js `. Diff with highlighting: ` ```diff js `.
- Collapsible section: `:::details Title` ... `:::`.
- Comment: `<!-- ... -->`, single line only. It is not rendered, so use it for notes to self.
- Embeds: a bare URL on its own line renders as a link card, and X, YouTube, and GitHub file or permalink URLs (including `#L1-L3` line ranges) render as embeds. `@[card](URL)` forces a card, and `@[gist]`, `@[codepen]`, `@[speakerdeck]`, `@[docswell]`, `@[slideshare]`, `@[figma]`, `@[codesandbox]`, `@[stackblitz]`, `@[jsfiddle]`, and `@[blueprintue]` take the service URL or ID.
- Footnotes: `[^1]` with `[^1]: text`, or inline `^[text]`.
- Image caption: an `*italic*` line immediately after the image. Image width: `![alt](URL =250x)`.
- Math: KaTeX with `$$` blocks and `$...$` inline.
- Mermaid: at most 2000 characters and 10 chains per block; click events are disabled.
- Message box: `:::message` or `:::message alert` ... `:::`. Nest boxes by adding a colon to the outer fence (`::::details` around `:::message`).

Style conventions for articles:

- Give every code fence a language, use a file name when showing file contents, prefer complete runnable examples over fragments, and comment complex examples in Japanese.
- Keep heading levels sequential, without skipping from `##` to `####`.
- Use `-` for bullet lists.
- Use inline code for commands and technical identifiers.
- Write descriptive link text rather than "here" or "click this".

Run `npm run lint` after editing an article; textlint checks Japanese technical-writing style and AI-writing patterns.

## Images

Put images under `images/<slug>/` and reference them with an absolute path such as `![Synology DSM storage manager screen](/images/<slug>/screenshot-1.png)`; relative paths do not resolve on Zenn. Give every image descriptive alt text. Zenn accepts `.gif`, `.jpeg`, `.jpg`, `.png`, and `.webp` up to 3 MB, and a file outside these limits fails the deploy. Removing an image from the repository removes it from Zenn, and a replaced image can take about a minute to update because of caching.

## Commands

```bash
npm install
npx zenn new:article --slug <slug> --title "<title>" --type tech --emoji ✨
npx zenn new:book --slug <slug>
npx zenn preview    # http://localhost:8000
npm run lint
```

## Publishing

Setting `published: true` and pushing publishes an article; keeping the same slug and pushing updates it. Include `[ci skip]` or `[skip ci]` in a commit message to push without deploying. Deleting a file does not delete the article from Zenn; delete it from the [dashboard](https://zenn.dev/dashboard).

## Commit Messages

Follow Conventional Commits with these scopes: `article` or `article:<slug>`, `book` or `book:<slug>`, and `content` for other content files. Choose the type as follows:

- `chore`: publishing, reverting to draft, scheduling, metadata such as topics or emoji, dependency and configuration updates.
- `docs`: repository documentation such as README.
- `feat`: a new article or book, or new content in one, such as a section or added information.
- `fix`: errors such as typos, wrong commands, broken links, technical inaccuracies, or broken formatting.
- `refactor`: reorganizing content for readability without changing what it says.
- `style`: formatting or notation consistency that does not affect content.

Make the subject concrete: `add filename to code blocks`, not `improve readability`. Explain the reason for a change in the body, and quote Japanese terms as they are when the change is about wording. Mark changes that break existing reader links or purchases with `!` and a `BREAKING CHANGE:` footer.

```
feat(article:docker-guide): add troubleshooting section

Add solutions for common permission errors and port conflicts
based on frequent reader questions.
```

```
fix: unify technical terminology

Change デプロイメント to デプロイ for consistency.
```

```
feat(article)!: split kubernetes article into a 3-part series

BREAKING CHANGE: Existing bookmark URLs may be affected.
```

```
chore(article:docker-guide): schedule publication for 2024-12-01
```

## Validation

Before pushing, run `uvx pre-commit run --all-files`, which includes `npm run lint`, and commit any files the hooks reformat. Stage new files first, because `--all-files` skips untracked files. CI runs the same hooks, and `core.hooksPath` points at git-defender, so `pre-commit install` cannot run them at commit time.
