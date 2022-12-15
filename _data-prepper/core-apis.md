---
layout: default
title: Core APIs
nav_order: 2
---

# Core APIs

All Data Prepper instances expose a server with some control APIs. By default, this server runs on port 4900. Some plugins, especially source plugins, may expose other servers. These will be on different ports, and their configurations are independent of the core API. For example, to shut down Data Prepper, you can run the following curl request:

```
curl -X POST http://localhost:4900/shutdown
```

## APIs

The following table lists the available APIs:

|             |                 |
|-------------|-----------------|
| API | Function |
| `GET /list`<br> `POST /list` | Returns a list of running pipelines. |
| `POST /shutdown` | Starts a graceful shutdown of Data Prepper. |
| `GET /metrics/prometheus`<br>`POST /metrics/prometheus`| Returns a scrape of the Data Prepper metrics in the prometheus text format. This API is provided as a `metricsRegistries` parameter in the Data Prepper configuration file `data-prepper-config.yaml` and has `Prometheus` as part of the registry. |
| `GET /metrics/sys`<br>`POST /metrics/sys` | Returns JVM metrics in the prometheus text format. This API is provided as a `metricsRegistries` parameter in the Data Prepper configuration file `data-prepper-config.yaml` and has `Prometheus` as part of the registry. |


## Configuring the server

You can configure your Data Prepper core APIs through the `data-prepper-config.yaml` file. 

### SSL/TLS connection

Many of the Getting Started guides for this project disable SSL on the endpoint. Disable SSL by using the following yaml configuration file:

```yaml
ssl: false
```

To enable SSL on your Data Prepper endpoint, configure your `data-prepper-config.yaml` file:

```yaml
ssl: true
keyStoreFilePath: "/usr/share/data-prepper/keystore.p12"
keyStorePassword: "secret"
privateKeyPassword: "secret"
```

For more information about configuring your Data Prepper server with SSL, see [Server Configuration](https://github.com/opensearch-project/data-prepper/blob/main/docs/configuration.md#server-configuration). If you are using a self-signed certificate, add the `-k` flag to quickly test sending curl requests for the core APIs with SSL. See the following example of the curl request:

```
curl -k -X POST https://localhost:4900/shutdown
```

### Authentication

The Data Prepper core APIs support HTTP Basic authentication. Set the username and password with the following configuration in `data-prepper-config.yaml`:

```yaml
authentication:
  http_basic:
    username: "myuser"
    password: "mys3cr3t"
```

You can disable authentication of core endpoints using the following configuration. Use this with caution because the shutdown API and others will be accessible to anybody with network access to your Data Prepper instance. Use the following yaml configuration file:

```yaml
authentication:
  unauthenticated:
```

## Peer Forwarder

Peer Forwarder can be configured to enable stateful aggregation across multiple Data Prepper nodes. For more information about configuring Peer Forwarder, see [Peer Forwarder Configuration](https://github.com/opensearch-project/data-prepper/blob/main/docs/peer_forwarder.md). It is supported by the `service_map_stateful`, `otel_trace_raw`, and `aggregate` processors.

## Shutdown timeouts

The Data Prepper `shutdown` API gives the sink and `ExecutorService` processor time to gracefully shut down and clear any remaining data when the API is invoked. The default shutdown timeout for the `ExecutorService` processor is 10 seconds, and can be configured with the following optional parameters:

```yaml
processorShutdownTimeout: "PT15M"
sinkShutdownTimeout: 30s
```

The values for these parameters are parsed into a `Duration` object via the [DataPrepperDurationDeserializer](https://github.com/opensearch-project/data-prepper/tree/main/data-prepper-core/src/main/java/org/opensearch/dataprepper/parser/DataPrepperDurationDeserializer.java). 