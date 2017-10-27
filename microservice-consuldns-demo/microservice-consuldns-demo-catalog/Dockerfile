FROM ewolff/docker-java
COPY target/microservice-consuldns-demo-catalog-0.0.1-SNAPSHOT.jar .
CMD /usr/bin/java -Dlogging.path=/log/ -Xmx400m -Xms400m -jar microservice-consuldns-demo-catalog-0.0.1-SNAPSHOT.jar
EXPOSE 8080
