# anymap-ts (MapLibre) — API subset used in this workshop

This page documents the **small set of anymap-ts functions** students use for mapping.

It is intentionally not a full anymap-ts manual — it focuses on the functions you’ll see in the workshop code.


## Install

```bash
pip install -e ".[notebooks]"
```


## Coordinates

All examples use WGS84 geographic coordinates in the order:

- `(lng, lat)` (longitude first)


## Import

The workshop uses the MapLibre backend through the `Map` widget:

```python
from anymap_ts import Map
```


## Create a map

```python
CITY_HALL_LNGLAT = (12.5683, 55.6761)

m = Map(
    center=CITY_HALL_LNGLAT,
    zoom=16.5,
    height="650px",
    width="100%",  # optional
    # style=...,    # optional (URL or style dict)
)
m
```

Workshop-relevant arguments:

- `center`: `(lng, lat)`
- `zoom`: float
- `height`: CSS string
- `width`: CSS string


## Markers (static points)

### `add_marker(lng, lat, ...) -> str`

Adds a marker to the map and returns its marker ID.

Used arguments in the workshop:

- `lng`, `lat`: floats
- `name`: stable marker ID (recommended)
- `color`: hex string (e.g. `"#2e7d32"`)
- `popup`: optional text/HTML

Example:

```python
marker_id = m.add_marker(
    12.5699,
    55.6763,
    name="coffee-1",
    color="#2e7d32",
    popup="Coffee shop 1",
)
```

### `remove_marker(marker_id) -> None`

Removes a marker by ID.

```python
m.remove_marker("coffee-1")
```


## Route animation (frontend playback)

### `animate_along_route(coords, ...) -> None`

Animates a marker along a precomputed route in the browser (smooth and low overhead).

The workshop uses:

- `coords`: list of `[lng, lat]` points
- `anim_id`: string identifier
- `duration`: milliseconds
- `loop`: bool
- `marker_color`, `marker_size`
- `show_trail`: bool

Example:

```python
coords = [
    [12.5683, 55.6761],
    [12.5684, 55.67612],
]

m.animate_along_route(
    coords,
    anim_id="walk-1",
    duration=45_000,
    loop=True,
    marker_color="#3388ff",
    marker_size=1.0,
    show_trail=False,
)
```

### `stop_animation(anim_id) -> None`

Stops a running animation.

```python
m.stop_animation("walk-1")
```


## Workshop extension: live marker movement

anymap-ts provides `add_marker(...)`, but does not expose a built-in “move an existing marker in place” method suitable for high-frequency streaming.

For that, the workshop includes a tiny extension:

```python
from simulated_city.maplibre_live import LiveMapLibreMap
```

### `LiveMapLibreMap.move_marker(marker_id, (lng, lat), color=None, popup=None) -> None`

Moves an existing marker in-place (or creates it if missing).

```python
m = LiveMapLibreMap(center=CITY_HALL_LNGLAT, zoom=16.5, height="650px")
m

m.move_marker("walker", CITY_HALL_LNGLAT, color="#ff0000")
m.move_marker("walker", (12.56835, 55.67615))
```

Practical note used in the workshop:

- Recoloring an existing marker is done by refresh:
  - `remove_marker(id)` then `add_marker(..., name=id, color=...)`


## MQTT / threading rule (important)

If coordinates come from MQTT:

- Do not call widget methods from an MQTT callback thread.
- Use: MQTT callback → queue → async consumer in the notebook → `move_marker(...)`.
