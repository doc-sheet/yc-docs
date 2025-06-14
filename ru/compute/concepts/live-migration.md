# Динамическая миграция

В {{ compute-name }} используется механизм динамической миграции, который позволяет перемещать запущенные [виртуальные машины](vm.md) между физическими серверами без остановки и перезагрузки. Пауза при этом обычно не превышает 10 секунд.

Динамическая миграция не изменяет никаких параметров ВМ. Процесс миграции в реальном времени переносит работающую ВМ с одного сервера на другой в той же [зоне доступности](../../overview/concepts/geo-scope.md). Все свойства ВМ, в том числе [внутренние и внешние IP-адреса](../../vpc/concepts/address.md), [метаданные](vm-metadata.md), память, состояние [сетей](../../vpc/concepts/network.md#network), операционных систем и приложений остаются неизменными.

{{ compute-name }} запускает динамическую миграцию в следующих случаях:
* Плановое техническое обслуживание и обновление оборудования.
* Внеплановое техническое обслуживание при возникновении неисправностей оборудования, в том числе CPU, сетевых карт, источников питания и т. д.
* Обновление ОС, BIOS и встроенного программного обеспечения.
* Изменение конфигурации ОС.
* Обновления безопасности.

## Ограничения {#limitations}

Динамическая миграция не поддерживается для:

* ВМ с [GPU](../concepts/gpus.md) ^1^.
* ВМ с [политикой обслуживания](../concepts/vm-policies.md) `RESTART` ^1^.
* [Прерываемых](../concepts/preemptible-vm.md) ВМ ^2^.
* ВМ управляемых СУБД с локальными SSD-дисками.
* ВМ на выделенных хостах с [локальными SSD-дисками](../concepts/dedicated-host.md#resource-disks).
* ВМ на выделенных хостах с [привязкой к конкретному хосту](../concepts/dedicated-host.md#bind-vm).

^1^ При обслуживании останавливаются и запускаются на новом сервере.
^2^ При обслуживании останавливаются.


## Смотрите также {#see-also}

* [Политики обслуживания виртуальных машин](./vm-policies.md).
* [Как перенести виртуальную машину в другую зону доступности](../operations/vm-control/vm-change-zone.md).
* [Как перенести виртуальную машину в другой каталог](../operations/vm-control/vm-change-folder.md).
* [Как перенести виртуальную машину в другое облако](../operations/vm-control/vm-change-cloud.md).
