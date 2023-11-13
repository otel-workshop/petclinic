# Lab 2: Set up Pet Clinic Sample App (Demonstrating Zero Configuration Auto Instrumentation for Java)

First thing we need to setup APM is… well, an application. For this exercise, we will use the Spring PetClinic application. This is a very popular sample java application built with Spring framework (Springboot). We will also demonstrate the zero configuration feature of Open Telemetry Collector by showing that instrumentation is injected automatically into any Java application without any additional configuration.

## Set up Pet Clinic Sample App


Login into your EC2 machine. You should be in the "/home/ubuntu" folder. Verify that by running this command.
```cmd
pwd
```

Next we will clone the PetClinic repository, then we will compile, build, package and test the application.

```cmd
git clone https://github.com/spring-projects/spring-petclinic
```

Change into the spring-petclinic directory:

```cmd
cd spring-petclinic && git checkout 276880e
```

Start a MySQL database for PetClinic to use:

```cmd
docker run -d -e MYSQL_USER=petclinic -e MYSQL_PASSWORD=petclinic -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=petclinic -p 3306:3306 docker.io/mysql:5.7.8
```

Next, run the maven command to compile/build/package PetClinic:

```cmd
./mvnw package -Dmaven.test.skip=true
```

Once the compilation is complete, you can run the application with the following command:
The unique identity of an application (or service) is defined by its application (or service) name and the environment in which it is deployed. 

The APP_NAME and ENV_NAME values can be any alpha-numberic values but ensure that you identify them uniformly to all instrumentation - APM, RUM, Synthetics etc. For this application, we will use the following values, but again you can really use any value as long as they are unique (since this backend UI is shared among all workshop participants) and that you use it consistently throughout the rest of this workshop labs.

```cmd
    export APP_NAME=$(hostname)-petclinic
    export ENV_NAME=$(hostname)-fe
```

Update APP_NAME and ENV_NAME in the below command with their chosen values
Resource attributes can also be added to every reported span. For example version=0.970. A comma separated list of resource attributes can also be defined e.g. key1=val1,key2=val2.
At a minimum, you should specify the **deployment.environment** resource attribute but others are optional.

```cmd
sudo java \
-Dotel.service.name=$APP_NAME \
-Dotel.resource.attributes=deployment.environment=$ENV_NAME,version=0.970 \
-Dsplunk.metrics.enabled=true \
-jar target/spring-petclinic-*.jar --spring.profiles.active=mysql
```

If you check the logs of the Splunk OpenTelemetry collector you will see that the collector automatically detected the application running and auto-instrumented it. You can view the logs using the following command:

```cmd
sudo tail -f /var/log/syslog
```

You can validate if the application is running by visiting http://<VM_IP_ADDRESS>:8080.

Once your validation is complete you can stop the application by pressing Ctrl-C.

## Generate Traffic

Next we will start a Docker container running Locust that will generate some simple traffic to the PetClinic application. Locust is a simple load testing tool that can be used to generate traffic to a web application.

```cmd
docker run --network="host" -d -p 8089:8089 -v /home/ubuntu/workshop/petclinic:/mnt/locust docker.io/locustio/locust -f /mnt/locust/locustfile.py --headless -u 10 -r 3 -H http://127.0.0.1:8080
```

## Next Step

[Go back to Main Page and Proceed to Lab 3](README.md)
