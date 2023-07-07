# PetClinic Custom Otel Instrumentation

## [Lab 1: Install the Open Telemetry Collector](install-otel-collector.md)

This lab walks you through an installation of Open Telemetry Collector on a Linux host. The Open Telemetry collector is
a proxy used to receive, process, and export telemetry data. It supports receiving data in several formats and can be
configured to send data to multiple backends. It is frequently conveniently bundled with a receiver to report host (
infrastructure) metrics and the ability to automatically download and apply auto instrumentation to Java, .Net
applications running on the same host.

## [Lab 2: APM Zero Configuration Auto Instrumentation for Java](apm-instrumentation.md)

In this lab, we will walk through the Java auto instrumentation for Pet Clinic sample application. We will see how APM
traces are captured in full fidelity including calls to database.

## [Lab 3: Real User Monitoring Instrumentation for Browser](rum-instrumentation.md)

In this lab, we will continue with the instrumentation for Pet Clinic sample application. We will add Open Telemetry
Javascript to monitor the frontend (GUI) pages and see how we can track entire user journeys in full fidelity and how
they are automatically connected to APM traces for end to end visibility.

## [Lab 4: Reduce MTTD with Custom Attribution (Span Tagging)](custom-tagging.md)

This lab walks through adding custom span tags to the petclinic application. You can use this to identify a spike in the
throughput of a certain enterprise customer, or the user suffering the highest latency, or to pinpoint the database
shard generating the most errors, which will demonstrate how adding appropriate span tags can reduce troubleshooting
times (MTTD) considerably.

