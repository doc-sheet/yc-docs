# Visualizing logs in {{ grafana-name }} using the {{ cloud-logging-full-name }} plugin


The [{{ cloud-logging-full-name }} plugin for {{ grafana-name }}](https://github.com/yandex-cloud/grafana-logs-plugin/tree/master) is an extension for {{ grafana-name }} that allows you to add [{{ cloud-logging-name }}](https://yandex.cloud/en/services/logging) as a data source.

To visualize logs:

1. [Install the plugin](#install-plugin).
1. [Create a service account](#create-account).
1. [Create an authorized key for the service account](#create-key).
1. [Create a log group](#create-group).
1. [Add records to the log group](#add-records).
1. [Connect a data source in {{ grafana-name }}](#connect-plugin).
1. [View the logs in {{ grafana-name }}](#see-logs).

If you no longer need the resources you created, [delete them](#clear-out).

## Get your cloud ready {#before-begin}

{% include [before-you-begin](../_tutorials_includes/before-you-begin.md) %}

### Required paid resources {#paid-resources}

The cost of resources includes a fee for logging operations and log storage in a log group (see [{{ cloud-logging-full-name }} pricing](../../logging/pricing.md)).

## Install the plugin {#install-plugin}

1. [Download](https://github.com/yandex-cloud/grafana-logs-plugin/releases) the archive with the latest plugin version.
1. Unpack the archive to the directory with plugins. The path to the directory with plugins is specified in the [{{ grafana-name }} configuration](https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/).

   ```bash
   unzip <path_to_archive> -d <path_to_plugin_directory>
   ```

   {% note info %}

   In macOS, after you unpack the plugin archive manually run the `/opt/homebrew/var/lib/grafana/plugins/yandexcloud-logging-datasource/yc-logs-plugin_darwin_arm64` file and [allow launching third-party applications](https://support.apple.com/en-us/HT202491#openanyway) in the system settings.

   {% endnote %}

1. Allow loading an unsigned plugin. To do this, specify the plugin name in the `allow_loading_unsigned_plugins` parameter of the {{ grafana-name }} configuration file:

   ```text
   allow_loading_unsigned_plugins = yandexcloud-logging-datasource
   ```

   For more information about loading unsigned plugins, see the [{{ grafana-name }} documentation](https://grafana.com/docs/grafana/latest/administration/plugin-management/#allow-unsigned-plugins).

1. Restart the {{ grafana-name }} server:

   {% list tabs group=operating_system %}

   - Linux {#linux}

     ```bash
     sudo systemctl restart grafana-server
     ```

   - Windows {#windows}

     1. Click **Win+R**.
     1. In the window that opens, enter `services.msc` and click **OK**.
     1. Right-click the line with `Grafana` and select **Restart**.

   - macOS {#macos}

     ```bash
     brew services restart grafana
     ```

   {% endlist %}

## Create a service account {#create-account}

{% list tabs group=instructions %}

- Management console {#console}

  1. In the [management console]({{ link-console-main }}), select the folder where you want to create a service account.
  1. In the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_iam }}**.
  1. Click **{{ ui-key.yacloud.iam.folder.service-accounts.button_add }}**.
  1. Enter a name for the service account: `grafana-plugin`.
  1. Click **{{ ui-key.yacloud.iam.folder.service-account.label_add-role }}** and select the `logging.reader` role.
  1. Click **{{ ui-key.yacloud.iam.folder.service-account.popup-robot_button_add }}**.

- CLI {#cli}

  {% include [cli-install](../../_includes/cli-install.md) %}

  {% include [default-catalogue](../../_includes/default-catalogue.md) %}

  1. Create a service account named `grafana-plugin`:

     ```bash
     yc iam service-account create --name grafana-plugin
     ```

     Result:

     ```text
     id: nfersamh4s**********
     folder_id: b1gc1t4cb6**********
     created_at: "2023-09-26T10:36:29.726397755Z"
     name: grafana-plugin
     ```

     Save the ID of the `grafana-plugin` service account (`id`) and the ID of the folder where you created it (`folder_id`).

  1. Assign the `logging.reader` role for the folder to the service account:

     ```bash
     yc resource-manager folder add-access-binding <folder_ID> \
       --role logging.reader \
       --subject serviceAccount:<service_account_ID>
     ```

     Result:

     ```text
     done (1s)
     ```

- {{ TF }} {#tf}

  
  If you do not have {{ TF }} yet, [install it and configure the {{ yandex-cloud }} provider](../../tutorials/infrastructure-management/terraform-quickstart.md#install-terraform).


  1. In the configuration file, describe the service account parameters:

     ```hcl
     resource "yandex_iam_service_account" "grafana-plugin" {
       name        = "grafana-plugin"
       folder_id   = "<folder_ID>"
     }

     resource "yandex_resourcemanager_folder_iam_member" "reader" {
       folder_id = "<folder_ID>"
       role      = "logging.reader"
       member    = "serviceAccount:${yandex_iam_service_account.grafana-plugin id}"
     }
     ```

     Where:

     * `name`: Service account name. This is a required parameter.
     * `folder_id`: [Folder ID](../../resource-manager/operations/folder/get-id.md). This is an optional parameter. It defaults to the value specified in the provider settings.
     * `role`: Role being assigned.

     For more information about `yandex_iam_service_account` properties, see [this {{ TF }} article]({{ tf-provider-resources-link }}/iam_service_account).

  1. Make sure the configuration files are correct.

     1. In the command line, navigate to the directory where you created the configuration file.
     1. Run a check using this command:

         ```bash
         terraform plan
         ```

     If the configuration description is correct, the terminal will display information about the service account. If the configuration contains any errors, {{ TF }} will point them out.

  1. Deploy the cloud resources.

  1. If the configuration does not contain any errors, run this command:

     ```bash
     terraform apply
     ```

  1. Confirm creating the service account: type `yes` in the terminal and press **Enter**.

     This will create the service account. You can check the new service account using the [management console]({{ link-console-main }}) or this [CLI](../../cli/quickstart.md) command:

     ```bash
     yc iam service-account list
     ```

- API {#api}

  To create a service account, use the [create](../../iam/api-ref/ServiceAccount/create.md) REST API method for the [ServiceAccount](../../iam/api-ref/ServiceAccount/index.md) resource or the [ServiceAccountService/Create](../../iam/api-ref/grpc/ServiceAccount/create.md) gRPC API call.

  To assign the `logging.reader` role for a folder to a service account, use the [setAccessBindings](../../iam/api-ref/ServiceAccount/setAccessBindings.md) method for the [ServiceAccount](../../iam/api-ref/ServiceAccount/index.md) resource or the [ServiceAccountService/SetAccessBindings](../../iam/api-ref/grpc/ServiceAccount/setAccessBindings.md) gRPC API call.

{% endlist %}

## Create an authorized key for a service account {#create-key}

{% list tabs group=instructions %}

- Management console {#console}

  1. In the [management console]({{ link-console-main }}), select the folder the service account belongs to.
  1. In the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_iam }}**.
  1. In the left-hand panel, select ![FaceRobot](../../_assets/console-icons/face-robot.svg) **{{ ui-key.yacloud.iam.label_service-accounts }}**.
  1. In the list that opens, select the `grafana-plugin` service account.
  1. Click **{{ ui-key.yacloud.iam.folder.service-account.overview.button_create-key-popup }}** in the top panel.
  1. Select **{{ ui-key.yacloud.iam.folder.service-account.overview.button_create_key }}**.
  1. Select the encryption algorithm.
  1. Enter a description of the key so that you can easily find it in the management console.
  1. Click **{{ ui-key.yacloud.iam.folder.service-account.overview.popup-key_button_create }}**.
  1. In the window that opens, click **{{ ui-key.yacloud.iam.folder.service-account.overview.action_download-keys-file }}**.
  1. Click **{{ ui-key.yacloud.iam.folder.service-account.overview.popup-key_button_close }}**.

- CLI {#cli}

  Create authorized keys for the `grafana-plugin` service account:

  ```bash
  yc iam key create --service-account-name grafana-plugin -o authorized_key.json
  ```

  If successful, a private key (`privateKey`) and a public key ID (`id`) will be written to the `authorized_key.json` file.

  Key file example:

  ```json
  {
     "id": "lfkoe35hsk**********",
     "service_account_id": "ajepg0mjt0**********",
     "created_at": "2023-10-10T10:04:56Z",
     "key_algorithm": "RSA_2048",
     "public_key": "-----BEGIN PUBLIC KEY-----\n...\n-----END PUBLIC KEY-----\n",
     "private_key": "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n"
  }
  ```

- {{ TF }} {#tf}

  1. In the configuration file, describe the resources you want to create:

     * `service_account_id`: Service account [ID](../../iam/operations/sa/get-id.md). This is a required parameter.
     * `description`: Key description. This is an optional parameter.
     * `key_algorithm`: Key generation algorithm. This is an optional parameter. The default algorithm is `RSA_2048`. For more information about the acceptable parameter values, see the [API documentation](../../iam/api-ref/Key/index.md).

     Here is an example of the configuration file structure:

     ```hcl
     resource "yandex_iam_service_account_key" "sa-auth-key" {
       service_account_id = "<service_account_ID>"
       description        = "<key_description>"
       key_algorithm      = "<key_generation_algorithm>"
     }
     ```

     For more information about the resources you can create with {{ TF }}, see the [relevant provider documentation]({{ tf-provider-resources-link }}/iam_service_account_key).

  1. Make sure the configuration files are correct.

     1. In the command line, navigate to the directory where you created the configuration file.
     1. Run a check using this command:

        ```bash
        terraform plan
        ```

     If the configuration description is correct, the terminal will display a list of the resources being created and their settings. If the configuration contains any errors, {{ TF }} will point them out.

  1. Deploy the cloud resources.

     1. If the configuration does not contain any errors, run this command:

        ```bash
        terraform apply
        ```

     1. Confirm creating the resources: type `yes` in the terminal and press **Enter**.

        This will create all the resources you need in the specified folder. You can check the new resources and their settings using the [management console]({{ link-console-main }}) and this CLI command:

        ```bash
        yc iam key list --service-account-id <service_account_ID>
        ```

- API {#api}

  To create an access key, use the [create](../../iam/api-ref/Key/create.md) REST API method for the [Key](../../iam/api-ref/Key/index.md) resource or the [KeyService/Create](../../iam/api-ref/grpc/Key/create.md) gRPC API call.

  Example of a request using cURL for the `create` REST API method:

  ```bash
  curl --request POST \
    --header 'Content-Type: application/json' \
    --header "Authorization: Bearer <IAM_token>" \
    --data '{"serviceAccountId": "<service_account_ID>"}' \
    https://iam.{{ api-host }}/iam/v1/keys
  ```

  Where:

  * `<IAM_token>`: IAM token of the user with permissions to create keys for the specified service account.
  * `<service_account_id>`: ID of the service account for which the keys are being created.

  If successful, the server response will contain the private key (`privateKey`) and public key ID (`id`). Save this data. You will not be able to get the secret key again.

  Example of a server response:

  ```json
  {
      "key": {
          "createdAt": "2023-10-10T10:55:00+00:00",
          "description": "",
          "id": "lfkoe35hsk**********",
          "keyAlgorithm": "RSA_2048",
          "publicKey": "-----BEGIN PUBLIC KEY-----\n...\n-----END PUBLIC KEY-----\n",
          "serviceAccountId": "ajepg0mjt0**********"
      },
      "privateKey": "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n"
  }
  ```

{% endlist %}

## Create a log group {#create-group}

{% list tabs group=instructions %}

- Management console {#console}

  1. In the [management console]({{ link-console-main }}), go to the folder where you created the `grafana-plugin` service account.
  1. Select **{{ cloud-logging-name }}**.
  1. Click **{{ ui-key.yacloud.logging.button_create-group }}**.
  1. Enter `grafana-plugin` as the log group name.
  1. Set the log group record retention period.
  1. Click **{{ ui-key.yacloud.logging.button_create-group }}**.

- CLI {#cli}

  To create a log group, run this command:

  ```bash
  yc logging group create \
    --name=grafana-plugin \
    --retention-period=<retention_period> \
  ```

  Where:

  * `--name`: Log group name.
  * `--retention-period`: Retention period for log group records.

  Result:

  ```text
  done (1s)
  id: af3flf29t8**********
  folder_id: aoek6qrs8t**********
  cloud_id: aoegtvhtp8**********
  created_at: "2023-09-26T09:56:38.970Z"
  name: grafana-plugin
  status: ACTIVE
  retention_period: 3600s
  ```

- {{ TF }} {#tf}

  1. In the configuration file, describe the log group parameters:

     ```hcl
     provider "yandex" {
       token     = "<OAuth_token>"
       cloud_id  = "<cloud_ID>"
       folder_id = "<folder_ID>"
       zone      = "{{ region-id }}-a"
     }

     resource "yandex_logging_group" "grafana-plugin" {
       name             = "grafana-plugin"
       folder_id        = "<folder_ID>"
       retention_period = "1h"
     }
     ```

     Where:

     * `name`: Log group name.
     * `folder_id`: [Folder ID](../../resource-manager/operations/folder/get-id.md).
     * `retention_period`: Retention period for log group records.

     For more information about `yandex_logging_group` properties, see [this {{ TF }} article]({{ tf-provider-resources-link }}/logging_group).

  1. Make sure the configuration files are correct.

     1. In the command line, navigate to the directory where you created the configuration file.
     1. Run a check using this command:

        ```bash
        terraform plan
        ```

     If the configuration description is correct, the terminal will display a list of the resources being created and their settings. If the configuration contains any errors, {{ TF }} will point them out.

  1. Deploy the cloud resources.

  1. If the configuration does not contain any errors, run this command:

     ```bash
     terraform apply
     ```

  1. Confirm creating the resources: type `yes` in the terminal and press **Enter**.

     This will create all the resources you need in the specified folder. You can check the new resources and their settings using the [management console]({{ link-console-main }}) or this [CLI](../../cli/quickstart.md) command:

     ```bash
     yc logging group list
     ```

- API {#api}

  To create a log group, use the [create](../../logging/api-ref/LogGroup/create.md) REST API method for the [LogGroup](../../logging/api-ref/LogGroup/index.md) resource or the [LogGroupService/Create](../../logging/api-ref/grpc/LogGroup/create.md) gRPC API call.

{% endlist %}

## Add records to the log group {#add-records}

{% list tabs group=instructions %}

- CLI {#cli}

  To add records to a log group, run this command:

  * Linux, macOS:

    ```bash
    yc logging write \
      --group-name=grafana-plugin \
      --message="My message" \
      --level=INFO
    ```

  * Windows (cmd):

    ```bash
    yc logging write ^
      --group-name=grafana-plugin ^
      --message="My message" ^
      --level=INFO
    ```

  * Windows (PowerShell):

    ```bash
    yc logging write `
      --group-name=grafana-plugin `
      --message="My message" `
      --level=INFO
    ```

    Where:

    * `--group-name`: Name of the log group to add records to.
    * `--message`: Message.
    * `--level`: Logging level.

   {% note info %}

   You can skip the `--group-name` and `--message` parameters and provide only the values, e.g., `grafana-plugin "My message"`.

   {% endnote %}

- API {#api}

  To add records to the log group, use the [LogIngestionService/Write](../../logging/api-ref/grpc/LogIngestion/write.md) gRPC API call.

{% endlist %}

## Connect a data source in {{ grafana-name }} {#connect-plugin}

1. In the browser, go to `http://localhost:3000/`.

   {% note info %}

   By default, {{ grafana-name }} uses port 3000, unless you [specified a different one](https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/#http_port) in the configuration file.

   {% endnote %}

1. In the left-hand panel, select **Connections** → **Add new connection**.
1. In the list of sources, select **{{ cloud-logging-full-name }}**.
1. Click **Add new data source**.
1. Under **Secret config**, in the **API Key** field, paste the contents of the `authorized_key.json` file with the [authorized keys](#create-key).
1. Under **SDK config**, in the **Folder ID** field, specify the ID of the folder with the `grafana-plugin` log group.
1. Click **Save & test**.

## View the logs in {{ grafana-name }} {#see-logs}

1. In the {{ grafana-name }} interface, select **Explore** in the left-hand panel.
1. In the top-left corner, select the **{{ cloud-logging-full-name }}** data source from the drop-down list.
1. In the query editor for the data source:

   1. Select the ID of the `grafana-plugin` log group in the **Group** field.
   1. Enter your query written in the [filter expression language](../../logging/concepts/filter.md) in the **Filter query** field.
   1. In the top-right corner, click **Run query**.

      You will see a histogram with log group records in the **Logs volume** section.

## How to delete the resources you created {#clear-out}

To stop paying for the resources you created, [delete the log group](../../logging/operations/delete-group.md).
