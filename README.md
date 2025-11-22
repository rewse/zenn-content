# Zenn Content Repository

GitHub-integrated repository for publishing technical articles and books on the Zenn platform. This repository provides a seamless workflow from local writing to automatic publication.

## Overview

This repository is integrated with the Zenn platform to manage and publish technical articles and books. Write content in Markdown format locally, and it will be automatically published to Zenn when pushed to GitHub.

### Key Features

- **GitHub Integration**: Automatic publication on push
- **Local Writing**: Write with your favorite editor
- **Preview**: Local preview with Zenn CLI
- **Image Management**: Centralized image storage in repository
- **Version Control**: Track changes with Git

## Tech Stack

- **zenn-cli**: Official CLI tool for Zenn platform
- **Node.js**: CommonJS modules
- **npm**: Package manager

## Setup

### Prerequisites

- Node.js installed
- GitHub account linked with Zenn account

### Installation

```bash
# Install dependencies
npm install
```

## Usage

### Create a New Article

```bash
# Create an article
npx zenn new:article

# Create with options
npx zenn new:article --slug article-slug --title "Title" --type tech --emoji ✨
```

### Create a New Book

```bash
# Create a book
npx zenn new:book

# Create with slug
npx zenn new:book --slug book-slug
```

### Local Preview

```bash
# Start preview server
npx zenn preview
```

Access http://localhost:8000 in your browser to preview content.

### Publish Articles

1. Set `published: true` in the article's front matter
2. Commit and push to GitHub repository
3. Content will be automatically published to Zenn

```bash
git add .
git commit -m "feat(article): add new article about docker"
git push origin main
```

## Project Structure

```
.
├── articles/           # Technical articles (Markdown)
├── books/             # Book content
├── images/            # Images for articles and books
│   └── article-slug/  # Organized by article
├── .kiro/             # Kiro AI settings
├── package.json       # Project configuration
└── README.md          # This file
```

## Article Front Matter

```yaml
---
title: "Article Title"
emoji: "😸"              # Icon emoji (1 character)
type: "tech"             # "tech" or "idea"
topics: ["docker", "aws", "devops"]  # Tags (max 5)
published: true          # Publication status
published_at: 2050-06-12 09:03  # Optional: scheduled publication
---
```

### Article Types

- **tech**: Technical content about programming, infrastructure, hardware, etc.
- **idea**: Career, management, abstract concepts, or information summaries

## Image Management

### Image Placement

```
images/
└── article-slug/
    ├── screenshot-1.png
    └── diagram.jpg
```

### Image Reference

```markdown
![Alt text](/images/article-slug/screenshot-1.png)
```

### Limitations

- File size: Max 3MB
- Supported formats: `.png` `.jpg` `.jpeg` `.gif` `.webp`

## Content Guidelines

### Recommended Article Characteristics

- Title matches content
- Clear overview at the beginning
- Detailed environment and reproduction conditions
- Clear target audience
- Personal experience and insights

### Content to Avoid

- Posts primarily for advertising or promotion
- Exaggerated clickbait titles
- Copyright-infringing content
- Unverified AI-generated content

## References

- [About Zenn](https://zenn.dev/about)
- [Community Guidelines](https://zenn.dev/guideline)
- [GitHub Integration Guide](https://zenn.dev/zenn/articles/connect-to-github)
- [Markdown Guide](https://zenn.dev/zenn/articles/markdown-guide)
- [Zenn CLI](https://zenn.dev/zenn/articles/install-zenn-cli)
