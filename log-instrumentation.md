# Lab 5: Configure application logs and forward them with Otel Collector

We will configure the Spring PetClinic application to write logs to a file in the filesystem and configure the Splunk OpenTelemetry Collector to read (tail) that log file and report the information to the Splunk Observability Platform.

## Generate application log file

The Spring PetClinic application can be configured to use a number of different java logging libraries. In this scenario, we are going to use logback. 

We just need to create a file named logback.xml in the configuration folder:

```cmd
vi /home/ubuntu/spring-petclinic/src/main/resources/logback.xml
```

Copy and paste the following XML content. Edit the log location (/home/ubuntu/spring-petclinic/spring-petclinic.log) as needed:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE xml>
<configuration scan="true" scanPeriod="30 seconds">
  <contextListener class="ch.qos.logback.classic.jul.LevelChangePropagator">
      <resetJUL>true</resetJUL>
  </contextListener>

  <logger name="org.springframework.samples.petclinic" level="info"/>

    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
      <encoder>
      <pattern>
        %d{yyyy-MM-dd HH:mm:ss} trace_id=%X{trace_id} span_id=%X{span_id} trace_flags=%X{trace_flags} service.name=%property{otel.resource.service.name} deployment.environment=%property{otel.resource.deployment.environment} message="%msg" %n
      </pattern>
    </encoder>
  </appender>

  <appender name="file" class="ch.qos.logback.core.rolling.RollingFileAppender">
          <file>/home/ubuntu/spring-petclinic/spring-petclinic.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <fileNamePattern>springLogFile.%d{yyyy-MM-dd}.log</fileNamePattern>
      <maxHistory>5</maxHistory>
      <totalSizeCap>1GB</totalSizeCap>
    </rollingPolicy>
    <encoder>
      <pattern>
        %d{yyyy-MM-dd HH:mm:ss} trace_id=%X{trace_id} span_id=%X{span_id} trace_flags=%X{trace_flags} service.name=%property{otel.resource.service.name} deployment.environment=%property{otel.resource.deployment.environment} message="%msg" %n
      </pattern>
    </encoder>
  </appender>

  <root level="info">
    <appender-ref ref="file" />
    <appender-ref ref="console" />
  </root>
</configuration>
```


Next,edit the **pom.xml** file to add the Mapped Diagnostics Context dependency used to propagate trace data to the logger. 

```cmd
vi /home/ubuntu/spring-petclinic/pom.xml
```

```xml
<dependency>
    <groupId>io.opentelemetry.instrumentation</groupId>
    <artifactId>opentelemetry-logback-mdc-1.0</artifactId>
    <version>1.31.0-alpha</version>
    <scope>runtime</scope>
</dependency>
```

Now we need to rebuild the application and run it again:

```cmd
./mvnw package -Dmaven.test.skip=true
```


## Open Telemetry Configuration

The Splunk OpenTelemetry Collector uses the Filelog receiver to consume logs. We will need to edit the collectors configuration file:

```cmd
sudo vi /etc/otel/collector/agent_config.yaml
```

Under receivers: create the Filelog Receiver (make sure to indent correctly):

```
receivers:
    filelog:
        include: [/home/ubuntu/spring-petclinic/spring-petclinic.log]
```

The under the service: section, find the logs: pipeline, replace fluentforward with filelog and optionally remove otlp (again, make sure to indent correctly):

```
    logs:
        receivers: [filelog]
```

Save the file and exit the editor. 

Next, we need to edit the HEC Token and HEC URL for the collector to use. We will edit the /etc/otel/collector/splunk-otel-collector.conf file:

```cmd
sudo vi /etc/otel/collector/splunk-otel-collector.conf
```

Replace the values for SPLUNK_HEC_URL and SPLUNK_HEC_TOKEN, then restart the collector to apply the changes:


```cmd
sudo systemctl restart splunk-otel-collector
```

## Sample Code  


Open the .../***src/main/java/org/springframework/samples/petclinic/owner/OwnerController.java*** file.

Add this import statement (import statements are usually found at the top of the java source file)
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
```

Add a static logger field to the **OwnerController** class
```java
private static final Logger logger = LoggerFactory.getLogger(OwnerController.class);
```

Find the following one or more log entries in the **OwnerController.java** file.
```java
@GetMapping("/owners/{ownerId}")
public ModelAndView showOwner(@PathVariable("ownerId") int ownerId) {
    ModelAndView mav = new ModelAndView("owners/ownerDetails");
    Owner owner = this.owners.findById(ownerId);

    Span currentSpan = Span.current();
    currentSpan.setAttribute("custom.owner.id", owner.getId());
    currentSpan.setAttribute("custom.owner.name", String.join(" ", owner.getFirstName(), owner.getLastName()));
    currentSpan.setAttribute("custom.owner.city", owner.getCity());
    currentSpan.setAttribute("custom.owner.phone", owner.getTelephone());

    currentSpan.addEvent(
            String.format("custom event at petclinic: an owner %s's record was viewed", owner.getFirstName()));

    mav.addObject(owner);
    logger.info("Owner record accessed for {} {}", owner.getFirstName(), owner.getLastName());
    return mav;
}
```

Now we need to rebuild the application and run it again:

```cmd
./mvnw spring-javaformat:apply
./mvnw package -Dmaven.test.skip=true
```

Once the rebuild has completed we can then run the application again:

```cmd
sudo java \
-Dotel.service.name=$APP_NAME \
-Dotel.resource.attributes=deployment.environment=$ENV_NAME,version=0.970 \
-Dsplunk.metrics.enabled=true \
-jar target/spring-petclinic-*.jar --spring.profiles.active=mysql
```


## View Logs in Log Observer

From the left hand menu click on Log Observer. Click on Index and select o11y-workshop-XXX.splunkcloud.com (where XXX will be the realm you are running in). On the right hand side select petclinic-workshop and then click Apply. You should see log messages being reported.

Next click Add Filter and search for the field service_name and select the value <your host name>-petclinic-service and click = (include). You should now see only the log messages from your PetClinic application.


## Summary

This the end of the exercise and we have certainly covered a lot of ground. At this point you should have metrics, traces (APM & RUM), logs, database query performance and code profiling being reported into Splunk Observability Cloud. Congratulations!