# Generator Documentation

`Milestone1.ipynb` is a Google Colab / Jupyter notebook that generates two synthetic streaming data feeds for the Real Time Food Delivery Analytics Pipeline project. Running all cells produces 10 output files: full JSON and AVRO datasets for each feed, 10 event sample files for quick inspection, and standalone AVRO schema files.

---

## Table of Contents

- [Output Files](#output-files)
- [Requirements](#requirements)
- [How to Run](#how-to-run)
- [Configuration](#configuration)
- [Edge Case Injection](#edge-case-injection)
- [How to Verify the Output](#how-to-verify-the-output)

---

## Output Files

All files are written to the current working directory (the same folder as the notebook).

| File | Format | Events | Description |
|---|---|---|---|
| `order_lifecycle_events.json` | JSON array | 4,000 | Full Feed A — order lifecycle stream |
| `order_lifecycle_events.avro` | AVRO binary | 4,000 | Full Feed A in binary AVRO format |
| `courier_status_events.json` | JSON array | 4,000 | Full Feed B — courier status/location stream |
| `courier_status_events.avro` | AVRO binary | 4,000 | Full Feed B in binary AVRO format |
| `sample_order_lifecycle_events.json` | JSON array | 10 | First 10 events of Feed A |
| `sample_order_lifecycle_events.avro` | AVRO binary | 10 | First 10 events of Feed A |
| `sample_courier_status_events.json` | JSON array | 10 | First 10 events of Feed B |
| `sample_courier_status_events.avro` | AVRO binary | 10 | First 10 events of Feed B |
| `order_lifecycle_events.avsc` | JSON (AVRO schema) | — | Standalone schema for Feed A |
| `courier_status_events.avsc` | JSON (AVRO schema) | — | Standalone schema for Feed B |

---

## Requirements

- Python **3.10** or higher
- [`fastavro`](https://fastavro.readthedocs.io/) for AVRO serialisation

The notebook installs the dependency automatically in its first cell:

```python
!pip install fastavro
```

If running locally outside Colab, install it manually first:

```bash
pip install fastavro
```

No other third party libraries are required. The generator uses only `random`, `datetime`, `json`, and `collections` from the Python standard library.

---

## How to Run

1. Open `Milestone1.ipynb` in Google Colab or Jupyter
2. Run all cells: **Runtime → Run all** (Colab) or **Kernel → Restart & Run All** (Jupyter)

The notebook prints a confirmation line for each file written:

```
✅ order_lifecycle_events.avsc written
✅ courier_status_events.avsc written
✅ sample_order_lifecycle_events.json written
✅ sample_courier_status_events.json written
✅ sample_order_lifecycle_events.avro written
✅ sample_courier_status_events.avro written

📂 All missing deliverables generated.
```

The JSON files (`order_lifecycle_events.json` and `courier_status_events.json`) are written during the main generation cell. The `.avsc`, sample JSON, and sample AVRO files are written in the final cell.

> **Reproducibility**: the generator uses `random.seed(7)`. Running the notebook from scratch always produces identical output.

---

## Configuration

There are no CLI arguments. All parameters are defined as **constants at the top of the second notebook cell** and can be edited directly before running. The sections below document every configurable value.

### Volume

```python
NUM_ORDER_EVENTS   = 4000   # total events in the order lifecycle stream
NUM_COURIER_EVENTS = 4000   # total events in the courier status/location stream

NUM_RESTAURANTS = 120       # distinct restaurant IDs (R0001–R0120)
NUM_COURIERS    = 300       # distinct courier IDs (C00001–C00300)
NUM_CUSTOMERS   = 800       # distinct customer IDs (U00001–U00800)
```

Feed A targets 4,000 events by generating approximately 667 orders (~6 events each), then trimming or extending to hit the exact count. Feed B generates exactly `NUM_COURIER_EVENTS` events directly.

### Time window

```python
START_DATE = datetime.datetime(2026, 2, 1, 0, 0, 0)
END_DATE   = datetime.datetime(2026, 3, 1, 0, 0, 0)
```

All events have `event_time` values within this one month window. Timestamps are ISO 8601 UTC strings.

### Geographic zones

```python
ZONES        = ["Z1_Center", "Z2_North", "Z3_South", "Z4_East", "Z5_West"]
ZONE_WEIGHTS = [0.40, 0.18, 0.15, 0.14, 0.13]
```

Zone centroids are fixed lat/lon coordinates loosely based on Madrid. Each event's location is jittered ±800 m from the centroid. `Z1_Center` receives 40% of all demand; the four outer zones share the rest.

### Peak hours

```python
LUNCH_HOURS  = list(range(12, 15))   # 12:00–14:00 — weight multiplier 2.3×
DINNER_HOURS = list(range(19, 23))   # 19:00–22:00 — weight multiplier 2.8×
WEEKEND_MULTIPLIER = 1.25            # applied on Saturday and Sunday
```

Peak weighting is implemented via rejection sampling. Off peak hours have a base weight of 1.0.

### Business parameters

```python
BASE_CANCEL_PROB    = 0.06   # baseline cancellation probability (6%)
SURGE_CANCEL_BONUS  = 0.05   # added during surge periods (+5%)
PROMO_CANCEL_BONUS  = -0.02  # subtracted during promo periods (−2%)
SURGE_PROB          = 0.08   # chance of a demand surge during lunch/dinner hours
PROMO_PROB          = 0.10   # chance of a promo being active on any given order
```

The effective cancellation probability for any order is `BASE_CANCEL_PROB + SURGE_CANCEL_BONUS (if surge) + PROMO_CANCEL_BONUS (if promo)`, clamped between 0.0 and 0.40.

---

## Edge Case Injection

These constants control how frequently intentional data quality issues are injected. Set any value to `0.0` to disable that edge case entirely.

```python
DUPLICATE_EVENT_PROB              = 0.02   # 2%  — exact duplicate events
LATE_EVENT_PROB                   = 0.05   # 5%  — ingestion_time 2–25 min after event_time
MISSING_STEP_PROB                 = 0.03   # 3%  — orders with a skipped lifecycle step (Feed A)
IMPOSSIBLE_DURATION_PROB          = 0.01   # 1%  — negative or extreme delivery durations (Feed A)
COURIER_OFFLINE_MID_DELIVERY_PROB = 0.02   # 2%  — courier goes OFFLINE with active order (Feed B)
```

When `LATE_EVENT_PROB` is not triggered, `ingestion_time` is `event_time` plus a normal 1–45 second processing delay.

| Constant | Feed | What it tests in Spark |
|---|---|---|
| `DUPLICATE_EVENT_PROB` | Both | `event_id` deduplication before windowing |
| `LATE_EVENT_PROB` | Both | Watermark tolerance; events >10 min late will be dropped |
| `MISSING_STEP_PROB` | A | Stateful operator resilience to incomplete lifecycle sequences |
| `IMPOSSIBLE_DURATION_PROB` | A | Anomaly detection — negative pickup gap or extreme travel time |
| `COURIER_OFFLINE_MID_DELIVERY_PROB` | B | Stream-stream join correctness when courier stream ends unexpectedly |

---

## How to Verify the Output

### Inspect a sample event (JSON)

```bash
python -c "
import json
events = json.load(open('sample_order_lifecycle_events.json'))
print(json.dumps(events[0], indent=2))
"
```

### Count events by type — Feed A

```bash
python -c "
import json
from collections import Counter
events = json.load(open('order_lifecycle_events.json'))
print(Counter(e['event_type'] for e in events))
"
```

Expected output (approximate, varies slightly due to shuffling and edge cases):

```
Counter({'DELIVERED': 892, 'READY': 882, 'ACCEPTED': 855, 'CREATED': 833,
         'PICKED_UP': 782, 'PREP_STARTED': 668, 'CANCELLED': 88})
```

### Count events by type — Feed B

```bash
python -c "
import json
from collections import Counter
events = json.load(open('courier_status_events.json'))
print(Counter(e['event_type'] for e in events))
"
```

### Read the AVRO file

```python
from fastavro import reader

with open('sample_order_lifecycle_events.avro', 'rb') as f:
    for record in reader(f):
        print(record)
        break
```

### Check for duplicates

```bash
python -c "
import json
events = json.load(open('order_lifecycle_events.json'))
ids = [e['event_id'] for e in events]
dupes = len(ids) - len(set(ids))
print(f'Duplicate event_ids: {dupes} (expected ~{int(len(ids)*0.02)})')
"
```

### Verify late events exist

```bash
python -c "
import json
from datetime import datetime
events = json.load(open('order_lifecycle_events.json'))
late = [e for e in events
        if (datetime.fromisoformat(e['ingestion_time']) - datetime.fromisoformat(e['event_time'])).seconds > 120]
print(f'Late events (>2 min lag): {len(late)} out of {len(events)}')
"
```

---

*Real Time Food Delivery Analytics Pipeline — Stream Analytics Course, Group 5*










