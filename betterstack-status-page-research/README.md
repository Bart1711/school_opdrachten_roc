# Better Stack status page: research and redesign proposal

Date: 2026-11-01

## 1. What it is

Better Stack (formerly Better Uptime) sells monitoring, logs, on-call and a hosted
status page in one product. The status page is fed by Better Stack's own uptime
monitors, so a monitor going down can flip a component on the page automatically.

### Typical layout of a Better Stack public page

1. Header with logo and a "Subscribe to updates" button
2. Overall banner ("All services are online" / degraded / outage)
3. Sections with components, each with a row of daily uptime bars and an uptime %
4. Hover on a bar shows that day's downtime and any incidents
5. Active incidents and scheduled maintenance above the component list
6. Incident history on a separate page
7. Footer "Powered by Better Stack" (removing it is a paid add-on)

### Features

- Automatic status from Better Stack monitors and heartbeats
- Custom subdomain through one CNAME record
- Subscribers by email, Slack, webhook and RSS
- Scheduled maintenance and incident updates
- Private pages via password, SSO or IP allowlist (paid)
- Custom CSS/JS (paid), embeddable charts

### Pricing (verify on betterstack.com/pricing, sources disagree)

| Item | Price found |
| --- | --- |
| Free plan | 10 monitors, 1 status page |
| Extra status page | about $12 to $15 per month |
| Custom CSS/JS | about $12 to $15 per page per month |
| Password protection | about $50 per page per month |
| SSO or IP allowlist | about $250 per month each |
| Remove "Powered by" (white label) | about $208 to $250 per page per month |
| Extra subscribers | about $40 per month per 1,000 |

## 2. What is wrong with it

Sources: G2 and Capterra reviews, competitor comparisons, general status page criticism.

1. **Add-on pricing is the biggest problem.** Basic things like hiding the
   "Powered by" footer or putting a password on the page are priced per page per
   month and add up fast. White label costing more than the whole base product is
   hard to defend.
2. **Limited customization on lower plans.** Reviewers ask for background colors,
   section reordering and layout control. Custom CSS is a paid add-on.
3. **Components are tied to monitors.** Reviewers want components with no linked
   monitor (for example "Customer support" or a third party dependency) and more
   control over manual status.
4. **No multi-language pages.** Only English is confirmed. A problem for any
   company with non-English customers, including Dutch ones.
5. **Thin data on the page.** No response time / latency graph per component,
   only up/down bars. A green bar says nothing about a service that is slow.
6. **"Sea of green" problem.** Like every status page, it only shows what the
   monitors see or what someone types in. Partial outages (login fails for 10% of
   users) often stay green. This is an industry wide problem, not unique to Better Stack.
7. **Analytics and webhooks gated behind paid plans** (Google Analytics on the
   page, outgoing webhooks).
8. **Probing from few regions.** openstatus claims Better Stack checks from 4
   regions round robin. A regional outage can be missed. (Competitor claim, verify.)
9. **Subscribing is the only way to get notified.** No browser push, no per
   component subscription without signing up.

## 3. How to remake it better

### Must have (fixes the real problems)

| Problem | Better solution |
| --- | --- |
| Expensive add-ons | Flat pricing: white label, password and custom domain included. If it is a school/portfolio project: everything free and self hostable |
| "Sea of green" | Show **live measured data** next to the manual status: latency, error rate, number of regions failing. Add "X users reported a problem in the last 30 min" button (like Downdetector, but first party) |
| Only up/down bars | Bars with 3+ states (operational, degraded, down, maintenance) plus a latency sparkline per component, p50/p95 on hover |
| Components tied to monitors | Components are independent. A component can have 0, 1 or many monitors, plus manual override |
| English only | i18n from day one. Language follows the browser (`Accept-Language`), switch in header. Incident updates can be written in multiple languages |
| Status page goes down with the product | Host it on a separate provider/CDN as static files plus a small JSON API, so it survives the main outage |

### Should have (makes it clearly better than Better Stack)

- **Per component subscriptions** by email, RSS/Atom, webhook, Slack, Discord and
  browser push, without needing an account.
- **Plain language incident template**: what is broken, who is affected, what
  users can do now, when the next update comes. Force a "next update at" time
  on every update so pages never go silent.
- **Region view**: small map or table showing which regions are affected.
- **Public JSON API and badge** (`/api/status.json`, `/badge.svg`) so developers
  can build on it. Atlassian Statuspage has this, Better Stack less clearly.
- **Accessibility**: status never shown by color alone (icon + text), WCAG AA
  contrast, keyboard navigation, screen reader labels on uptime bars. Most status
  pages fail this because the bars are pure colored divs.
