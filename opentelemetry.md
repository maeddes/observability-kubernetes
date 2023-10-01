# Intro

Examples based on repo: https://github.com/maeddes/technologyconsulting-containerexerciseapp.git (branch otelagent)

```bash
git clone -b otelagent https://github.com/maeddes/technologyconsulting-containerexerciseapp.git
```

Show codebase a little and architecture of application.
Build apps and container

```bash
mvn clean install -Dspring.profiles.active=dev -f todobackend/pom.xml
mvn clean install -f todoui/pom.xml
docker build -f Dockerfile-todoui-orig -t maeddes/todoui:orig .
docker build -f Dockerfile-todobackend-orig -t maeddes/todobackend:orig .
```

Show the initial docker composed setup

```bash
docker compose -f docker-compose-orig.yaml up
```

## Links

Jaeger: http://localhost:16686/

App Frontend: http://localhost:8090/
App Backend: http://localhost:8080/todos/

# Standard OpenTelemetry instrumentation

Add the opentelemetry library to the Dockerfile and the Java process

Backend:

```dockerfile
FROM eclipse-temurin:17
RUN mkdir -p /opt/todobackend
WORKDIR /opt/todobackend
# COPY opentelemetry-javaagent.jar /opt/todobackend
# ADD https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/latest/download/opentelemetry-javaagent.jar .
COPY todobackend/target/todobackend-0.0.1-SNAPSHOT.jar /opt/todobackend
CMD ["java", "-jar", "-javaagent:/opt/todobackend/opentelemetry-javaagent.jar", "todobackend-0.0.1-SNAPSHOT.jar"]
```

Frontend:

```dockerfile
FROM eclipse-temurin:17
RUN mkdir -p /opt/todoui
WORKDIR /opt/todoui
COPY opentelemetry-javaagent.jar /opt/todoui
#ADD https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/latest/download/opentelemetry-javaagent.jar .
COPY todoui/target/todoui-0.0.1-SNAPSHOT.jar /opt/todoui
CMD ["java", "-javaagent:/opt/todoui/opentelemetry-javaagent.jar", "-jar", "todoui-0.0.1-SNAPSHOT.jar"]
```

Build new container images

```bash
docker build -f Dockerfile-todobackend-otel -t maeddes/todobackend:otel .
docker build -f Dockerfile-todoui-otel -t maeddes/todoui:otel .
```

Configure environment variables in docker compose

```bash
      OTEL_EXPORTER_OTLP_ENDPOINT: http://jaeger:4317
      OTEL_RESOURCE_ATTRIBUTES: service.name=todobackend
```

```bash
      OTEL_EXPORTER_OTLP_ENDPOINT: http://jaeger:4317
      OTEL_RESOURCE_ATTRIBUTES: service.name=todoui
```


```bash
docker compose -f docker-compose-otel.yaml up
```

## Flows

Show Jaeger again - probably no changes

Jaeger: http://localhost:16686/

Access Backend first and just query

App Backend: http://localhost:8080/todos/

Add a Todo via curl

```bash
curl -X POST localhost:8080/todos/test
```

Validate in Jaeger

Access Frontend and perform steps.
Validate in Jaeger again.

Show various CRD operations and accordings spans.

Create an Item in the UI ad then delete it via curl

```bash
curl -X DELETE localhost:8080/todos/ABC
```

Try to delete via UI and generate error.
Analyse via Jaeger.

# SDK OpenTelemetry implementation

Add another method to the backend.

```java
	@PostMapping("/todos/{todo}")
	String addTodo(@PathVariable String todo){

		this.someUselessMethod(todo);
		//todoRepository.save(new Todo(todo));
		return todo;

	}

	String someUselessMethod(String todo){

		todoRepository.save(new Todo(todo));
		return todo;

	}
```

rebuild

```bash
mvn clean install -Dspring.profiles.active=dev -f todobackend/pom.xml
docker build -f Dockerfile-todobackend-otel -t maeddes/todobackend:otel .
```

restart docker compose and show things
new method not visible in traces

Add maven dependency to pom.xml

```xml
		<dependency>
			<groupId>io.opentelemetry.instrumentation</groupId>
			<artifactId>opentelemetry-instrumentation-annotations</artifactId>
			<version>1.29.0</version>
		</dependency>
```

Add annotations in code


```java
	@WithSpan
	String someUselessMethod(@SpanAttribute String todo){

		todoRepository.save(new Todo(todo));
		return todo;

	}
```

add problems to code


```java
	@WithSpan
	String someUselessMethod(@SpanAttribute String todo){

		todoRepository.save(new Todo(todo));
		if(todo.equals("slow")){
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		} 		
		if(todo.equals("fail")){
			System.out.println("Failing ...");
			System.exit(1);
		} 
		return todo;

	}
```