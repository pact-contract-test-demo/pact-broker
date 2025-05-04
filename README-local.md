## Demo: a local Pact Broker

Setup:

```shell
docker network create pact-broker
docker compose up
```

## CLI (examples)

```shell
docker compose --profile cli run --rm pact-client pact-broker help
docker compose --profile cli run --rm pact-client pact-broker list-environments
```

## Consumer: publish

```shell
npm install
npm run test:pact
```

```shell
export PACT_PROVIDER=pactflow-example-provider
export PACTICIPANT="pactflow-example-consumer"
export ENVIRONMENT=production
export GIT_COMMIT=$(git rev-parse HEAD)
export GIT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
docker compose --profile cli --file ../pact-broker/docker-compose.yaml run --rm --volume ${PWD}:/w pact-client \
  publish /w/pacts \
  --consumer-app-version ${GIT_COMMIT} \
  --branch ${GIT_BRANCH}
```

## Consumer: can-i-deploy

> it is expected to fail because the provider has not verified the pact(s), yet

### Linux

```shell
export PACT_PROVIDER=pactflow-example-provider
export PACTICIPANT="pactflow-example-consumer"
export ENVIRONMENT=production
export GIT_COMMIT=$(git rev-parse HEAD)
export GIT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
docker compose --profile cli --file ../pact-broker/docker-compose.yaml run --rm pact-client \
  broker can-i-deploy \
  --pacticipant ${PACTICIPANT} \
  --version ${GIT_COMMIT} \
  --to-environment ${ENVIRONMENT} \
  --retry-while-unknown 10 \
  --retry-interval 2
```

## Provider: verify

```shell
npm install
```

```shell
export PACT_DO_NOT_TRACK=true
export PACT_BROKER_PUBLISH_VERIFICATION_RESULTS=true
export PACT_BROKER_BASE_URL="http://localhost:9292/"
export PACT_BROKER_USERNAME=agile-glue-rw
export PACT_BROKER_PASSWORD=agile-glue-rw-pass
export GIT_COMMIT=$(git rev-parse HEAD)
export GIT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
npm run test
```

## Provider: can-i-deploy

```shell
export PACTICIPANT="pactflow-example-provider"
export ENVIRONMENT=production
export PACT_DO_NOT_TRACK=true
export PACT_BROKER_BASE_URL="http://localhost:9292/"
export PACT_BROKER_USERNAME=agile-glue-rw
export PACT_BROKER_PASSWORD=agile-glue-rw-pass
export GIT_COMMIT=$(git rev-parse HEAD)
export GIT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
docker compose --profile cli --file ../pact-broker/docker-compose.yaml run --rm pact-client \
  broker can-i-deploy \
  --pacticipant ${PACTICIPANT} \
  --version ${GIT_COMMIT} \
  --to-environment ${ENVIRONMENT}
```

## Provider: deploy

> assumes another task with the actual deployment just before the "record-deployment" step below

```shell
export PACTICIPANT="pactflow-example-provider"
export ENVIRONMENT=production
export PACT_DO_NOT_TRACK=true
export PACT_BROKER_BASE_URL="http://localhost:9292/"
export PACT_BROKER_USERNAME=agile-glue-rw
export PACT_BROKER_PASSWORD=agile-glue-rw-pass
export GIT_COMMIT=$(git rev-parse HEAD)
export GIT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
docker compose --profile cli --file ../pact-broker/docker-compose.yaml run --rm pact-client \
  broker record-deployment \
  --pacticipant ${PACTICIPANT} \
  --version ${GIT_COMMIT} \
  --environment ${ENVIRONMENT}
```

> Consumer: "can-i-deploy" should now succeed (see above)

## Consumer: deploy

> assumes another task with the actual deployment just before the "record-deployment" step below

```shell
export PACT_PROVIDER=pactflow-example-provider
export PACTICIPANT="pactflow-example-consumer"
export ENVIRONMENT=production
export GIT_COMMIT=$(git rev-parse HEAD)
export GIT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
docker compose --profile cli --file ../pact-broker/docker-compose.yaml run --rm pact-client \
  broker record-deployment \
  --pacticipant ${PACTICIPANT} \
  --version ${GIT_COMMIT} \
  --environment ${ENVIRONMENT}
```

## Broker

Shutdown/cleanup:

```shell
docker compose down --remove-orphans --volumes
docker network rm pact-broker
```
