# Glossary

## Ecosystem
A distributed execution topology of cooperating apps, services, and surfaces. An ecosystem is broader than any single app and may be centered around a workflow.

## Workflow
A cohesive unit of functionality that may span multiple apps and services. A workflow can be ephemeral, batch-oriented, or long-running.

## App
A bounded executable or business capability. An app may participate in one or more workflows.

## Service
A functional capability exposed for reuse by apps or workflows. A service is defined by what it does, not by its transport or surface, and may exist on multiple surfaces.

## Surface
A concrete runtime substrate such as local Python, Docker, Kubernetes, WSL, or AWS. A surface hosts app or service instances but is not itself the service.

## Transport
The communication mechanism used to access a service, such as SDK, gRPC, REST, or Kafka. Transport is incidental to service identity and may change without changing the service's function.
