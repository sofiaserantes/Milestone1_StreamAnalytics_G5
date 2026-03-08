# Real-Time Food Delivery Analytics Pipeline
## Stream Analytics — Milestone 1: Data Feed Design & Generation
### Project Overview
This repository implements the data generation layer for a real-time analytics pipeline simulating an on-demand food delivery platform (think Glovo, Uber Eats, Deliveroo).
We design and generate two synthetic streaming data feeds that represent the core operational dynamics of the platform. These feeds will be ingested into Azure Event Hubs in Milestone 2 and processed with Spark Structured Streaming.
### Team Structure
Members of Group 5:
- Bernarda Andrade
- Javier Comin
- Nour Farhat
- Sofía Serantes
- Tessa Correig
- Rakan Hourani
### Feed A: Order Lifecycle Events
Captures every state transition an order goes through, from the moment a customer places it to final delivery or cancellation
#### Schema Feed A
CREATED → ACCEPTED → PREP_STARTED → READY → PICKED_UP → DELIVERED
                                                        ↘ CANCELLED (any stage)
#### Feed B: Courier Status/Location Events
Captures the continuous status and location updates of couriers, emitted on every state change and periodically as location pings.
#### Schema Feed B
ONLINE (IDLE) → ASSIGNED → EN_ROUTE_TO_RESTAURANT → WAITING → EN_ROUTE_TO_CUSTOMER → IDLE
                                                                                     ↘ OFFLINE

