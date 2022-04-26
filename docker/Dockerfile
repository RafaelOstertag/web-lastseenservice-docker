FROM curlimages/curl:latest AS download
ARG VERSION
ARG REPOSITORY_USER
ARG REPOSITORY_PASSWORD
RUN curl -Lf -u "${REPOSITORY_USER}:${REPOSITORY_PASSWORD}"  -o /tmp/lastseenservice.jar https://rafaelostertag.jfrog.io/artifactory/maven/ch/guengel/webtools/lastseenservice/${VERSION}/lastseenservice-${VERSION}-jar-with-dependencies.jar

FROM --platform=$BUILDPLATFORM ghcr.io/graalvm/graalvm-ce:ol8-java11-22 AS executable
ARG BUILDPLATFORM
ARG VERSION
WORKDIR /tmp
RUN gu install native-image
COPY --from=download /tmp/lastseenservice.jar /tmp/
RUN native-image --verbose -jar lastseenservice.jar

FROM --platform=$BUILDPLATFORM debian:11.2-slim AS runnable
ARG BUILDPLATFORM
ARG USERID=10000
RUN apt install libstdc++6 &&\
    useradd -U -d /app -m -u $USERID lastseenservice &&\
    rm -rf /var/lib/apt/*

COPY --from=executable /tmp/lastseenservice /app/lastseenservice

WORKDIR /app/
USER $USERID
ENTRYPOINT ["/app/lastseenservice"]

