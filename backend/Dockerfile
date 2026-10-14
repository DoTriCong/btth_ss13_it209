# syntax=docker/dockerfile:1

# ---------- Stage 1: build ----------
FROM gradle:8.14.3-jdk21 AS build
WORKDIR /workspace

# Copy build files first so the dependency layer is cached
COPY build.gradle settings.gradle ./
RUN gradle dependencies --no-daemon -q > /dev/null 2>&1 || true

COPY src ./src
# Tests are run separately (./gradlew test); the image build only packages the jar
RUN gradle bootJar -x test --no-daemon \
 && cp "$(ls build/libs/*.jar | grep -v -- '-plain' | head -n 1)" /workspace/app.jar

# ---------- Stage 2: runtime ----------
FROM eclipse-temurin:21-jre
RUN apt-get update \
 && apt-get install -y --no-install-recommends curl \
 && rm -rf /var/lib/apt/lists/* \
 && groupadd --system app && useradd --system --gid app --no-create-home app
WORKDIR /app
COPY --from=build /workspace/app.jar app.jar
USER app

EXPOSE 8080
HEALTHCHECK --interval=15s --timeout=5s --start-period=60s --retries=5 \
  CMD curl -fs http://localhost:8080/actuator/health | grep -q '"status":"UP"' || exit 1

ENTRYPOINT ["java", "-XX:MaxRAMPercentage=75", "-jar", "app.jar"]
