# README — Build this with Claude Code

This kit (`CLAUDE.md`, `SPEC.md`, this file) is a ready handoff for **Claude Code** to build an interactive 3D + economics web app for the project. Follow the steps in order.

---

## Prerequisites
- A **Claude paid subscription or API key** (Claude Code uses your Claude account).
- A terminal. (Mac: Terminal · Windows: PowerShell · Linux: your shell.)

## 1 · Install Claude Code
**Recommended — native installer (no Node.js needed):**
- macOS / Linux / WSL: `curl -fsSL https://claude.ai/install.sh | bash`
- Windows (PowerShell): `irm https://claude.ai/install.ps1 | iex`
- (macOS/Linux alt) Homebrew: `brew install --cask claude-code`

**Or via npm** (requires **Node.js 18+** installed first): `npm install -g @anthropic-ai/claude-code`

Then verify: `claude --version`. Docs: https://docs.claude.com/en/docs/claude-code/overview

> Note: you'll still want **Node.js 18+** anyway, because the *app itself* is a Vite/React project that runs on Node. Get it from nodejs.org (LTS) if you don't have it.

## 2 · Export your architect's model to GLB
The 3D view needs a web-friendly **glTF/GLB**. From your software:
- **Blender:** File → Export → glTF 2.0 (.glb). Easiest if your model is already here.
- **SketchUp:** install a glTF export extension, or export to Collada/FBX → import to Blender → export GLB.
- **Rhino:** File → Export → `.gltf/.glb` (native in recent versions).
- **Revit:** no native glTF — use a glTF exporter plugin, or export FBX → Blender → GLB. (Twinmotion/Datasmith is another route.)
- **3ds Max:** use the Babylon/glTF exporter.

**Optimize it for the web** (big models stall on phones): aim for **under ~30–50 MB**. Run it through a compressor:
```
npx gltf-transform optimize input.glb site.glb --compress draco
```
(or `npx gltfpack -i input.glb -o site.glb -cc`). Bake/limit textures to ~2K.

## 3 · Set up the project folder
```
mkdir wellness-garden && cd wellness-garden
# copy CLAUDE.md, SPEC.md, README.md into this folder
mkdir -p public/models public/renders
cp /path/to/site.glb public/models/site.glb
cp /path/to/*.jpg     public/renders/      # your renders & photos
```

## 4 · Build it with Claude Code
From inside the folder, run `claude`, then give it these prompts in sequence (it reads `CLAUDE.md` + `SPEC.md` automatically):

1. *"Read CLAUDE.md and SPEC.md. Scaffold the Vite + React + TypeScript + Tailwind project with three.js / react-three-fiber / drei installed, and the app shell with the three tabs. Get `npm run dev` working."*
2. *"Build View 1, the interactive Site Map, from the zones table in SPEC.md. Make zones clickable with a detail panel."*
3. *"Build View 2, the Economics calculator, exactly to the formulas and defaults in SPEC.md, with live recalculation and the KPI band."*
4. *"Build View 3, the 3D Walkthrough, using my model at public/models/site.glb and the Viewer3D reference in SPEC.md. Add the hotspots and a placeholder if the model is missing."*
5. *"Wire renders from public/renders into the site-map panels and the 3D hotspots. Polish the styling to match the design tokens."*
6. *"Deploy to Vercel and give me the URL."*

Tips:
- Drop a reference image into the folder and say *"match this look"* when polishing.
- If a hotspot sits in the wrong spot, tell Claude Code the rough position or ask it to add a small dev overlay that prints the camera/cursor 3D coordinates so you can dial them in.
- Commit to git early (`git init`) so you can roll back: *"initialize git and commit after each working step."*

## 5 · Iterate
Everything's now natural-language editable: *"add a day/night lighting toggle to the 3D view," "add a 360° auto-rotate," "let me filter the economics by year-1 ramp vs stabilized,"* etc.

---

### What Claude Code can and can't do here
- ✅ Builds the entire app, the viewer, the calculator, hotspots, styling, and deploy.
- ✅ Loads and optimizes your GLB, wires your renders.
- ⚠️ It does **not** create 3D geometry — that's your architect's model (which you have). Fidelity of the walkthrough = fidelity of that model.
