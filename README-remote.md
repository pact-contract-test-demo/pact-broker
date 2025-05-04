## Demo: a remote Pact Broker

## CLI (examples)

```shell
docker compose --profile cli run --rm pact-client-remote pact-broker help
docker compose --profile cli run --rm pact-client-remote pact-broker list-environments
```

## CLI: create and test a webhook (contract-requiring-verification-published)

Shared environment variables for the CLI commands:

```shell
export PACT_BROKER_BASE_URL=https://.../
export PACT_BROKER_USERNAME=...
export PACT_BROKER_PASSWORD=...

# can be generated with: 'docker compose --profile cli --file ./docker-compose.yaml run --rm pact-client-remote broker generate-uuid'
export CONTRACT_REQUIRING_VERIFICATION_PUBLISHED_WEBHOOK_UUID=b4b91d0d-74a8-44ef-8102-e64ac5ce6c60
```

### Create a webhook for contract-requiring-verification-published

```shell
export GITHUB_REPO=pact-contract-test-demo/example-provider
export PACT_PROVIDER_NAME=pactflow-example-provider

# GitHub Personal Access Token or Fine Grained Access Token with repo:write grants
export GH_TOKEN=github_pat_...

docker compose --profile cli \
  --file ./docker-compose.yaml \
  run \
  --rm \
  --volume ${PWD}:/data \
  pact-client-remote \
  broker create-or-update-webhook \
  --broker-base-url=${PACT_BROKER_BASE_URL} \
  "https://api.github.com/repos/${GITHUB_REPO}/dispatches" \
  --header 'Content-Type: application/json' 'Accept: application/vnd.github.everest-preview+json' "Authorization: Bearer ${GH_TOKEN}" \
  --request POST \
  --data @/data/webhook-template.json \
  --uuid ${CONTRACT_REQUIRING_VERIFICATION_PUBLISHED_WEBHOOK_UUID} \
  --provider ${PACT_PROVIDER_NAME} \
  --contract-requiring-verification-published \
  --description "contract_requiring_verification_published for ${PACT_PROVIDER_NAME}"
```

### Test the webhook

```shell
docker compose \
  --profile cli \
  --file docker-compose.yaml \
  run \
  --rm \
  pact-client-remote \
  broker test-webhook \
  --broker-base-url=${PACT_BROKER_BASE_URL} \
  --uuid ${CONTRACT_REQUIRING_VERIFICATION_PUBLISHED_WEBHOOK_UUID}
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
export PACT_BROKER_BASE_URL="https://.../"
export PACT_BROKER_USERNAME=...
export PACT_BROKER_PASSWORD=...
export GIT_COMMIT=$(git rev-parse HEAD)
export GIT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
docker compose --profile cli --file ../pact-broker/docker-compose.yaml run --rm --volume ${PWD}:/w pact-client-remote \
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
export PACT_BROKER_BASE_URL="https://.../"
export PACT_BROKER_USERNAME=...
export PACT_BROKER_PASSWORD=...
export GIT_COMMIT=$(git rev-parse HEAD)
export GIT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
docker compose --profile cli --file ../pact-broker/docker-compose.yaml run --rm pact-client-remote \
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
export PACT_BROKER_BASE_URL="https://.../"
export PACT_BROKER_USERNAME=...
export PACT_BROKER_PASSWORD=...
export GIT_COMMIT=$(git rev-parse HEAD)
export GIT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
npm run test
```

## Provider: can-i-deploy

```shell
export PACTICIPANT="pactflow-example-provider"
export ENVIRONMENT=production
export PACT_DO_NOT_TRACK=true
export PACT_BROKER_BASE_URL="https://.../"
export PACT_BROKER_USERNAME=...
export PACT_BROKER_PASSWORD=...
export GIT_COMMIT=$(git rev-parse HEAD)
export GIT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
docker compose --profile cli --file ../pact-broker/docker-compose.yaml run --rm pact-client-remote \
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
export PACT_BROKER_BASE_URL="https://.../"
export PACT_BROKER_USERNAME=...
export PACT_BROKER_PASSWORD=...
export GIT_COMMIT=$(git rev-parse HEAD)
export GIT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
docker compose --profile cli --file ../pact-broker/docker-compose.yaml run --rm pact-client-remote \
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
export PACT_BROKER_BASE_URL="https://.../"
export PACT_BROKER_USERNAME=...
export PACT_BROKER_PASSWORD=...
export GIT_COMMIT=$(git rev-parse HEAD)
export GIT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
docker compose --profile cli --file ../pact-broker/docker-compose.yaml run --rm pact-client-remote \
  broker record-deployment \
  --pacticipant ${PACTICIPANT} \
  --version ${GIT_COMMIT} \
  --environment ${ENVIRONMENT}
```
