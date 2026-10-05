<div align="center">

# Calcash

**Know the net profit of every sale before you list it.**

A free profit and pricing calculator for sellers on **Mercado Livre**, **Shopee** and **Amazon Brazil** — fees, taxes and shipping included.

![React](https://img.shields.io/badge/React-18-61dafb?logo=react&logoColor=white)
![Create React App](https://img.shields.io/badge/CRA-5-09d3ac?logo=createreactapp&logoColor=white)
![Framer Motion](https://img.shields.io/badge/Framer_Motion-11-0055ff?logo=framer&logoColor=white)
![Bilingual](https://img.shields.io/badge/i18n-PT--BR%20%7C%20EN-5850fe)
![License](https://img.shields.io/badge/license-ISC-38ae59)

<img src="docs/img/home-en.png" alt="Calcash home page — dark theme with a glassmorphism calculator preview" width="860">

</div>

---

## Why Calcash

Every marketplace charges differently: a commission that changes by category or price bracket, a fixed fee per item, shipping rules, plus the invoice tax (NF-e) you owe on each sale. Pricing by gut feeling is how sellers end up **selling at a loss without noticing**.

Calcash flips the question around. Instead of "how much will I make at this price?", you tell it **the margin you want** and it returns **the price to list** — and the profit you'll keep per sale.

- **Free and unlimited** — no sign-up, no login, no paywall.
- **One calculator per marketplace**, each applying that platform's own rules.
- **Bilingual** — Portuguese (Brazil) and English, picked automatically from the visitor's region.
- **Fast, static, dark** — a single-page React app with a polished dark UI and no backend required.

## Features

### Calculators

| Marketplace | Route | What it models |
| --- | --- | --- |
| **Mercado Livre** | `/Calculadora` | Classic / Premium commission (you enter the %), NF-e tax, selling expenses, seller-paid shipping, and the fixed per-unit cost on items under R$ 79 (≈ R$ 6.75). |
| **Shopee** | `/CalculadoraShopee` | The 2026 commission table, auto-detected by price bracket (commission % + fixed fee per item). No fee fields to fill in — it picks the right bracket for you and shows which one it applied. |
| **Amazon Brazil** | `/CalculadoraAmazon` | Category commission (you enter the %), NF-e tax, selling expenses, and DBA shipping estimated from price tier and product weight. |

All three share the same pricing model:

```text
price  = (product cost + selling expenses + fixed fees) / (1 − (tax % + commission % + margin %) / 100)
profit = price × margin %
```

Because the fixed fee can depend on the price itself (e.g. Shopee brackets, Mercado Livre's under-R$ 79 cost, Amazon's DBA tiers), the calculators resolve it iteratively until the price settles inside a consistent bracket.

> **Heads-up:** marketplace fee tables change often and some vary by weight, dimensions or state of origin. Results are **estimates** — every calculator links to the official fee page, and Calcash is not a substitute for an accountant.

### Home page

A dark, motion-driven landing page built around a small design system (see [`.claude/skills/frontend-design/SKILL.md`](.claude/skills/frontend-design/SKILL.md)):

- **Hero** — headline, CTAs and a glassmorphism preview of a worked example (R$ 50 cost, 8 % tax, 20 % fee, 15 % margin → list at **R$ 87.72**, keep **R$ 13.16**).
- **Tools** — one card per marketplace, straight into its calculator.
- **Social proof** — usage stats and seller quotes.
- **FAQ** — accessible accordion.
- **Footer** — closing call to action and contact links.

The navbar is fixed and scroll-aware (glass effect once you scroll), with a full-screen mobile menu.

### Language and region switch

The site is fully translated into **Portuguese (Brazil)** and **English**.

1. **First visit** — the visitor's country is detected from their IP address. **Brazil → Portuguese; anywhere else → English.**
2. **Manual override** — the `PT | EN` switch in the navbar always wins, and the choice is remembered in `localStorage`.
3. **Shareable links** — append `?lang=en` or `?lang=pt` to any URL to force a language (and remember it).
4. **Graceful fallbacks** — if geolocation is unavailable, the browser's language is used. The UI never blocks on the network: it renders immediately with a best guess and corrects itself if the detected region disagrees.

Country detection tries, in order:

1. `/api/region` — a tiny [Vercel serverless function](api/region.js) that reads the `x-vercel-ip-country` header. First-party, no third party sees visitor IPs.
2. [`api.country.is`](https://api.country.is) — public fallback for hosts without that function (local dev, GitHub Pages, Docker).
3. `navigator.language`.

The detected country is cached, so the lookup runs once per visitor. `<html lang>`, the page title and the meta description follow the active language.

> The calculators model **Brazilian** marketplaces, so amounts are always in Brazilian reais (R$), in both languages.

## Getting started

**Requirements:** Node.js 16+ and npm.

```bash
git clone https://github.com/thaianramalho/Calcash.git
cd Calcash
npm install
npm start
```

The app runs at <http://localhost:3000> (set `PORT=3001` if that port is taken).

| Command | What it does |
| --- | --- |
| `npm start` | Dev server with hot reload. |
| `npm run build` | Production build into `build/`. |
| `npm test` | Test runner (Create React App / Jest). |

> **Build gotcha:** CI platforms such as Vercel build with `CI=true`, and Create React App treats **every ESLint warning as an error** in that mode. Before pushing, verify with:
>
> ```bash
> CI=true npm run build
> ```
>
> (PowerShell: `$env:CI="true"; npm run build`)

### Docker

```bash
docker build -t calcash .
docker run -p 3000:3000 calcash
```

### Deploying

Calcash is a static single-page app and deploys anywhere that serves `build/` with an SPA fallback to `index.html`. It is set up for **Vercel**, which also gives you real IP-based region detection through `api/region.js`. On other hosts, detection transparently falls back to the public geolocation API and then to the browser language.

## Project structure

```text
api/
  region.js                  Vercel function: visitor country from the edge header
public/
  img/                       Marketplace logos
src/
  i18n/
    pt.js · en.js            All user-facing copy, one dictionary per language
    index.js                 I18nProvider, useI18n(), region detection
    LangSwitch.js            PT | EN switch used by both navbars
  componentes/
    pagina-inicial/          Home page (composes the sections below)
    Navbar/ · Navbar2/       Home navbar · calculator-page navbar
    inicio/                  Hero
    ferramentas/             Marketplace cards
    prova-social/            Stats and quotes
    ajuda/                   FAQ
    footer/                  CTA + footer
    Calculadora/             Mercado Livre, Shopee and Amazon calculators
      CalcParts.js           Shared field, title, buttons and result cards
  lib/                       Motion presets and the scroll-reveal helper
  styles/tokens.css          Design tokens (colors, type scale, 8px grid)
```

## Contributing

### Edit copy or add a language

All text lives in `src/i18n/`. To change wording, edit `pt.js` and `en.js` (keep both files on the same key structure). To add a language, create a new dictionary with that structure, register it in `DICTIONARIES` in `src/i18n/index.js`, and add a button to `LangSwitch.js`.

### Update a marketplace's fees

Fee logic sits at the top of each calculator in `src/componentes/Calculadora/`:

- Shopee — the `FAIXAS_SHOPEE` bracket table.
- Amazon — the `freteAmazon()` function (DBA tiers).
- Mercado Livre — the under-R$ 79 fixed cost inside `calcular()`.

If you change a table, update the matching disclaimer in the dictionaries so users know which year the numbers come from.

### Design

Before touching the UI, read the design system in `.claude/skills/frontend-design/SKILL.md` and reuse the tokens in `src/styles/tokens.css` rather than hard-coding colors or spacing.

## Roadmap and known gaps

- The usage stats and seller quotes in the social-proof section are **placeholder content** — replace them with real data before launch.
- Mercado Livre and Amazon commissions are entered manually (they vary by category); a category picker would remove that step.
- An experimental Mercado Livre listing lookup (`/BuscarAnuncio`) is routed but not linked from the UI, and it is not translated.
- No automated tests yet for the pricing formulas.

## Disclaimer

Calcash is an independent project and is **not affiliated with Mercado Livre, Shopee or Amazon**. Marketplace names and logos belong to their respective owners. Figures are estimates for pricing guidance only.

## License

ISC — see `package.json`.
