# Non-European CRMA Strategic Projects Map

Interactive map of the 13 non-European strategic mining projects selected by the European Commission on 25 March 2025 under the [Critical Raw Materials Act (CRMA)](https://single-market-economy.ec.europa.eu/sectors/raw-materials/areas-specific-interest/critical-raw-materials/strategic-projects-under-crma/selected-projects_en).

Built amateurly so anyone can see the status, environmental, and community situation of each project at a glance.
I used Ollama to help with coding and logic, checked and refined everything by hand.

## What this is

The EU's Critical Raw Materials Act (CRMA) designates strategic mining and processing projects globally to secure Europe's supply of minerals needed for the green and digital transition. This map covers the **13 projects located outside Europe** from the first selection round.

Each project is tracked across four dimensions:
- **Overview** — who is promoting it, which companies are involved, what it does
- **Production** — what volumes are projected and for which applications
- **EIA & Licensing** — environmental assessment and permit status
- **Community** — documented conflicts with indigenous peoples or local communities

**Status definitions used:**
| Status | Meaning |
|---|---|
| `active` | Project is currently advancing (permitting, construction, or production) |
| `halted` | Project has been suspended but not formally cancelled |
| `cancelled` | Project licences have been cancelled or the project formally abandoned |

---

## Repository structure

```
crma-non-eu-projects/
├── index.html      # Map interface — all layout, styling, and logic
├── data.js         # Project data — edit this to update project information
└── README.md       # This file
```

---

## How the data file works (`data.js`)

The file declares a single global JavaScript array called `PROJECTS`. Each entry is one project object. Here is what each field means:

```js
{
  id: 6,                          // Unique number shown on the map marker
  name: "Kobaloni Energy Zambia", // Full project name
  status: "active",               // active | halted | cancelled | pending
  type: "Processing",             // Extraction | Processing | Integrated
  country: "Zambia",              // Country where project is located
  lat: -15.4961583,               // Latitude (decimal degrees)
  lng: 28.3846278,                // Longitude (decimal degrees)

  // Visual styling on map
  mat: "Cobalt",                  // Material label — must match a filter pill
  neon: "#2979ff",                // Bright colour used for glow and legend dot
  fill: "#1a56cc",                // Darker fill used for marker background

  // Community conflict
  conflict: true,                 // true = red ring on marker + shows in conflict filter
  conflictDetail: "...",          // Full text shown in the Community tab
  conflictLink: "https://...",    // URL for source link (leave "" to hide button)

  // Detail panel content
  promoter: "...",                // Lead company or entity
  linked: "...",                  // Linked companies and shareholders
  notes: "...",                   // Background notes on the promoter
  desc: "...",                    // Project description shown in Overview tab
  applications: "...",            // End uses of the material
  eia: "...",                     // EIA status + details (shown in EIA tab)
  lic: "...",                     // Licensing status + details (shown in EIA tab)
  prod: "...",                    // Production target (shown in Production tab)

  web: "https://..."              // Project website URL
}
```

At the bottom of the file, the status label lookup table:
```js
var SL = {active:"Active", halted:"Halted", cancelled:"Cancelled"};
```

**To update a project:** open `data.js`, find the object with the right `id`, change the relevant fields, save, and push. GitHub Pages will rebuild in about 60 seconds.

**To add a project:** copy an existing object, give it a new `id`, fill in all fields, and add it to the array. Also add a filter pill in `index.html` if the material is new.

**Material colour reference:**

| Material | `neon` (glow) | `fill` (marker) |
|---|---|---|
| Graphite | `#00e5ff` | `#007a99` |
| Nickel | `#76ff03` | `#3d8a00` |
| Cobalt | `#2979ff` | `#1a56cc` |
| Lithium / Boron | `#d500f9` | `#8800b3` |
| Copper | `#ff6d00` | `#b34d00` |
| Rare Earth Elements | `#ff1744` | `#b30000` |
| Tungsten | `#ffab00` | `#997a00` |

---

## How the HTML file works (`index.html`)

The file is self-contained except for loading `data.js`. It has three sections:

### 1. `<style>` — Visual design

All CSS is in one `<style>` block at the top. Key sections:
- **CSS variables** (`:root`) — colours, sidebar width, header height
- **Header** — dark brown bar with EU flag, title, search input, hamburger button
- **Sidebar** — right-side panel with filter pills and scrollable project list
- **Detail panel** — floating card bottom-left with tabbed content (Overview / Production / EIA / Community)
- **Markers** — `.mk`, `.mkc` (conflict ring), `.mkn` (normal)
- **Responsive breakpoints** — `@media (max-width: 900px)` for tablet, `@media (max-width: 600px)` for mobile (sidebar becomes full-screen drawer, detail becomes bottom sheet)

### 2. `<body>` — HTML structure

```
<header>
  EU logo | Site title | Search bar | Hamburger button
</header>

<div id="map">          ← Leaflet map fills the whole screen behind everything
</div>

<div class="cnt">       ← "13 projects" counter badge (top-left of map)
</div>

<div class="sb-reopen"> ← Thin arrow tab to reopen sidebar when closed
</div>

<aside class="sidebar"> ← Right-side filter + project list panel
  Filter pills (All / Graphite / Nickel / ... / Community Conflict)
  Scrollable list of project cards
</aside>

<div class="detail">    ← Floating detail card (appears when you click a project)
  Coloured top bar
  Project name + location + status badges
  Four tab buttons (Overview / Production / EIA & Licensing / Community)
  Tab content area (filled dynamically by JavaScript)
</div>
```

### 3. `<script>` — Application logic

The entire app logic runs inside a self-executing function `(function(){ ... })()` to avoid polluting the global scope. Steps in order:

**Map setup**
```js
var map = L.map('map', { center:[20,10], zoom:2 });
// Layer 1: ESRI satellite imagery (real satellite photos, no API key needed)
L.tileLayer('https://server.arcgisonline.com/.../World_Imagery/...').addTo(map);
// Layer 2: CartoDB label overlay (country/city names on top of satellite)
L.tileLayer('https://{s}.basemaps.cartocdn.com/light_only_labels/...').addTo(map);
```

**Marker creation**
Loops through `PROJECTS`, creates a Leaflet `divIcon` for each one (a styled `<div>` with the project ID number), adds it to the map, and stores a reference in `MK[p.id]` so markers can be shown/hidden during filtering.

**`applyFilters()` — the core filter function**
Called every time a pill is clicked or the search input changes. Filters `PROJECTS` by:
1. Material match — `p.mat.toLowerCase().indexOf(mat)` (uses `indexOf` not `===` so "Nickel, Cobalt" correctly matches the "Nickel" pill)
2. Conflict toggle — only shows projects where `p.conflict === true`
3. Search text — checks name, country, material, and promoter fields

Then re-renders the sidebar card list and shows/hides map markers to match.

**`openDetail(p)` — opens the detail card**
Sets the coloured top bar, name, location, status badges, stores `p.id` on the panel element, calls `fillTabs(p)` to render all four tab contents, switches to the Overview tab, highlights the sidebar card, and animates the panel open.

**`fillTabs(p)` — builds tab content**
Constructs HTML strings for all four tabs using the project's data fields. EIA and licensing status labels are colour-coded (green = granted/completed, amber = pending, red = cancelled/not approved).

**`switchTab(t)` — tab navigation**
Toggles the `.on` class on tab buttons and content panes to show the selected tab.

**Sidebar and detail close buttons**
The ✕ on the sidebar adds `.hidden` class (CSS `transform: translateX(100%)` slides it off-screen). The ✕ on the detail panel removes `.open` class (CSS `display:none`).

---

## Dependencies (all loaded from CDN, no installation needed)

| Library | Version | Purpose |
|---|---|---|
| [Leaflet.js](https://leafletjs.com) | 1.9.4 | Interactive map |
| [Montserrat](https://fonts.google.com/specimen/Montserrat) | Google Fonts | Typography |
| ESRI World Imagery | — | Satellite tile layer |
| CartoDB Light Labels | — | Country/city name overlay |

No build tools, no npm, no framework. Open the files in any text editor to make changes.


