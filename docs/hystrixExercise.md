# Hystrix Exercise

In this exercise, we will add Hystrix support to our three individual web services and enable a gracefull fall back in the case of errors. We will also install and configure a Hystrix dashboard that we can use to monitor the health of our circuits.

## Enable Hystrix

All changes will be made in the `movie-aggregator-service` project.

1. Open `pom.xml` and uncomment the dependency for `spring-cloud-starter-netflix-hystrix`
1. Open `src/main/resources/application.yml` and un-comment the properties related to Hystrix
1. Make a new package `microservice.workshop.movieaggregatorservice.service.fallback`
1. Make a new class `MovieAwardServiceFallback` in the fallback package. The class should implement the `MovieAwardService` interface, and return an empty List for the single method
1. Make a new class `MovieCastServiceFallback` in the fallback package. The class should implement the `MovieCastService` interface, and return an empty List for the single method
1. Make a new class `MovieServiceFallback` in the fallback package. The class should implement the `MovieService` interface, and return an empty Optional for the single method
1. Open the `MovieAwardService` interface and alter the `@FeignClient` annotation to add the fallback implementation
1. Open the `MovieCastService` interface and alter the `@FeignClient` annotation to add the fallback implementation
1. Open the `MovieService` interface and alter the `@FeignClient` annotation to add the fallback implementation
1. Open `MovieAggregatorServiceApplication` and make the following changes:
    1. Add the `@EnableCircuitBreaker` annotation to the class
    1. Add three methods, each annotated with `@Bean` that will return a new instance of the three fallback classes.  For example:

```java
    @Bean
    public MovieAwardServiceFallback movieAwardServiceFallback() {
        return new MovieAwardServiceFallback();
    }
```

Start the `movie-aggregator-service` and the traffic simulator. Everything should look normal. Then kill the `movie-cast-service` application. You should see that the aggregator service is still running, but not returning cast members.

Now open a browser to [http://localhost:8080/actuator/health](http://localhost:8080/actuator/health). You should see a report from Hystrix that the movie-cast-service cricuit is open (meaning that the target is down).

Now open a browser to [http://localhost:8080/actuator/hystrix.stream](http://localhost:8080/actuator/hystrix.stream). You should see a never ending stream of data from Hystrix about the status of the circuit breakers. This is hard to grasp, so we'll use the Hystrix dashboard.

## Hystrix Dashboard

There is a simple Spring boot starter that will stand up a hystrix server with almost no coding required. We'll use the Spring initializer web site to create the server.

1. Navigate to [https://start.spring.io](https://start.spring.io)
1. Create a Maven project with Java and the latest version of SpringBoot (2.1.1 at the time of writing)
1. Specify group: `microservice.workshop`
1. Specify artifact: `hystrix-dashboard`
1. For dependencies, add the following:
    - Hystrix Dashboard
1. Generate the project (causes a download)
1. Unzip the downloaded file somewhere convenient
1. Add the new project to your IDE workspace
    - Eclipse: File->Import->Existing Maven Project
    - IntelliJ: File->New->Module From Existing Sources...
    - VS Code: File->Add Folder to Workspace
1. Open `microservice.workshop.hystrixdashboard.HystrixDashboardApplication.java` and add the `@EnableHystrixDashboard` annotation to the class
1. Navigate to `src/main/resources`, then rename `application.properties` to `application.yml`
1. Enter the following values in `application.yml`:

```yml
server:
  port: 9090
```

Once these changes are made, start the `hystrix-dashboard` application. Follow these seps to see the hystrix dashboard:

1. Open [http://localhost:9090/hystrix](http://localhost:9090/hystrix)
1. The dashboard needs to know what stream to consume. Enter `http://localhost:8080/actuator/hystrix.stream`, then press the `Monitor Stream` button. You should see the Hystrix dashboard with the three circuits displayed. The movie cast cisrcuit is open, the other two are closed.
1. Once you feel comfortable with the information shown on this screen, restart the `movie-cast-service` application. It may take a minute or so, but you should eventually see the circuit close in the dashboard and see movie cast information being returned in the traffic simulator.