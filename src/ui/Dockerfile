# =========================
# Build Stage
# =========================
FROM public.ecr.aws/amazonlinux/amazonlinux:2023 AS build-env

# Install build dependencies
RUN dnf --setopt=install_weak_deps=False install -q -y \
    maven \
    java-21-amazon-corretto-headless \
    which \
    tar \
    gzip \
    && dnf clean all

VOLUME /tmp
WORKDIR /

# Copy Maven wrapper and config
COPY .mvn .mvn
COPY mvnw .
COPY pom.xml .

# 🔴 FIX: make mvnw executable BEFORE running it
RUN chmod +x mvnw \
    && ./mvnw dependency:go-offline -B -q

# Copy source and build
COPY ./src ./src

RUN ./mvnw -DskipTests package -q \
    && mv /target/ui-0.0.1-SNAPSHOT.jar /app.jar


# =========================
# Runtime Stage
# =========================
FROM public.ecr.aws/amazonlinux/amazonlinux:2023

# Install runtime dependencies only
RUN dnf --setopt=install_weak_deps=False install -q -y \
    java-21-amazon-corretto-headless \
    shadow-utils \
    && dnf clean all

# Use full curl (needed for telnet://)
RUN dnf -q -y swap libcurl-minimal libcurl-full \
    && dnf -q -y swap curl-minimal curl-full

# Create non-root user
ENV APPUSER=appuser
ENV APPUID=1000
ENV APPGID=1000

RUN useradd \
    --home "/app" \
    --create-home \
    --user-group \
    --uid "$APPUID" \
    "$APPUSER"

ENV JAVA_TOOL_OPTIONS=
ENV SPRING_PROFILES_ACTIVE=prod

WORKDIR /app
USER appuser

# Copy licenses and built JAR
COPY ./ATTRIBUTION.md ./LICENSES.md
COPY --chown=appuser:appuser --from=build-env /app.jar .

EXPOSE 8080

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar /app/app.jar"]
