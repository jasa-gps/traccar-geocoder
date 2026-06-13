# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## graphify

This project has a knowledge graph at graphify-out/ with god nodes, community structure, and cross-file relationships.

Rules:
- For codebase questions, first run `graphify query "<question>"` when graphify-out/graph.json exists. Use `graphify path "<A>" "<B>"` for relationships and `graphify explain "<concept>"` for focused concepts. These return a scoped subgraph, usually much smaller than GRAPH_REPORT.md or raw grep output.
- If graphify-out/wiki/index.md exists, use it for broad navigation instead of raw source browsing.
- Read graphify-out/GRAPH_REPORT.md only for broad architecture review or when query/path/explain do not surface enough context.
- After modifying code, run `graphify update .` to keep the graph current (AST-only, no API cost).

## Overview

Self-hosted reverse geocoder built from OpenStreetMap data. Two cooperating components share an on-disk binary index:

- **Builder** (`builder/`, C++17) — parses OSM PBF files with libosmium and emits a compact binary index keyed by S2 geometry cells.
- **Server** (`server/`, Rust + axum) — memory-maps the index and answers `/reverse` and `/snap` HTTP(S) queries with sub-millisecond latency.

The two components are coupled only through the binary file format. Any change to a struct layout, field, or file must be mirrored on both sides or the server reads garbage.

## Build & Run

```bash
# Builder (C++) — needs libosmium, protozero, s2geometry, zlib, bzip2, expat
mkdir build && cd build && cmake ../builder && make -j$(nproc)

# Server (Rust)
cargo build --release --manifest-path server/Cargo.toml

# Create the index from one or more PBF files
./build/build-index <data-dir>/index input.osm.pbf [input2.osm.pbf ...]

# Serve (HTTP)
./server/target/release/query-server <data-dir> [bind-address]
# Serve (auto HTTPS via Let's Encrypt)
./server/target/release/query-server <data-dir> --domain geocoder.example.com
```

Builder/server CLI flags: `--street-level`, `--admin-level` (S2 cell levels; must match between build and serve — defaults 17 and 10), `--search-distance` (server only, default 75 m), `--domain` / `--cache` (server HTTPS).

Docker (`Dockerfile`, 3 stages: C++ build → Rust build → slim runtime) is the primary distribution path. `entrypoint.sh` runs `auto` by default (download PBFs from `$PBF_URLS` → build index → serve); subcommands `build` and `serve` run individual phases. Index build is skipped if `<data-dir>/index/geo_cells.bin` already exists.

The cell-level/distance flags are also settable via Docker env vars (read by `entrypoint.sh`, but **not listed in the README env-var table**): `STREET_LEVEL` and `ADMIN_LEVEL` (passed to both `build` and `serve` — keep them identical across the two phases or lookups break) and `SEARCH_DISTANCE` (serve only).

There is no test suite in this repo. Verify changes by building the index from a small PBF (e.g. Monaco from Geofabrik) and querying `/reverse` and `/snap`.

## Architecture

### Builder (`builder/src/build_index.cpp`)

Two-pass OSM pipeline via libosmium `BuildHandler`:
1. Pass 1 collects node coordinates, place nodes, and admin/address metadata.
2. Pass 2 resolves ways (streets, interpolation lines, building addresses) against the cached coordinates.

Key stages: string interning (`StringPool`) to dedupe all text into `strings.bin`; S2 cell covering of streets/addresses/admin polygons; polygon simplification (Douglas–Peucker, `dp_simplify`); `resolve_semantic_levels` which maps each admin polygon to an output field; `write_index` which serializes the 14 binary files.

`builder/src/address_levels.hpp` is a flattened, per-country mapping of OSM `admin_level` / `place` tags to the 7 output semantics (None/Country/State/County/City/Suburb/Postcode), derived from Nominatim's `address-levels.json`. The `Semantic` enum here must stay in sync with the `SEM_*` constants in `server/src/main.rs`.

### Server (`server/src/main.rs` + `auth.rs`)

`main.rs` mmaps the index (`Index::load`), then for each query computes the S2 cell at the configured level plus neighbors, scans candidate entries, and picks the nearest street/address/interpolation. `reverse_geocode` builds a Nominatim-format response; `snap_to_road` returns the closest point on a street. Admin fields come from point-in-polygon tests against admin polygons, with a place-node fallback within ~20 km.

The `#[repr(C)]` structs (`WayHeader`, `AddrPoint`, `InterpWay`, `AdminPolygon`, etc.) and `SEM_*` constants at the top of `main.rs` **must byte-match** the C++ writer. When editing the format, change both sides and bump nothing silently.

`auth.rs` holds the API-key/user system: a JSON `Db` (`geocoder.json` in the data dir — note `entrypoint.sh` migrates it out of the old `index/` location), bcrypt password hashing, in-memory sessions, and per-key/per-IP rate limiting (`RateState`, atomic counters). It also serves the web dashboard (`server/src/web/*.html`) for creating the first admin account, generating API keys, and managing users. `/reverse` and `/snap` require a valid `key`; responses are `401` (bad key) / `429` (rate limited) / `404` (snap: nothing in range).

## Conventions

- Commit/branch/PR messages: short, lowercase, prefixed (`feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`). Example: `fix: handle empty token list`.
