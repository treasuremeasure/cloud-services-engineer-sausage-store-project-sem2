# syntax=docker/dockerfile:1.4

############################
# 1.  BUILD  (Maven) stage #
############################
FROM --platform=${BUILDPLATFORM} maven:3.8.2-openjdk-16-slim AS build

ARG VERSION=dev
WORKDIR /src

# сначала тянем зависимости — хороший кеш‑слой
COPY pom.xml .
RUN mvn -B dependency:go-offline

# копируем исходники и собираем
COPY src ./src
RUN mvn -B package -DskipTests -Dversion.application=${VERSION}

############################
# 2.  RUNTIME stage (JRE)  #
############################
FROM --platform=${TARGETPLATFORM} amazoncorretto:17-alpine3.18

ARG VERSION=dev
WORKDIR /app

# создаём не‑root пользователя
RUN addgroup -S jaruser\ 
 && adduser  -S jaruser -G jaruser \
 && apk add --no-cache dumb-init curl \
 && mkdir -p /logs/reports \
 && chown -R jaruser:jaruser /logs /app

# копируем JAR из build‑стадии
COPY --chown=jaruser:jaruser \
     --from=build /src/target/sausage-store-${VERSION}.jar /app/sausage-store.jar

EXPOSE 8080
USER jaruser
ENTRYPOINT ["dumb-init","java","-jar","-Dmyserver.bindPort=8080","/app/sausage-store.jar"]
