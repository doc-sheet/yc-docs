# Access management in {{ load-testing-name }}

{{ load-testing-name }} uses [roles](../../iam/concepts/access-control/roles.md) to manage access permissions.

In this section, you will learn:
* [What resources you can assign a role for](#resources).
* [What roles this service has](#roles-list).
* [What roles are required](#required-roles) for specific actions.

{% include [about-access-management](../../_includes/iam/about-access-management.md) %}

{% include [roles-assign](../../_includes/iam/roles-assign.md) %}

## Resources supporting role assignment {#resources}

{% include [basic-resources](../../_includes/iam/basic-resources-for-access-control.md) %}

## What roles exist in this service {#roles-list}

{% include [roles-intro](../../_includes/roles-intro.md) %}

![image](../../_assets/load-testing/service-roles-hierarchy.svg)

### Service roles {#service-roles}

#### loadtesting.viewer {#loadtesting-viewer}

{% include [loadtesting.viewer](../../_roles/loadtesting/viewer.md) %}

#### loadtesting.editor {#loadtesting-editor}

{% include [loadtesting.editor](../../_roles/loadtesting/editor.md) %}

#### loadtesting.admin {#loadtesting-admin}

{% include [loadtesting.admin](../../_roles/loadtesting/admin.md) %}

#### loadtesting.loadTester {#loadtesting-loadtester}

{% include [loadtesting.loadtester](../../_roles/loadtesting/loadTester.md) %}

#### loadtesting.generatorClient {#loadtesting-generatorclient}

{% include [loadtesting.generatorclient](../../_roles/loadtesting/generatorClient.md) %}

#### loadtesting.externalAgent {#loadtesting-externalagent}

{% include [loadtesting.externalagent](../../_roles/loadtesting/externalAgent.md) %}

### Primitive roles {#primitive-roles}

{% include [roles-primitive](../../_includes/roles-primitive.md) %}

{% include [primitive-roles-footnote](../../_includes/primitive-roles-footnote.md) %}

#### What's next {#next}

* [How to assign a role](../../iam/operations/roles/grant.md).
* [How to revoke a role](../../iam/operations/roles/revoke.md).
* [Learn more about access management in {{ yandex-cloud }}](../../iam/concepts/access-control/index.md).
* [Learn more about role inheritance](../../resource-manager/concepts/resources-hierarchy.md#access-rights-inheritance).
