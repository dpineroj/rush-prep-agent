# rush-prep-agent

A prep tablet for restaurant kitchens. It merges DoorDash and Grubhub order history, predicts when the next rushes will hit, and shows the shift lead how many of each dish to have ready, with the reasoning behind every number.

Built in one day by Team Dreadheads at [NOVA 2026](https://nova.scottylabs.org/), the generative-AI hackathon at Carnegie Mellon University, for the DoorDash sponsor track.

> All data in this repo is synthetic. The forecasts demonstrate the pipeline; they don't prove real-world accuracy.

## The problem

During a rush, orders arrive from several delivery apps at once, and the kitchen guesses what to start. Food either sits waiting or the rush begins with nothing ready. DoorDash asked teams to tackle order preparation timing and inventory forecasting; this project targets both.

## What it does

The shift lead checks one read-only tablet between rushes:

- **Next rush, rest of today, tomorrow:** each predicted busy window and the expected count of each dish.
- **Further ahead:** seven-day dish totals, labeled so nobody treats them as one pile to cook now.
- **Setup:** which past days the forecast used, forecast vs actual totals, and the limitations.

Rushes are found from the order history: runs of 15-minute slots expected at 1.5× the normal level or more. An optional AI agent reviews the forecast with read-only tools. It can only adjust it within 0.5×–2.0× and must give a reason for each change. Python recomputes every number, and without an API key the app falls back to the plain forecast and says so.

## How it's built

```text
DoorDash + Grubhub report CSVs (synthetic)
  -> importer -> PostgreSQL
  -> FastAPI sales API (/orders, /dishes/top, /rush-hours, /tablet)
  -> rush detection + optional OpenRouter agent with tool calls
  -> tablet frontend (Vite)
```

| Folder | What's there |
| --- | --- |
| `frontend/` | The tablet app (Vite, vanilla JS, Playwright tests) |
| `backend/nova_backend/` | Sales backend: PostgreSQL import, read API, rush detection, rush-hour agent, tablet endpoints |
| `backend/services/`, `backend/main.py` | A second agent prototype: ingredient-level prep sheet, batch timeline and backtest (see [AGENT_README](backend/AGENT_README.md)) |
| `data/` | Synthetic datasets and their generators |
| `wireframes/` | The clickable tablet wireframe used for the pitch |
| `docs/`, `handoffs/`, `presentation/` | Hackathon context, specs, design notes, demo script |
| `GLOSSARY.md` | Shared vocabulary: dish, rush, prep list, service window |

## Run it

With Docker and Node/npm installed:

```sh
./scripts/demo.sh
```

Then open http://127.0.0.1:5173. See [docs/local-demo.md](docs/local-demo.md) for details and tests. To enable the AI agent, copy `backend/.env.example` to `backend/.env` and add an [OpenRouter](https://openrouter.ai) key. Never commit `.env`.

## Team

| Member | Focus |
| --- | --- |
| David (dpineroj) | Recommendations spec; prep-planning agent (ingredient forecasts, batch timeline, backtest) |
| Ryan Joyce | Sales backend, data integration, tablet API |
| Kurt Schimmel | Tablet design, wireframe, frontend, presentation |
| Jon Okorie | Pre-rush stock check spec |
| morningjoe | Synthetic data, order analytics, rush-hour agent |

This repository is a snapshot of the team's hackathon repo, published with one commit instead of the full history.

## Limitations

- Synthetic data only; no real restaurant or provider account is connected.
- Delivery-platform reports lag a day or more, so plans are made from history, not live orders.
- The main demo forecasts dish sales, not ingredients or prep instructions; the ingredient-level prototype isn't connected to the tablet.
