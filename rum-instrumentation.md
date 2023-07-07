# Lab 3: Real User Monitoring with Browser Instrumentation

## Enable RUM

For the Real User Monitoring (RUM) instrumentation, we will add
the [Open Telemetry Javascript](https://github.com/signalfx/splunk-otel-js-web) snippet in the pages, we will use the
wizard again **Data Management → Add Integration → RUM Instrumentation → Browser Instrumentation**.

Select the preconfigured **RUM ACCESS TOKEN** from the dropdown, click **Next**. Enter **App name** and **Environment** using the
following syntax:

* <mark style="background-color: #FDFDC9">[hostname]</mark>-petclinic-service - replacing **[hostname]** with your actual hostname.
* <mark style="background-color: #FDFDC9">[hostname]</mark>-petclinic-env - replacing **[hostname]** with your actual hostname.

Then you’ll need to select the workshop RUM token and define the application and environment names. The wizard will then
show a snipped of HTML code that needs to be place at the top at the pages in the **&lt;head&gt;** section. In this example we are
using:

* Application Name: <mark style="background-color: #FDFDC9">[hostname]</mark>-petclinic-service
* Environment: <mark style="background-color: #FDFDC9">[hostname]</mark>-petclinic-env

Copy the generated code snippet in the wizard or copy and edit the snippet below accordingly. You need to
replace **&lt;REALM&gt;**, **&lt;RUM_ACCESS_TOKEN&gt;** and **&lt;hostname&gt;** with the actual values.

```cmd
<script src="https://cdn.signalfx.com/o11y-gdi-rum/latest/splunk-otel-web.js" crossorigin="anonymous"></script>
<script>
SplunkRum.init({
    beaconUrl: "https://rum-ingest.<REALM>.signalfx.com/v1/rum",
    rumAuth: "<RUM_ACCESS_TOKEN>",
    app: "<hostname>-petclinic-service",
    environment: "<hostname>-petclinic-env"
    });
</script>
```

The Spring PetClinic application uses a single HTML page as the "layout" page, that is reused across all pages of the
application. This is the perfect location to insert the Splunk RUM Instrumentation Library as it will be loaded in all
pages automatically

Let’s then edit the layout page:

```cmd
vi src/main/resources/templates/fragments/layout.html
```

Next, insert the snippet we generated above in the <mark style="background-color: #FDFDC9">&lt;head&gt;</mark> section of the page. Now we need to rebuild the application
and run it again:

## Rebuild PetClinic

Run the <mark style="background-color: #FDFDC9">maven</mark> command to compile/build/package PetClinic:

```cmd
./mvnw package -Dmaven.test.skip=true
```

```cmd
java \
-Dotel.service.name=$(hostname)-petclinic-service \
-Dsplunk.profiler.enabled=true \
-Dsplunk.profiler.memory.enabled=true \
-Dsplunk.metrics.enabled=true \
-Dotel.resource.attributes=deployment.environment=$(hostname)-petclinic,version=0.314 \
-jar target/spring-petclinic-*.jar --spring.profiles.active=mysql
```

Then let’s visit the application using a browser to generate real-user traffic http://&lt;VM_IP_ADDRESS&gt;:8080, now we
should see RUM traces being reported.

Let’s visit RUM and see some of the traces and metrics **Hamburger Menu → RUM** and you will see some of the Spring
PetClinic URLs showing up in the UI.

When you drill down into a RUM trace you will see a link to APM in the spans. Clicking on the trace ID will take you to
the corresponding APM trace for the current RUM trace.