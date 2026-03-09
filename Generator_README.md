# Feed Generation
This document explains how to install, configure, and run the synthetic data generator for the two streaming feeds used in this project.

Milestone1_ipynb produces two synthetic streaming data feeds that simulate the operational dynamics of a real-time food delivery platform

## Feed A
OrderLifecycle with Output order_lifecycle_events.json/.avro and 4,000 events

## Feed B
CourierStatus with Output courier_status_events.json/.avro and 4,000 events

It also writes smaller sample files (first 10 events of each feed) for quick inspection:
sample_order_lifecycle_events.json / .avro
sample_courier_status_events.json / .avro

## Requirements
Python 3.10 or higher
fastavro for AVRO serialisation
## Install the dependency:
bashpip install fastavro
## How to run
bashcd generator
python Milestone1_ipynb

All output files are written to the current working directory. The script prints a confirmation line for each file created:
✅ order_lifecycle_events.json written
✅ order_lifecycle_events.avro written
✅ courier_status_events.json written
✅ courier_status_events.avro written
✅ sample_order_lifecycle_events.json written
✅ sample_order_lifecycle_events.avro written
✅ sample_courier_status_events.json written
✅ sample_courier_status_events.avro written
📂 All missing deliverables generated.

