# Copilot Workspace Instructions

## Project Overview

**Soc Ops** is a Social Bingo game for in-person mixers. Players find people who match questions on a 5×5 board and get 5 in a row to win. Built with Python (FastAPI + Jinja2 + HTMX) — no JavaScript framework, no build step.

## Key Commands

```bash
uv run uvicorn app.main:app --reload --host 0.0.0.0 --port 8000  # Dev server
uv run pytest                                                       # Run tests
uv run ruff check .                                                 # Lint
uv run ruff format .                                                # Format
```

Requires Python 3.13+ and [uv](https://docs.astral.sh/uv/). Run `uv sync` to install dependencies.

## Architecture

```
app/
├── main.py          # FastAPI routes & HTMX endpoints
├── models.py        # Pydantic models: GameState (StrEnum), BingoSquareData, BingoLine
├── game_logic.py    # Pure functions: generate_board(), toggle_square(), check_bingo()
├── game_service.py  # GameSession dataclass — owns mutable game state per session
├── data.py          # QUESTIONS list and FREE_SPACE constant
├── templates/       # Jinja2 templates
│   ├── base.html
│   ├── home.html
│   └── components/  # bingo_board, bingo_modal, game_screen, start_screen
└── static/
    ├── css/app.css  # Custom Tailwind-like utility classes (no CDN)
    └── js/htmx.min.js
tests/
├── test_api.py         # FastAPI TestClient (httpx) endpoint tests
└── test_game_logic.py  # Pure unit tests for game_logic.py
```

### Request flow

1. All state lives in `GameSession` (in-memory, keyed by a session cookie UUID).
2. HTMX sends `POST` requests; routes return **partial HTML** (a single component template).
3. Full-page loads return `home.html` which wraps the active component.

## Conventions

- **Python style**: snake_case, type hints on all public functions, Pydantic models are `frozen=True`.
- **Immutability in game logic**: `toggle_square()` and `generate_board()` return new lists — never mutate in place.
- **No JavaScript**: all interactivity via HTMX attributes (`hx-post`, `hx-target`, `hx-swap`).
- **CSS**: use utility classes from `app/static/css/app.css`. Add new utilities there when needed — do **not** add inline styles or import external CSS. See [css-utilities.instructions.md](instructions/css-utilities.instructions.md).
- **Templates**: keep logic out of templates. Pass computed values (e.g. `winning_square_ids`) from the route or session property.
- **Session middleware**: `SessionMiddleware` (starlette) with a signed cookie. Session data is keyed by `session_id` (UUID hex).

## Pre-commit Checklist

- [ ] `uv run ruff check .` — zero errors
- [ ] `uv run pytest` — all tests pass
- [ ] Type hints present on new functions
- [ ] No unused imports or variables

## Pitfalls to Avoid

- **Don't add a JS framework** — HTMX is the only JS dependency. New interactions should use `hx-*` attributes.
- **Don't import Tailwind or other CSS** — only `app.css` is loaded. Check that any new utility class you need exists there first.
- **Don't mutate `BingoSquareData`** — it is a frozen Pydantic model; use `.model_copy(update={...})`.
- **Don't store state outside `GameSession`** — `game_logic.py` functions are stateless by design.
- **`QUESTIONS` must have ≥ 24 entries** — `generate_board()` samples exactly 24 (the 5×5 grid minus the free space).

## Extending the App

- **New questions/themes**: edit `app/data.py` — replace or extend `QUESTIONS`.
- **New routes**: add to `app/main.py`; return a partial template for HTMX endpoints.
- **New game states**: extend `GameState` (StrEnum) in `models.py` and handle in `GameSession`.
- **Frontend redesign**: edit templates in `app/templates/` and utilities in `app/static/css/app.css`. See [frontend-design.instructions.md](instructions/frontend-design.instructions.md).

## Workshop

Step-by-step lab guides live in [`workshop/`](../workshop/). Start with `00-overview.md`. Reference solutions are in [`.solutions/`](../.solutions/) — consult them when stuck, not as a shortcut.
