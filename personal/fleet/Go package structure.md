Some random reference from my colleague

## To be purposeful, _packages must provide_, not contain

## Usecases
1. APM abstraction
In DH, each app has a need to collect telemetry data such as tracing and metric. We create an abstraction layer and hide them inside a pkg called apm. Once a breaking chage happens underneath (an otel version upgrade or vendor switch), the abstraction layer protect the app from the breaking change. If it has leaked any of its dependency toward the caller (e.g pkg that import apm for usage), there is a risk of breaking change happening.

#golang