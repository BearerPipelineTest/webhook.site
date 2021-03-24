---
layout: home
title: About
nav_order: 0
---

# About Webhook.site

[![Docker Cloud Build Status](https://img.shields.io/docker/cloud/build/fredsted/webhook.site.svg)](https://hub.docker.com/r/fredsted/webhook.site)
[![GitHub last commit](https://img.shields.io/github/last-commit/fredsted/webhook.site.svg)](https://github.com/fredsted/webhook.site/commits/master)

With [Webhook.site](https://webhook.site), you instantly get a unique, random URL that you can use to test and debug Webhooks and HTTP requests, as well as to create your own workflows using the [Custom Actions](/custom-actions.html) graphical editor or [WebhookScript](/webhookscript.html), a simple scripting language, to transform, validate and process HTTP requests. 

What are people using it for?

* Receive Webhooks without needing an internet-facing Web server
* Send Webhooks to a server that's behind a firewall or private subnet
* Transforming Webhooks into other formats, and re-sending them to different systems
* Connect different APIs that aren't compatible
* Building contact forms that send emails
* Instantly build APIs without needing infrastructure

## Company information

Webhook.site is operated by Webhook ApS (VAT ID: DK41561718.)

Address: M.P. Bruuns Gade 52-4, 8000 Aarhus, Denmark.

Founded and built by Simon Fredsted ([@fredsted](https://twitter.com/fredsted)).

## Open Source

There are two versions of Webhook.site: 

* The completely open-source, MIT-licensed version is available on [Github](https://github.com/fredsted/webhook.site), which can be self-hosted using e.g. Docker, is great for testing Webhooks, but doesn't include features like Custom Actions.

* The cloud version at https://webhook.site which has more features, some of them requiring a paid subscription.

## Acknowledgements

* The app was built with [Laravel](https://laravel.com) for the API and Angular.js for the frontend SPA. 
* WebhookScript based on [Primi](https://github.com/smuuf/primi) Copyright (c) Přemysl Karbula. 
* The WebhookScript editor is using the [Ace](https://ace.c9.io). 
* JSONPath extraction provided by [FlowCommunications](https://github.com/FlowCommunications/JSONPath). 
* This documentation site uses MkDocs with the [Material](https://squidfunk.github.io/mkdocs-material/) theme.