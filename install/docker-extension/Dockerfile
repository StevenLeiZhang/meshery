FROM golang:1.17-alpine AS builder
ENV CGO_ENABLED=0
RUN apk add --update make
WORKDIR /backend
COPY go.* .
ARG TARGETARCH
RUN --mount=type=cache,target=/go/pkg/mod \
  --mount=type=cache,target=/root/.cache/go-build \
  go mod download
COPY . .
RUN --mount=type=cache,target=/go/pkg/mod \
  --mount=type=cache,target=/root/.cache/go-build \
  make bin

FROM node:14.17-alpine3.13 AS client-builder
WORKDIR /ui
# cache packages in layer
COPY ui/package.json /ui/package.json
COPY ui/package-lock.json /ui/package-lock.json
ARG TARGETARCH
RUN --mount=type=cache,target=/usr/src/app/.npm-${TARGETARCH} \
  npm set cache /usr/src/app/.npm-${TARGETARCH} && \
  npm ci
# install
COPY ui /ui
RUN npm run build

FROM alpine
ARG GIT_STRIPPED_VERSION
ARG GIT_VERSION
ARG RELEASE_CHANNEL
LABEL org.opencontainers.image.title="meshery" \
  org.opencontainers.image.description="Design and operate your cloud native deployments with the extensible management plane, Meshery." \
  org.opencontainers.image.vendor="Layer5, Inc." \
  com.docker.desktop.extension.api.version=">= 0.2.0" \
  com.docker.extension.screenshots="[{\"alt\": \"Meshery extension for docker desktop\", \"url\": \"https://raw.githubusercontent.com/layer5io/layer5/master/src/assets/images/docker-extension-for-meshery.png\"}]"\
  com.docker.extension.detailed-description="The Docker Extension for Meshery extends Docker Desktop’s position as the cloud native developer’s go-to Kubernetes environment with easy access to the next layer of cloud native infrastructure: service meshes."\
  com.docker.desktop.extension.icon="https://raw.githubusercontent.com/meshery/meshery/master/install/docker-extension/docs/img/meshery-logo-light.svg" \
  com.docker.extension.publisher-url="https://meshery.io" \
  com.docker.extension.additional-urls="[{\"title\":\"Documentation\",\"url\":\"https://docs.meshery.io\"},{\"title\":\"Project\",\"url\":\"https://layer5.io/meshery\"}]"
COPY --from=builder /backend/bin/service /
COPY docker-compose.yaml .
COPY metadata.json .
RUN mkdir -p /mesheryctl/darwin && mkdir -p /mesheryctl/windows && mkdir -p /mesheryctl/linux
ADD https://github.com/meshery/meshery/releases/download/${GIT_VERSION}/mesheryctl_${GIT_STRIPPED_VERSION}_Darwin_x86_64.zip /mesheryctl/darwin/
ADD https://github.com/meshery/meshery/releases/download/${GIT_VERSION}/mesheryctl_${GIT_STRIPPED_VERSION}_Windows_x86_64.zip /mesheryctl/windows/
ADD https://github.com/meshery/meshery/releases/download/${GIT_VERSION}/mesheryctl_${GIT_STRIPPED_VERSION}_Linux_x86_64.zip /mesheryctl/linux/
RUN unzip /mesheryctl/darwin/mesheryctl_${GIT_STRIPPED_VERSION}_Darwin_x86_64.zip -d /mesheryctl/darwin/
RUN unzip /mesheryctl/windows/mesheryctl_${GIT_STRIPPED_VERSION}_Windows_x86_64.zip -d /mesheryctl/windows/
RUN unzip /mesheryctl/linux/mesheryctl_${GIT_STRIPPED_VERSION}_Linux_x86_64.zip -d /mesheryctl/linux/
COPY meshery-logo-light.svg .
COPY --from=client-builder /ui/build ui
CMD ./service -socket /run/guest-services/extension-meshery.sock
