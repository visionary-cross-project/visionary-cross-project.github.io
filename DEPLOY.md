# Deployment & domain migration

## Current setup (LIVE)

All four sites are GitHub Pages on the single domain **visionary-cross-project.github.io**
(path-based / "project pages"):

| Site      | URL                                            | Repo                                                    |
|-----------|------------------------------------------------|---------------------------------------------------------|
| Hub       | https://visionary-cross-project.github.io/     | visionary-cross-project/visionary-cross-project.github.io |
| Ruthwell  | …/ruthwell                                     | visionary-cross-project/ruthwell                        |
| Bewcastle | …/bewcastle                                    | visionary-cross-project/bewcastle                       |
| Brussels  | …/brussels                                     | visionary-cross-project/brussels                        |

Shared chrome (header/nav/footer + styles) lives in THIS repo's `_includes`,
`_layouts`, `_sass`, and `assets/css`. The cross repos pull it via `remote_theme`
and keep no local copies. All links are **root-relative** (`/ruthwell`,
`/frontmatter/...`), so the live setup is domain-agnostic.

Switching to a custom domain with the **same path model** (e.g.
`visionarycross.ca/ruthwell`) is trivial: change `url:` in each repo's
`_config.yml`, set the custom domain on the hub repo, point DNS. Done.

---

## Target: SUBDOMAINS on a custom domain

Example with `visionarycross.ca`:

| Site      | URL                                |
|-----------|------------------------------------|
| Hub       | https://visionarycross.ca          |
| Ruthwell  | https://ruthwell.visionarycross.ca |
| Bewcastle | https://bewcastle.visionarycross.ca|
| Brussels  | https://brussels.visionarycross.ca |

> **This cannot be done while on github.io.** Project pages must stay path-based
> (`baseurl: /ruthwell`). The subdomain model needs `baseurl: ""` and full-URL
> cross-links, which would break the github.io sites. Do it ONLY after the
> domain + DNS are live — it is a one-time cutover.

### 1. Per-repo `_config.yml` changes

**Hub** (apex domain):
- `url: "https://visionarycross.ca"`
- `baseurl: ""`  *(unchanged)*
- leave `hub_url` unset (the hub IS the apex)
- crosses → full subdomain URLs:
  ```yaml
  crosses:
    ruthwell:  "https://ruthwell.visionarycross.ca"
    bewcastle: "https://bewcastle.visionarycross.ca"
    brussels:  "https://brussels.visionarycross.ca"
  ```

**Each cross** (Ruthwell shown; mirror for Bewcastle/Brussels):
- `url: "https://ruthwell.visionarycross.ca"`
- `baseurl: ""`   ← **changed** from the project-page path
- `hub_url: "https://visionarycross.ca"`
- crosses (self is root-relative, the others are full URLs):
  ```yaml
  crosses:
    ruthwell:  ""                                    # self
    bewcastle: "https://bewcastle.visionarycross.ca"
    brussels:  "https://brussels.visionarycross.ca"
  ```

### 2. Shared theme change (this repo's `_includes`)

Re-add the `{{ site.hub_url }}` prefix to the hub-local links (frontmatter /
backmatter / home) in `nav.html`, `footer.html`, `header.html`. Empty `hub_url`
→ root-relative (hub itself); set `hub_url` → full URLs (the crosses, which are
now cross-origin). This mechanism was prototyped earlier and reverted; re-applying
it is a small, mechanical edit.

### 3. GitHub Pages settings + DNS

For EACH repo: **Settings → Pages → Custom domain** → its hostname (writes a
`CNAME` file). Then **Enforce HTTPS** once the cert issues.

DNS at the registrar:
- Apex `visionarycross.ca` → A records:
  `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`
  (and AAAA `2606:50c0:8000::153` … `8003::153` for IPv6).
- Each subdomain (`ruthwell`, `bewcastle`, `brussels`) → CNAME →
  `visionary-cross-project.github.io.`

### 4. Order of operations

1. Add DNS records first; let them propagate.
2. Set the custom domain per repo (start with the hub).
3. Commit the `_config.yml` + theme changes; push; let each repo rebuild.
4. Verify each subdomain loads and that cross-links + the shared nav work.
5. Enable "Enforce HTTPS" on each repo.

### Reverting
Undo the config edits (crosses back to root-relative, `baseurl` back to the
project path, drop `hub_url`, remove custom domains). Because all in-page links
are root-relative, the path model is the low-effort default to fall back to.
