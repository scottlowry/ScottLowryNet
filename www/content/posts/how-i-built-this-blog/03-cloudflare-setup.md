---
title: "How I Built This Blog: Part 3"
date: 2025-06-14
draft: false
summary: "An overview of how this blog uses Cloudflare for DNS and web analytics"
tags: ["Azure", "Cloudflare", "web development", "blog"]
series: ["Blog Development"]
series_weight: 3
---

# Introduction

In part three of this series, I'll walk through how I created a custom domain for the Azure Static Web App to use with Cloudflare DNS. I'll also show how I use Cloudflare's web analytics to gather site traffic data.

# Why Cloudflare?

I could use Azure DNS or any other DNS provider like AWS Route 53, GoDaddy, etc. but I already have my domain registered through Cloudflare for use with their Zero Trust for Teams service.

# Custom Domain

I navigated to the static web app in Azure portal and then selected `Custom domains` under `Settings`. Then I clicked `+ Add` and selected `Custom domain on other DNS`. I chose to use `www.scottlowry.net` instead of the root domain and here's why:
- Using root domain requires TXT record validation which can take hours to confirm
- Using root domain requires CNAME flattening which I don't want to use even though Cloudflare supports it
- [Microsoft recommends using a subdomain](https://learn.microsoft.com/en-us/azure/static-web-apps/custom-domain-external#create-a-cname-record-on-your-domain-registrar-account)

After creating the custom domain, I was given a CNAME record to create in Cloudflare. I created it making sure to not have Cloudflare proxy it. Next, I created a rule to route requests for the root domain to `www` subdomain.

# Route Rule

In the Cloudflare admin portal, I navigated to the `Rules` section of my domain's settings. I then created the Redirect Rule shown below.
![Cloudflare Redirect Rule Config](/assets/posts/how-i-built-this-blog/cloudflare-redirect-rule-config.png#center)

I then also created a proxied `A` record for `@` pointing to `192.0.2.1`. This is for the redirect roule to work as it needs Cloudflare to proxy the redirect.

# Web Analytics

There's a section for metrics for the static site in the Azure portal but it doesn't give much useful information for content creators. Azure's built-in metrics focus primarily on infrastructure-level data like bandwidth usage and request counts, which are more relevant for operations teams than bloggers trying to understand their audience.

Cloudflare offers Web Analytics as part of registering my domain with them so it was an easy choice to use that for gathering more meaningful metrics. It's very feature rich and these are just some of the things it offers:
- **No cookies or fingerprinting**: Cloudflare Web Analytics is privacy-respecting and doesn't require cookie banners or GDPR compliance worries
- **Page views and unique visitors**: Clear metrics on what content resonates with readers
- **Top pages**: See which blog posts are most popular
- **Referrer data**: Understand how people discover your content (search engines, social media, direct links)
- **Geographic insights**: Learn where your readers are located without compromising their privacy
- **Single script tag**: Just add one lightweight JavaScript snippet to your Hugo templates
- **Real-time data**: See traffic as it happens, unlike Azure's delayed metrics
- **Time-based analysis**: See when your content gets the most engagement

## Implementation

Enabling was a quick set up in the Cloudflare admin portal. I navigated to `Web Analytics` under the `Analytics & Logs` section of my domain's settings and enabled it. By default, it injects a JS Snippet to track visitors if the domain is proxied by Cloudflare. This is great except it doesn't track any direct visits to `www` since it's not proxied. I instead opted to manually insert the JS Snippet myself using Hugo's `extend_head.html` partial. Using `extend_head.html` is the recommended approach as it keeps analytics separate from base templates and won't be overwritten during theme updates. Now visits are tracked for both visits to root and `www`. 

# Final Thoughts

Now I have my own Hugo blog hosted for free in Azure, automated with GitHub, and generating meaningful metrics for it in Cloudflare. It was all relatively easy to set up and now I'm enjoying blogging. 