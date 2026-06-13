# Graph Report - .  (2026-06-13)

## Corpus Check
- Corpus is ~12,357 words - fits in a single context window. You may not need a graph.

## Summary
- 262 nodes · 532 edges · 11 communities
- Extraction: 97% EXTRACTED · 3% INFERRED · 0% AMBIGUOUS · INFERRED: 15 edges (avg confidence: 0.82)
- Token cost: 85,000 input · 5,765 output

## Community Hubs (Navigation)
- [[_COMMUNITY_Query Server & Index Types|Query Server & Index Types]]
- [[_COMMUNITY_OSM Index Builder (C++)|OSM Index Builder (C++)]]
- [[_COMMUNITY_Auth & User Management|Auth & User Management]]
- [[_COMMUNITY_Request Handlers & Deployment|Request Handlers & Deployment]]
- [[_COMMUNITY_Address Resolution & Formatting|Address Resolution & Formatting]]
- [[_COMMUNITY_Admin Level Mapping (C++)|Admin Level Mapping (C++)]]
- [[_COMMUNITY_Rate Limiting|Rate Limiting]]
- [[_COMMUNITY_AddrPoint Schema|AddrPoint Schema]]
- [[_COMMUNITY_AdminPolygon Schema|AdminPolygon Schema]]
- [[_COMMUNITY_Interpolation Ways|Interpolation Ways]]
- [[_COMMUNITY_PlaceNode Schema|PlaceNode Schema]]

## God Nodes (most connected - your core abstractions)
1. `Db` - 17 edges
2. `Index` - 16 edges
3. `reverse_geocode()` - 16 edges
4. `snap_to_road()` - 16 edges
5. `BuildHandler` - 12 edges
6. `String` - 12 edges
7. `dashboard()` - 12 edges
8. `get_session_cookie()` - 11 edges
9. `login_submit()` - 11 edges
10. `delete_token()` - 11 edges

## Surprising Connections (you probably didn't know these)
- `docker-compose geocoder service` --references--> `serve()`  [INFERRED]
  docker-compose.yml → entrypoint.sh
- `Per-country admin_level mapping` --references--> `Nominatim response format`  [INFERRED]
  builder/src/address_levels.hpp → README.md
- `format_rules (per-country format)` --semantically_similar_to--> `Per-country admin_level mapping`  [INFERRED] [semantically similar]
  server/src/main.rs → builder/src/address_levels.hpp
- `Traccar Geocoder (project overview)` --references--> `build-index main (OSM two-pass pipeline)`  [EXTRACTED]
  README.md → builder/src/build_index.cpp
- `point_in_polygon (Rust ray casting)` --semantically_similar_to--> `point_in_polygon_f64`  [INFERRED] [semantically similar]
  server/src/main.rs → builder/src/build_index.cpp

## Import Cycles
- 1-file cycle: `server/src/auth.rs -> server/src/auth.rs`
- 1-file cycle: `server/src/main.rs -> server/src/main.rs`

## Hyperedges (group relationships)
- **Two-stage build-then-serve geocoder pipeline** — build_index_main, build_index_binary_index_format, main_index, entrypoint_serve [INFERRED 0.85]
- **C++/Rust binary struct ABI contract** — build_index_adminpolygon, main_adminpolygon, build_index_addrpoint, main_addrpoint [INFERRED 0.85]
- **API request auth and rate-limit flow** — main_reverse_geocode, main_authorize, auth_validate_token, auth_check_rate [EXTRACTED 1.00]

## Communities (11 total, 0 thin omitted)

### Community 0 - "Query Server & Index Types"
Cohesion: 0.08
Nodes (49): Address, AdminResult, ConnectInfo, Cow, Db, FnMut, Mmap, Query (+41 more)

### Community 1 - "OSM Index Builder (C++)"
Cohesion: 0.07
Nodes (48): Area, Semantic, string, handler::Handler, Map, Node, pair, S2CellId (+40 more)

### Community 2 - "Auth & User Management"
Cohesion: 0.21
Nodes (30): Form, HashMap, HeaderMap, HeaderName, Path, Arc, Extension, Option (+22 more)

### Community 3 - "Request Handlers & Deployment"
Cohesion: 0.09
Nodes (31): check_rate (rate limiter), create_token handler, dashboard handler, Db (auth database), login_submit handler, RateLimiter / RateState, auth::router (dashboard routes), Db::validate_token (+23 more)

### Community 4 - "Address Resolution & Formatting"
Cohesion: 0.11
Nodes (22): Per-country admin_level mapping, lookup_admin, lookup_place, Semantic enum (address levels), add_admin_polygon, AddrPoint binary struct (C++), AdminPolygon binary struct (C++), BuildHandler (OSM pass 2 handler) (+14 more)

### Community 5 - "Admin Level Mapping (C++)"
Cohesion: 0.17
Nodes (14): Semantic, CountryRules, admin, admin_count, country_code, LevelEntry, admin_level, semantic (+6 more)

### Community 6 - "Rate Limiting"
Cohesion: 0.22
Nodes (8): AtomicU32, AtomicU64, Default, RateLimiter, Result, Self, check_rate(), RateState

### Community 7 - "AddrPoint Schema"
Cohesion: 0.25
Nodes (8): AddrPoint, city_id, housenumber_id, lat, lng, postcode_id, street_id, suburb_id

### Community 8 - "AdminPolygon Schema"
Cohesion: 0.29
Nodes (7): AdminPolygon, area, country_code, name_id, semantic, vertex_count, vertex_offset

### Community 9 - "Interpolation Ways"
Cohesion: 0.29
Nodes (7): InterpWay, end_number, interpolation, node_count, node_offset, start_number, street_id

### Community 10 - "PlaceNode Schema"
Cohesion: 0.40
Nodes (5): PlaceNode, country_code, lng, name_id, semantic

## Knowledge Gaps
- **70 isolated node(s):** `admin_level`, `semantic`, `place_value`, `polygon_semantic`, `node_semantic` (+65 more)
  These have ≤1 connection - possible missing edges or undocumented components.

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **Why does `AddrPoint` connect `AddrPoint Schema` to `OSM Index Builder (C++)`?**
  _High betweenness centrality (0.020) - this node is a cross-community bridge._
- **Why does `build-index main (OSM two-pass pipeline)` connect `Request Handlers & Deployment` to `Address Resolution & Formatting`?**
  _High betweenness centrality (0.018) - this node is a cross-community bridge._
- **What connects `admin_level`, `semantic`, `place_value` to the rest of the system?**
  _70 weakly-connected nodes found - possible documentation gaps or missing edges._
- **Should `Query Server & Index Types` be split into smaller, more focused modules?**
  _Cohesion score 0.07879428873611846 - nodes in this community are weakly interconnected._
- **Should `OSM Index Builder (C++)` be split into smaller, more focused modules?**
  _Cohesion score 0.07288135593220339 - nodes in this community are weakly interconnected._
- **Should `Request Handlers & Deployment` be split into smaller, more focused modules?**
  _Cohesion score 0.08870967741935484 - nodes in this community are weakly interconnected._
- **Should `Address Resolution & Formatting` be split into smaller, more focused modules?**
  _Cohesion score 0.10822510822510822 - nodes in this community are weakly interconnected._