FROM bellsoft/liberica-openjdk-alpine:17 as builder
ADD . /src
WORKDIR /src
RUN ./mvnw package -DskipTests

FROM alpine:3.16.2 as packager

RUN  apk update \
  && apk upgrade \
  && apk add ca-certificates \
  && update-ca-certificates \
  && apk add --update coreutils \
  && apk add --no-cache binutils \
  && rm -rf /var/cache/apk/* \
  && apk add --update openjdk17 openjdk17-jmods tzdata curl unzip bash \
  && apk add --no-cache nss \
  && rm -rf /var/cache/apk/*

ENV JAVA_MINIMAL="/opt/java-minimal"
RUN /usr/lib/jvm/java-17-openjdk/bin/jlink  \
    --verbose \
    --add-modules \
        java.base,java.sql,java.naming,java.desktop,java.management,java.security.jgss,java.instrument \
    --compress 2 --strip-debug --no-header-files --no-man-pages \
    --release-info="add:IMPLEMENTOR=radistao:IMPLEMENTOR_VERSION=radistao_JRE" \
    --output "$JAVA_MINIMAL"

FROM alpine:3.16.2
LABEL maintainer="author@mail.com"
ENV JAVA_HOME=/opt/java-minimal
ENV PATH="$PATH:$JAVA_HOME/bin"
COPY --from=packager "$JAVA_HOME" "$JAVA_HOME"
COPY --from=builder /src/target/counter-service-*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app.jar"]