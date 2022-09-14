./gradlew build && java -jar build/libs/gs-spring-boot-docker-0.1.0.jar

docker build -t daeuk/spring .

docker run -p 8080:8080 daeuk/spring