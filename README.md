# Openfire Terms of Service Plugin

This is a plugin for [Openfire](https://www.igniterealtime.org/projects/openfire/), a cross-platform real-time collaboration server based on the XMPP protocol developed by the [Ignite Realtime](https://www.igniterealtime.org/) community. It adds functionality to require users to accept a terms of service before they can log in.

## Reporting Issues

Issues may be reported to the [forums](https://discourse.igniterealtime.org) or via this repo's [Github Issues](https://github.com/igniterealtime/openfire-termsofservice-plugin).

## Overview

Requires users to explicitly accept the current terms of service before their authentication completes, and serves the terms themselves publicly over HTTP, so a user (or their client) can read them without needing to authenticate first.

The only enforcement mechanism this plugin currently ships is a SASL2 task, implementing the "SASL2 Terms of Service Acceptance Task" specification (a ProtoXEP awaiting a XEP number at the time of writing). The plugin's document repository and acceptance records are deliberately kept independent of that mechanism, so that a future non-SASL2 way of obtaining acceptance can be added later without changing either.

## Installation

Copy `termsofservice.jar` into the plugins directory of your Openfire installation. The plugin will then be automatically deployed. To upgrade to a new version, copy the new `termsofservice.jar` file over the existing file.

This plugin exposes a public web service, which will not work if the Web Binding service of Openfire is disabled. It can be enabled on **Server → Server Settings → Web Binding**.

### Getting started

1. Go to **Server → Server Settings → Terms of Service Texts** and create a draft: give it a version identifier (e.g. `2026-06`) and write its text as Markdown.
2. Activate that draft. It becomes the current version, and can no longer be edited (start a new draft, or copy it forward, for the next revision).
3. Go to **Server → Server Settings → Terms of Service**, confirm the announced URL looks right for your deployment, and enable the requirement.

### The public endpoint

The current version's text is served at the URL shown on the settings page (by default `https://<this server>/termsofservice`), in whichever of HTML, Markdown, or plain text the requester's `Accept` header prefers. If your server sits behind a reverse proxy, or is otherwise reachable under a different address than Openfire itself is configured with, override the announced protocol, host, port, or path on the settings page so that the URL handed to clients is one they can actually reach.

### Reviewing and managing a user's acceptance

Open any user's properties page and look for **Terms of Service** among the other per-user options (Profile Information, Password, and so on). It shows every version that account has accepted, when, and how (through the SASL2 task, or recorded manually by an administrator), and lets an administrator record an acceptance on the user's behalf, for example because it was obtained outside of Openfire entirely.

### Behaviour worth knowing about

* Anonymous authentication is asked to accept on every connection, since there is no account to record acceptance against.
* If the server cannot durably persist an acceptance (for example a transient database failure), authentication is still allowed to complete for that one session; the account is asked again on its next connection.
* A client that does not implement the SASL2 task cannot authenticate while the requirement applies to its account. Per the base SASL2 specification (XEP-0388), it will still receive a human-readable explanation before it has to abort.
* Only clients speaking SASL2 (XEP-0388) are affected by the SASL2 task. If this server also permits the legacy SASL profile of RFC 6120, or any other authentication path that bypasses SASL2, this requirement is not enforced on that path; see the specification's Security Considerations.
* An acceptance obtained some other way (a support interaction, a signed agreement) can be recorded by an administrator on the user's behalf, and counts the same as one obtained through the SASL2 task.

### Extending this plugin

How a user comes to accept the terms is deliberately decoupled from the fact that they did: any code with access to `TosAcceptanceService` can record an acceptance, tagged with a mechanism label of its choosing. The SASL2 task and the admin-console "record on behalf of" action are the two mechanisms this plugin ships with, not the only two it supports — a web login flow or an email-confirmation link, for example, could be added later as a further caller of the same service, with no change needed to this plugin's database schema, its document repository, or the SASL2 task itself.
