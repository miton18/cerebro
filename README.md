# cerebro
Infrastructure automation made fast

## How it works
*cerebro* use a client-server model, an agent is installed on every host and a main server is started somewhere.
When an agent start, it generate a secret key and try to contact his master, use [mTLS](https://github.com/islishude/grpc-mtls-example/blob/master/client/main.go)

> Cerebro server can easily perform a first SSH connection in order to install and put his IP on the target host for Cerebro agent configuration.
> Agent and Server us gRPC.

environment data are store on the server under `environment/` directory, every environment has 2 informations kind `/environment/prod/host` relative to host scoped informations and `/environment/prod/roles` which holds informations about a group of server (webs ones, databases or proxy ones) (thx Ansible).

Every thing you need to do to configure your hosts is called *actions* an action must be idempotent, because the server can send several time the same action to the host.

Actions are groupped into roles, roles match a group of hosts and apply a collection of action

## Actions
Every action must fill the following spec, see the first examples

|Name|Version|Help|Request payload|Result payload|
|----|-------|----|---------------|--------------|
|file|v1|Copy a templatized file onto the target host|path/name/content/mode|-|

An action is a simple as writing a Golang struct

```go
type Action interface{
    Name() string
    Version() semver.Version
    Help() string
    Execute(Request) Response // TODO
    Schema() string // OpenAPI Schema
}

type Request interface{
    DryRun() bool
    onChanges() []Action
    Payload() map[string]interface{}
}

type Response interface{
    HasChanged() bool
    Payload() map[string]interface{}
}

```
## Reporters

You can also link roles or actions to *reporters*, reporters are like monitoring probes, it allow you, for example, to start an NGINX server and be sure it is always up.

```go
type Reporter interface {
  Name() string
  Version() semver.Version
  Help() string
  Execute(Request) Response
  Schema() string // OpenAPI Schema
}
```
