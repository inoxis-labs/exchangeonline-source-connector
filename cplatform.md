# Microsoft Exchange Online Source Connector for Confluent Platform

This Kafka *source* connector seamlessly integrates Microsoft Exchange Online (M365) mailboxes with
Kafka, by leveraging the Microsoft Graph API to fetch emails and publish them to Kafka topics. This
enables real-time email processing, making it an essential component for organizations looking to
streamline their email data workflows.

## Features

The Microsoft Exchange Online Source connector offers the following features:

- [At least once delivery](#at-least-once-delivery)
- [Supports one task](#supports-one-task)
- [Microsoft Exchange Online Resources](#microsoft-exchange-online-resources)

### At least once delivery

This connector guarantees that emails are delivered to the Kafka topic at least once. If the
connector restarts, there may be some duplicate records in the Kafka topic.

### Supports one task

The Microsoft Exchange Online Source connector supports running one task, one mailbox is covered by
one task.

### Microsoft Exchange Online resources

The connector supports fetching the following resources:

- **emails**: Emails from a Exchange Online
  mailbox.

## Limitations

- Supports only one task in community edition.
- Email body is limited to 1MB.

## Install the Microsoft Exchange Online Source Connector

You can install this connector by using
the [Confluent Connect Plugin Install](https://docs.confluent.io/platform/current/connect/userguide.html)
command or by manually downloading the ZIP file.

### Prerequisites

- The connector must be installed on every machine where Connect will run.
- Kafka Broker: Confluent Platform 3.3.0 or later.
- Connect: Confluent Platform 4.1.0 or later.
- Java 1.8.
- You must have an Azure App Registration with access to Microsoft Graph API.
- An installation of the latest connector version. Run the following command:

```bash
confluent connect plugin install *TBD* <inoxis/ExchangeOnlineSourceConnector:latest>
```

To install a specific version:

```bash
confluent connect plugin install *TBD* <inoxis/ExchangeOnlineSourceConnector:1.2.4>
```

### Install the Connector Manually

**TBD**
[Download and extract the ZIP file](https://www.confluent.io/hub/inoxis/ExchangeOnlineSourceConnector)
for your connector and follow
the [manual connector installation instructions](https://docs.confluent.io/platform/current/connect/userguide.html#connect-installing-plugins).

## Quick Start

In this quick start, you will configure the Microsoft Exchange Online Source connector to copy
emails from the Exchange mailbox to the Kafka topic.

### Start Confluent

Start the Confluent services using the
following [Confluent CLI](https://docs.confluent.io/confluent-cli/current/index.html) command:

```bash
confluent local services start
```

**Important**: Do not use the Confluent CLI in production environments. **TBD**

### Property-based Example

Configure the `exchangeonline-source-quickstart.properties` file with the following properties:

```properties
name=ExchangeOnlineSourceConnector
tasks.max=1
connector.class=io.inoxis.comms.kafkaconnect.exchangeonline.ExchangeOnlineEmailSourceConnector
exchangeonline.tenant.id=XXXXXXXX
exchangeonline.client.id=XXXXXXXX
exchangeonline.client.secret=XXXXXXXX
kafka.topic=m365-emails
```

Next, load the Source connector:

```bash
./bin/confluent local services connect connector load ExchangeOnlineSourceConnector --config ./etc/kafka-connect-jira/exchangeonline-source-quickstart.properties
```

### Configuration Properties

In addition to the common Kafka
Connect [source-related](https://kafka.apache.org/documentation.html#sourceconnectconfigs)
configuration options, this connector defines the following configuration properties.

| Property                                | Required | Default  | Description                                                                                                                                                                                                                       |
|-----------------------------------------|----------|----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `kafka.topic`                           | **Yes**  |          | Kafka topic to publish the emails to.                                                                                                                                                                                             |
| `tasks.max`                             | **Yes**  | 1        | Maximum number of tasks to use for this connector. To achieve maximum throughput, set this to the same number as the mailboxes configured to monitor.                                                                             |
| `email.mailboxes`                       | **Yes**  |          | List of mailboxes to monitor, separated by comma. Example: `a@sample.com,b@sample.com`                                                                                                                                            |
| `email.new.offset`                      | **Yes**  | earliest | Possible values are earliest and latest. This setting is used when the mailbox is being monitored for the first time. `earliest` - fetches emails from the oldest email in the mailbox. `latest` - fetches from current timestamp |
| `exchangeonline.tenant.id`              | **Yes**  |          | Azure App's Tenant ID (Instructions for setting up the app can be found in Azure-AD-App-Setup-Instructions.docx.                                                                                                                  |
| `exchangeonline.client.id`              | **Yes**  |          | Azure App's Client ID.                                                                                                                                                                                                            |
| `exchangeonline.client.secret`          | **Yes**  |          | Azure App's secret.                                                                                                                                                                                                               |
| `email.repository.enabled`              | **Yes**  | `false`  | This enables saving of raw emails(in mime format, .eml) to S3.                                                                                                                                                                    |
| `email.repository.s3.region`            | No       |          | AWS region of the the S3 bucket.                                                                                                                                                                                                  |
| `email.repository.s3.bucket`            | No       |          | The S3 bucket to use to save the raw emails.                                                                                                                                                                                      |
| `email.repository.s3.access.key.id`     | No       |          | AWS Access Key to access S3                                                                                                                                                                                                       |
| `email.repository.s3.secret.access.key` | No       |          | AWS Access Key Secret.                                                                                                                                                                                                            |
| `email.repository.s3.endpoint.url`      | No       |          | An optional AWS endpoint URL, useful for testing with Localstack.                                                                                                                                                                 |

### REST-based Examples

Use the following configuration when using API to start the connector:

#### With Default Configuration

The following minimal configuration configures the connector with default values, publishing emails
to topic
`"m365-emails"`, and saving raw emails to S3.

```bash
curl -X POST http://localhost:8083/connectors -H "Content-Type: application/json" -d '{
  "name": "ExchangeOnlineSourceConnector",
  "config": {
    "connector.class": "io.inoxis.comms.kafkaconnect.exchangeonline.ExchangeOnlineEmailSourceConnector",
    "tasks.max": 1,
    "kafka.topic": "m365-emails",
    "email.mailboxes": "a@sample.com",
    "email.new.offset": "earliest",
    "exchangeonline.tenant.id": "XXXXXXXX",
    "exchangeonline.client.id": "XXXXXXXX",
    "exchangeonline.client.secret": "XXXXXXXX",
    "email.repository.enabled": "true",
    "email.repository.s3.region": "us-east-1",
    "email.repository.s3.bucket": "my-emails-bucket",
    "email.repository.s3.access.key": "XXXXXXXX",
    "email.repository.s3.secret.access.key": "XXXXXXXX"
  }
}'
```

#### With Default Configuration and Multiple Mailboxes

```bash
curl -X POST http://localhost:8083/connectors -H "Content-Type: application/json" -d '{
  "name": "ExchangeOnlineSourceConnector",
  "config": {
    "connector.class": "io.inoxis.comms.kafkaconnect.exchangeonline.ExchangeOnlineEmailSourceConnector",
    "tasks.max": 2,
    "kafka.topic": "m365-emails",
    "email.mailboxes": "a@sample.com,b@sample.com",
    "email.new.offset": "earliest",
    "exchangeonline.tenant.id": "XXXXXXXX",
    "exchangeonline.client.id": "XXXXXXXX",
    "exchangeonline.client.secret": "XXXXXXXX",
    "email.repository.enabled": "true",
    "email.repository.s3.region": "us-east-1",
    "email.repository.s3.bucket": "my-emails-bucket",
    "email.repository.s3.access.key": "XXXXXXXX",
    "email.repository.s3.secret.access.key": "XXXXXXXX"
  }
}'
```

#### JSON Encoding

Below is an example of a JSON Encoding, in the same way Avro Encoding can also be configured.

```bash
curl -X POST http://localhost:8083/connectors -H "Content-Type: application/json" -d '{
  "name": "ExchangeOnlineSourceConnector",
  "config": {
    "connector.class": "io.inoxis.comms.kafkaconnect.exchangeonline.ExchangeOnlineEmailSourceConnector",
    "tasks.max": 2,
    "kafka.topic": "m365-emails",
    "email.mailboxes": "a@sample.com,b@sample.com",
    "email.new.offset": "earliest",
    "exchangeonline.tenant.id": "XXXXXXXX",
    "exchangeonline.client.id": "XXXXXXXX",
    "exchangeonline.client.secret": "XXXXXXXX",
    "email.repository.enabled": "true",
    "email.repository.s3.region": "us-east-1",
    "email.repository.s3.bucket": "my-emails-bucket",
    "email.repository.s3.access.key": "XXXXXXXX",
    "email.repository.s3.secret.access.key": "XXXXXXXX",
    "key.converter": "org.apache.kafka.connect.storage.StringConverter",
    "value.converter": "org.apache.kafka.connect.json.JsonConverter",
    "value.converter.schemas.enable": false
  }
}'
```

## Error handling

Since Apache Kafka 2.0, Kafka Connect has included error handling options for connectors. Here is a
brief overview of the error handling options available in Kafka
Connect: https://www.confluent.io/en-gb/blog/kafka-connect-deep-dive-error-handling-dead-letter-queues/

## Logging and tracking

ExchangeOnlineEmailSourceConnector adds ```source.poll.id``` to MDC context for each poll and the
same id is also populated into the headers of each kafka message published in that poll. The id can
be used to correlate the log messages to published kafka messages.

Please refer to [Logging](https://kafka.apache.org/documentation.html#connect_logging) on how to
enable context printing in the logs (look for ```connector.context```).

## License

**TBD**
_You can use this connector for a 30-day trial period without a license key_.

After 30 days, you must purchase a connector subscription, which
includes [Confluent enterprise license](https://www.confluent.io/subscription/) keys for subscribers
and enterprise-level support for Confluent Platform and your connectors. If you are a subscriber,
you can contact Confluent Support at [support@confluent.io](mailto:support@confluent.io) for more
information.

For license properties,
see [Confluent Platform license](configuration_options.html#jira-source-connector-license-config).
For information about the license topic,
see [License topic configuration](configuration_options.html#jira-source-license-topic-configuration).