# Триггер для {{ cloud-logging-name }}, который вызывает контейнер {{ serverless-containers-name }}

[Триггер](../trigger/) для {{ cloud-logging-name }} запускает [контейнер](../container.md) {{ serverless-containers-name }}, когда в [лог-группу](../../../logging/concepts/log-group.md) добавляют записи.

Триггеру для {{ cloud-logging-name }} необходим [сервисный аккаунт](../../../iam/concepts/users/service-accounts.md) для чтения из лог-группы и вызова контейнера.

О том, как создать триггер для {{ cloud-logging-name }}, читайте в инструкции [{#T}](../../operations/cloud-logging-trigger-create.md).

{% include [batching-messages](../../../_includes/serverless-containers/batching-messages.md) %}

## Роли, необходимые для корректной работы триггера для {{ cloud-logging-name }} {#roles}

* Для создания триггера вам необходимо разрешение на сервисный аккаунт, от имени которого триггер выполняет операцию. Это разрешение входит в роли [iam.serviceAccounts.user](../../../iam/concepts/access-control/roles#sa-user), [editor](../../../iam/concepts/access-control/roles#editor) и выше.
* Для работы триггера сервисному аккаунту необходимы роли:
    * `{{ roles-serverless-containers-invoker }}` на контейнер, который вызывает триггер.
    * `{{ roles-logging-reader }}` на лог-группу, при добавлении записей в которую вызывается триггер.

## Формат сообщения от триггера для {{ cloud-logging-name }} {#format}

После того как триггер сработает, он отправит в контейнер следующее сообщение:

{% include [logging-format](../../../_includes/functions/logging-format.md) %}

## См. также {#see-also}

* [{#T}](../../../functions/concepts/trigger/cloud-logging-trigger.md)
* [{#T}](../../../api-gateway/concepts/trigger/cloud-logging-trigger.md)
