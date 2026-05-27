#Это так называемый мультистейдж docker-файл. Сначала делаем образ, который соберёт нам java-приложение,
#а затем из него берём собранный артефакт и кладём его в новый образ, чтобы оставить в нём меньше лишнего

# build
FROM maven:3.9.9-eclipse-temurin-17 AS builder
# задаём переменную VERSION, которая будет использоваться для сборки проекта
ARG VERSION=latest
ENV VERSION=${VERSION}
WORKDIR /usr/src/app
COPY ./src ./src
COPY *.xml ./

RUN mvn package -DskipTests

# release
FROM eclipse-temurin:17-jre-jammy
ARG VERSION=latest
WORKDIR /app
# создаём пользователя jaruser
RUN groupadd --system jaruser \
    && useradd --system --gid jaruser --home-dir /nonexistent --shell /usr/sbin/nologin jaruser \
    && apt-get update \
    && apt-get install -y --no-install-recommends dumb-init curl \
    && rm -rf /var/lib/apt/lists/* \
    && mkdir -p /logs/reports \
    && chown -R jaruser:jaruser /logs
COPY --chown=jaruser:jaruser \
     --from=builder /usr/src/app/target/sausage-store-${VERSION}.jar ./sausage-store.jar
EXPOSE 8080
# приложение будет запускаться под пользователем jaruser
USER jaruser
ENTRYPOINT ["dumb-init", "java", "-jar", "-Dmyserver.bindPort=8080", "./sausage-store.jar"]
