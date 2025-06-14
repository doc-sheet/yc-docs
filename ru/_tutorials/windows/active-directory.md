# Развертывание Active Directory


{% include [ms-disclaimer](../../_includes/ms-disclaimer.md) %}



В сценарии приводится пример развертывания Active Directory в {{ yandex-cloud }}.

Чтобы развернуть инфраструктуру Active Directory:
1. [Подготовьте облако к работе](#before-you-begin).
1. [Создайте облачную сеть и подсети](#create-network).
1. [Создайте скрипт для управления локальной учетной записью администратора](#admin-script).
1. [Создайте виртуальную машину для Active Directory](#ad-vm).
1. [Создайте ВМ для бастионного хоста](#jump-server-vm).
1. [Установите и настройте Active Directory](#install-ad).
1. [Настройте второй контроллер домена](#install-ad-2).
1. [Проверьте работу Active Directory](#test-ad).

Если инфраструктура вам больше не нужна, [удалите все используемые ею ресурсы](#clear-out).

## Подготовьте облако к работе {#before-you-begin}

{% include [before-you-begin](../_tutorials_includes/before-you-begin.md) %}

{% include [ms-additional-data-note](../_tutorials_includes/ms-additional-data-note.md) %}

### Необходимые платные ресурсы {#paid-resources}

В стоимость инсталляции Active Directory входят:
* Плата за постоянно запущенные [ВМ](../../compute/concepts/vm.md) (см. [тарифы {{ compute-full-name }}](../../compute/pricing.md)).
* Плата за использование динамических или статических [публичных IP-адресов](../../vpc/concepts/address.md#public-addresses) (см. [тарифы {{ vpc-full-name }}](../../vpc/pricing.md)).
* Стоимость исходящего трафика из {{ yandex-cloud }} в интернет (см. [тарифы {{ compute-name }}](../../compute/pricing.md)).

## Создайте облачную сеть и подсети {#create-network}

Создайте [облачную сеть](../../vpc/concepts/network.md#network) `ad-network` с [подсетями](../../vpc/concepts/network.md#subnet) во всех [зонах доступности](../../overview/concepts/geo-scope.md), где будут находиться ВМ.
1. Создайте облачную сеть:

   {% list tabs group=instructions %}

   - Консоль управления {#console}

     Чтобы создать облачную сеть:
     1. Откройте раздел **{{ vpc-name }}** в [каталоге](../../resource-manager/concepts/resources-hierarchy.md#folder), где требуется создать облачную сеть.
     1. Нажмите кнопку **Создать сеть.**
     1. Задайте имя сети: `ad-network`.
     1. Нажмите кнопку **Создать сеть**.

   - CLI {#cli}

     {% include [cli-install](../../_includes/cli-install.md) %}

     {% include [default-catalogue](../../_includes/default-catalogue.md) %}

     Чтобы создать облачную сеть, выполните команду:

     ```bash
     yc vpc network create --name ad-network
     ```

   {% endlist %}

1. Создайте три подсети в сети `ad-network`:

   {% list tabs group=instructions %}

   - Консоль управления {#console}

       Чтобы создать подсеть:
       1. Откройте раздел **{{ vpc-name }}** в каталоге, где требуется создать подсеть.
       1. Нажмите на имя облачной сети.
       1. Нажмите кнопку **Добавить подсеть**.
       1. Заполните форму: введите имя подсети `ad-subnet-a`, выберите зону доступности `{{ region-id }}-a` из выпадающего списка.
       1. Введите CIDR подсети: IP-адрес и маску подсети `10.1.0.0/16`.
       1. Нажмите кнопку **Создать подсеть**.

       Повторите шаги еще для двух подсетей:
       * Название: `ad-subnet-b`. Зона доступности: `{{ region-id }}-b`. CIDR: `10.2.0.0/16`.
       * Название: `ad-subnet-d`. Зона доступности: `{{ region-id }}-d`. CIDR: `10.3.0.0/16`.

   - CLI {#cli}

       Чтобы создать подсети, выполните команды:

       ```bash
       yc vpc subnet create \
         --name ad-subnet-a \
         --zone {{ region-id }}-a \
         --network-name ad-network \
         --range 10.1.0.0/16

       yc vpc subnet create \
         --name ad-subnet-b \
         --zone {{ region-id }}-b \
         --network-name ad-network \
         --range 10.2.0.0/16

       yc vpc subnet create \
         --name ad-subnet-d \
         --zone {{ region-id }}-d \
         --network-name ad-network \
         --range 10.3.0.0/16
       ```

   {% endlist %}

## Создайте скрипт для управления локальной учетной записью администратора {#admin-script}

При создании ВМ через CLI необходимо устанавливать пароль для локальной учетной записи администратора.

Для этого в корневой директории командной строки создайте файл с названием `setpass` и без расширения. Скопируйте в файл скрипт и укажите ваш пароль:

```powershell
#ps1
Get-LocalUser | Where-Object SID -like *-500 | Set-LocalUser -Password (ConvertTo-SecureString "<ваш пароль>" -AsPlainText -Force)
```

Пароль должен соответствовать [требованиям к сложности]({{ ms.docs }}/windows/security/threat-protection/security-policy-settings/password-must-meet-complexity-requirements#справочник).

Подробные рекомендации по защите Active Directory читайте на [сайте разработчика]({{ ms.docs }}/windows-server/identity/ad-ds/plan/security-best-practices/best-practices-for-securing-active-directory).

## Создайте ВМ для Active Directory {#ad-vm}

Создайте две ВМ для контроллеров домена Active Directory. Эти ВМ не будут иметь доступа в интернет.

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. На странице каталога в [консоли управления]({{ link-console-main }}) нажмите кнопку **{{ ui-key.yacloud.iam.folder.dashboard.button_add }}** и выберите `{{ ui-key.yacloud.iam.folder.dashboard.value_compute }}`.

  1. В блоке **{{ ui-key.yacloud.compute.instances.create.section_image }}**:

      * Перейдите на вкладку **{{ ui-key.yacloud.compute.instances.create.image_value_custom_new }}**.
      * Нажмите кнопку **{{ ui-key.yacloud.common.select }}** и в открывшемся окне выберите **{{ ui-key.yacloud.common.create }}**.
      * В поле **{{ ui-key.yacloud.compute.instances.create-disk.field_source }}** выберите `{{ ui-key.yacloud.compute.instances.create-disk.value_source-image }}` и в списке ниже выберите образ **Windows Server 2022 Datacenter**. Как загрузить свой образ для продуктов Microsoft подробнее см. в разделе [Импортировать нужный образ](../../microsoft/byol.md#how-to-import).
      * (Опционально) В поле **{{ ui-key.yacloud.compute.field_additional_vt356 }}** включите опцию **{{ ui-key.yacloud.compute.field_disk-autodelete_qZn4x }}**, если вы хотите автоматически удалять этот диск при удалении ВМ.
      * Нажмите кнопку **{{ ui-key.yacloud.compute.component.instance-storage-dialog.button_add-disk }}**.

  1. В блоке **{{ ui-key.yacloud.k8s.node-groups.create.section_allocation-policy }}** выберите зону доступности `{{ region-id }}-a`.
  1. В блоке **{{ ui-key.yacloud.compute.instances.create.section_storages }}** задайте размер загрузочного [диска](../../compute/concepts/disk.md) `50 {{ ui-key.yacloud.common.units.label_gigabyte }}`.
  1. В блоке **{{ ui-key.yacloud.compute.instances.create.section_platform }}** перейдите на вкладку `{{ ui-key.yacloud.component.compute.resources.label_tab-custom }}` и укажите необходимую [платформу](../../compute/concepts/vm-platforms.md), количество vCPU и объем RAM:

      * **{{ ui-key.yacloud.component.compute.resources.field_platform }}** — `Intel Ice Lake`.
      * **{{ ui-key.yacloud.component.compute.resources.field_cores }}** — `4`.
      * **{{ ui-key.yacloud.component.compute.resources.field_core-fraction }}** — `100%`.
      * **{{ ui-key.yacloud.component.compute.resources.field_memory }}** — `8 {{ ui-key.yacloud.common.units.label_gigabyte }}`.
  1. В блоке **{{ ui-key.yacloud.compute.instances.create.section_network }}** укажите:

      * **{{ ui-key.yacloud.component.compute.network-select.field_subnetwork }}** — `ad-subnet-a`.
      * **{{ ui-key.yacloud.component.compute.network-select.field_external }}** — `{{ ui-key.yacloud.component.compute.network-select.switch_none }}`.
      * Разверните блок **{{ ui-key.yacloud.component.compute.network-select.section_additional }}** и в поле **{{ ui-key.yacloud.component.internal-v4-address-field.field_internal-ipv4-address }}** выберите `{{ ui-key.yacloud.component.compute.network-select.switch_manual }}`.
      * В появившемся поле для ввода укажите `10.1.0.3`.
  1. В блоке **{{ ui-key.yacloud.compute.instances.create.section_base }}** задайте имя ВМ: `ad-vm-a`.
  1. Нажмите кнопку **{{ ui-key.yacloud.compute.instances.create.button_create }}**.

  {% include [vm-reset-password-windows-operations](../../_includes/compute/reset-vm-password-windows-operations.md) %}

  Повторите шаги для ВМ с именем `ad-vm-b` в зоне доступности `{{ region-id }}-b`, подключите ее к подсети `ad-subnet-b` и вручную укажите внутренний IP-адрес `10.2.0.3`.

- CLI {#cli}

  ```bash
  yc compute instance create \
    --name ad-vm-a \
    --hostname ad-vm-a \
    --memory 8 \
    --cores 4 \
    --zone {{ region-id }}-a \
    --network-interface subnet-name=ad-subnet-a,ipv4-address=10.1.0.3 \
    --create-boot-disk image-folder-id=standard-images,image-family=windows-2022-dc-gvlk \
    --metadata-from-file user-data=setpass

  yc compute instance create \
    --name ad-vm-b \
    --hostname ad-vm-b \
    --memory 8 \
    --cores 4 \
    --zone {{ region-id }}-b \
    --network-interface subnet-name=ad-subnet-b,ipv4-address=10.2.0.3 \
    --create-boot-disk image-folder-id=standard-images,image-family=windows-2022-dc-gvlk \
    --metadata-from-file user-data=setpass
  ```

  {% include [cli-metadata-variables-substitution-notice](../../_includes/compute/create/cli-metadata-variables-substitution-notice.md) %}

{% endlist %}

## Создайте ВМ для бастионного хоста {#jump-server-vm}

Для настройки ВМ с Active Directory будет использоваться файловый сервер с выходом в интернет.

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. На странице каталога в [консоли управления]({{ link-console-main }}) нажмите кнопку **{{ ui-key.yacloud.iam.folder.dashboard.button_add }}** и выберите `{{ ui-key.yacloud.iam.folder.dashboard.value_compute }}`.
  1. В блоке **{{ ui-key.yacloud.compute.instances.create.section_image }}**:

      * Перейдите на вкладку **{{ ui-key.yacloud.compute.instances.create.image_value_custom_new }}**.
      * Нажмите кнопку **{{ ui-key.yacloud.common.select }}** и в открывшемся окне выберите **{{ ui-key.yacloud.common.create }}**.
      * В поле **{{ ui-key.yacloud.compute.instances.create-disk.field_source }}** выберите `{{ ui-key.yacloud.compute.instances.create-disk.value_source-image }}` и в списке ниже выберите образ **Windows Server 2022 Datacenter**. Как загрузить свой образ для продуктов Microsoft подробнее см. в разделе [Импортировать нужный образ](../../microsoft/byol.md#how-to-import).
      * (Опционально) В поле **{{ ui-key.yacloud.compute.field_additional_vt356 }}** включите опцию **{{ ui-key.yacloud.compute.field_disk-autodelete_qZn4x }}**, если вы хотите автоматически удалять этот диск при удалении ВМ.
      * Нажмите кнопку **{{ ui-key.yacloud.compute.component.instance-storage-dialog.button_add-disk }}**.
  
  1. В блоке **{{ ui-key.yacloud.k8s.node-groups.create.section_allocation-policy }}** выберите зону доступности `{{ region-id }}-d`.
  1. В блоке **{{ ui-key.yacloud.compute.instances.create.section_storages }}** задайте размер загрузочного диска `50 {{ ui-key.yacloud.common.units.label_gigabyte }}`.
  1. В блоке **{{ ui-key.yacloud.compute.instances.create.section_platform }}** перейдите на вкладку `{{ ui-key.yacloud.component.compute.resources.label_tab-custom }}` и укажите необходимую платформу, количество vCPU и объем RAM:

      * **{{ ui-key.yacloud.component.compute.resources.field_platform }}** — `Intel Ice Lake`.
      * **{{ ui-key.yacloud.component.compute.resources.field_cores }}** — `2`.
      * **{{ ui-key.yacloud.component.compute.resources.field_core-fraction }}** — `100%`.
      * **{{ ui-key.yacloud.component.compute.resources.field_memory }}** — `4 {{ ui-key.yacloud.common.units.label_gigabyte }}`.
  1. В блоке **{{ ui-key.yacloud.compute.instances.create.section_network }}**:
  
      * В поле **{{ ui-key.yacloud.component.compute.network-select.field_subnetwork }}** выберите подсеть `ad-subnet-d`.
      * В поле **{{ ui-key.yacloud.component.compute.network-select.field_external }}** оставьте значение `{{ ui-key.yacloud.component.compute.network-select.switch_auto }}`.

  1. В блоке **{{ ui-key.yacloud.compute.instances.create.section_base }}** задайте имя ВМ: `jump-server-vm`.
  1. Нажмите кнопку **{{ ui-key.yacloud.compute.instances.create.button_create }}**.

  {% include [vm-reset-password-windows-operations](../../_includes/compute/reset-vm-password-windows-operations.md) %}

- CLI {#cli}

  ```bash
  yc compute instance create \
    --name jump-server-vm \
    --hostname jump-server-vm \
    --memory 4 \
    --cores 2 \
    --zone {{ region-id }}-d \
    --network-interface subnet-name=ad-subnet-d,nat-ip-version=ipv4 \
    --create-boot-disk image-folder-id=standard-images,image-family=windows-2022-dc-gvlk \
    --metadata-from-file user-data=setpass
  ```

{% endlist %}

## Установите и настройте Active Directory {#install-ad}

У машин с Active Directory нет доступа в интернет, поэтому их следует настраивать через ВМ `jump-server-vm` с помощью [RDP](https://ru.wikipedia.org/wiki/Remote_Desktop_Protocol).
1. [Подключитесь к ВМ](../../compute/operations/vm-connect/rdp.md) `jump-server-vm` с помощью RDP. Используйте логин `Administrator` и ваш пароль.
1. Запустите RDP и подключитесь к ВМ `ad-vm-a` — используйте ее локальный IP-адрес, имя пользователя `Administrator` и ваш пароль.
1. Запустите PowerShell и задайте статический IP-адрес:

   ```powershell
   netsh interface ip set address "eth0" static 10.1.0.3 255.255.255.0 10.1.0.1
   ```

1. Установите роли Active Directory:

   ```powershell
   Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
   ```

   Результат:

   ```text
   Success  Restart Needed  Exit Code  Feature Result
   -------  --------------  ---------  --------------
   True     No              Success    {Active Directory Domain Services, Group P...
   ```

1. Создайте лес Active Directory:

   ```powershell
   Install-ADDSForest -DomainName 'yantoso.net' -Force:$true
   ```

   Затем введите пароль и подтвердите его.

   Windows перезапустится автоматически. Снова подключитесь к `ad-vm-a` и откройте PowerShell.
1. Переименуйте сайт по умолчанию в `{{ region-id }}-a`:

   ```powershell
   Get-ADReplicationSite 'Default-First-Site-Name' | Rename-ADObject -NewName '{{ region-id }}-a'
   ```

1. Создайте еще два сайта для других зон доступности:

   ```powershell
   New-ADReplicationSite '{{ region-id }}-b'
   New-ADReplicationSite '{{ region-id }}-d'
   ```

1. Создайте подсети и привяжите их к сайтам:

   ```powershell
   New-ADReplicationSubnet -Name '10.1.0.0/16' -Site '{{ region-id }}-a'
   New-ADReplicationSubnet -Name '10.2.0.0/16' -Site '{{ region-id }}-b'
   New-ADReplicationSubnet -Name '10.3.0.0/16' -Site '{{ region-id }}-d'
   ```

1. Переименуйте сайт-линк и настройте репликацию:

   ```powershell
   Get-ADReplicationSiteLink 'DEFAULTIPSITELINK' | `
       Set-ADReplicationSiteLink -SitesIncluded @{Add='{{ region-id }}-b'} -ReplicationFrequencyInMinutes 15 -PassThru | `
       Set-ADObject -Replace @{options = $($_.options -bor 1)} -PassThru | `
       Rename-ADObject -NewName '{{ region-id }}'
   ```

1. Укажите сервер переадресации DNS:

   ```powershell
   Set-DnsServerForwarder '10.1.0.2'
   ```

1. Настройте DNS-клиент:

   ```powershell
   Get-NetAdapter | Set-DnsClientServerAddress -ServerAddresses "10.2.0.3,127.0.0.1"
   ```

## Настройте второй контроллер домена {#install-ad-2}

1. Подключитесь к ВМ `jump-server-vm` с помощью RDP.
1. С помощью RDP подключитесь к ВМ `ad-vm-b` — используйте ее локальный IP-адрес, имя пользователя `Administrator` и ваш пароль.
1. Установите роли Active Directory:

   ```powershell
   Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
   ```

   Результат:

   ```text
   Success  Restart Needed  Exit Code       Feature Result
   -------  --------------  ---------       --------------
   True     No              NoChangeNeeded  {}
   ```

1. Настройте DNS-клиент:

   ```powershell
   Get-NetAdapter | Set-DnsClientServerAddress -ServerAddresses "10.1.0.3,127.0.0.1"
   ```

1. Настройте статический IP-адрес:

   ```powershell
   netsh interface ip set address "eth0" static 10.2.0.3 255.255.255.0 10.2.0.1
   ```

1. Добавьте контроллер в домен:

   ```powershell
   Install-ADDSDomainController `
       -Credential (Get-Credential "yantoso\Administrator") `
       -DomainName 'yantoso.net' `
       -Force:$true
   ```

   Затем введите пароль и подтвердите его.

   Windows перезапустится автоматически. Снова подключитесь к `ad-vm-b` и откройте PowerShell.
1. Укажите сервер переадресации DNS:

   ```powershell
   Set-DnsServerForwarder '10.2.0.2'
   ```

## Проверьте работу Active Directory {#test-ad}

1. Подключитесь к ВМ `jump-server-vm` с помощью RDP.
1. С помощью RDP подключитесь к ВМ `ad-vm-b` — используйте ее локальный IP-адрес, имя пользователя `Administrator` и ваш пароль. Запустите PowerShell.
1. Создайте тестового пользователя:

   ```powershell
   New-ADUser testUser
   ```

1. Убедитесь, что пользователь присутствует на обоих серверах:

   ```powershell
   Get-ADUser testUser -Server 10.1.0.3
   Get-ADUser testUser -Server 10.2.0.3
   ```

   Результаты обеих команд должны совпадать:

   ```text
   DistinguishedName : CN=testUser,CN=Users,DC=yantoso,DC=net
   Enabled           : False
   GivenName         :
   Name              : testUser
   ObjectClass       : user
   ObjectGUID        : 7202f41a-(...)-2d168ecd5271
   SamAccountName    : testUser
   SID               : S-1-5-21-(...)-1105
   Surname           :
   UserPrincipalName :
   ```

## Как удалить созданные ресурсы {#clear-out}

Чтобы перестать платить за развернутые серверы, достаточно удалить все созданные [ВМ](../../compute/operations/vm-control/vm-delete.md).
