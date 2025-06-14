---
title: Триггеры в {{ api-gw-full-name }}. Обзор
description: Триггер — условие, при выполнении которого автоматически рассылается сообщение в соединения по протоколу WebSocket. Триггеры позволяют автоматизировать работу с другими сервисами {{ yandex-cloud }}, например Yandex Object Storage, Yandex Message Queue и Yandex IoT Core.
---

# Триггеры в {{ api-gw-full-name }}. Обзор

_Триггер_ — условие, при выполнении которого сообщения автоматически отправляются в [WebSocket-соединения](../extensions/websocket.md), подключенные к API-шлюзу по пути, указанному пользователем. Сам API-шлюз при этом не вызывается.

Триггеры позволяют автоматизировать работу с другими сервисами {{ yandex-cloud }}, например: {{ objstorage-full-name }}, {{ message-queue-full-name }} и {{ container-registry-full-name }}.

{% include [trigger-time](../../../_includes/functions/trigger-time.md) %}

В {{ api-gw-full-name }} доступны следующие типы триггеров: 
* [таймер](timer.md);
* [триггер для {{ message-queue-name }}](ymq-trigger.md);
* [триггер для {{ objstorage-name }}](os-trigger.md);
* [триггер для {{ container-registry-name }}](cr-trigger.md);
* [триггер для {{ cloud-logging-name }}](cloud-logging-trigger.md);
* [триггер для {{ iot-name }}](iot-core-trigger.md);
* [триггер для бюджетов](budget-trigger.md);
* [триггер для {{ yds-name }}](data-streams-trigger.md);
* [триггер для почты](mail-trigger.md).

{% include [trigger-intro-note](../../../_includes/functions/trigger-intro-note.md) %}


## Особенности отправки сообщений триггером {#invoke}

Триггеры отправляют сообщения с учетом установленных [квот и лимитов](../limits.md).

При отправке сообщений триггером существуют следующие особенности:
* Триггер изменяет формат сообщений перед их отправкой в WebSocket-соединения. У каждого типа триггера свой формат сообщений, подробнее об этом читайте в разделе с описанием соответствующего триггера.
* Если сообщение не удалось отправить или никто не подключен по пути, который указан в настройках триггера, сообщение теряется без возможности повторить отправку.
* Сервисному аккаунту, от имени которого сообщения будут отправляться в WebSocket-соединения, необходима роль `api-gateway.websocketBroadcaster`. Другие роли, необходимые для корректной работы триггера, зависят от его типа. Подробнее см. описание соответствующего триггера.


## Группирование сообщений {#batch-messages}

Вы можете настроить группирование сообщений перед их отправкой в WebSocket-соединения у следующих триггеров:

* триггер для {{ message-queue-name }};
* триггер для {{ cloud-logging-name }};
* триггер для {{ objstorage-name }};
* триггер для {{ container-registry-name }};
* триггер для {{ iot-name }};
* триггер для {{ yds-name }};
* триггер для почты.

Это позволит отправлять сообщения не по одному, а группой. Для этого в параметрах триггера задайте:

* **{{ ui-key.yacloud.serverless-functions.triggers.form.field_cutoff }}** — время, в течение которого происходит группирование сообщений. Число сообщений при этом не превышает указанный размер группы.
* **{{ ui-key.yacloud.serverless-functions.triggers.form.field_size }}** — количество сообщений для группирования.

Подробнее о группировании сообщений читайте в инструкциях по созданию соответствующего триггера.


## См. также {#see-also}

* [Триггеры для запуска контейнера {{ serverless-containers-name }}](../../../serverless-containers/concepts/trigger/index.md)
* [Триггеры для запуска функции {{ sf-name }}](../../../functions/concepts/trigger/index.md)
