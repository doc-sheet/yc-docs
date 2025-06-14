{% list tabs group=instructions %}

- CLI {#cli}

    {% include [cli-install](../cli-install.md) %}

    {% include [default-catalogue](../default-catalogue.md) %}

    To add records to a log group, run this command:

    * Linux, MacOS:
        ```bash
        yc logging write \
          --group-name=default \
          --message="My message" \
          --level=INFO \
          --json-payload='{"request_id": "1234"}'
        ```
    * Windows (cmd):
        ```cmd
        yc logging write ^
          --group-name=default ^
          --message="My message" ^
          --level=INFO ^
          --json-payload="{"request_id": "1234"}"
        ```
    * Windows (PowerShell):
        ```powershell
        yc logging write `
          --group-name=default `
          --message="My message" `
          --level=INFO `
          --json-payload='"{ \"request_id\": \"1234\" }"'
        ```
        
    Where:

    * `--group-name`: Name of the log group where you want to add records. If you leave it unspecified, the system will add records to the [default log group](../../logging/concepts/log-group.md) in the current folder. You can specify `--group-id` instead of `--group-name`. 
    * `--message`: Message.
    * `--level`: Logging level.
    * `--json-payload`: Additional information in JSON format.

   {% note info %}

   You can skip the `--group-name`, `--message`, and `--json-payload` parameters and provide only the values in the right order, e.g., `default "My message" '{"request_id": "1234"}'`.

   {% endnote %}

- API {#api}

  To add records to a log group, use the [LogIngestionService/Write](../../logging/api-ref/grpc/LogIngestion/write.md) gRPC API call.

{% endlist %}