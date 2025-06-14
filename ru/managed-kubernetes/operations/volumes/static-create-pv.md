---
title: Как создать под со статически подготовленным томом в {{ managed-k8s-full-name }}
description: Следуя данной инструкции, вы сможете создать под со статически подготовленным томом.
---

# Статическая подготовка тома


Создайте [под](../../concepts/index.md#pod) со статически подготовленным [томом](../../concepts/volume.md):
1. [Создайте объект PersistentVolume](#create-pv).
1. [Создайте объект PersistentVolumeClaim](#create-claim).
1. [Создайте под](#create-pod).

{% note tip %}

Вы можете использовать [бакет](../../../storage/concepts/bucket.md) {{ objstorage-full-name }} в качестве хранилища для пода. Подробнее см. в разделе [{#T}](s3-csi-integration.md).

{% endnote %}

## Перед началом работы {#before-you-begin}

1. {% include [Install and configure kubectl](../../../_includes/managed-kubernetes/kubectl-install.md) %}

1. Узнайте уникальный идентификатор [диска](../../../compute/concepts/disk.md), который будет использован для создания объекта `PersistentVolume`:
   1. Если у вас еще нет диска, [создайте его](../../../compute/operations/disk-create/empty.md).

      {% note warning %}

      Диск должен быть размещен в той же [зоне доступности](../../../overview/concepts/geo-scope.md), что и [узлы группы](../../concepts/index.md#node-group), на которой будут работать поды.

      {% endnote %}

   1. Получите идентификатор диска (колонка `ID`):

      ```bash
      yc compute disk list
      ```

      Результат:

      ```text
      +----------------------+------+------------+-------------------+--------+--------------+-------------+
      |          ID          | NAME |    SIZE    |       ZONE        | STATUS | INSTANCE IDS | DESCRIPTION |
      +----------------------+------+------------+-------------------+--------+--------------+-------------+
      | ef3ouo4sgl86******** | k8s  | 4294967296 | {{ region-id }}-a     | READY  |              |             |
      +----------------------+------+------------+-------------------+--------+--------------+-------------+
      ```

1. Посмотрите доступные [классы хранилищ](manage-storage-class.md) и выберите подходящий:

   ```bash
   kubectl get storageclass
   ```

   Результат:

   ```text
   NAME                          PROVISIONER                    RECLAIMPOLICY  VOLUMEBINDINGMODE     ALLOWVOLUMEEXPANSION  AGE
   yc-network-hdd (default)      disk-csi-driver.mks.ycloud.io  Delete         WaitForFirstConsumer  true                  12d
   yc-network-nvme               disk-csi-driver.mks.ycloud.io  Delete         WaitForFirstConsumer  true                  12d
   yc-network-ssd                disk-csi-driver.mks.ycloud.io  Delete         WaitForFirstConsumer  true                  12d
   yc-network-ssd-nonreplicated  disk-csi-driver.mks.ycloud.io  Delete         WaitForFirstConsumer  true                  12d
   ```

   {% note info %}

   Не путайте [классы хранилищ {{ k8s }}](manage-storage-class.md) и [типы дисков {{ compute-full-name }}](../../../compute/concepts/disk.md#disks_types).

   {% endnote %}

## Создайте объект PersistentVolume {#create-pv}

1. Сохраните спецификацию для создания объекта `PersistentVolume` в YAML-файл с названием `test-pv.yaml`.

   Подробнее о спецификации читайте в [документации {{ k8s }}](https://kubernetes.io/docs/reference/kubernetes-api/config-and-storage-resources/persistent-volume-v1/).

   При указании параметра `spec.capacity.storage` убедитесь, что задан точный объем диска. {{ CSI }} не обеспечивает проверку объема диска для статически подготовленных томов.

   Для создания объекта `PersistentVolume` на основе существующего облачного диска в параметре `volumeHandle` укажите уникальный идентификатор необходимого диска.

   {% note info %}

   Если не указать параметр `storageClassName`, будет использован класс хранилищ по умолчанию: `yc-network-hdd`. Как изменить класс по умолчанию читайте в разделе [{#T}](manage-storage-class.md#sc-default).

   {% endnote %}

   Подробнее о спецификации для создания объекта `PersistentVolumeClaim` читайте в [документации {{ k8s }}](https://kubernetes.io/docs/reference/kubernetes-api/config-and-storage-resources/persistent-volume-claim-v1/).

   ```yaml
   apiVersion: v1
   kind: PersistentVolume
   metadata:
     name: <имя_PersistentVolume>
   spec:
     capacity:
       storage: <размер_PersistentVolume>
     accessModes:
       - ReadWriteOnce
     csi:
       driver: disk-csi-driver.mks.ycloud.io
       fsType: ext4
       volumeHandle: <идентификатор_диска>
     storageClassName: <имя_класса_хранилища>
   ```

1. Выполните команду:

   ```bash
   kubectl create -f test-pv.yaml
   ```

   Результат:

   ```text
   persistentvolume/<имя_PersistentVolume> created
   ```

1. Посмотрите информацию о созданном объекте `PersistentVolume`:

   ```bash
   kubectl describe persistentvolume <имя_PersistentVolume>
   ```

   Результат:

   ```text
   Name:            <имя_PersistentVolume>
   Labels:          <none>
   Annotations:     <none>
   Finalizers:      [kubernetes.io/pv-protection]
   StorageClass:    <имя_класса_хранилища>
   Status:          Available
   ...
   ```

## Создайте объект PersistentVolumeClaim {#create-claim}

1. Сохраните спецификацию для создания объекта `PersistentVolumeClaim` YAML-файл с названием `test-claim.yaml`.

   Подробнее о спецификации читайте в [документации {{ k8s }}](https://kubernetes.io/docs/reference/kubernetes-api/config-and-storage-resources/persistent-volume-claim-v1/).

   ```yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: <имя_PersistentVolumeClaim>
   spec:
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: <размер_PersistentVolumeClaim>
     storageClassName: <имя_класса_хранилища>
     volumeName: <имя_PersistentVolume>
   ```

   {% note info %}

   Размер `PersistentVolumeClaim` должен быть меньше или равен размеру `PersistentVolume`.

   {% endnote %}

1. Выполните команду:

   ```bash
   kubectl create -f test-claim.yaml
   ```

   Результат:

   ```text
   persistentvolumeclaim/<имя_PersistentVolumeClaim> created
   ```

1. Посмотрите информацию о созданном `PersistentVolumeClaim`:

   ```bash
   kubectl describe persistentvolumeclaim <имя_PersistentVolumeClaim>
   ```

   Результат:

   ```text
   Name:          <имя_PersistentVolumeClaim>
   Namespace:     default
   StorageClass:  <имя_класса_хранилища>
   Status:        Bound
   Volume:        <имя_PersistentVolume>
   ...
   ```

## Создайте под со статически подготовленным томом {#create-pod}

1. Создайте файл `test-pod.yaml` с манифестом пода, использующего `PersistentVolumeClaim`:

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: test-pod
   spec:
     containers:
     - name: app
       image: ubuntu
       command: ["/bin/sh"]
       args: ["-c", "while true; do echo $(date -u) >> /data/out.txt; sleep 5; done"]
       volumeMounts:
       - name: persistent-storage
         mountPath: /data
     volumes:
     - name: persistent-storage
       persistentVolumeClaim:
         claimName: <имя_PersistentVolumeClaim>
   ```

   Подробнее о спецификации читайте в [документации {{ k8s }}](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/).
1. Выполните команду:

   ```bash
   kubectl create -f test-pod.yaml
   ```

   Результат:

   ```text
   pod/test-pod created
   ```

1. Посмотрите информацию о созданном поде:

   ```bash
   kubectl describe pod test-pod
   ```

   Результат:

   ```text
   Name:       test-pod
   Namespace:  default
   Priority:   0
   ...
     ----    ------                  ----  ----                     -------
     Normal  Scheduled               20m   default-scheduler        Successfully assigned default/test-pod to cl1jtehftl7q********-icut
     Normal  SuccessfulAttachVolume  20m   attachdetach-controller  AttachVolume.Attach succeeded for volume "<имя_PersistentVolume>"
   ```

После этого рядом с используемым диском в консоли управления в **{{ ui-key.yacloud.iam.folder.dashboard.label_compute }}** в разделе **{{ ui-key.yacloud.compute.disks_ddfdb }}** появится надпись **{{ ui-key.yacloud.compute.disks.label_disk-used }}**.

## Как удалить том {#delete-volume}

Диски в {{ compute-name }} не удаляются автоматически при удалении `PersistentVolume`. Чтобы полностью удалить том:
1. Удалите объект `PersistentVolumeClaim`:

   ```bash
   kubectl delete pvc <идентификатор_объекта_PersistentVolumeClaim>
   ```

1. Удалите объект `PersistentVolume`:

   ```bash
   kubectl delete pv <идентификатор_объекта_PersistentVolume>
   ```

1. [Удалите диск](../../../compute/operations/disk-control/delete.md) в {{ compute-name }}, связанный с объектом `PersistentVolume`.

### См. также {#see-also}

* [{#T}](../../concepts/volume.md)
* [{#T}](./encrypted-disks.md)
* [{#T}](./dynamic-create-pv.md)
* [{#T}](./manage-storage-class.md)