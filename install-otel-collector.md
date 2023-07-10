# Lab 1: Install the Open Telemetry Collector

## Open Telemetry Collector

The OpenTelemetry Collector is the core component of instrumenting infrastructure and applications. Its role is to
collect and send:

* Infrastructure metrics (disk, cpu, memory, etc)
* Application Performance Monitoring (APM) traces
* Profiling data
* Host and application logs

## Install the Open Telemetry Collector

Create the ACCESS_TOKEN and REALM environment variables to use in the proceeding OpenTelemetry Collector install
command.

For instance, if your realm is us1, you would type export REALM=us1 and for eu0 type export REALM=eu0 etc.

```cmd
export ACCESS_TOKEN="<replace_with_O11y-Workshop-ACCESS_TOKEN>"
export REALM="<replace_with_REALM>"
```

We can then go ahead and install the Collector. There are two additional parameters passed to the install script, they
are --with-instrumentation and --deployment-environment. The --with-instrumentation option the installer will install
the agent from the Splunk distribution of OpenTelemetry Java, which is then loaded automatically when the PetClinic Java
application starts up. No configuration required!

```cmd
curl -sSL https://dl.signalfx.com/splunk-otel-collector.sh > /tmp/splunk-otel-collector.sh && \
sudo sh /tmp/splunk-otel-collector.sh --with-instrumentation --deployment-environment prod --realm $REALM -- $ACCESS_TOKEN
```

On AWS/EC2 machines, we need to run and additional step to patch the collector to expose the hostname of the instance

```cmd
sudo sed -i 's/gcp, ecs, ec2, azure, system/system, gcp, ecs, ec2, azure/g' /etc/otel/collector/agent_config.yaml
```

Restart the collector

```cmd
sudo systemctl restart splunk-otel-collector
```

Check that the collector service is up and running

```cmd
sudo systemctl status splunk-otel-collector
```

Once the install is completed, you can navigate to the **Hosts with agent installed** dashboard to see the data from your host, Dashboards → Hosts with agent installed.

Use the dashboard filter and select **host.name** and type or select the hostname of your virtual machine. Once you see data flowing for your host, we are then ready to get started with the APM component.

## Next Step

[Go back to Main Page and Proceed to Lab 2](README.md)