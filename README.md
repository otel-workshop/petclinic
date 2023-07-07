# PetClinic Custom Otel Instrumentation

## [Lab 1: Install the Open Telemetry Collector](install-otel-collector.md)

This lab walks you through an installation of Open Telemetry Collector on a Linux host. An Open Telemetry collector is a
proxy used to receive, process, and export telemetry data. It supports receiving data in several formats and can be
configured to send data to multiple backends. It is frequently conveniently bundled with a receiver to report host (
infrastructure) metrics and the ability to automatically download and apply auto instrumentation to Java, .Net applications.

## [Lab 4: Reduce MTTD with Custom Attribution (Span Tagging)](custom-tagging.md)

This lab walks through adding custom span tags to the petclinic application. You can use this to identify a spike in the
throughput of a certain enterprise customer, or the user suffering the highest latency, or to pinpoint the database
shard generating the most errors, which will demonstrate how adding appropriate span tags can reduce troubleshooting
times (MTTD) considerably.

