# pact-broker

Docs: https://docs.pact.io/pact_broker

- https://github.com/pact-foundation/pact_broker
- https://github.com/pact-foundation/pact_broker/wiki
- https://github.com/pact-foundation/pact_broker-client
- https://docs.pact.io/pact_broker/webhooks
- https://docs.pact.io/pact_broker/webhooks/template_library
- https://docs.pact.io/pact_broker/webhooks#example-cicd-and-webhook-configuration
- https://docs.pactflow.io/docs/workshops/ci-cd
- https://github.com/pact-foundation/pact-broker-docker

## Demos

- see [README-local.md](README-local.md) for a local setup 
- see [README-remote.md](README-remote.md) for a remote setup including webhooks

## Flows: Consumer

> with a changed contract

```mermaid
sequenceDiagram
    participant ConsumerCI as Consumer CI
    participant PactBroker as Pact Broker
    participant ProviderCI as Provider CI

    ConsumerCI-->>ConsumerCI: test, create contracts
    ConsumerCI->>PactBroker: publish contract
    PactBroker->>+ProviderCI: webhook: verify changes
    ProviderCI->>+PactBroker: fetch contracts
    PactBroker-->>-ProviderCI: contracts requiring verification
    ConsumerCI->>+PactBroker: can-i-deploy
    ProviderCI-->>ProviderCI: verify
    ProviderCI-->>-PactBroker: publish results
    PactBroker-->>-ConsumerCI: verified (or timeout)
    ConsumerCI-->>ConsumerCI: deploy
    ConsumerCI->>+PactBroker: record-deployment
```

> without changed contracts

```mermaid
sequenceDiagram
    participant ConsumerCI as Consumer CI
    participant PactBroker as Pact Broker
    participant ProviderCI as Provider CI

    ConsumerCI-->>ConsumerCI: test, create contracts
    ConsumerCI->>PactBroker: publish contract
    ConsumerCI->>+PactBroker: can-i-deploy
    PactBroker-->>-ConsumerCI: verified
    ConsumerCI-->>ConsumerCI: deploy
    ConsumerCI->>+PactBroker: record-deployment
```

## Flows: Producer

> with a changed contract

```mermaid
sequenceDiagram
    participant ConsumerCI as Consumer CI
    participant PactBroker as Pact Broker
    participant ProviderCI as Provider CI

    ProviderCI-->>ProviderCI: test
    ProviderCI->>+PactBroker: fetch contracts
    PactBroker-->>-ProviderCI: contracts requiring verification
    ProviderCI-->>ProviderCI: verify
    ProviderCI->>PactBroker: publish results
    ProviderCI->>PactBroker: can-i-deploy
    ProviderCI-->>ProviderCI: deploy
    ProviderCI->>+PactBroker: record-deployment
```

> without changed contracts

```mermaid
sequenceDiagram
    participant ConsumerCI as Consumer CI
    participant PactBroker as Pact Broker
    participant ProviderCI as Provider CI

    ProviderCI-->>ProviderCI: test
    ProviderCI->>+PactBroker: fetch contracts
    PactBroker-->>-ProviderCI: contracts requiring verification
    ProviderCI->>PactBroker: can-i-deploy
    ProviderCI-->>ProviderCI: deploy
    ProviderCI->>+PactBroker: record-deployment
```
