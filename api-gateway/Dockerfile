FROM gradle:7.2-jdk11 as build
ARG VERSION=latest
WORKDIR /app
COPY . ./
RUN gradle bootJar

#PROD
FROM adoptopenjdk:11-jre-openj9

ENV SERVICE_PROFILES dummy
ENV INITIAL_MEMORY 64m
ENV MAX_MEMORY 512m
ENV MAX_RAM_PERCENTAGE 50

RUN apt-get update \
  && apt-get install -y --no-install-recommends curl jq \
  && rm -rf /var/lib/apt/lists/*

COPY --from=build /app/build/libs/*.jar /app.jar

EXPOSE 8765

ENTRYPOINT ["java", "-XX:+UseSerialGC", "-Xshareclasses", "-Xquickstart", "-Xms${INITIAL_MEMORY}", "-Xmx${MAX_MEMORY}", "-XX:MaxRAMPercentage=${MAX_RAM_PERCENTAGE}", "-jar", "-Dspring.profiles.active=${SERVICE_PROFILES}", "/app.jar"]

HEALTHCHECK --start-period=30s --interval=30s --timeout=3s --retries=3 \
  CMD curl --silent --fail --request GET http://localhost:8765/actuator/health \
  | jq --exit-status '.status == "UP"' || exit 1
