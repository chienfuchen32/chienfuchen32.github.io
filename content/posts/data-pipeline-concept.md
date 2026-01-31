---
title: "Data Pipeline Concept"
date: 2026-01-31T20:54:15+08:00
draft: true
tags: ["Data pipeline"]
---
# Data Pipeline

## Requirements
* data processing job work just-in-time
* design should be a scalable solutions
* data consistency and syncronization
* common module thinking
* need some friendly integration 
* continuously review
* create data pipeline SLA
* dependency and flow
* high availability

## Design
* relation database
* hot & cold data layer
* raw data and aggregation tier
* data migration plan
* data TTL & lifecycle

## How to
* Application
  * data producer & consumer
  * data model and governance
  * schema migration framework
* Kafka
  * steaming data processing
* PostgreSQL
  * master data storage
  * ACID
* Clickhouse
  * multiple mergeTree table engine
  * materialized view
  * user defined function
* ETL tools:
  * Airflow
  * Spark
