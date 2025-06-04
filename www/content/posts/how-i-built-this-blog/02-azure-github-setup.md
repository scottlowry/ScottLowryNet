---
title: "How I Built This Blog: Part 2"
date: 2025-06-03
draft: false
summary: "An overview of how this blog is hosted and deployed using Azure Static Web Apps and GitHub Actions."
tags: ["Azure", "GitHub", "CI/CD", "Hugo", "web development", "blog"]
series: ["Blog Development"]
series_weight: 2
---

# Introduction

In part two of this series, I'll walk through how I set up hosting and deployment for this blog using Azure Static Web Apps (free tier) and GitHub Actions. This combination provides a robust, automated deployment pipeline that's perfect for static sites like this Hugo blog.

## Why Azure Static Web Apps?

Azure Static Web Apps offers several advantages for hosting a Hugo blog:

- Free tier with generous limits (100GB bandwidth/month, 1GB storage)
- Built-in CI/CD with GitHub Actions
- Global CDN for fast content delivery
- Free SSL certificates
- No server management required

## Setting Up Azure Static Web Apps

1. First, I created a new Static Web App in the Azure Portal:
   - Navigated to Azure Portal and selected "Create a resource"
   - Searched for "Static Web App" and selected it
   - Chose the free tier plan
   - Connected my GitHub repository
   - Configured the build settings (more on this below)

![Azure Static Web App Config](/assets/posts/how-i-built-this-blog/azure-static-web-app-config.png#center)

## GitHub Actions Workflow Configuration

The deployment is handled automatically through GitHub Actions. Azure Static Web Apps creates a workflow file (`.github/workflows/azure-static-web-apps-*.yml`) that handles the build and deployment process. Here's the key configuration I used:

```yaml
app_location: "www"           # Where my Hugo site source code lives
output_location: "public"     # Where Hugo builds the site
```

### Hugo Version Configuration

One crucial aspect of the setup was specifying the Hugo version in the workflow file. This is particularly important when using themes like PaperMod that require specific Hugo versions. I added this to the workflow's environment variables:

```yaml
env:
  HUGO_VERSION: 0.147.3       # Required for PaperMod theme
```

## The Deployment Process

The workflow automatically:
1. Triggers on pushes to main branch
2. Builds the Hugo site
3. Deploys to Azure's global CDN

## Benefits of This Setup

- **Automated Deployments**: Every push to main automatically deploys to production
- **Cost-Effective**: The free tier is more than sufficient for a personal blog
- **Low Maintenance**: No server management or SSL certificate renewal needed
- **Fast Performance**: Global CDN ensures quick content delivery worldwide

## Next Steps

In the next post, I'll cover setting up a custom domain and gathering web analytics using Cloudflare DNS.

