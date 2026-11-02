# kafka-connect-library

Kafka Connect plugins and add-ons OCI library.

Each library is a `FROM scratch` OCI image that contains only the plugin or JAR files, meant to be mounted
as an [image volume](https://kubernetes.io/docs/tasks/configure-pod-container/image-volumes/) in a
Kafka Connect cluster managed by the [kafka-connect-operator](https://github.com/b1zzu/kafka-connect-operator).

## Libraries

Every library is published as `ghcr.io/b1zzu/kafka-connect-library/<library>`.

| Library | Contents | Operator field |
|---|---|---|
| [debezium-postgres](https://debezium.io/documentation/reference/stable/connectors/postgresql.html) | `/debezium-postgres/`: Debezium PostgreSQL connector bundled with the [AWS Advanced JDBC Wrapper](https://github.com/aws/aws-advanced-jdbc-wrapper) | `spec.plugins` |
| [jmx-exporter](https://github.com/prometheus/jmx_exporter) | `/jmx_prometheus_javaagent.jar` | `spec.metrics.jmxExporter` (default) |
| [kafka-connect-datagen](https://github.com/confluentinc/kafka-connect-datagen) | `/kafka-connect-datagen/`: Datagen source connector for generating test data | `spec.plugins` |
| [log4j-layout-template-json](https://logging.apache.org/log4j/2.x/manual/json-template-layout.html) | `/log4j-layout-template-json.jar` | `spec.logging.log4jJsonLayout` (default) |
| [msk-iam-auth](https://github.com/aws/aws-msk-iam-auth) | `/aws-msk-iam-auth-all.jar` | `spec.libraries` |
| [sample-plugins](https://kafka.apache.org/) | `/connect-file.jar`, `/connect-mirror.jar` | `spec.plugins` |

## Tags

Every image is published for `linux/amd64` and `linux/arm64` with two tags:

- `<version>`: the upstream versions of the library joined with `-`, e.g. `msk-iam-auth:2.3.9` or `debezium-postgres:3.7.0.Final-4.5.0`
- `latest`: the most recent build from `main`

Upstream versions are kept up-to-date by [Renovate](https://docs.renovatebot.com/).

## Usage

Mount the images in a Kafka Connect cluster managed by the operator using the field from the [Libraries](#libraries) table:

```yaml
apiVersion: kafka-connect.b1zzu.net/v1alpha1
kind: Cluster
metadata:
  name: my-cluster
spec:
  plugins:
    - name: debezium-postgres
      image: ghcr.io/b1zzu/kafka-connect-library/debezium-postgres:3.7.0.Final-4.5.0
  libraries:
    - name: msk-iam-auth
      image: ghcr.io/b1zzu/kafka-connect-library/msk-iam-auth:2.3.9
```

`jmx-exporter` and `log4j-layout-template-json` are the default images of `spec.metrics.jmxExporter` and
`spec.logging.log4jJsonLayout`, enable them with an empty object:

```yaml
spec:
  metrics:
    jmxExporter: {}
  logging:
    log4jJsonLayout: {}
```

## Adding a library

1. Create `containers/<name>/Dockerfile` that copies the files into a `FROM scratch` image.
2. Declare each upstream version as an `ARG <NAME>_VERSION=<version>` in the Dockerfile, their values joined with `-` are used as the image tag.
3. Add a Renovate custom manager and group for the version in [renovate.json](renovate.json).
4. Add a row to the [Libraries](#libraries) table.

The [Build and Push Libraries](.github/workflows/build-push.yml) workflow builds the changed libraries on pull requests and pushes them to `ghcr.io/b1zzu/kafka-connect-library/<name>` on `main`.
