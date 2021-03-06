FROM mcr.microsoft.com/dotnet/core/sdk:3.1 as build-env

COPY . .

ENV DOTNET_SKIP_FIRST_TIME_EXPERIENCE 1
ENV DOTNET_CLI_TELEMETRY_OPTOUT 1

# Build self contained

RUN dotnet publish -c Release src/Microsoft.Crank.Agent --output /app

# Build runtime image
FROM mcr.microsoft.com/dotnet/core/aspnet:3.1

# Install dependencies
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        git \
        procps \
        cgroup-bin \
        curl

# Install docker client
ENV DOCKER_VERSION 17.09.0-ce
RUN cd /tmp \
    && curl https://download.docker.com/linux/static/stable/x86_64/docker-$DOCKER_VERSION.tgz -o docker.tgz \
    && tar xvzf docker.tgz \
    && cp docker/docker /usr/bin \
    && rm -rf docker.tgz docker

# Install perfcollect
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        zip \
        unzip \
        linux-perf \
        lttng-tools \
        lttng-modules-dkms \
        liblttng-ust-dev \
        binutils

RUN cp /usr/bin/perf_4.19 /usr/bin/perf

# Downloading perfcollect - This branch contains SIGINT fixes
ADD https://raw.githubusercontent.com/sebastienros/perfview/sebros/sigint/src/perfcollect/perfcollect /usr/bin/perfcollect

RUN chmod +x /usr/bin/perfcollect

COPY --from=build-env /app /app

ENTRYPOINT [ "/app/crank-agent", "--url", "http://*:5001" ]
