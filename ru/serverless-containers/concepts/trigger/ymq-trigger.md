# Триггер для {{ message-queue-short-name }}, который передает сообщения в контейнер {{ serverless-containers-name }}

[Триггер](../trigger/) для {{ message-queue-short-name }} предназначен для разгрузки [очереди сообщений](../../../message-queue/concepts/queue.md). Он принимает сообщения из очереди и передает их в [контейнер](../container.md) {{ serverless-containers-name }} для обработки. После успешной обработки триггер удаляет сообщения из очереди, а при ошибке — возвращает сообщения в очередь через [таймаут видимости](../../../message-queue/concepts/visibility-timeout.md). Если для очереди не настроена [Dead Letter Queue](../../../message-queue/concepts/dlq.md), сообщение будет повторно передаваться в контейнер, пока успешно не обработается или не закончится срок его хранения.

{% include [ymq-trigger-note.md](../../../_includes/functions/ymq-trigger-note.md) %}

Запросы к очереди сообщений при работе триггера не тарифицируются.

Триггеру для {{ message-queue-short-name }} необходимы [сервисные аккаунты](../../../iam/concepts/users/service-accounts.md) для чтения из очереди сообщений и вызова контейнера. Вы можете использовать один и тот же сервисный аккаунт для обеих операций. 

О том, как создать триггер для {{ message-queue-short-name }}, читайте в инструкции [{#T}](../../operations/ymq-trigger-create.md).

{% include [batching-messages](../../../_includes/serverless-containers/batching-messages.md) %}

## Роли, необходимые для корректной работы триггера для {{ message-queue-short-name }} {#roles}

* Для создания триггера вам необходимы: 
    * Роль `{{ roles-viewer }}` на каталог с очередью сообщений, из которой триггер читает сообщения.
    * Роль `{{ roles-viewer }}` на каталог с контейнером, которые вызывает триггер.
    * Разрешение на сервисный аккаунт, от имени которого триггер выполняет операции. Это разрешение входит в роли [iam.serviceAccounts.user](../../../iam/security/index.md#iam-serviceAccounts-user), [editor](../../../iam/roles-reference.md#editor) и выше.
* Для работы триггера сервисным аккаунтам необходимы роли: 
    * `{{ roles-editor }}` на каталог с очередью сообщений, из которой триггер читает сообщения.
    * `serverless.containers.invoker` на каталог с контейнером, который вызывает триггер.

Подробнее об [управлении доступом](../../security/index.md).

## Формат сообщения от триггера {{ message-queue-short-name }} {#format}

После того как триггер примет сообщение из очереди, он передаст его в контейнер в следующем формате: 

{% include [ymq-format](../../../_includes/functions/ymq-format.md) %}


## См. также {#see-also}

* [{#T}](../../../functions/concepts/trigger/ymq-trigger.md)
* [{#T}](../../../api-gateway/concepts/trigger/ymq-trigger.md)