- **Dark mode** following the system setting.
- **Postmortem link** on resolved incidents.
- **Uptime calculated honestly**: show degraded time separately instead of
  counting it as 100% up.

### Nice to have

- Embeddable widget for in-app banners ("We are having issues with payments")
- Status history export (CSV) for SLA claims
- Git based config (components and incidents as files in a repo, like openstatus
  or Upptime), so changes are reviewed and versioned

## 4. Suggested layout for the remake

```
+--------------------------------------------------------------+
| Logo                     [NL | EN]  [Dark]  [Subscribe v]     |
+--------------------------------------------------------------+
| ! Payments degraded since 14:02 (CET)                         |
|   Some card payments fail. Next update at 14:45.              |
|   [I'm affected too: 213 reports]                             |
+--------------------------------------------------------------+
| Website          Operational  [||||||||||||||||||] 99.98%     |
|                  latency ~~~~~~~ p95 180 ms                   |
| Payments  v      Degraded     [|||||||||||||||:||] 99.71%     |
|   Card payments  Degraded     (3 of 5 regions failing)        |
|   iDEAL          Operational                                  |
| API              Operational  [||||||||||||||||||] 100%       |
+--------------------------------------------------------------+
| Scheduled maintenance: DB upgrade, Sat 02:00-03:00            |
+--------------------------------------------------------------+
| Past incidents (filter by component, export CSV)              |
+--------------------------------------------------------------+
| RSS  |  JSON API  |  Status badge                             |
+--------------------------------------------------------------+
```

## 5. Tech stack suggestion (if built as a project)

- Frontend: static HTML/CSS/JS or a small framework, served from a CDN separate
  from the main app
- Backend: PHP (Yii, matches this course) or Node, with MySQL tables:
  `components`, `monitors`, `checks`, `incidents`, `incident_updates`,
  `subscribers`, `user_reports`, `translations`
- Checker: cron job that pings each monitor every minute and stores status + latency
- Notifications: email via SMTP, webhooks via simple HTTP POST queue

## 6. Honest verdict

Better Stack's status page is good enough and quick to set up, and the free tier
is generous for one page. Its weak points are the per page add-on pricing,
limited customization, English only, and only showing up/down. A remake wins
by being cheaper/free on the basics, showing real measured data instead of
just green bars, supporting multiple languages, and being accessible. Building
a better looking page alone is not enough; the data and honesty are where it
can actually beat Better Stack.

## Limitations of this research

The Better Stack website itself could not be loaded from the research
environment, so details come from review sites (G2, Capterra), comparison
sites and competitor blogs. Prices differ between sources and some reviews are
old. Check betterstack.com/pricing and the docs before relying on exact numbers.

## Sources

- [Better Stack pricing](https://betterstack.com/pricing)
- [Better Stack status page product page](https://betterstack.com/status-page.md)
- [Hyperping: Better Stack pricing](https://hyperping.com/blog/betterstack-pricing)
- [Capterra: Better Stack pricing](https://www.capterra.com/p/202150/Better-Stack/pricing/)
- [Capterra: Better Stack reviews](https://capterra.com/p/202150/Better-Stack/reviews/)
- [G2: Better Stack reviews](https://www.g2.com/products/better-stack/reviews)
- [G2: Better Stack vs Statuspage](https://www.g2.com/compare/better-stack-vs-statuspage)
- [getpulsesignal: Better Stack vs Statuspage](https://getpulsesignal.com/compare/betterstack-vs-statuspage)
- [openstatus: Better Stack alternative](https://www.openstatus.dev/compare/betterstack)
- [Better Stack vs Instatus](https://betterstack.com/community/comparisons/better-stack-vs-instatus/)
- [Better Stack vs Statuspage](https://betterstack.com/community/comparisons/better-stack-vs-statuspage/)
- [Status page providers overview](https://vibereference.vercel.app/cloud-and-hosting/status-page-providers)
- [PagerDuty: status page guide](https://www.pagerduty.com/?p=100612)
- [Rootly: better incident status pages](https://rootly.com/blog/the-ultimate-guide-to-creating-better-incident-status-pages.md)
- [StatusGator: why status pages suck](https://statusgator.com/blog/why-status-pages-suck/)
- [Alert24: why status pages lie](https://alert24-marketing.pages.dev/blog/why-status-pages-lie)
- [ThousandEyes internet report](https://thousandeyes.com/blog/internet-report-weekly-pulse-apr-11)
- [The Register: Apple outage status](https://www.theregister.com/2023/05/15/apple_outage_status/)
