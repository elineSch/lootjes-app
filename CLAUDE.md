# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

**Activiteitenpot** is een gezinsactiviteiten-app gebouwd als single-file HTML/CSS/JS webapplicatie. De app is beschikbaar via GitHub Pages en gebruikt Supabase als gedeelde database zodat alle gezinsleden real-time dezelfde data zien.

- **Live URL:** https://elinesch.github.io/lootjes-app/
- **GitHub repo:** github.com/elinesch/lootjes-app
- **Supabase project:** ynaczklprjncwkpsjmoi

## Ontwikkeling

De app bestaat uit één bestand: `index.html`. Er is geen build-stap nodig.

**Preview starten:**
```
npx serve C:\Users\eline\Documents\LootjesApp
```
Of via Claude Code's launch.json (naam: `lootjes-app`, poort 3000).

**Deployen:** Het bestand handmatig uploaden via github.com/elinesch/lootjes-app → index.html → potlood-icoontje → Ctrl+A → plakken → Commit changes.

## Architectuur

Alles zit in `index.html` in drie secties:

### CSS (`:root` variabelen)
Kleuren worden beheerd via CSS custom properties:
- `--a1` / `--a2`: de twee gradientkleuren
- `--grad`: het kleurverloop (gebruikt op header, knoppen, tabs)
- `--bg` / `--bg2` / `--bg3`: achtergrondkleuren
- Gebruiker kan kleuren aanpassen via de instellingen-tab; worden opgeslagen in Supabase

### HTML (tabbladen)
Vier tabbladen: `tab-trekken`, `tab-activiteiten`, `tab-geschiedenis`, `tab-instellingen`. Plus twee overlays: het PIN-scherm (toegang app, code: **1802**) en het instellingen-PIN-scherm (code: **0210**).

### JavaScript (datamodel)
Data wordt opgeslagen in Supabase (tabel: `activities`) als key-value paren én lokaal als cache in `localStorage`:

| Supabase key | Inhoud |
|---|---|
| `acts` | Array van activiteiten |
| `members` | Array van gezinsleden |
| `history` | Array van getrokken activiteiten |
| `settings` | Object met kleuren, naam, knoptekst |
| `currentDraw` | Huidig getrokken lootje (null als leeg) |

**LocalStorage keys:** `loot-acts5`, `loot-hist4`, `loot-settings7`, `loot-members2`

> Wil je een dataschema-wijziging doorvoeren die bestaande data overschrijft? Verhoog dan het versienummer in de relevante key (bijv. `loot-acts5` → `loot-acts6`).

### Supabase integratie
- `save()` slaat altijd op in zowel localStorage als Supabase
- `syncFromSupabase()` haalt data op bij laden; Supabase data heeft prioriteit over localStorage
- Real-time updates via `db.channel('avonturenpot')` — wijzigingen van andere apparaten verschijnen direct

### Activiteiten datastructuur
```js
{
  name: string,
  emoji: string,
  seasons: ['altijd'] | ['lente','zomer','herfst','winter'] // combinaties mogelijk
  dur: 'kort' | 'u12' | 'halve' | 'hele',
  freq: 'altijd' | 'week' | 'maand' | 'kwartaal' | 'jaar' | 'eenmalig',
  blocked: boolean // optioneel, tijdelijk blokkeren
}
```

### Filtersysteem (Trekken-tab)
- **Seizoensfilter:** multi-select via `activeSeasons[]` array. Lege array = alles. Activiteiten met `seasons:['altijd']` komen altijd mee bij elk seizoensfilter.
- **Duurtijdfilter:** exact match op `dur` waarde
- **Frequentielogica:** `freqAvailable(act)` controleert of een activiteit beschikbaar is op basis van de laatste keer dat hij gedaan is (via `history` met `timestamp`)

## Gezinsleden
- Dennis 🤠, Eline 🤩, Ayla 😍, Roan 😎
- Beurt-systeem: `settings.currentMember` wijst naar index in `members[]`; na "Gedaan!" wordt automatisch doorgeschoven
