# SPEC.md — Build Specification

The product is a 3-view web app (see `CLAUDE.md` for stack & design tokens). Build in this order: (1) shell + Site Map, (2) Economics, (3) 3D Walkthrough. Acceptance criteria are in **bold**.

---

## App shell
- Top bar: project name "The Wellness Garden", address line, and three tabs: **Site Map · 3D Walkthrough · Economics**.
- Remember the active tab between reloads. Responsive down to ~380px.

---

## VIEW 1 — Site Map (interactive 2D plan)
A top-down schematic of the ~170 × 173 ft lot. Render as SVG. Each zone is a colored, clickable block; clicking opens a side/bottom panel with the zone's name, area, description, and a small gallery of renders.

**Streets (label the edges):** Knox Street = top/north (main public entrance) · N Central Expressway = right/east (back: bath garden + signage) · Armstrong Avenue = left/west (parking access + members' entrance) · service alley = bottom/south.

**Zones** (`src/data/zones.ts`):
| id | label | area | entrance | color token | notes |
|----|-------|------|----------|-------------|-------|
| grocer | Grocer + Market Hall · Café + Coffee + Tonic | ~4,200 sf | public, off Knox | clay | open to public |
| courtyard | Social Courtyard — terrace · lounge · firepit · daybeds · communal tables · padel viewing | ~4,000 sf | — | sand | the heart |
| padel | Padel Court (glass, regulation) | ~2,300 sf | from courtyard | dark green #2F5233 | spectator-facing |
| bath | Bath House (2-story) — lockers · showers · indoor sauna · reformer + yoga above | ~4,000 sf | members, off Armstrong | teal | |
| garden | Private Bathing Garden — sauna · cold plunge · warm soak · fire | ~3,000 sf | from bath | garden #9FB089 (dashed) | screened on N Central edge |
| parking | Surface Parking | ~18 stalls | off Armstrong | grey #B7AE9F | back of house |

**Acceptance:** **all zones render in correct relative positions, each is clickable, and the panel shows that zone's info + at least a placeholder image slot.**

---

## VIEW 2 — Economics (live calculator)
Port the established model. Inputs on the left, live results + revenue/cost breakdown bars on the right; a sticky KPI band (Revenue, Costs, EBITDA + margin, Rent burden %, Payback).

**Inputs & defaults** (`src/data/econDefaults.ts`):
- members 800 · memberPrice 250 · memberBasis = "year" (toggle year/month → multiplier 1 or 12)
- bathVisitsDay 100 · bathPrice 42
- classesDay 8 · classAttend 11 · classPrice 38
- padelHrsDay 8 · padelRate 80
- grocerDay 6000 · grocerCOGSpct 55
- treatmentsYr 300000 · eventsYr 400000 · days 360
- laborPct 32 · svcSuppliesPct 10 · rent 900000 · utilities 240000 · amenities 120000
- marketingPct 4 · processingPct 2.5 · insurance 80000 · maintenance 100000 · tech 60000 · admin 120000
- buildout 3000000

**Formulas:**
```
memberRev  = members * memberPrice * (basis==="month" ? 12 : 1)
bathRev    = bathVisitsDay * bathPrice * days
classRev   = classesDay * classAttend * classPrice * days
padelRev   = padelHrsDay * padelRate * days
grocerRev  = grocerDay * days
totalRev   = memberRev + bathRev + classRev + padelRev + grocerRev + treatmentsYr + eventsYr

svcRev        = bathRev + classRev + padelRev + treatmentsYr
cogs          = grocerRev * grocerCOGSpct/100
svcSupplies   = svcRev * svcSuppliesPct/100
labor         = totalRev * laborPct/100
marketing     = totalRev * marketingPct/100
processing    = totalRev * processingPct/100
fixed         = rent + utilities + amenities + insurance + maintenance + tech + admin
totalCost     = cogs + svcSupplies + labor + marketing + processing + fixed

ebitda   = totalRev - totalCost
margin   = ebitda / totalRev
rentPct  = rent / totalRev
payback  = ebitda > 0 ? buildout / ebitda : null   // years
```
**Acceptance:** **changing any input updates all outputs instantly; base case yields ≈ $6.0M revenue, ≈ $560k EBITDA (~9% margin), ~5-yr payback; the member basis toggle to "month" swings EBITDA past ~$1.9M.**
Include a disclaimer: illustrative model, not financial advice or a forecast.

---

## VIEW 3 — 3D Walkthrough
Load `/public/models/site.glb` and let the user orbit/pan/zoom. Add hotspots (clickable markers at 3D coordinates) that open a card with a render + room name. Graceful placeholder if the GLB is absent.

**Hotspots** (`src/data/hotspots.ts`): array of `{ id, label, position:[x,y,z], render:"/renders/xxx.jpg", body }`. Start with bathhouse, courtyard, padel, grocer. The owner will fine-tune coordinates against the real model.

**Reference component** (`src/components/Viewer3D.tsx`) — start from this:
```tsx
import { Suspense } from "react";
import { Canvas } from "@react-three/fiber";
import { OrbitControls, useGLTF, Html, Environment, Bounds, useProgress } from "@react-three/drei";

function Model({ url }: { url: string }) {
  const { scene } = useGLTF(url);
  return <primitive object={scene} />;
}

function Hotspot({ position, label, onClick }: any) {
  return (
    <Html position={position} center distanceFactor={10}>
      <button onClick={onClick}
        className="rounded-full bg-[#3E6B66] text-white text-xs px-3 py-1 shadow-lg hover:scale-105 transition">
        {label}
      </button>
    </Html>
  );
}

function Loader() {
  const { progress } = useProgress();
  return <Html center className="text-[#4A4338] text-sm">Loading model… {Math.round(progress)}%</Html>;
}

export default function Viewer3D({ hotspots, onSelect }: any) {
  const url = "/models/site.glb";
  return (
    <Canvas camera={{ position: [12, 8, 12], fov: 45 }} style={{ height: "70vh" }}>
      <ambientLight intensity={0.6} />
      <directionalLight position={[10, 15, 10]} intensity={1.1} />
      <Suspense fallback={<Loader />}>
        <Bounds fit clip margin={1.2}>
          <Model url={url} />
        </Bounds>
        {hotspots.map((h: any) => (
          <Hotspot key={h.id} position={h.position} label={h.label} onClick={() => onSelect(h)} />
        ))}
        <Environment preset="city" />
      </Suspense>
      <OrbitControls makeDefault enableDamping />
    </Canvas>
  );
}
useGLTF.preload("/models/site.glb");
```
*(If `site.glb` is missing, wrap `<Model>` in an error boundary that renders a labeled placeholder box instead — don't crash.)*

**Acceptance:** **the model loads and is orbitable on desktop + touch; ≥3 hotspots open a render card; missing model shows a placeholder.**

---

## File structure (target)
```
src/
  App.tsx                # shell + tabs
  components/SiteMap.tsx
  components/Economics.tsx
  components/Viewer3D.tsx
  components/ZonePanel.tsx
  data/zones.ts
  data/hotspots.ts
  data/econDefaults.ts
  lib/format.ts          # currency/number helpers
public/
  models/site.glb        # owner provides
  renders/*.jpg          # owner provides
```
