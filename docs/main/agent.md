---
layout: docs
---

## Agent

### Install pre-built binaries

The easiest way to install yorkie is from pre-built binaries:

1. Download the compressed archive file for your platform from [Releases](https://github.com/yorkie-team/yorkie/releases).
2. Unpack the archive file. This results in a directory containing the binaries.
3. Add the executable binaries to your path. For example, rename and/or move the binaries to a directory in your path (like /usr/local/bin), or add the directory created by the previous step to your path.
4. From a shell, test that yorkie is in your path:
```bash
$ yorkie --version
Yorkie: 0.1.8
...
```

### Commands (CLI)

Yorkie is controlled via a very easy to use command-line interface (CLI). Yorkie is only a single command-line application: `yorkie`. This application then takes a subcommand such as `agent`.

The yorkie CLI is a well-behaved command line application. In erroneous cases, a non-zero exit status will be returned. It also responds to `-h` and `---help` as you'd most likely expect.

To get help for any specific command, pass the `-h` flag to the relevant subcommand. For example, to see help about the members subcommand:

```bash
$ yorkie agent -h
Starts yorkie agent

Usage:
  yorkie agent [options] [flags]

Flags:
      --auth-webhook-cache-auth-ttl duration      TTL value to set when caching authorized webhook response. (default 10s)
      --auth-webhook-cache-unauth-ttl duration    TTL value to set when caching unauthorized webhook response. (default 10s)
      --auth-webhook-max-wait-interval duration   Maximum wait interval for authorization webhook. (default 3s)
      --auth-webhook-methods strings              List of methods that require authorization checks. If no value is specified, all methods will be checked.
      --auth-webhook-url string                   URL of remote service to query authorization
      --authorization-webhook-max-retries uint    Maximum number of retries for an authorization webhook. (default 10)
      --backend-snapshot-interval uint            Interval of changes to create a snapshot (default 100)
      --backend-snapshot-threshold uint           Threshold that determines if changes should be sent with snapshot when the number of changes is greater than this value. (default 500)
  -c, --config string                             Config path
      --etcd-endpoints strings                    Comma separated list of etcd endpoints
  -h, --help                                      help for agent
      --metrics-port int                          Metrics port (default 11102)
      --mongo-connection-timeout duration         Mongo DB's connection timeout (default 5s)
      --mongo-connection-uri string               MongoDB's connection URI (default "mongodb://localhost:27017")
      --mongo-ping-timeout duration               Mongo DB's ping timeout (default 5s)
      --mongo-yorkie-database string              Yorkie's database name in MongoDB (default "yorkie-meta")
      --rpc-cert-file string                      RPC certification file's path
      --rpc-key-file string                       RPC key file's path
      --rpc-port int                              RPC port (default 11101)
```

### Running Agent

Next, let's start a Yorkie agent. Agent runs until they're told to quit and handle the communication of maintenance tasks of Agent. and start the agent:

```bash
$ ./bin/yorkie agent
```

*NOTE: Yorkie uses MongoDB to store it's data. To start MongoDB, `docker-compose -f docker/docker-compose.yml up --build -d` in [the project root](https://github.com/yorkie-team/yorkie).*

Use the `--mongo-connection-uri` option to change he MongoDB connectionURI.

```bash
$ ./bin/yorkie agent --mongo-connection-uri mongodb://localhost:27017
```
