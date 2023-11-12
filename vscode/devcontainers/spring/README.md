# spring demo

bootstrap project - lauch test application
```bash
mkdir -p test-project && cd test-project && curl -G https://start.spring.io/starter.tgz \
-d bootVersion=3.1.5 \
-d dependencies=web,actuator,devtools,testcontainers \
-d type=gradle-project | tar -xzf - && \
echo "management.endpoints.web.exposure.include=*" > src/main/resources/application.properties
```

adding testcontainers
```bash
sed -i -e "s/^\(.*testImplementation.*org.testcontainers.*\)/\ttestImplementation 'io.github.microcks:microcks-testcontainers:0.1.3'\n\1/" build.gradle && \
cp ../order-service-openapi.yaml src/main/resources && \
mkdir -p src/test/resources/third-parties/ && \
cp ../apipastries* src/test/resources/third-parties && \
cp ../UserSignedUpAPI-asyncapi-0.1.1.yml src/test/resources/third-parties && \
cp ../Pastry.java src/main/java/com/example/demo && \
sed -i -e 's/package org.acme.order.client.model/package com.example.demo/' src/main/java/com/example/demo/Pastry.java && \

cp ../PastryAPIClient.java src/main/java/com/example/demo && \
sed -i -e 's/package org.acme.order.client/package com.example.demo/' src/main/java/com/example/demo/PastryAPIClient.java && \
sed -i -e 's/import org.acme.order.client.model/import com.example.demo/' src/main/java/com/example/demo/PastryAPIClient.java && \

cp ../PastryAPIClientTests.java src/test/java/com/example/demo && \
sed -i -e 's/package org.acme.order.client/package com.example.demo/' src/test/java/com/example/demo/PastryAPIClientTests.java && \
sed -i -e 's/import org.acme.order.client.model.Pastry/import com.example.demo.Pastry/' src/test/java/com/example/demo/PastryAPIClientTests.java && \
sed -i -e 's#target/test-classes#build/resources/test#' src/test/java/com/example/demo/PastryAPIClientTests.java && \
echo "spring.jackson.default-property-inclusion=non_null" >> src/main/resources/application.properties && \
echo "pastries.baseUrl=http://localhost:9090/rest/API Pastries/0.0.1" >> src/main/resources/application.properties && \
gradle test --info
```

running test application
```bash
gradle bootTestRun --args='--hostname=spring.local'
curl localhost:8080/actuator/env/hostname | jq
```

```bash
gradle bootBuildImage
docker run --rm -p 8081:8080 demo:0.0.1-SNAPSHOT
curl localhost:8081/actuator/env/hostname | jq
```

## References

* [Microcks: Open source Kubernetes Native tool for API Mocking and Testing](https://www.youtube.com/live/4ObZu9Gh9Xk?si=8OWjbDi0dcBxs8mg)
* [Mocking and contract-testing in your Inner Loop with Microcks - Part 2: Unit testing with TestContainers](https://itnext.io/mocking-and-contract-testing-in-your-inner-loop-with-microcks-part-2-unit-testing-with-860a86cb4b4c)
* [Testcontainers at development time](https://spring.io/blog/2023/06/23/improved-testcontainers-support-in-spring-boot-3-1#testcontainers-at-development-time)