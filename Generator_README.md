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

Install the dependency:
bashpip install fastavro

