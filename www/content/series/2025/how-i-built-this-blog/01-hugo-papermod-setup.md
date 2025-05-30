---
title: "How I Built This Blog: Part 1"
date: 2025-05-27
draft: false
summary: "An overview of how this blog is built using Hugo with PaperMod theme."
tags: ["Hugo", "web development", "static site", "blog"]
series: ["Blog Development"]
series_weight: 1
ShowPostNavLinks: true
---

# Introduction

This article is part one in a series about how I use [Hugo](https://gohugo.io/), a powerful static site generator written in Go, to build this blog. I'll also dive into using [Azure Static Web Apps](https://azure.microsoft.com/en-us/products/app-service/static) to host it for free and [GitHub Actions](https://github.com/features/actions) for CI/CD in a later article. 

## Why Hugo?

After a 20+ year break from web development, I wanted to start a blog without diving back into the complexity of traditional web stacks. Hugo was the perfect solution because it offers:
- **Simplicity**: Write content in Markdown, and Hugo handles the rest
- **Performance**: Static sites are fast and secure
- **Flexibility**: Easy to customize and extend
- **Maintenance**: No database or server-side code to maintain
- **Version Control**: All content and configuration can be tracked in Git

Markdown is so much easier to use than HTML. Check out the [Markdown Cheat Sheet](https://www.markdownguide.org/cheat-sheet/) for a quick overview.

## Theme and Structure

This blog uses the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme, which provides:
- Clean, modern design
- Dark/light mode support
- Responsive layout
- Built-in search functionality
- Social media integration
- Code syntax highlighting

The blog follows a standard Hugo directory structure, with posts organized by year and month:

```
www/
├── config.yaml        # Main configuration file
├── content/          # Where all the content lives
│   ├── posts/        # Blog posts in Markdown
│   │   └── YYYY/     # Posts organized by year
│   │       └── MM/   # Posts organized by month
│   └── series/       # Posts organized in series
│       └── YYYY/     # Series organized by year
├── static/           # Static assets (images, etc.)
│   ├── assets/       # Site-wide assets
│   └── posts/        # Post-specific assets
├── themes/           # Theme files
└── layouts/          # Custom layout overrides
```

This organization helps maintain a clean content structure and makes it easy to manage posts chronologically.

## Installation and Configuration

Setting up Hugo on my Mac was straightforward. Here's how I did it:

1. First, I installed Hugo using [Homebrew](https://brew.sh/):
   ```bash
   brew install hugo
   ```

2. I verified the installation and checked the version:
   ```bash
   hugo version
   ```

3. I created a new Hugo site in my desired directory:
   ```bash
   hugo new site www
   cd www
   ```

4. I added the PaperMod theme as a Git submodule:
   ```bash
   git init
   git submodule add https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
   ```

5. I created a new `config.yaml` file in `www` and configured it for my needs. You can see the current configuration in my [GitHub repository](https://github.com/scottlowry/ScottLowryNet/blob/main/www/config.yaml).

6. I created my first post in markdown, which you can view [here](https://raw.githubusercontent.com/scottlowry/ScottLowryNet/refs/heads/main/www/content/posts/2025/May/hello-world.md). The front matter and structure are similar to what you'd find in any Hugo blog, but customized for my needs.

7. I then used Hugo's built-in development server to view the draft:
   ```bash
   hugo server -D
   ```
   This starts a local development server at `http://localhost:1313` with draft posts enabled, allowing me to preview my content in real-time using a web browser. The development server provides live reload, so any changes to content or configuration are immediately visible in the browser. This makes it easy to iterate on the design and content before deploying.

## Conclusion

I wanted a blog with no backend, minimal risk of breakage, and full version control. Hugo and PaperMod delivered. Everything's in Git, builds locally in seconds, and stays fast at any scale. It's exactly what I was looking for after years away from web dev.

If you're interested in starting your own Hugo blog, I recommend checking out the [official Hugo documentation](https://gohugo.io/documentation/).

## What's Next?

In the next article, I'll cover how this blog is deployed using Azure Static Web Apps and GitHub Actions. This setup provides a robust, free hosting solution that includes automatic deployments, global CDN, and built-in security features.