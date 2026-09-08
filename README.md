# FlowPath by Friven — documentation site

The public site for the **FlowPath by Friven** Power BI custom visual: user guide, privacy policy, and terms
of use. Served by GitHub Pages at
<https://frivensolutions.github.io/flow-map-visual-docs/>.

This repository holds no source code. The visual itself lives in a separate, private repository —
its source is shared with Microsoft's certification reviewers, who require access to it, but is not
published. See the [terms of use](https://frivensolutions.github.io/flow-map-visual-docs/terms.html).

The split exists because AppSource requires publicly reachable URLs for the privacy policy, help
documentation and support, and GitHub Pages only serves public repositories on the free tier.

**Support runs to support@friven.dev**, not through this repository's issues — see
[the support page](https://frivensolutions.github.io/flow-map-visual-docs/support.html).

## Files

| File | Purpose |
|---|---|
| `index.html` | Landing page |
| `guide.html` | User guide — fields, formatting, licensed features |
| `support.html` | Support contact and what to include in a report |
| `privacy.html` | Privacy policy (required by AppSource) |
| `terms.html` | Terms of use — points at the Microsoft Standard Contract |
| `style.css` | Shared styling, light and dark |
| `.nojekyll` | Serve the HTML as written rather than through Jekyll |

Plain static HTML with no build step: edit and push.

## Enabling Pages

Settings → Pages → Source: **Deploy from a branch** → `main` / `/ (root)`.

---

© Friven Solutions · support@friven.dev
