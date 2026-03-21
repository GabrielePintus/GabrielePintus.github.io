---
layout: post
title: "How I sync my Obsidian vault for free"
date: 2026-03-21
categories: homelab
---

Obsidian Sync costs $8 a month. I already have a NAS at home, so I set up the same thing for free using Syncthing.

## The setup

Three machines are involved: my laptop, my desktop, and the NAS. All three are configured as **Send & Receive**, but the NAS is the hub. The laptop and desktop do not sync directly with each other. Each of them only syncs with the NAS, and the NAS propagates changes between them.

<style>
.sync-card {
  background: var(--bg-secondary, #f5f5f5);
  border: 1px solid #ddd;
  border-radius: 12px;
  padding: 32px;
  margin: 24px 0;
  max-width: 480px;
  margin-left: auto;
  margin-right: auto;
}
.sync-card h3 {
  font-size: 14px;
  font-weight: 500;
  color: #666;
  margin: 0 0 20px 0;
  letter-spacing: 0.02em;
}

.c-purple rect { fill: #EEEDFE; stroke: #534AB7; stroke-width: 0.5px; }
.c-purple text { fill: #3C3489; }
.c-teal   rect { fill: #E1F5EE; stroke: #0F6E56; stroke-width: 0.5px; }
.c-teal   text { fill: #085041; }
.c-coral  rect { fill: #FAECE7; stroke: #993C1D; stroke-width: 0.5px; }
.c-coral  text { fill: #712B13; }

@media (prefers-color-scheme: dark) {
  .sync-card { background: #242424; border-color: #333; }
  .sync-card h3 { color: #aaa; }
  .c-purple rect { fill: #3C3489; stroke: #AFA9EC; }
  .c-purple text { fill: #CECBF6; }
  .c-teal   rect { fill: #085041; stroke: #5DCAA5; }
  .c-teal   text { fill: #9FE1CB; }
  .c-coral  rect { fill: #712B13; stroke: #F0997B; }
  .c-coral  text { fill: #F5C4B3; }
}

.sync-label-main { font-size: 14px; font-weight: 500; font-family: system-ui, sans-serif; }
.sync-label-sub  { font-size: 12px; font-family: system-ui, sans-serif; }
.sync-label-note { font-size: 12px; fill: #999; font-family: system-ui, sans-serif; }
</style>
<div class="sync-card">
  <h3>Syncthing topology: Laptop ↔ NAS ↔ Desktop</h3>
  <svg width="100%" viewBox="0 0 640 280" xmlns="http://www.w3.org/2000/svg">
    <defs>
      <marker id="st-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
        <path d="M2 1L8 5L2 9" fill="none" stroke="#888" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
      </marker>
    </defs>
    <!-- Laptop -->
    <g class="c-purple">
      <rect x="30" y="90" width="150" height="64" rx="8"/>
      <text class="sync-label-main" x="105" y="115" text-anchor="middle" dominant-baseline="central">Laptop</text>
      <text class="sync-label-sub"  x="105" y="135" text-anchor="middle" dominant-baseline="central">Mobile device</text>
    </g>
    <!-- NAS -->
    <g class="c-teal">
      <rect x="245" y="90" width="150" height="64" rx="8"/>
      <text class="sync-label-main" x="320" y="115" text-anchor="middle" dominant-baseline="central">NAS</text>
      <text class="sync-label-sub"  x="320" y="135" text-anchor="middle" dominant-baseline="central">Always on hub</text>
    </g>
    <!-- Desktop -->
    <g class="c-coral">
      <rect x="460" y="90" width="150" height="64" rx="8"/>
      <text class="sync-label-main" x="535" y="115" text-anchor="middle" dominant-baseline="central">Desktop</text>
      <text class="sync-label-sub"  x="535" y="135" text-anchor="middle" dominant-baseline="central">Workstation</text>
    </g>
    <!-- Laptop <-> NAS -->
    <line x1="180" y1="117" x2="243" y2="117"
          stroke="#888" stroke-width="1.5" fill="none"
          marker-end="url(#st-arrow)" marker-start="url(#st-arrow)"/>
    <!-- NAS <-> Desktop -->
    <line x1="397" y1="127" x2="458" y2="127"
          stroke="#888" stroke-width="1.5" fill="none"
          marker-end="url(#st-arrow)" marker-start="url(#st-arrow)"/>
    <text class="sync-label-note" x="320" y="176" text-anchor="middle">no direct laptop ↔ desktop connection</text>
    <!-- Legend -->
    <line x1="30" y1="228" x2="80" y2="228"
          stroke="#888" stroke-width="1.5"
          marker-end="url(#st-arrow)" marker-start="url(#st-arrow)"/>
    <text class="sync-label-note" x="88" y="232">active sync link</text>
  </svg>
</div>

The laptop and desktop watch the same `~/Obsidian/` folder. On the NAS, the same vault lives in a dedicated dataset.

The NAS is always on. The laptop and desktop are not, and I rarely use both at the same time. This means that whenever I switch machines, the NAS has already received the latest changes from the other one. The two computers never need to be online at the same time to stay in sync.

## Avoiding conflicts

Obsidian writes a lot of cache and workspace state that has nothing to do with your notes. Syncing those files causes constant spurious conflicts. This is the small `.stignore` file I use in the root of the vault:

```
.obsidian/workspace.json
.obsidian/workspace-mobile.json
.obsidian/cache
.trash/
```

## Versioning

On the NAS, the Obsidian folder lives in a dedicated ZFS dataset (`MainPool/SyncThing`). TrueNAS takes automatic snapshots on three schedules:

| Schedule | Frequency |
|---|---|
| Hourly | Every hour, 00:00–23:59 |
| Daily | Every day at 00:00 |
| Weekly | Every Sunday at 00:00 |

On top of that, Syncthing's own **Staggered versioning** is enabled on the NAS copy for 365 days. This means that if I accidentally delete or overwrite a note, I can still recover it from the NAS, even if both the laptop and desktop have already synced the change. I can restore it either from Syncthing's version history or from a ZFS snapshot.

## Local only

Since all devices are on the same home network, I disabled **Global Discovery** and **Relaying**. The service is not exposed to the internet, and there is no dependency on Syncthing's external infrastructure. I use static peer addresses, so each device connects only to the NAS. When I am away with the laptop, I can still sync by connecting back home through the **WireGuard VPN** configured on my FritzBox router.
