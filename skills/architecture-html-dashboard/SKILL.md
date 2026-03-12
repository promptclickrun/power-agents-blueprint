---
name: architecture-html-dashboard
description: >
  Generates a self-contained, interactive HTML dashboard for Power Platform and
  Copilot Studio architecture documentation. Features Mermaid diagrams, dark theme,
  responsive layout, clickable stat cards with slide-in detail drawers, and tabbed content.
---

# Skill: Architecture HTML Dashboard Template

> Generates a self-contained, interactive HTML dashboard for Power Platform and Copilot Studio architecture documentation. Features Mermaid diagrams, dark theme, responsive layout, clickable stat cards with slide-in detail drawers, and tabbed content.

---

## When to Use This Skill

Use this skill when generating an architecture analysis as a **single self-contained HTML file**. The template provides:

- Dark-themed responsive dashboard (mobile, tablet, desktop, print)
- Hero section with solution name, badge, and description
- Clickable stat cards → slide-in modal drawers with expandable detail cards
- Five tabbed content panels: Architecture, ERD, Data Flows, Components, Notes
- Mermaid v11+ diagrams with proper hidden-tab rendering
- Zoom controls for diagrams
- Full print stylesheet

---

## Critical Rendering Rules

### Mermaid in Hidden Tabs

**Mermaid cannot render into `display: none` containers.** Follow this exact sequence:

1. All `.tab-content` elements start **visible** (no `display: none` on page load)
2. Initialize Mermaid with `startOnLoad: false`
3. Call `mermaid.run()` explicitly
4. In the `.then()` callback, call `switchTab('arch')` to hide inactive tabs
5. Tab CSS uses `.tab-content.hidden { display: none; }` — the `hidden` class is only added AFTER Mermaid renders

```html
<!-- CSS: Tabs start visible, hidden class added by JS after render -->
.tab-content { animation: fadeIn 0.3s ease; }
.tab-content.hidden { display: none; }
.tab-content.active { display: block; }
```

```javascript
// Initialize without auto-start
mermaid.initialize({ startOnLoad: false, theme: 'dark', /* ... */ });

// Render all diagrams while visible, THEN hide inactive tabs
mermaid.run().then(() => {
    switchTab('arch'); // This hides non-active tabs
});
```

### ERD Syntax Rules (Mermaid v11+)

- Use ONLY `string`, `int`, `float`, `boolean` as attribute types
- Do NOT use `guid`, `datetime`, `uniqueidentifier` — map them to `string`
- Only `PK` and `FK` are valid annotations — do NOT use `UK`, `NN`, `UNIQUE`
- Relationship labels MUST be in double quotes: `ENTITY_A ||--o{ ENTITY_B : "label"`
- Valid relationship markers: `||` (exactly one), `|o` (zero or one), `}|` (one or more), `}o` (zero or more)
- Test every entity name, attribute name, and relationship for valid syntax

### Architecture Diagram Rules (Mermaid Flowchart)

- Use `graph TB` for top-to-bottom layout
- Group components into `subgraph` blocks with clear labels
- Use `-->` for control flow, `-.->` for data/query flow
- Use `classDef` for color coding each layer
- Use `\n` for line breaks in node labels (not `<br>`)
- Keep node IDs short (e.g., `FA_S1`, `T_IN`) — put descriptive text in labels

---

## Full HTML Template

