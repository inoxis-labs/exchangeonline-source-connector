# Kafka Connector for Microsoft Exchange Online

This Kafka _source_ connector seamlessly integrates Microsoft Exchange Online (M365) mailboxes with
Kafka, by leveraging the Microsoft Graph API to fetch emails and publish them to Kafka topics. This
enables real-time email processing, making it an essential component for organizations looking to
streamline their email data workflows.

## Note

- This Quick Start is for the fully-managed Confluent Cloud connector. If you are installing the
  connector locally for Confluent Platform, see **TBD
  ** [Microsoft Exchange Online Source Connector for Confluent Platform](https://docs.confluent.io/kafka-connectors/msexchange-online/current/).
- If you require private networking for fully-managed connectors, make sure to set up the proper
  networking beforehand. For more information,
  see [Manage Networking for Confluent Cloud Connectors](networking/internet-resource.html#clusters-connect-cloud).

## Features

The Microsoft Exchange Online Source connector supports the following features:

- **At least once delivery**: The connector guarantees that records are delivered at least once to
  the Kafka topic.
- **Supports multiple tasks**: The connector supports running one or more tasks. More tasks may
  improve performance. One mailbox is covered by one task only.
- **Offset management capabilities**: Supports offset management. For more information,
  see [Manage offsets](https://docs.confluent.io/cloud/current/connectors/offsets.html#manage-connector-offsets).

For more information and examples to use with the Confluent Cloud API for Connect, see
the [Confluent Cloud API for Connect Usage Examples](#connect-api-section.html#ccloud-connect-api).

## Limitations

Be sure to review the following information:

- connector limitations,
  see [Microsoft Exchange Online Source Connector limitations](#exchange-online-source-limits).
- If you plan to use one or more Single Message Transforms (SMTs),
  see [SMT Limitations](single-message-transforms.html#cc-single-message-transforms-limitations).
- If you plan to use Confluent Cloud Schema Registry,
  see [Schema Registry Enabled Environments](limits.html#connect-ccloud-environment-limits).

## Exchange Online Source Limits

- Supports only one task in community edition.
- Email body is limited to 1MB.

## Microsoft Exchange Online resources

The connector supports fetching the following resources:

- **emails**: Emails from a Exchange Online
  mailbox.

## Quick Start

Use this quick start to get up and running with the Confluent Cloud Microsoft Exchange Online Source
connector. The quick start provides the basics of selecting the connector and configuring it to get
emails from one mailbox.

### Prerequisites

- Authorized access to a [Confluent Cloud](https://www.confluent.io/confluent-cloud/) cluster on
  AWS, Azure, or Google Cloud.
- The Confluent CLI installed and configured for the cluster.
  See [Install the Confluent CLI](https://docs.confluent.io/confluent-cli/current/install.html).
- [Schema Registry](../get-started/schema-registry.html#cloud-sr-config) must be enabled for Schema
  Registry-based formats (e.g., Avro, JSON_SR, Protobuf).
- You must have an Azure App Registration with access to Microsoft Graph API.

### Steps to Set Up the Connector

1. Launch your Confluent Cloud cluster.
   See
   the [Quick Start for Confluent Cloud](https://docs.confluent.io/cloud/current/get-started/index.html#cloud-quickstart)
   for installation instructions.

2. Add a connector via the Confluent Cloud Console.
   In the left navigation menu, click **Connectors**. If you already have connectors in your
   cluster, click **+ Add connector**.

3. Select your connector.
   Click **Microsoft Exchange Online Source** connector card.

4. Enter the connector details and validate the configuration.

At the **Add Microsoft Exchange Online Source Connector** screen, complete the following:

**Kafka Credentials**:

1. Select the way you want to provide Kafka Cluster credentials. You can choose one of the following
   options:

- **My account**: This setting allows your connector to globally access everything that you have
  access to. With a user account, the connector uses an API key and secret to access the Kafka
  cluster. This option is not recommended for production.
- **Service account**: This setting limits the access for your connector by using a service account.
  This option is recommended for production.
- **Use an existing API key**: This setting allows you to specify an API key and a secret pair. You
  can use an existing pair or create a new one. This method is not recommended for production
  environments.

**Configuration**:
Enter the following configuration details:

- Email Configuration:
  - **Mailboxes to poll**: Enter the list of mailboxes to monitor, separated by commas. Example:
    `abc@mail.com,xyz@mail.com`.
  - **Preferred content type** for email: `text` or `html`.
  - **Where to start** reading emails from for new mailboxes: `earliest` or `latest`.
- Exchange Online Configuration:
  - **Tenant ID**: Azure App's Tenant ID (Instructions for setting up the app can be found in
    Azure-AD-App-Setup-Instructions.docx.
  - **Client ID**: Azure App's Client ID.
  - **Client Secret**: Azure App's secret.
- Email Repository Configuration:
  - **Enable Email Repository**: Enable saving of raw emails (in mime format, .eml) to S3.
  - **S3 Region**: AWS region of the S3 bucket.
  - **S3 Bucket**: The S3 bucket to use to save the raw emails.
  - **S3 Access Key ID**: AWS Access Key to access S3.
  - **S3 Secret Access Key**: AWS Access Key Secret.
- General Configuration:
  - **Topic Name**: Enter the topic name to which the emails to be written.
  - **Tasks max**: The number of tasks to run. To achieve maximum throughput, set this to the same
    number as the mailboxes configured to monitor.

**Review and Launch**:

1. Verify the connection details by previewing the running configuration.
2. After you’ve validated that the properties are configured to your satisfaction, click **Launch**.
   The status for the connector should go from **Provisioning** to **Running**.
3. Check for records being produced in the Kafka topic.
   Verify that records are being produced in the Kafka topic.

For more information and examples to use with the Confluent Cloud API for Connect, see
the [Confluent Cloud API for Connect Usage Examples](https://docs.confluent.io/cloud/current/connectors/connect-api-section.html#ccloud-connect-api)
section.

### Configuration Properties

In addition to the common Kafka
Connect [source-related](https://kafka.apache.org/documentation.html#sourceconnectconfigs)
configuration options, this connector defines the following configuration properties.

| Property                                | Required | Default  | Description                                                                                                                                                                                                                       |
| --------------------------------------- | -------- | -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
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

## Error handling

Since Apache Kafka 2.0, Kafka Connect has included error handling options for connectors. Here is a
brief overview of the error handling options available in Kafka
Connect: https://www.confluent.io/en-gb/blog/kafka-connect-deep-dive-error-handling-dead-letter-queues/

## Logging and tracking

ExchangeOnlineEmailSourceConnector adds `source.poll.id` to MDC context for each poll and the
same id is also populated into the headers of each kafka message published in that poll. The id can
be used to correlate the log messages to published kafka messages.

Please refer to [Logging](https://kafka.apache.org/documentation.html#connect_logging) on how to
enable context printing in the logs (look for `connector.context`).
