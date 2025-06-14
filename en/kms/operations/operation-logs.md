# Viewing operations with {{ kms-name }} resources

The system logs all actions with {{ kms-name }} resources as a list of operations. Each operation gets its own unique ID.

## Getting a list of operations {#get-operations}

{% list tabs group=instructions %}

- Management console {#console}

  Below, you can learn how to get a list of operations for a symmetric key. The same steps apply to other resources as well.

  1. In the [management console]({{ link-console-main }}), open the folder containing the symmetric key.
  1. Select **{{ ui-key.yacloud.iam.folder.dashboard.label_kms }}**.
  1. In the left-hand panel, select ![image](../../_assets/console-icons/key.svg) **{{ ui-key.yacloud.kms.switch_symmetric-keys }}**.
  1. Select the key you need.
  1. Go to the ![image](../../_assets/console-icons/list-check.svg) **Operations** panel for the selected key.

     You will see a list of operations with the selected symmetric key.

- CLI {#cli}

  {% include [cli-install](../../_includes/cli-install.md) %}

  {% include [default-catalogue](../../_includes/default-catalogue.md) %}

  To get the list of operations for the {{ kms-name }} resource, run this command:

  ```bash
  yc kms <resource_type> list-operations <resource_name_or_ID>
  ```

  **Example**

  Getting a list of operations for the symmetric key:

  ```bash
  yc kms symmetric-key list-operations epdplu8jn7sr********
  ```

  Result:

  ```text
  +----------------------+---------------------+----------------------+---------------------+--------+----------------------+
  |          ID          |     CREATED AT      |      CREATED BY      |     MODIFIED AT     | STATUS |     DESCRIPTION      |
  +----------------------+---------------------+----------------------+---------------------+--------+----------------------+
  | abji19ucm6b1******** | 2024-03-26 10:33:12 | aje9k8luj4qf******** | 2024-03-26 10:33:12 | DONE   | update key           |
  | abjluv2sk1sp******** | 2024-03-26 10:19:52 | aje9k8luj4qf******** | 2024-03-26 10:19:52 | DONE   | create symmetric key |
  +----------------------+---------------------+----------------------+---------------------+--------+----------------------+
  ```

  By default, operation information is displayed as text. To get more detailed information, specify the `yaml` or `json` output data format using the `--format` parameter:

  ```bash
  yc kms symmetric-key list-operations epdplu8jn7sr******** --format yaml
  ```

  Result:

  ```text
  - id: abji19ucm6b1********
    description: update key
    created_at: "2024-03-26T10:33:12.360289758Z"
    created_by: aje9k8luj4qf********
    modified_at: "2024-03-26T10:33:12.360314054Z"
    done: true
    metadata:
      '@type': type.googleapis.com/yandex.cloud.kms.v1.UpdateSymmetricKeyMetadata
      key_id: abjip2tl1frh********
    response:
      '@type': type.googleapis.com/google.protobuf.Empty
      value: {}
  - id: abjluv2sk1sp********
    description: create symmetric key
    created_at: "2024-03-26T10:19:52.794225911Z"
    created_by: aje9k8luj4qf********
    modified_at: "2024-03-26T10:19:52.794265993Z"
    done: true
    metadata:
      '@type': type.googleapis.com/yandex.cloud.kms.v1.CreateSymmetricKeyMetadata
      key_id: abjip2tl1frh********
      primary_version_id: abjg82vbog68********
    response:
      '@type': type.googleapis.com/google.protobuf.Empty
      value: {}
  ```

- API {#api}

  Use the `listOperations` REST API method for the relevant resource or the `<service>/ListOperations` gRPC API call.

  For example, to obtain a list of operations for a symmetric key, use either the REST API [listOperations](../api-ref/SymmetricKey/listOperations.md) for the [SymmetricKey](../api-ref/SymmetricKey/index.md) resource or the [SymmetricKeyService/ListOperations](../api-ref/grpc/SymmetricKey/listOperations.md) gRPC API call.

{% endlist %}

## Getting operation details {#get-operations-info}

1. [Get a list of operations](#get-operations) for the resource.
1. Copy the operation ID.
1. Get operation details:

    {% list tabs group=instructions %}

    - CLI {#cli}

      {% include [cli-install](../../_includes/cli-install.md) %}

      {% include [default-catalogue](../../_includes/default-catalogue.md) %}

      Run this command:

      ```bash
      yc operation get <operation_ID>
      ```

      Result:

      ```text
      id: abji19ucm6b1********
      description: update key
      created_at: "2024-03-26T10:33:12.360Z"
      created_by: aje9k8luj4qf********
      modified_at: "2024-03-26T10:33:12.360Z"
      done: true
      metadata:
        '@type': type.googleapis.com/yandex.cloud.kms.v1.UpdateSymmetricKeyMetadata
        key_id: abjip2tl1frh********
      response:
        '@type': type.googleapis.com/yandex.cloud.kms.v1.SymmetricKey
        id: abjip2tl1frh********
        folder_id: b1g681qpemb4********
        created_at: "2024-03-26T10:19:52Z"
        name: sym-key
        status: ACTIVE
        primary_version:
          id: abjg82vbog68kv5jvj6s
          key_id: abjip2tl1frh********
          status: ACTIVE
          algorithm: AES_256
          created_at: "2024-03-26T10:19:52Z"
          primary: true
        default_algorithm: AES_192
      ```

   - API {#api}

     Use the [OperationService/Get](../api-ref/grpc/Operation/get.md) gRPC API call.

   {% endlist %}

#### See also {#see-also}

* [{#T}](../../api-design-guide/concepts/about-async.md)
