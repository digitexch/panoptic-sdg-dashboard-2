# Panoptic SDG Dashboard

![Panoptic scoring framework](docs/scoring_framework.svg)

*The scoring framework: principles → weighted sub-principles → evaluator scores → weighted points → project score and band, and how the points land on the SDGs. Regenerated on every build (`docs/scoring_framework.svg`).*

```
data/Panoptic_SDG_database_department.xlsx   ← the only place data is edited
        │  python3 build.py
        ▼
panoptic_dashboard_data.json            ← generated feed (never edit by hand)
        │  injected into dashboard.template.html at __DATA__
        ▼
panoptic-sdg-dashboard.html             ← generated page (open in a browser, or publish)
```

## Files
| File | Role | Edit? |
|---|---|---|
| `data/Panoptic_SDG_database_department.xlsx` | Source of truth: principles, sub-principles, weights, target links, indicators, prescripts, projects and evaluator scores | **Yes** |
| `build.py` | Reads the workbook, validates it, writes the JSON, builds the page. `FEATURES` at the top switches releases on or off | Switches; feed shape |
| `dashboard.template.html` | The dashboard UI (layout, 3D scene, panel, matrix). Contains one `__DATA__` placeholder | Only to change the UI |
| `panoptic_dashboard_data.json` | Generated feed; also usable by other tools (Power BI, a web app) | No |
| `panoptic-sdg-dashboard.html` | Generated, self-contained dashboard (not committed; the workflow builds it) | No |
| `assets/sdg/sdg-01.png … sdg-17.png` | SDG icons, injected into the page at `__ASSETS__` (kept out of the JSON feed). SDG icons: United Nations; use follows the UN SDG icon guidelines | Only to replace icons |
| `docs/build_framework.py` → `docs/scoring_framework.svg` | The framework diagram, drawn from the feed on every build | No |

## Routine changes (workbook only)
- **Add or remove a link:** add/delete a row on *Target links*. Fill Link ID (`PP2.1|11.5`), Principle, Sub-principle, SDG, Target, Target short name, Target text, Alignment (`Primary`/`Secondary`). Then add that target's indicators on *Indicator links* if it is new.
- **Change primary ↔ secondary:** edit the Alignment cell.
- **Change a weight:** *Sub-principles* column E. Weights must still total 100.
- **Edit wording or prescripts:** *Sub-principles* (criterion, question, rationale, Prescripts).

- **Discuss coverage gaps:** *SDG target coverage* lists all 169 targets with the sub-principles linked to each (P/S) and a Coverage status (Primary link / Secondary only / Gap); *Coverage summary* rolls it up per SDG. Record the team's view in the amber columns; a gap only closes when a row is added on *Target links*.
- **Add a project assessment:** a row on *Projects* (new ID, e.g. `P002`), then one row per evaluator and sub-principle on *Project scores* (score 1–5 in steps of 0.5, optional comment). The dashboard's *New assessment* screen can copy these rows for you.

Then run `python3 build.py`, commit the workbook together with the regenerated `panoptic_dashboard_data.json`, and push (see *Publishing* below).

## Release switches (`FEATURES` in build.py)
- `indicators: False` — SDG indicators stay in the workbook but are left out of the feed and the UI. Set to `True` for the release that brings them back.
- `scoring: True` — the *Score projects* mode.

## Scoring rules
Score = Σ (score ÷ 5 × weight) ÷ Σ weights scored, with the department weights (total 100). Several evaluators: scores are averaged per sub-principle. 80–100% bankable portfolio · 50–79% return to step 2 · below 50% rework. *Where the score lands* shares each sub-principle's points across its linked SDG targets (primary links count double); the SDG arcs in the model rise in proportion.

## Checks the build runs (it stops and lists problems)
Weights total 100 · every link's sub-principle exists · scores are 1–5 in half steps, each evaluator scores a sub-principle once and the project exists on *Projects* · Alignment is Primary or Secondary · Link ID = sub|target · target sits under its SDG · no duplicate links · every target has text and at least one indicator · 17 SDGs present.

## JSON feed shape
- `principles[]` id, name, short, w (sum of its sub weights)
- `subs[]` id, p, name, short, w, crit, q, why, pres
- `sdgs[]` n, name, title, c (official colour)
- `targets{code}` g (SDG), s (short name), t (UNSD text)
- `links[]` s (sub), t (target), a (`P`/`S`), l (department label)
- `inds{target}`, `also{indicator}` — only when `indicators` is on
- `projects[]` id, name, dept, loc, sector, stage, lead, date, evals[] {n: evaluator, s: {sub: score}, c: {sub: comment}}
- `meta` framework, source, prepared, workbook, features

## Publishing on GitHub

`.github/workflows/publish.yml` does the publishing:

- **Pull request:** runs `build.py`, so a change with broken data cannot be merged. Nothing is published.
- **Push to `main`:** builds the page and deploys it to GitHub Pages at `https://<owner>.github.io/<repo>/`.
- **Public build:** the workflow sets `PANOPTIC_PUBLIC=1`, which replaces evaluator names with *Evaluator 1…n* and leaves out evaluator comments and the project lead. Set it to `"0"` in the workflow to publish them.

### Day-to-day
1. `git pull`
2. Edit `data/Panoptic_SDG_database_department.xlsx` (close Excel afterwards)
3. `python3 build.py` (fix anything it lists)
4. `git add -A && git commit -m "What changed and why" && git push` (or open a pull request)
5. The site updates a minute or two after the push to `main` (watch the *Actions* tab)

### Visibility
The workbook in this repository contains evaluator names and comments. Keep the **repository private**. GitHub Pages from a private repository needs GitHub Pro, Team or Enterprise. The published site itself is public unless the organisation uses GitHub Enterprise Cloud with private Pages.

Setup: `pip install -r requirements.txt` (Python 3.10+).