Replace all `{{PLACEHOLDER}}` values with real data from your analysis. The template is the structural skeleton — populate every section with solution-specific content.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{SOLUTION_NAME}} — Architecture & ERD</title>
    <script src="https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.min.js"></script>
    <style>
        /* === RESET & VARIABLES === */
        *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
        :root {
            --bg: #0a0e1a;
            --surface: #111827;
            --surface-alt: #1a2236;
            --border: #1e2d4a;
            --border-hover: #3b5998;
            --text: #e2e8f0;
            --text-muted: #8896b3;
            --accent-blue: #3b82f6;
            --accent-purple: #8b5cf6;
            --accent-green: #10b981;
            --accent-orange: #f59e0b;
            --accent-red: #ef4444;
            --accent-cyan: #06b6d4;
            --glow-blue: rgba(59, 130, 246, 0.15);
            --glow-purple: rgba(139, 92, 246, 0.15);
        }

        body {
            font-family: 'Segoe UI', -apple-system, BlinkMacSystemFont, sans-serif;
            background: var(--bg); color: var(--text); line-height: 1.6; overflow-x: hidden;
        }

        body::before {
            content: ''; position: fixed; top: 0; left: 0; right: 0; bottom: 0;
            background:
                radial-gradient(ellipse 80% 50% at 20% 20%, rgba(59,130,246,0.06) 0%, transparent 60%),
                radial-gradient(ellipse 60% 40% at 80% 80%, rgba(139,92,246,0.06) 0%, transparent 60%);
            pointer-events: none; z-index: 0;
        }

        .container { max-width: 1400px; margin: 0 auto; padding: 2rem 1.5rem; position: relative; z-index: 1; }

        /* === HERO === */
        .hero { text-align: center; padding: 3rem 1rem 2rem; margin-bottom: 2rem; }
        .hero-badge {
            display: inline-flex; align-items: center; gap: 0.5rem;
            padding: 0.4rem 1rem; background: linear-gradient(135deg, var(--glow-blue), var(--glow-purple));
            border: 1px solid var(--border); border-radius: 999px;
            font-size: 0.8rem; color: var(--text-muted); text-transform: uppercase; letter-spacing: 0.1em; margin-bottom: 1.5rem;
        }
        .hero-badge .dot { width: 6px; height: 6px; background: var(--accent-green); border-radius: 50%; animation: pulse 2s ease-in-out infinite; }
        @keyframes pulse { 0%, 100% { opacity: 1; } 50% { opacity: 0.4; } }
        .hero h1 {
            font-size: clamp(2rem, 5vw, 3.2rem); font-weight: 700;
            background: linear-gradient(135deg, #fff 0%, #94a3b8 100%);
            -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text; margin-bottom: 0.75rem;
        }
        .hero p { font-size: 1.1rem; color: var(--text-muted); max-width: 700px; margin: 0 auto; }

        /* === STAT CARDS (clickable) === */
        .stats-row { display: grid; grid-template-columns: repeat(auto-fit, minmax(160px, 1fr)); gap: 1rem; margin-bottom: 2.5rem; }
        .stat-card {
            background: var(--surface); border: 1px solid var(--border); border-radius: 12px;
            padding: 1.25rem; text-align: center; transition: all 0.3s; cursor: pointer; position: relative; overflow: hidden;
        }
        .stat-card:hover { border-color: var(--border-hover); transform: translateY(-3px); box-shadow: 0 8px 24px rgba(0,0,0,0.3); }
        .stat-card::after {
            content: 'Click for details →'; position: absolute; bottom: 0; left: 0; right: 0;
            padding: 0.3rem; font-size: 0.65rem; color: var(--text-muted); background: var(--surface-alt);
            opacity: 0; transform: translateY(100%); transition: all 0.25s;
        }
        .stat-card:hover::after { opacity: 1; transform: translateY(0); }
        .stat-value { font-size: 2rem; font-weight: 700; line-height: 1; margin-bottom: 0.35rem; }
        .stat-label { font-size: 0.78rem; color: var(--text-muted); text-transform: uppercase; letter-spacing: 0.05em; }

        /* Color each stat card's value */
        .stat-card:nth-child(1) .stat-value { color: var(--accent-blue); }
        .stat-card:nth-child(2) .stat-value { color: var(--accent-purple); }
        .stat-card:nth-child(3) .stat-value { color: var(--accent-green); }
        .stat-card:nth-child(4) .stat-value { color: var(--accent-orange); }
        .stat-card:nth-child(5) .stat-value { color: var(--accent-cyan); }
        .stat-card:nth-child(6) .stat-value { color: var(--accent-red); }

        /* === MODAL DRAWER === */
        .modal-overlay {
            position: fixed; top: 0; left: 0; right: 0; bottom: 0;
            background: rgba(0,0,0,0.65); backdrop-filter: blur(4px);
            z-index: 1000; opacity: 0; pointer-events: none; transition: opacity 0.3s;
        }
        .modal-overlay.open { opacity: 1; pointer-events: all; }
        .modal-drawer {
            position: fixed; top: 0; right: 0; bottom: 0; width: min(560px, 90vw);
            background: var(--surface); border-left: 1px solid var(--border); z-index: 1001;
            transform: translateX(100%); transition: transform 0.35s cubic-bezier(0.16, 1, 0.3, 1);
            display: flex; flex-direction: column; box-shadow: -8px 0 40px rgba(0,0,0,0.4);
        }
        .modal-drawer.open { transform: translateX(0); }
        .modal-header {
            display: flex; align-items: center; gap: 0.75rem;
            padding: 1.25rem 1.5rem; border-bottom: 1px solid var(--border); background: var(--surface-alt); flex-shrink: 0;
        }
        .modal-header-icon { width: 40px; height: 40px; border-radius: 10px; display: flex; align-items: center; justify-content: center; font-size: 1.3rem; flex-shrink: 0; }
        .modal-header h2 { font-size: 1.1rem; font-weight: 600; flex: 1; }
        .modal-header .modal-count { padding: 0.2rem 0.65rem; border-radius: 999px; font-size: 0.75rem; font-weight: 600; }
        .modal-close {
            width: 32px; height: 32px; display: flex; align-items: center; justify-content: center;
            background: transparent; border: 1px solid var(--border); border-radius: 6px;
            color: var(--text-muted); cursor: pointer; font-size: 1.1rem; transition: all 0.2s; font-family: inherit;
        }
        .modal-close:hover { background: rgba(239,68,68,0.15); border-color: var(--accent-red); color: var(--accent-red); }
        .modal-body { flex: 1; overflow-y: auto; padding: 1.5rem; }
        .modal-body::-webkit-scrollbar { width: 6px; }
        .modal-body::-webkit-scrollbar-thumb { background: var(--border); border-radius: 3px; }

        /* Detail cards inside modal */
        .detail-card { background: var(--bg); border: 1px solid var(--border); border-radius: 12px; margin-bottom: 1rem; overflow: hidden; transition: border-color 0.2s; }
        .detail-card:hover { border-color: var(--border-hover); }
        .detail-card-header { display: flex; align-items: center; gap: 0.75rem; padding: 1rem 1.25rem; cursor: pointer; transition: background 0.2s; }
        .detail-card-header:hover { background: rgba(255,255,255,0.02); }
        .detail-card-icon { width: 32px; height: 32px; border-radius: 8px; display: flex; align-items: center; justify-content: center; font-size: 0.95rem; flex-shrink: 0; }
        .detail-card-header h4 { font-size: 0.9rem; font-weight: 600; flex: 1; }
        .detail-card-header .detail-tag { padding: 0.15rem 0.55rem; border-radius: 999px; font-size: 0.65rem; font-weight: 500; text-transform: uppercase; letter-spacing: 0.04em; }
        .detail-card-chevron { color: var(--text-muted); transition: transform 0.2s; font-size: 0.8rem; }
        .detail-card.expanded .detail-card-chevron { transform: rotate(90deg); }
        .detail-card-body { max-height: 0; overflow: hidden; transition: max-height 0.35s ease, padding 0.35s ease; padding: 0 1.25rem; }
        .detail-card.expanded .detail-card-body { max-height: 600px; padding: 0 1.25rem 1.25rem; border-top: 1px solid var(--border); }
        .detail-row { display: flex; padding: 0.5rem 0; font-size: 0.83rem; border-bottom: 1px solid rgba(255,255,255,0.04); }
        .detail-row:last-child { border-bottom: none; }
        .detail-row-label { color: var(--text-muted); min-width: 120px; flex-shrink: 0; }
        .detail-row-value { color: var(--text); }
        .detail-row-value code { background: rgba(59,130,246,0.12); color: var(--accent-blue); padding: 0.1rem 0.4rem; border-radius: 4px; font-size: 0.78rem; font-family: 'Cascadia Code', 'Fira Code', monospace; }

        /* === TABS === */
        .tabs { display: flex; gap: 0.25rem; margin-bottom: 0; border-bottom: 1px solid var(--border); overflow-x: auto; -webkit-overflow-scrolling: touch; }
        .tab-btn {
            padding: 0.75rem 1.5rem; background: transparent; border: none; color: var(--text-muted);
            font-size: 0.9rem; font-weight: 500; cursor: pointer; border-bottom: 2px solid transparent;
            transition: all 0.25s; white-space: nowrap; font-family: inherit;
        }
        .tab-btn:hover { color: var(--text); background: rgba(255,255,255,0.03); }
        .tab-btn.active { color: var(--accent-blue); border-bottom-color: var(--accent-blue); }
        .tab-content { animation: fadeIn 0.3s ease; }
        .tab-content.hidden { display: none; }
        .tab-content.active { display: block; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(8px); } to { opacity: 1; transform: translateY(0); } }

        /* === DIAGRAM PANELS === */
        .diagram-panel { background: var(--surface); border: 1px solid var(--border); border-radius: 16px; margin-top: 1.5rem; overflow: hidden; }
        .diagram-panel-header { display: flex; align-items: center; justify-content: space-between; padding: 1rem 1.5rem; border-bottom: 1px solid var(--border); background: var(--surface-alt); }
        .diagram-panel-header h3 { font-size: 1rem; font-weight: 600; display: flex; align-items: center; gap: 0.5rem; }
        .diagram-panel-body { padding: 2rem; overflow-x: auto; min-height: 400px; display: flex; align-items: center; justify-content: center; }
        .diagram-panel-body .mermaid { width: 100%; }
        .diagram-panel-body .mermaid svg { max-width: 100%; height: auto; }
        .legend { display: flex; flex-wrap: wrap; gap: 1rem; padding: 1rem 1.5rem; border-top: 1px solid var(--border); background: var(--surface-alt); }
        .legend-item { display: flex; align-items: center; gap: 0.4rem; font-size: 0.78rem; color: var(--text-muted); }
        .legend-dot { width: 10px; height: 10px; border-radius: 3px; }

        /* Zoom controls */
        .zoom-controls { display: flex; gap: 0.25rem; }
        .zoom-btn {
            width: 32px; height: 32px; display: flex; align-items: center; justify-content: center;
            background: var(--surface); border: 1px solid var(--border); border-radius: 6px;
            color: var(--text-muted); cursor: pointer; font-size: 1rem; font-family: inherit; transition: all 0.2s;
        }
        .zoom-btn:hover { border-color: var(--border-hover); color: var(--text); }

        /* === INVENTORY CARDS === */
        .inventory-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(340px, 1fr)); gap: 1.25rem; margin-top: 1.5rem; }
        .inv-card { background: var(--surface); border: 1px solid var(--border); border-radius: 12px; overflow: hidden; transition: border-color 0.3s, box-shadow 0.3s; }
        .inv-card:hover { border-color: var(--border-hover); box-shadow: 0 4px 24px rgba(0,0,0,0.2); }
        .inv-card-header { padding: 1rem 1.25rem; border-bottom: 1px solid var(--border); display: flex; align-items: center; gap: 0.75rem; }
        .inv-icon { width: 36px; height: 36px; border-radius: 8px; display: flex; align-items: center; justify-content: center; font-size: 1.1rem; flex-shrink: 0; }
        .inv-card-header h4 { font-size: 0.95rem; font-weight: 600; }
        .inv-card-header .inv-tag { margin-left: auto; padding: 0.15rem 0.6rem; border-radius: 999px; font-size: 0.7rem; font-weight: 500; text-transform: uppercase; letter-spacing: 0.04em; }
        .inv-card-body { padding: 1rem 1.25rem; }
        .inv-card-body ul { list-style: none; }
        .inv-card-body li { padding: 0.4rem 0; font-size: 0.85rem; color: var(--text-muted); border-bottom: 1px solid rgba(255,255,255,0.04); display: flex; align-items: flex-start; gap: 0.5rem; }
        .inv-card-body li:last-child { border-bottom: none; }
        .inv-card-body li::before { content: '›'; color: var(--accent-blue); font-weight: 700; flex-shrink: 0; margin-top: -1px; }
        .inv-card-body li strong { color: var(--text); font-weight: 500; }

        /* === DATA FLOW === */
        .flow-section { margin-top: 1.5rem; }
        .flow-card { background: var(--surface); border: 1px solid var(--border); border-radius: 16px; padding: 2rem; margin-bottom: 1.5rem; }
        .flow-card h3 { font-size: 1.1rem; margin-bottom: 1.25rem; display: flex; align-items: center; gap: 0.5rem; }
        .flow-steps { position: relative; padding-left: 2rem; }
        .flow-steps::before { content: ''; position: absolute; left: 11px; top: 0; bottom: 0; width: 2px; background: linear-gradient(180deg, var(--accent-blue), var(--accent-purple), var(--accent-green)); border-radius: 1px; }
        .flow-step { position: relative; padding: 0.6rem 0 0.6rem 1rem; font-size: 0.9rem; color: var(--text-muted); }
        .flow-step::before { content: ''; position: absolute; left: -1.97rem; top: 0.85rem; width: 10px; height: 10px; border-radius: 50%; border: 2px solid var(--accent-blue); background: var(--bg); }
        .flow-step.decision::before { border-color: var(--accent-orange); background: var(--accent-orange); }
        .flow-step.success::before { border-color: var(--accent-green); background: var(--accent-green); }
        .flow-step.error::before { border-color: var(--accent-red); background: var(--accent-red); }
        .flow-step strong { color: var(--text); }
        .flow-branch { margin: 0.5rem 0 0.5rem 1rem; padding: 0.75rem 1rem; border-radius: 8px; font-size: 0.85rem; }
        .flow-branch.success-branch { background: rgba(16,185,129,0.08); border-left: 3px solid var(--accent-green); color: #6ee7b7; }
        .flow-branch.error-branch { background: rgba(239,68,68,0.08); border-left: 3px solid var(--accent-red); color: #fca5a5; }

        /* === NOTES === */
        .notes-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); gap: 1.25rem; margin-top: 1.5rem; }
        .note-card { background: var(--surface); border: 1px solid var(--border); border-radius: 12px; padding: 1.5rem; }
        .note-card h4 { font-size: 0.95rem; margin-bottom: 0.75rem; display: flex; align-items: center; gap: 0.5rem; }
        .note-card ul { list-style: none; }
        .note-card li { padding: 0.35rem 0; font-size: 0.85rem; color: var(--text-muted); line-height: 1.5; }

        /* === RESPONSIVE === */
        @media (max-width: 768px) {
            .container { padding: 1rem; }
            .hero { padding: 2rem 0.5rem 1rem; }
            .stats-row { grid-template-columns: repeat(3, 1fr); gap: 0.5rem; }
            .stat-card { padding: 0.75rem; }
            .stat-value { font-size: 1.5rem; }
            .stat-card::after { display: none; }
            .inventory-grid { grid-template-columns: 1fr; }
            .diagram-panel-body { padding: 1rem; }
            .tabs { gap: 0; }
            .tab-btn { padding: 0.65rem 1rem; font-size: 0.82rem; }
        }
        @media (max-width: 480px) {
            .stats-row { grid-template-columns: repeat(2, 1fr); }
            .legend { gap: 0.5rem; }
            .notes-grid { grid-template-columns: 1fr; }
            .modal-drawer { width: 100vw; }
        }
        @media print {
            body { background: #fff; color: #000; }
            .tabs, .zoom-controls { display: none; }
            .tab-content { display: block !important; page-break-inside: avoid; }
            .diagram-panel { border: 1px solid #ccc; }
            .modal-overlay, .modal-drawer { display: none !important; }
        }
    </style>
</head>
<body>

<!-- Modal elements -->
<div class="modal-overlay" id="modalOverlay" onclick="closeModal()"></div>
<div class="modal-drawer" id="modalDrawer">
    <div class="modal-header" id="modalHeader"></div>
    <div class="modal-body" id="modalBody"></div>
</div>

<div class="container">
    <div class="hero">
        <div class="hero-badge"><span class="dot"></span> Production Architecture</div>
        <h1>{{SOLUTION_NAME}}</h1>
        <p>{{SOLUTION_DESCRIPTION}}</p>
    </div>

    <div class="stats-row">
        <!-- Each stat card calls openModal('key') -->
        <div class="stat-card" onclick="openModal('{{KEY}}')">
            <div class="stat-value">{{VALUE}}</div>
            <div class="stat-label">{{LABEL}}</div>
        </div>
        <!-- Repeat for each stat... -->
    </div>

    <div class="tabs" role="tablist">
        <button class="tab-btn active" data-tab="arch" role="tab">Architecture</button>
        <button class="tab-btn" data-tab="erd" role="tab">ERD</button>
        <button class="tab-btn" data-tab="flow" role="tab">Data Flows</button>
        <button class="tab-btn" data-tab="inventory" role="tab">Components</button>
        <button class="tab-btn" data-tab="notes" role="tab">Notes</button>
    </div>

    <!-- All tab-content divs start WITHOUT .hidden — Mermaid renders first -->
    <div class="tab-content active" id="tab-arch">
        <!-- Architecture diagram panel with zoom controls and legend -->
    </div>
    <div class="tab-content" id="tab-erd">
        <!-- ERD diagram panel with zoom controls and legend -->
    </div>
    <div class="tab-content" id="tab-flow">
        <!-- Data flow cards with timeline steps and decision branches -->
    </div>
    <div class="tab-content" id="tab-inventory">
        <!-- Inventory grid cards -->
    </div>
    <div class="tab-content" id="tab-notes">
        <!-- Notes grid: Strengths, Considerations, Dependencies, Security, Env Vars, ROI -->
    </div>
</div>

<script>
    // Modal data object — one key per stat card
    const modalData = {
        // Each key contains: icon, iconBg, iconColor, title, count, countBg, countColor, items[]
        // Each item: icon, iconBg, iconColor, name, tag, tagBg, tagColor, details[]
        // Each detail: { label, value } where value can contain <code> and <strong> HTML
    };

    function openModal(key) {
        const data = modalData[key]; if (!data) return;
        document.getElementById('modalHeader').innerHTML = `
            <div class="modal-header-icon" style="background:${data.iconBg}; color:${data.iconColor}">${data.icon}</div>
            <h2>${data.title}</h2>
            <span class="modal-count" style="background:${data.countBg}; color:${data.countColor}">${data.count}</span>
            <button class="modal-close" onclick="closeModal()" title="Close">✕</button>`;
        document.getElementById('modalBody').innerHTML = data.items.map((item, i) => `
            <div class="detail-card ${i === 0 ? 'expanded' : ''}" onclick="toggleCard(this)">
                <div class="detail-card-header">
                    <div class="detail-card-icon" style="background:${item.iconBg}; color:${item.iconColor}">${item.icon}</div>
                    <h4>${item.name}</h4>
                    <span class="detail-tag" style="background:${item.tagBg}; color:${item.tagColor}">${item.tag}</span>
                    <span class="detail-card-chevron">▶</span>
                </div>
                <div class="detail-card-body">
                    ${item.details.map(d => `<div class="detail-row"><span class="detail-row-label">${d.label}</span><span class="detail-row-value">${d.value}</span></div>`).join('')}
                </div>
            </div>`).join('');
        document.getElementById('modalOverlay').classList.add('open');
        document.getElementById('modalDrawer').classList.add('open');
    }
    function closeModal() {
        document.getElementById('modalOverlay').classList.remove('open');
        document.getElementById('modalDrawer').classList.remove('open');
    }
    function toggleCard(el) { el.classList.toggle('expanded'); }
    document.addEventListener('keydown', e => { if (e.key === 'Escape') closeModal(); });

    // Tab switching
    document.querySelectorAll('.tab-btn').forEach(btn => {
        btn.addEventListener('click', () => switchTab(btn.dataset.tab));
    });
    function switchTab(tabName) {
        document.querySelectorAll('.tab-btn').forEach(b => b.classList.toggle('active', b.dataset.tab === tabName));
        document.querySelectorAll('.tab-content').forEach(tc => {
            const id = tc.id.replace('tab-', '');
            tc.classList.toggle('hidden', id !== tabName);
            tc.classList.toggle('active', id === tabName);
        });
    }

    // Zoom
    const zoomLevels = {};
    function zoom(id, delta) {
        const el = document.getElementById(id);
        if (!el) return;
        zoomLevels[id] = Math.min(2.5, Math.max(0.3, (zoomLevels[id] || 1) + delta));
        el.style.transform = `scale(${zoomLevels[id]})`;
        el.style.transformOrigin = 'top center';
    }

    // Mermaid init — CRITICAL: render while all tabs visible, then hide
    mermaid.initialize({
        startOnLoad: false, theme: 'dark',
        themeVariables: { primaryColor: '#1e3a5f', primaryBorderColor: '#3b82f6', primaryTextColor: '#e2e8f0', lineColor: '#3b82f6', secondaryColor: '#1a2236', tertiaryColor: '#111827' },
        flowchart: { htmlLabels: true, curve: 'basis', padding: 15, nodeSpacing: 40, rankSpacing: 60 },
        er: { layoutDirection: 'TB', minEntityWidth: 180, minEntityHeight: 40, entityPadding: 12, fontSize: 13, useMaxWidth: true }
    });
    mermaid.run().then(() => { switchTab('arch'); });
</script>
</body>
</html>
```

---

## Template Placeholders Reference

| Placeholder | Description | Example |
|---|---|---|
| `{{SOLUTION_NAME}}` | Solution display name | `Manufacturing HR Copilot` |
| `{{SOLUTION_DESCRIPTION}}` | 1-2 sentence overview | `AI-powered intake processing...` |
| `{{KEY}}` | Modal data key for stat card | `agents`, `aiModels`, `flows` |
| `{{VALUE}}` | Stat card numeric value | `2`, `6`, `13` |
| `{{LABEL}}` | Stat card label text | `AI Models`, `Cloud Flows` |

---

## Content Population Guide

### Stat Cards
Create one `<div class="stat-card">` per major metric. Typical cards: Agents, AI Models, Flows, MCPs/Connectors, Channels, Data Entities. Each must have a corresponding `modalData` key.

### modalData Structure
```javascript
const modalData = {
    agents: {
        icon: '🤖', iconBg: 'rgba(59,130,246,0.15)', iconColor: '#3b82f6',
        title: 'Copilot Studio Agents', count: '2 agents',
        countBg: 'rgba(59,130,246,0.15)', countColor: '#3b82f6',
        items: [
            {
                icon: '🎯', iconBg: 'rgba(139,92,246,0.15)', iconColor: '#8b5cf6',
                name: 'Agent Name', tag: 'PRIMARY', tagBg: 'rgba(139,92,246,0.15)', tagColor: '#8b5cf6',
                details: [
                    { label: 'Schema', value: '<code>cr665_agent</code>' },
                    { label: 'Purpose', value: 'Routes incoming requests...' }
                ]
            }
        ]
    }
};
```

### Tabs
- **Architecture**: Mermaid `graph TB` flowchart in a `.diagram-panel`
- **ERD**: Mermaid `erDiagram` in a `.diagram-panel`
- **Data Flows**: `.flow-card` elements with `.flow-steps` timeline
- **Components**: `.inventory-grid` with `.inv-card` elements per category
- **Notes**: `.notes-grid` with `.note-card` elements (Strengths, Considerations, Dependencies, Security, ROI)
