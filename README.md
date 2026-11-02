# kafka-connect-library

Kafka Connect plugins and add-ons OCI library.

Each library is a `FROM scratch` OCI image that contains only the plugin or JAR files, meant to be mounted
as an [image volume](https://kubernetes.io/docs/tasks/configure-pod-container/image-volumes/) in a
[kafka-connect-operator](https://github.com/b1zzu/kafka-connect-operator) cluster.

## Libraries

| Library | Image | Contents | Operator field | Upstream |
|---|---|---|---|---|
| debezium-postgres | `ghcr.io/b1zzu/kafka-connect-library/debezium-postgres` | `/debezium-postgres/`: Debezium PostgreSQL connector bundled with the AWS Advanced JDBC Wrapper | `spec.plugins` | [Debezium](https://debezium.io/documentation/reference/stable/connectors/postgresql.html), [AWS Advanced JDBC Wrapper](https://github.com/aws/aws-advanced-jdbc-wrapper) |
| jmx-exporter | `ghcr.io/b1zzu/kafka-connect-library/jmx-exporter` | `/jmx_prometheus_javaagent.jar` | `spec.metrics.jmxExporter` (default) | [prometheus/jmx_exporter](https://github.com/prometheus/jmx_exporter) |
| kafka-connect-datagen | `ghcr.io/b1zzu/kafka-connect-library/kafka-connect-datagen` | `/kafka-connect-datagen/`: Datagen source connector for generating test data | `spec.plugins` | [confluentinc/kafka-connect-datagen](https://github.com/confluentinc/kafka-connect-datagen) |
| log4j-layout-template-json | `ghcr.io/b1zzu/kafka-connect-library/log4j-layout-template-json` | `/log4j-layout-template-json.jar` | `spec.logging.log4jJsonLayout` (default) | [Log4j JSON Template Layout](https://logging.apache.org/log4j/2.x/manual/json-template-layout.html) |
| msk-iam-auth | `ghcr.io/b1zzu/kafka-connect-library/msk-iam-auth` | `/aws-msk-iam-auth-all.jar` | `spec.libraries` | [aws/aws-msk-iam-auth](https://github.com/aws/aws-msk-iam-auth) |
| sample-plugins | `ghcr.io/b1zzu/kafka-connect-library/sample-plugins` | `/connect-file.jar`, `/connect-mirror.jar` | `spec.plugins` | [Apache Kafka](https://kafka.apache.org/) |

## Tags

Every image is published for `linux/amd64` and `linux/arm64` with two tags:

- `<version>`: the upstream version of the library, e.g. `msk-iam-auth:2.3.9`
- `latest`: the most recent build from `main`

Upstream versions are kept up-to-date by [Renovate](https://docs.renovatebot.com/).

## Usage

Mount the images in a kafka-connect-operator cluster using the field from the [Libraries](#libraries) table:

```yaml
apiVersion: kafka-connect.b1zzu.net/v1alpha1
kind: Cluster
metadata:
  name: my-cluster
spec:
  plugins:
    - name: debezium-postgres
      image: ghcr.io/b1zzu/kafka-connect-library/debezium-postgres:3.7.0.Final
  libraries:
    - name: msk-iam-auth
      image: ghcr.io/b1zzu/kafka-connect-library/msk-iam-auth:2.3.9
```

`jmx-exporter` and `log4j-layout-template-json` are the operator defaults for metrics and JSON logging, no configuration needed.

## Adding a library

1. Create `containers/<name>/Dockerfile` that copies the files into a `FROM scratch` image.
2. Declare the upstream version as the first `ARG <NAME>_VERSION=<version>` of the Dockerfile, it is used as the image tag.
3. Add a Renovate custom manager and group for the version in [renovate.json](renovate.json).
4. Add a row to the [Libraries](#libraries) table.

The [Build and Push Libraries](.github/workflows/build-push.yml) workflow builds the changed libraries on pull requests and pushes them to `ghcr.io/b1zzu/kafka-connect-library/<name>` on `main`.
