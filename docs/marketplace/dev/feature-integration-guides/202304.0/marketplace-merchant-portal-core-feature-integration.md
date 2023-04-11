---
title: Marketplace Merchant Portal Core feature integration
last_updated: Feb 20, 2023
description: Integrate the Merchant Portal Core feature into a Spryker project.
template: feature-integration-guide-template
related:
  - title: Marketplace Merchant Portal Core feature walkthrough
    link: docs/marketplace/dev/feature-walkthroughs/page.version/marketplace-merchant-portal-core-feature-walkthrough/marketplace-merchant-portal-core-feature-walkthrough.html
---

This document describes how to integrate the Marketplace Merchant Portal Core feature into a Spryker project.

## Install feature core

Follow the steps below to install the Merchant Portal Core feature core.

### Prerequisites

To start feature integration, integrate the required features:

| NAME | VERSION          | INTEGRATION GUIDE |
| -------------------- |------------------| ---------|
| Spryker Core | {{page.version}} | [Spryker Core feature integration](/docs/scos/dev/feature-integration-guides/{{page.version}}/spryker-core-feature-integration.html) |
| Spryker Core Back Office | {{page.version}} | [Install the Spryker Core Back Office feature](/docs/pbc/all/identity-access-management/{{page.version}}/install-and-upgrade/install-the-spryker-core-back-office-feature.html) |
| Marketplace Merchant | {{page.version}} | [Marketplace Merchant feature integration](/docs/marketplace/dev/feature-integration-guides/{{page.version}}/marketplace-merchant-feature-integration.html) |
| Acl | {{page.version}} | [Install the ACL feature](/docs/pbc/all/user-management/{{page.version}}/install-and-upgrade/install-the-acl-feature.html) |

### 1) Install the required modules using Composer

```bash
composer require spryker-feature/marketplace-merchantportal-core:"{{page.version}}" --update-with-dependencies
```

```bash
composer require spryker/security-merchant-portal-gui-extension
```

{% info_block warningBox "Verification" %}

Make sure that the following modules have been installed:

| MODULE | EXPECTED DIRECTORY |
|-|-|
| ZedUi  | vendor/spryker/zed-ui |
| GuiTable | vendor/spryker/gui-table |
| AclMerchantPortal   | vendor/spryker/acl-merchant-portal  |
| MerchantPortalApplication   | vendor/spryker/merchant-portal-application  |
| MerchantUserPasswordResetMail   | vendor/spryker/merchant-user-password-reset-mail  |
| Navigation   | vendor/spryker/navigation  |
| SecurityMerchantPortalGui  | vendor/spryker/security-merchant-portal-gui |
| UserMerchantPortalGui | vendor/spryker/user-merchant-portal-gui |
| UserMerchantPortalGuiExtension | spryker/user-merchant-portal-gui-extension |
| SecurityMerchantPortalGuiExtension | spryker/security-merchant-portal-gui-extension |

{% endinfo_block %}

### 2) Set up configuration

Add the following configuration:

| CONFIGURATION                        | SPECIFICATION                                                               | NAMESPACE        |
|--------------------------------------|-----------------------------------------------------------------------------|------------------|
| GuiTableConfig::getDefaultTimezone() | Defines default timezone for formatting the `DateTime` data to the ISO 8601 format. | Pyz\Zed\GuiTable |

**src/Pyz/Zed/GuiTable/GuiTableConfig.php**

```php
<?php

namespace Pyz\Zed\GuiTable;

use Spryker\Zed\GuiTable\GuiTableConfig as SprykerGuiTableConfig;

class GuiTableConfig extends SprykerGuiTableConfig
{
    /**
     * @return string|null
     */
    public function getDefaultTimezone(): ?string
    {
        return 'UTC';
    }
}
```

### 3) Set up the database schema

**src/Pyz/Zed/Merchant/Persistence/Propel/Schema/spy_merchant.schema.xml**

```xml
<?xml version="1.0"?>
<database xmlns="spryker:schema-01" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" name="zed" xsi:schemaLocation="spryker:schema-01 https://static.spryker.com/schema-01.xsd" namespace="Orm\Zed\Merchant\Persistence" package="src.Orm.Zed.Merchant.Persistence">
    <table name="spy_merchant">
        <behavior name="event">
            <parameter name="spy_merchant-name" column="name"/>
            <parameter name="spy_merchant-is_active" column="is_active"/>
        </behavior>
        <behavior name="\Spryker\Zed\AclEntity\Persistence\Propel\Behavior\AclEntityBehavior"/>
    </table>

</database>
```

**src/Pyz/Zed/MerchantUser/Persistence/Propel/Schema/spy_merchant_user.schema.xml**

```xml
<?xml version="1.0"?>
<database xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" name="zed" xsi:noNamespaceSchemaLocation="http://static.spryker.com/schema-01.xsd" namespace="Orm\Zed\MerchantUser\Persistence" package="src.Orm.Zed.MerchantUser.Persistence">

    <table name="spy_merchant_user">
        <behavior name="\Spryker\Zed\AclEntity\Persistence\Propel\Behavior\AclEntityBehavior"/>
    </table>

</database>
```

Apply database changes and generate entity and transfer changes:

```bash
console transfer:generate
console propel:install
console transfer:generate
```

### 4) Set up behavior

Set up behavior as follows:

#### Integrate the following plugins

| PLUGIN  | SPECIFICATION | PREREQUISITES | NAMESPACE |
|---|---| --- |---|
| MerchantUserSecurityPlugin | Sets security firewalls (rules, handlers) for Marketplace users. |  | Spryker\Zed\SecurityMerchantPortalGui\Communication\Plugin\Security    |
| BooleanToStringTwigPlugin | Adds a new Twig function for converting Boolean to String. |  | Spryker\Zed\ZedUi\Communication\Plugin\Twig |
| ZedUiNavigationTwigPlugin | Adds a new Twig function for rendering navigation using web components.      |  | Spryker\Zed\ZedUi\Communication\Plugin  |
| GuiTableApplicationPlugin  | Enables `GuiTable` infrastructure for Zed. |  | Spryker\Zed\GuiTable\Communication\Plugin\Application     |
| GuiTableConfigurationTwigPlugin    | Adds a new Twig function for rendering `GuiTableConfiguration` for the `GuiTable` web component.  |  | Spryker\Zed\GuiTable\Communication\Plugin\Twig  |
| SecurityTokenUpdateMerchantUserPostChangePlugin | Rewrites Symfony security token. |  | Spryker\Zed\SecurityMerchantPortalGui\Communication\Plugin\UserMerchantPortalGui |
| MerchantPortalConfigurationAclEntityMetadataConfigExpanderPlugin       | Expands provided Acl Entity Metadata with merchant order composite, merchant product composite, merchant composite, product offer composit data, merchant read global entities and allowlist entities. |  | Spryker\Zed\AclMerchantPortal\Communication\Plugin\AclEntity   |
| MerchantAclEntitiesMerchantPostCreatePlugin     | Creates ACL group, ACL role, ACL rules, ACL entity rules and ACL entity segment for a provided merchant.  |  | Spryker\Zed\AclMerchantPortal\Communication\Plugin\Merchant    |
| MerchantUserAclEntitiesMerchantUserPostCreatePlugin | Creates ACL group, ACL role, ACL rules, ACL entity rules, and ACL entity segment for a provided merchant user. |  | Spryker\Zed\AclMerchantPortal\Communication\Plugin\MerchantUser |
| AclMerchantPortalMerchantUserRoleFilterPreConditionPlugin | Checks if the Symfony security authentication roles must be filtered out.  |  | Spryker\Zed\AclMerchantPortal\Communication\Plugin\MerchantUser |
| MerchantUserUserRoleFilterPlugin   | Filters `ROLE_BACK_OFFICE_USER` to prevent a merchant user from loging in to the Back Office.  |  | Spryker\Zed\MerchantUser\Communication\Plugin\SecurityGui |
| ProductViewerForOfferCreationAclInstallerPlugin | Provide `ProductViewerForOfferCreation` roles with rules and groups to create on installation. |  | Spryker\Zed\AclMerchantPortal\Communication\Plugin\MerchantUser |
| AclGroupMerchantUserLoginRestrictionPlugin | Checks if the merchant user login is restricted.  |  | Spryker\Zed\AclMerchantPortal\Communication\Plugin\SecurityMerchantPortalGui     |

**src/Pyz/Zed/Twig/TwigDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\Twig;

use Spryker\Zed\GuiTable\Communication\Plugin\Twig\GuiTableConfigurationTwigPlugin;
use Spryker\Zed\ZedUi\Communication\Plugin\Twig\BooleanToStringTwigPlugin;
use Spryker\Zed\ZedUi\Communication\Plugin\ZedUiNavigationTwigPlugin;
use Spryker\Zed\Twig\TwigDependencyProvider as SprykerTwigDependencyProvider;

class TwigDependencyProvider extends SprykerTwigDependencyProvider
{
    /**
     * @return array<\Spryker\Shared\TwigExtension\Dependency\Plugin\TwigPluginInterface>
     */
    protected function getTwigPlugins(): array
    {
        return [
            new ZedUiNavigationTwigPlugin(),
            new BooleanToStringTwigPlugin(),
            new GuiTableConfigurationTwigPlugin()
        ];
    }
}
```

**src/Pyz/Zed/Application/ApplicationDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\Application;

use Spryker\Zed\Application\ApplicationDependencyProvider as SprykerApplicationDependencyProvider;
use Spryker\Zed\GuiTable\Communication\Plugin\Application\GuiTableApplicationPlugin;

class ApplicationDependencyProvider extends SprykerApplicationDependencyProvider
{

    /**
     * @return array<\Spryker\Shared\ApplicationExtension\Dependency\Plugin\ApplicationPluginInterface>
     */
    protected function getApplicationPlugins(): array
    {
        return [  
            new GuiTableApplicationPlugin(),
        ];
    }
}
```

**src/Pyz/Zed/Security/SecurityDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\Security;

use Spryker\Zed\Security\SecurityDependencyProvider as SprykerSecurityDependencyProvider;
use Spryker\Zed\SecurityGui\Communication\Plugin\Security\UserSecurityPlugin;
use Spryker\Zed\SecurityMerchantPortalGui\Communication\Plugin\Security\MerchantUserSecurityPlugin;
use Spryker\Zed\User\Communication\Plugin\Security\UserSessionHandlerSecurityPlugin;

class SecurityDependencyProvider extends SprykerSecurityDependencyProvider
{
    /**
     * @return array<\Spryker\Shared\SecurityExtension\Dependency\Plugin\SecurityPluginInterface>
     */
    protected function getSecurityPlugins(): array
    {
        return [
            new UserSessionHandlerSecurityPlugin(),
            new MerchantUserSecurityPlugin(),
            new UserSecurityPlugin(),
        ];
    }
}
```

**src/Pyz/Zed/SecurityGui/SecurityGuiDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\SecurityGui;

use Spryker\Zed\MerchantUser\Communication\Plugin\SecurityGui\MerchantUserUserRoleFilterPlugin;
use Spryker\Zed\SecurityGui\SecurityGuiDependencyProvider as SprykerSecurityGuiDependencyProvider;

class SecurityGuiDependencyProvider extends SprykerSecurityGuiDependencyProvider
{
    /**
     * @return array<\Spryker\Zed\SecurityGuiExtension\Dependency\Plugin\UserRoleFilterPluginInterface>
     */
    protected function getUserRoleFilterPlugins(): array
    {
        return [
            new MerchantUserUserRoleFilterPlugin(),
        ];
    }
}
```

{% info_block warningBox "Verification" %}

Ensure that merchant users can log in to the Back Office dashboard.

{% endinfo_block %}

**src/Pyz/Zed/UserMerchantPortalGui/UserMerchantPortalGuiDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\UserMerchantPortalGui;

use Spryker\Zed\SecurityMerchantPortalGui\Communication\Plugin\UserMerchantPortalGui\SecurityTokenUpdateMerchantUserPostChangePlugin;
use Spryker\Zed\UserMerchantPortalGui\UserMerchantPortalGuiDependencyProvider as SprykerUserMerchantPortalGuiDependencyProvider;

class UserMerchantPortalGuiDependencyProvider extends SprykerUserMerchantPortalGuiDependencyProvider
{
    /**
     * @return array<\Spryker\Zed\UserMerchantPortalGuiExtension\Dependency\Plugin\MerchantUserPostChangePluginInterface>
     */
    public function getMerchantUserPostChangePlugins(): array
    {
        return [
            new SecurityTokenUpdateMerchantUserPostChangePlugin(),
        ];
    }
}

```

**src/Pyz/Zed/AclEntity/AclEntityDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\AclEntity;

use Spryker\Zed\AclEntity\AclEntityDependencyProvider as SprykerAclEntityDependencyProvider;
use Spryker\Zed\AclMerchantPortal\Communication\Plugin\AclEntity\MerchantPortalConfigurationAclEntityMetadataConfigExpanderPlugin;

class AclEntityDependencyProvider extends SprykerAclEntityDependencyProvider
{
    /**
     * @return array<\Spryker\Zed\AclEntityExtension\Dependency\Plugin\AclEntityMetadataConfigExpanderPluginInterface>
     */
    protected function getAclEntityMetadataCollectionExpanderPlugins(): array
    {
        return [
            new MerchantPortalConfigurationAclEntityMetadataConfigExpanderPlugin(),
        ];
    }
}
```
**src/Pyz/Zed/Merchant/MerchantDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\Merchant;

use Spryker\Zed\AclMerchantPortal\Communication\Plugin\Merchant\MerchantAclEntitiesMerchantPostCreatePlugin;
use Spryker\Zed\Merchant\MerchantDependencyProvider as SprykerMerchantDependencyProvider;

class MerchantDependencyProvider extends SprykerMerchantDependencyProvider
{
    /**
     * @return array<\Spryker\Zed\MerchantExtension\Dependency\Plugin\MerchantPostCreatePluginInterface>
     */
    protected function getMerchantPostCreatePlugins(): array
    {
        return [
            new MerchantAclEntitiesMerchantPostCreatePlugin(),
        ];
    }
}
```

**src/Pyz/Zed/MerchantUser/MerchantUserDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\MerchantUser;

use Spryker\Zed\AclMerchantPortal\Communication\Plugin\MerchantUser\AclMerchantPortalMerchantUserRoleFilterPreConditionPlugin;
use Spryker\Zed\AclMerchantPortal\Communication\Plugin\MerchantUser\MerchantUserAclEntitiesMerchantUserPostCreatePlugin;
use Spryker\Zed\MerchantUser\MerchantUserDependencyProvider as SprykerMerchantUserDependencyProvider;

class MerchantUserDependencyProvider extends SprykerMerchantUserDependencyProvider
{
    /**
     * @return array<\Spryker\Zed\MerchantUserExtension\Dependency\Plugin\MerchantUserPostCreatePluginInterface>
     */
    protected function getMerchantUserPostCreatePlugins(): array
    {
        return [
            new MerchantUserAclEntitiesMerchantUserPostCreatePlugin(),
        ];
    }

    /**
     * @return array<\Spryker\Zed\MerchantUserExtension\Dependency\Plugin\MerchantUserRoleFilterPreConditionPluginInterface>
     */
    protected function getMerchantUserRoleFilterPreConditionPlugins(): array
    {
        return [
            new AclMerchantPortalMerchantUserRoleFilterPreConditionPlugin(),
        ];
    }
}
```

{% info_block warningBox "Verification" %}

Ensure that non-merchant users whose Acl Group has Back Office allowed Acl Group Reference (see `AclMerchantPortalConfig::getBackofficeAllowedAclGroupReferences()`) can log in to the Back Office.

{% endinfo_block %}

**src/Pyz/Zed/Acl/AclDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\Acl;

use Spryker\Zed\Acl\AclDependencyProvider as SprykerAclDependencyProvider;
use Spryker\Zed\AclMerchantPortal\Communication\Plugin\MerchantUser\ProductViewerForOfferCreationAclInstallerPlugin;

class AclDependencyProvider extends SprykerAclDependencyProvider
{
    /**
     * @return array<\Spryker\Zed\AclExtension\Dependency\Plugin\AclInstallerPluginInterface>
     */
    protected function getAclInstallerPlugins(): array
    {
        return [
            new ProductViewerForOfferCreationAclInstallerPlugin(),
        ];
    }
}
```

**src/Pyz/Zed/SecurityMerchantPortalGui/SecurityMerchantPortalGuiDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\SecurityMerchantPortalGui;

use Spryker\Zed\AclMerchantPortal\Communication\Plugin\SecurityMerchantPortalGui\AclGroupMerchantUserLoginRestrictionPlugin;
use Spryker\Zed\SecurityMerchantPortalGui\SecurityMerchantPortalGuiDependencyProvider as SprykerSecurityMerchantPortalGuiDependencyProvider;

class SecurityMerchantPortalGuiDependencyProvider extends SprykerSecurityMerchantPortalGuiDependencyProvider
{
    /**
     * @return array<\Spryker\Zed\SecurityMerchantPortalGuiExtension\Dependency\Plugin\MerchantUserLoginRestrictionPluginInterface>
     */
    protected function getMerchantUserLoginRestrictionPlugins(): array
    {
        return [
             new AclGroupMerchantUserLoginRestrictionPlugin(),
        ];
    }
}
```

#### Enable Merchant Portal infrastructural plugins

1. Enable the following plugins:

<details open><summary markdown='span'>src/Pyz/Zed/MerchantPortalApplication/MerchantPortalApplicationDependencyProvider.php</summary>

```php
<?php

namespace Pyz\Zed\MerchantPortalApplication;

use Spryker\Zed\AclEntity\Communication\Plugin\Application\AclEntityApplicationPlugin;
use Spryker\Zed\ErrorHandler\Communication\Plugin\Application\ErrorHandlerApplicationPlugin;
use Spryker\Zed\EventDispatcher\Communication\Plugin\Application\EventDispatcherApplicationPlugin;
use Spryker\Zed\Form\Communication\Plugin\Application\FormApplicationPlugin;
use Spryker\Zed\GuiTable\Communication\Plugin\Application\GuiTableApplicationPlugin;
use Spryker\Zed\Http\Communication\Plugin\Application\HttpApplicationPlugin;
use Spryker\Zed\Locale\Communication\Plugin\Application\LocaleApplicationPlugin;
use Spryker\Zed\MerchantPortalApplication\MerchantPortalApplicationDependencyProvider as SprykerMerchantPortalApplicationDependencyProvider;
use Spryker\Zed\Messenger\Communication\Plugin\Application\MessengerApplicationPlugin;
use Spryker\Zed\Propel\Communication\Plugin\Application\PropelApplicationPlugin;
use Spryker\Zed\Router\Communication\Plugin\Application\MerchantPortalRouterApplicationPlugin;
use Spryker\Zed\Security\Communication\Plugin\Application\SecurityApplicationPlugin;
use Spryker\Zed\Session\Communication\Plugin\Application\SessionApplicationPlugin;
use Spryker\Zed\Translator\Communication\Plugin\Application\TranslatorApplicationPlugin;
use Spryker\Zed\Twig\Communication\Plugin\Application\TwigApplicationPlugin;
use Spryker\Zed\Validator\Communication\Plugin\Application\ValidatorApplicationPlugin;
use Spryker\Zed\ZedUi\Communication\Plugin\Application\ZedUiApplicationPlugin;

class MerchantPortalApplicationDependencyProvider extends SprykerMerchantPortalApplicationDependencyProvider
{
    /**
     * @return array<\Spryker\Shared\ApplicationExtension\Dependency\Plugin\ApplicationPluginInterface>
     */
    protected function getMerchantPortalApplicationPlugins(): array
    {
        return [
            new SessionApplicationPlugin(),
            new TwigApplicationPlugin(),
            new EventDispatcherApplicationPlugin(),
            new LocaleApplicationPlugin(),
            new TranslatorApplicationPlugin(),
            new MessengerApplicationPlugin(),
            new PropelApplicationPlugin(),
            new MerchantPortalRouterApplicationPlugin(),
            new HttpApplicationPlugin(),
            new ErrorHandlerApplicationPlugin(),
            new FormApplicationPlugin(),
            new ValidatorApplicationPlugin(),
            new GuiTableApplicationPlugin(),
            new SecurityApplicationPlugin(),
            new ZedUiApplicationPlugin(),
            new AclEntityApplicationPlugin(),
        ];
    }
}
```

</details>

**src/Pyz/Zed/MerchantPortalApplication/Communication/Bootstrap/MerchantPortalBootstrap.php**
```php
<?php

namespace Pyz\Zed\MerchantPortalApplication\Communication\Bootstrap;

use Spryker\Zed\MerchantPortalApplication\Communication\Bootstrap\MerchantPortalBootstrap as MerchantPortalApplicationBootstrap;

class MerchantPortalBootstrap extends MerchantPortalApplicationBootstrap
{
}
```

**src/Pyz/Zed/MerchantPortalApplication/Communication/MerchantPortalApplicationCommunicationFactory.php**

```php
<?php

namespace Pyz\Zed\MerchantPortalApplication\Communication;

use Spryker\Zed\MerchantPortalApplication\Communication\MerchantPortalApplicationCommunicationFactory as SprykerMerchantPortalApplicationCommunicationFactory;

class MerchantPortalApplicationCommunicationFactory extends SprykerMerchantPortalApplicationCommunicationFactory
{
}
```

**src/Pyz/Zed/Router/RouterDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\Router;

use Spryker\Zed\Router\Communication\Plugin\Router\MerchantPortalRouterPlugin;
use Spryker\Zed\Router\RouterDependencyProvider as SprykerRouterDependencyProvider;

class RouterDependencyProvider extends SprykerRouterDependencyProvider
{
    /**
     * @return array|array<\Spryker\Zed\RouterExtension\Dependency\Plugin\RouterPluginInterface>
     */
    protected function getMerchantPortalRouterPlugins(): array
    {
        return [
            new MerchantPortalRouterPlugin()
        ];
    }
}
```

2. Open access to the Merchant Portal login page by default:

**config/Shared/config_default.php**

```php
<?php

$config[AclConstants::ACL_DEFAULT_RULES][] = [
  [
    'bundle' => 'security-merchant-portal-gui',
    'controller' => 'login',
    'action' => 'index',
    'type' => 'allow',
  ],

];
```

3. Add a console command for warming up the *Merchant Portal* router cache:

**src/Pyz/Zed/Console/ConsoleDependencyProvider.php**
```php
<?php

namespace Pyz\Zed\Console;

use Spryker\Zed\Console\ConsoleDependencyProvider as SprykerConsoleDependencyProvider;
use Spryker\Zed\Router\Communication\Plugin\Console\MerchantPortalRouterCacheWarmUpConsole;

class ConsoleDependencyProvider extends SprykerConsoleDependencyProvider
{
    /**
     * @param \Spryker\Zed\Kernel\Container $container
     *
     * @return array<\Symfony\Component\Console\Command\Command>
     */
    protected function getConsoleCommands(Container $container): array
    {
        $commands = [
            new MerchantPortalRouterCacheWarmUpConsole(),
        ];

        return $commands;
    }
}
```

**config/install/docker.yml**
```yaml
env:
    NEW_RELIC_ENABLED: 0

sections:
    build:
      router-cache-warmup-merchant-portal:
        command: 'vendor/bin/console router:cache:warm-up:merchant-portal'
```

### 5) Set up transfer objects

Generate transfer objects:

```bash
console transfer:generate
```

{% info_block warningBox "Verification" %}

Make sure that the following changes have been applied in transfer objects:

| TRANSFER  | TYPE  | EVENT | PATH  |
| ----------- | ----- | ------- | -------------------- |
| GuiTableDataRequest | class | Created | src/Generated/Shared/Transfer/GuiTableDataRequestTransfer |
| GuiTableConfiguration | class | Created | src/Generated/Shared/Transfer/GuiTableConfigurationTransfer |
| GuiTableColumnConfiguration | class | Created | src/Generated/Shared/Transfer/GuiTableColumnConfigurationTransfer |
| GuiTableTitleConfiguration | class | Created | src/Generated/Shared/Transfer/GuiTableTitleConfigurationTransfer |
| GuiTableDataSourceConfiguration | class | Created | src/Generated/Shared/Transfer/GuiTableDataSourceConfigurationTransfer |
| GuiTableRowActionsConfiguration | class | Created | src/Generated/Shared/Transfer/GuiTableRowActionsConfigurationTransfer |
| GuiTableBatchActionsConfiguration | class | Created | src/Generated/Shared/Transfer/GuiTableBatchActionsConfigurationTransfer |
| GuiTablePaginationConfiguration | class | Created | src/Generated/Shared/Transfer/GuiTablePaginationConfigurationTransfer |
| GuiTableSearchConfiguration | class | Created | src/Generated/Shared/Transfer/GuiTableSearchConfigurationTransfer |
| GuiTableFiltersConfiguration | class | Created | src/Generated/Shared/Transfer/GuiTableFiltersConfigurationTransfer |
| GuiTableItemSelectionConfiguration | class | Created | src/Generated/Shared/Transfer/GuiTableItemSelectionConfigurationTransfer |
| GuiTableSyncStateUrlConfiguration | class | Created | src/Generated/Shared/Transfer/GuiTableSyncStateUrlConfigurationTransfer |
| GuiTableEditableConfiguration | class | Created | src/Generated/Shared/Transfer/GuiTableEditableConfigurationTransfer |
| GuiTableEditableCreateConfiguration | class | Created | src/Generated/Shared/Transfer/GuiTableEditableCreateConfigurationTransfer |
| GuiTableEditableUpdateConfiguration | class | Created | src/Generated/Shared/Transfer/GuiTableEditableUpdateConfigurationTransfer |
| GuiTableEditableButton | class | Created | src/Generated/Shared/Transfer/GuiTableEditableButtonTransfer |
| GuiTableEditableUrl | class | Created | src/Generated/Shared/Transfer/GuiTableEditableUrlTransfer |
| GuiTableEditableInitialData | class | Created | src/Generated/Shared/Transfer/GuiTableEditableInitialDataTransfer |
| GuiTableEditableDataError | class | Created | src/Generated/Shared/Transfer/GuiTableEditableDataErrorTransfer |
| GuiTableDataResponse | class | Created | src/Generated/Shared/Transfer/GuiTableDataResponseTransfer |
| GuiTableRowDataResponse | class | Created | src/Generated/Shared/Transfer/GuiTableRowDataResponseTransfer |
| GuiTableDataResponsePayload | class | Created | src/Generated/Shared/Transfer/GuiTableDataResponsePayloadTransfer |
| SelectGuiTableFilterTypeOptions | class | Created | src/Generated/Shared/Transfer/SelectGuiTableFilterTypeOptionsTransfer |
| OptionSelectGuiTableFilterTypeOptions | class | Created | src/Generated/Shared/Transfer/OptionSelectGuiTableFilterTypeOptionsTransfer |
| GuiTableFilter | class | Created | src/Generated/Shared/Transfer/GuiTableFilterTransfer |
| GuiTableRowAction | class | Created | src/Generated/Shared/Transfer/GuiTableRowActionTransfer |
| GuiTableRowActionOptions | class | Created | src/Generated/Shared/Transfer/GuiTableRowActionOptionsTransfer |
| DateRangeGuiTableFilterTypeOptions | class | Created | src/Generated/Shared/Transfer/DateRangeGuiTableFilterTypeOptionsTransfer |
| CriteriaRangeFilter | class | Created | src/Generated/Shared/Transfer/CriteriaRangeFilterTransfer |
| GuiTableBatchAction | class | Created | src/Generated/Shared/Transfer/GuiTableBatchActionTransfer |
| GuiTableBatchActionOptions | class | Created | src/Generated/Shared/Transfer/GuiTableBatchActionOptionsTransfer |
| GuiTableColumnConfiguratorConfiguration | class | Created | src/Generated/Shared/Transfer/GuiTableColumnConfiguratorConfigurationTransfer |
| ZedUiFormResponseAction | class | Created | src/Generated/Shared/Transfer/ZedUiFormResponseActionTransfer |

{% endinfo_block %}

## Install feature frontend

Follow the steps below to install the Merchant Portal Core feature frontend.

### Prerequisites

Environment requirements:
- NPM v6 (higher versions have problems with workspace)
- NodeJs v12-14
- Yarn v2 (or latest Yarn v1)

Spryker requirements:

To start builder integration, check versions of Spryker packages:

| NAME | VERSION |
| --------------------------- | --------- |
| Discount (optional)         | 9.7.4 or later  |
| Gui (optional)              | 3.30.2 or later |
| Product Relation (optional) | 2.4.3 or later  |

### 1) Install the required modules using Composer

```bash
composer require spryker/dashboard-merchant-portal-gui:"^1.4.0" --update-with-dependencies
```

| MODULE | EXPECTED DIRECTORY |
|-|-|
| DashboardMerchantPortalGui   | vendor/spryker/dashboard-merchant-portal-gui  |
| DashboardMerchantPortalGuiExtension | vendor/spryker/dashboard-merchant-portal-gui-extension |

### 2) Set up transfer objects

Generate transfer changes:

```bash
console transfer:generate
```

{% info_block warningBox "Verification" %}

Make sure that the following changes have been applied in transfer objects:

| TRANSFER | TYPE | EVENT | PATH |
|-|-|-|-|
| MerchantDashboardCard | class | Created | src/Generated/Shared/Transfer/MerchantDashboardCardTransfer |
| MerchantDashboardActionButton | class | Created | src/Generated/Shared/Transfer/MerchantDashboardActionButtonTransfer |

{% endinfo_block %}

### 3) Build the navigation cache

Execute the following command:

```bash
console navigation:build-cache
```
{% info_block warningBox "Verification" %}

Make sure that Merchant Portal has the **Dashboard** menu.

{% endinfo_block %}


### 4) Set up Marketplace builder configs

1. Add the following files to the root folder:

```bash
wget -O angular.json https://raw.githubusercontent.com/spryker-shop/suite/1.9.0/angular.json
wget -O nx.json https://raw.githubusercontent.com/spryker-shop/suite/1.9.0/nx.json
wget -O .browserslistrc https://raw.githubusercontent.com/spryker-shop/suite/1.9.0/.browserslistrc
```

2. Rename default `tsconfig.json` to `tsconfig.base.json`. Create additional `tsconfig` files (`tsconfig.yves.json`, `tsconfig.mp.json`)

```bash
mv tsconfig.json tsconfig.base.json
wget -O tsconfig.yves.json https://raw.githubusercontent.com/spryker-shop/suite/1.9.0/tsconfig.yves.json
wget -O tsconfig.mp.json https://raw.githubusercontent.com/spryker-shop/suite/1.9.0/tsconfig.mp.json
```

3. Add `vendor/**` and `**/node_modules/**` to exclude option in `tslint.json`.

4. Add the `tslint.mp.json` file:

```bash
wget -O tslint.mp.json https://raw.githubusercontent.com/spryker-shop/suite/1.9.0/tslint.mp.json
```

5. Install npm dependencies:

```bash
npm i rxjs@~7.4.0 zone.js@~0.11.4 @webcomponents/custom-elements@~1.3.1 @webcomponents/webcomponents-platform@~1.0.1 @webcomponents/webcomponentsjs@~2.4.0
```

6. Install npm dev dependencies:

```bash
npm i -D @angular-builders/custom-webpack@~12.1.3 @angular-devkit/build-angular@~12.2.16 @angular/cli@~12.2.16 @angular/common@~12.2.16 @angular/compiler@~12.2.16 @angular/compiler-cli@~12.2.16 @angular/core@~12.2.16 @angular/language-service@~12.2.16 @angular/platform-browser@~12.2.16 @angular/platform-browser-dynamic@~12.2.16 @babel/plugin-proposal-class-properties@~7.10.4 @babel/plugin-transform-runtime@~7.10.5 @babel/preset-typescript@~7.10.4 @jsdevtools/file-path-filter@~3.0.2 @nrwl/cli@~12.10.1 @nrwl/jest@~12.10.1 @nrwl/tao@~12.10.1 @nrwl/workspace@~12.10.1 @spryker/oryx-for-zed@~2.11.3 @types/jest@~27.0.2 @types/node@~14.14.33 @types/webpack@~4.41.17 jest@~27.2.3 jest-preset-angular@~9.0.3 node-sass@~4.14.1 npm-run-all@~4.1.5 rimraf@~3.0.2 ts-jest@~27.0.5 ts-node@~9.1.1 tslib@~2.0.0 typescript@~4.2.4
```

7. Update `package.json` with the following fields:

**package.json**

```json
{
    "workspaces": [
        "vendor/spryker/*",
        "vendor/spryker/*/assets/Zed"
    ],
    "scripts": {
        "mp:build": "ng build",
        "mp:build:watch": "ng build --watch",
        "mp:build:production": "ng build --prod",
        "mp:test": "ng test",
        "mp:lint": "ng lint",
        "mp:clean": "run-s mp:clean:*",
        "mp:clean:dist": "rimraf public/MerchantPortal/assets/js",
        "mp:update:paths": "node ./frontend/merchant-portal/update-config-paths",
        "postinstall": "npm run mp:update:paths"
    },
    "engines": {
        "node": ">=12.0.0",
        "npm": ">=6.9.0"
    },
    "resolutions": {
        "typescript": "4.2.4",
        "fsevents": "2.1.3"
    }
}
```

8. For Yves, in the `globalSettings.paths` object, update `frontend/settings.js` to point to an updated `tsconfig`:

**frontend/settings.js**

```js
const globalSettings = {
  ...
  paths: {
    tsConfig: './tsconfig.yves.json',
    ...
  }
};
```

9. Add a `.yarnrc.yml` file:

**.yarnrc.yml**

```yaml
nodeLinker: node-modules

plugins:
    - path: .yarn/plugins/@yarnpkg/plugin-workspace-tools.js
      spec: '@yarnpkg/plugin-workspace-tools'
    - path: .yarn/plugins/@yarnpkg/plugin-interactive-tools.cjs
      spec: '@yarnpkg/plugin-interactive-tools'

yarnPath: .yarn/releases/yarn-2.3.3.js
```

10. Add the `.yarn` folder and download `plugin-workspace-tools.js` and `yarn-2.0.0-rc.32.js`:

```bash
mkdir .yarn && mkdir .yarn/plugins && mkdir .yarn/releases
wget -O .yarn/plugins/@yarnpkg/plugin-workspace-tools.js https://raw.githubusercontent.com/spryker-shop/suite/1.9.0/.yarn/plugins/%40yarnpkg/plugin-workspace-tools.js
wget -O .yarn/plugins/@yarnpkg/plugin-interactive-tools.cjs https://raw.githubusercontent.com/spryker-shop/suite/1.8.0/.yarn/plugins/%40yarnpkg/plugin-interactive-tools.cjs
wget -O .yarn/releases/yarn-2.3.3.js https://raw.githubusercontent.com/spryker-shop/suite/1.9.0/.yarn/releases/yarn-2.3.3.js
```

11. Run commands from the root of the project:

```bash
npm i -g yarn @angular/cli@12.2.16
```

12. To check if the yarn has been installed correctly, run `yarn -v`. Version 1.22.*x* is global (outside of the project) and 2.*x.x* is at least in the project.

`ng --version` must show Angular CLI: 12.2.16 version.

13. Install project dependencies:

```bash
yarn install
```

{% info_block warningBox "Warning" %}

If you get `Missing write access to node_modules/mp-profile`, delete this *file* and make a *folder* with the same name.

{% endinfo_block %}

14. Check if the marketplace packages are located in the `node_modules/@spryker` folder—for example, utils.

### 5) Install Marketplace builder

1. Add the `merchant-portal` folder and builder files:

```bash
mkdir frontend/merchant-portal
wget -O frontend/merchant-portal/entry-points.js https://raw.githubusercontent.com/spryker-shop/suite/1.9.0/frontend/merchant-portal/entry-points.js
wget -O frontend/merchant-portal/html-transform.js https://raw.githubusercontent.com/spryker-shop/suite/1.9.0/frontend/merchant-portal/html-transform.js
wget -O frontend/merchant-portal/jest.config.js https://raw.githubusercontent.com/spryker-shop/suite/1.9.0/frontend/merchant-portal/jest.config.js
wget -O frontend/merchant-portal/jest.preset.js https://raw.githubusercontent.com/spryker-shop/suite/1.9.0/frontend/merchant-portal/jest.preset.js
wget -O frontend/merchant-portal/mp-paths.js https://raw.githubusercontent.com/spryker-shop/suite/1.9.0/frontend/merchant-portal/mp-paths.js
wget -O frontend/merchant-portal/test-setup.ts https://raw.githubusercontent.com/spryker-shop/suite/1.9.0/frontend/merchant-portal/test-setup.ts
wget -O frontend/merchant-portal/tsconfig.spec.json https://raw.githubusercontent.com/spryker-shop/suite/1.9.0/frontend/merchant-portal/tsconfig.spec.json
wget -O frontend/merchant-portal/update-config-paths.js https://raw.githubusercontent.com/spryker-shop/suite/1.9.0/frontend/merchant-portal/update-config-paths.js
wget -O frontend/merchant-portal/utils.js https://raw.githubusercontent.com/spryker-shop/suite/1.9.0/frontend/merchant-portal/utils.js
wget -O frontend/merchant-portal/webpack.config.ts https://raw.githubusercontent.com/spryker-shop/suite/1.9.0/frontend/merchant-portal/webpack.config.ts
```

**frontend/merchant-portal/webpack.config.ts**

```ts
import {
    CustomWebpackBrowserSchema,
    TargetOptions,
} from "@angular-builders/custom-webpack";
import * as webpack from "webpack";

import { getMPEntryPointsMap } from "./entry-points";

export default async (
    config: webpack.Configuration,
    options: CustomWebpackBrowserSchema,
    targetOptions: TargetOptions
): Promise<webpack.Configuration> => {
    console.log("Resolving entry points...");

    const entryPointsMap = await getMPEntryPointsMap();

    console.log(`Found ${Object.keys(entryPointsMap).length} entry point(s)!`);

    config.entry = {
        ...(config.entry as any),
        ...entryPointsMap,
    };

    return config;
};
```

### 6) Add files for the Merchant Portal entry point

**public/MerchantPortal/index.php**

```php
<?php

use Pyz\Zed\MerchantPortalApplication\Communication\Bootstrap\MerchantPortalBootstrap;
use Spryker\Shared\Config\Application\Environment;
use Spryker\Shared\ErrorHandler\ErrorHandlerEnvironment;

define('APPLICATION', 'MERCHANT_PORTAL');
defined('APPLICATION_ROOT_DIR') || define('APPLICATION_ROOT_DIR', dirname(__DIR__, 2));

require_once APPLICATION_ROOT_DIR . '/vendor/autoload.php';

Environment::initialize();

$errorHandlerEnvironment = new ErrorHandlerEnvironment();
$errorHandlerEnvironment->initialize();

$bootstrap = new MerchantPortalBootstrap();
$bootstrap
    ->boot()
    ->run();
```

**public/MerchantPortal/maintenance/index.html**

```html
<!DOCTYPE html>
<html lang="en-US" xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>Spryker Merchant Portal - Maintenance</title>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
        <meta name="description" content="" />
        <meta name="keywords" content="" />
        <link href="http://fonts.googleapis.com/css?family=PT+Mono" rel="stylesheet" type="text/css" />
    </head>
    <style>
        body {
            font-family: 'PT Mono', sans-serif;
        }
        #so-doc {
            margin: 0 auto;
            width: 960px;
        }
    </style>
    <body>
        <div id="so-doc">
            <div>
                <pre>
                PAGE UNDER CONSTRUCTION!

                Come back in a few minutes...
                </pre>
            </div>
        </div>
    </body>
</html>
```

**public/MerchantPortal/maintenance/maintenance.php**

```php
<?php

/**
 * Copyright © 2017-present Spryker Systems GmbH. All rights reserved.
 * Use of this software requires acceptance of the Evaluation License Agreement. See the LICENSE file.
 */

if (file_exists(__DIR__ . '/maintenance.marker')) {
    http_response_code(503);
    echo file_get_contents(__DIR__ . '/index.html');
    exit(1);
}
```

**src/Pyz/Zed/ZedUi/Presentation/Components/app/app.module.ts**

```ts
import { HttpClientModule } from '@angular/common/http';
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';

import { DefaultMerchantPortalConfigModule, RootMerchantPortalModule } from '@mp/zed-ui';
import { DefaultTableConfigModule } from '@mp/gui-table';

@NgModule({
    imports: [
        BrowserModule,
        BrowserAnimationsModule,
        HttpClientModule,
        RootMerchantPortalModule,
        DefaultMerchantPortalConfigModule,
        DefaultTableConfigModule,
    ],
})
export class AppModule extends RootMerchantPortalModule {}
```

**src/Pyz/Zed/ZedUi/Presentation/Components/environments/environment.prod.ts**

```ts
export const environment = {
    production: true,
};
```

**src/Pyz/Zed/ZedUi/Presentation/Components/environments/environment.ts**

```ts
export const environment = {
    production: false,
};
```

**src/Pyz/Zed/ZedUi/Presentation/Components/index.html**

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="utf-8" />
        <title>ZedUi</title>
        <base href="/" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
    </head>
    <body></body>
</html>
```

**src/Pyz/Zed/ZedUi/Presentation/Components/main.ts**

```ts
import { enableProdMode } from '@angular/core';
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';

import { AppModule } from './app/app.module';
import { environment } from './environments/environment';

if (environment.production) {
    enableProdMode();
}

platformBrowserDynamic()
    .bootstrapModule(AppModule)
    /* tslint:disable-next-line: no-console */
    .catch((error) => console.error(error));
```

**src/Pyz/Zed/ZedUi/Presentation/Components/polyfills.ts**

```ts
import '@mp/polyfills';
```

{% info_block warningBox "Verification" %}

Ensure `yarn run mp:build` runs successfully. If it doesn't work, try the full rebuild:

`rm -rf node_modules && yarn cache clean --all && npm cache clean --force && yarn install && yarn mp:build`

{% endinfo_block %}

### 6) Adjust deployment configs

To configure deployment configuration to automatically install and build Merchant Portal, change frontend dependencies and installation commands in the deployment YAML:

1. Remove existing Yves dependencies' installation commands from deployment Yaml: `dependencies-install` and `yves-isntall-dependencies`.
2. Add required console commands:

**src/Pyz/Zed/Console/ConsoleDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\Console;

use Spryker\Zed\Console\ConsoleDependencyProvider as SprykerConsoleDependencyProvider;
use Spryker\Zed\SetupFrontend\Communication\Console\MerchantPortalBuildFrontendConsole;
use Spryker\Zed\SetupFrontend\Communication\Console\MerchantPortalInstallDependenciesConsole;

class ConsoleDependencyProvider extends SprykerConsoleDependencyProvider
{
    /**
     * @param \Spryker\Zed\Kernel\Container $container
     *
     * @return array<\Symfony\Component\Console\Command\Command>
     */
    protected function getConsoleCommands(Container $container): array
    {
        $commands = [
            new MerchantPortalInstallDependenciesConsole(),
            new MerchantPortalBuildFrontendConsole(),
        ];

        return $commands;
    }
}

```

3. Add the Merchant Portal installation command:

build-static:

 ```yaml
 merchant-portal-install-dependencies:
 command: 'console frontend:mp:install-dependencies | tail -100 && echo "Output trimmed, only last 100 lines shown."'
 ```

4. Add the Merchant Portal build command:
  - build-static-production:
    ```yaml
    merchant-portal-build-frontend:
      command: 'vendor/bin/console frontend:mp:build -e production'
      timeout: 1600
    ```

  - build-static-development:
    ```yaml
    merchant-portal-build-frontend:
      command: 'vendor/bin/console frontend:mp:build'
      timeout: 1600
    ```

### 7) Set up ACL behavior

Each feature/module with a persistent relation to the merchant must expand the ACL configuration. Based on your built-in features, you need to integrate the required plugins:

#### Integrate the ACL rule plugins

| PLUGIN                                                                             | SPECIFICATION                                                                         | PREREQUISITES | NAMESPACE                                                                                            |
|------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------|---------------|------------------------------------------------------------------------------------------------------|
| DashboardMerchantPortalGuiMerchantAclRuleExpanderPlugin                            | Adds `dashboard-merchant-portal-gui` to list of `AclRules`.                           | None          | Spryker\Zed\DashboardMerchantPortalGui\Communication\Plugin\AclMerchantPortal                        |
| MerchantProfileMerchantPortalGuiMerchantAclRuleExpanderPlugin                      | Adds `merchant-profile-merchant-portal-gui` to list of `AclRules`.                    | None          | Spryker\Zed\MerchantProfileMerchantPortalGui\Communication\Plugin\AclMerchantPortal                  |
| ProductOfferMerchantPortalGuiMerchantAclRuleExpanderPlugin                         | Adds `product-offer-merchant-portal-gui` to list of `AclRules`.                       | None          | Spryker\Zed\ProductOfferMerchantPortalGui\Communication\Plugin\AclMerchantPortal                     |
| ProductMerchantPortalGuiMerchantAclRuleExpanderPlugin                              | Adds `product-merchant-portal-gui` to list of `AclRules`.                             | None          | Spryker\Zed\ProductMerchantPortalGui\Communication\Plugin\AclMerchantPortal                          |
| SalesMerchantPortalGuiMerchantAclRuleExpanderPlugin                                | Adds `sales-merchant-portal-gui` to list of `AclRules`.                               | None          | Spryker\Zed\SalesMerchantPortalGui\Communication\Plugin\AclMerchantPortal                            |
| DummyMerchantPortalGuiMerchantAclRuleExpanderPlugin                                | Adds `dummy-merchant-portal-gui` to list of `AclRules`.                               | None          | Spryker\Zed\DummyMerchantPortalGui\Communication\Plugin\AclMerchantPortal                            |
| PriceProductMerchantRelationshipMerchantPortalGuiMerchantAclRuleExpanderPlugin     | Adds `price-product-merchant-relationship-merchant-portal-gui` to list of `AclRules`. | None          | Spryker\Zed\PriceProductMerchantRelationshipMerchantPortalGui\Communication\Plugin\AclMerchantPortal |
| SecurityMerchantPortalGuiMerchantUserAclRuleExpanderPlugin                         | Adds `security-merchant-portal-gui` to list of `AclRules`.                            | None          | Spryker\Zed\SecurityMerchantPortalGui\Communication\Plugin\AclMerchantPortal                         |
| UserMerchantPortalGuiMerchantUserAclRuleExpanderPlugin                             | Adds `user-merchant-portal-gui` to list of `AclRules`.                                | None          | Spryker\Zed\UserMerchantPortalGui\Communication\Plugin\AclMerchantPortal                             |
| MerchantUserMerchantUserAclEntityRuleExpanderPlugin                                | Expands set of `AclEntityRule` transfer objects with merchant user composite data.    | None          | Spryker\Zed\MerchantUser\Communication\Plugin\AclMerchantPortal                                      |


#### Integrate the ACL configuration plugins

| PLUGIN                                             | SPECIFICATION                                                                         | PREREQUISITES | NAMESPACE                                                                                            |
|----------------------------------------------------|---------------------------------------------------------------------------------------|---------------|------------------------------------------------------------------------------------------------------|
| AclEntityAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\AclEntity\Communication\Plugin\AclMerchantPortal                                         |
| AclEntityConfigurationExpanderPlugin    | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\\Communication\Plugin\AclMerchantPortal                                                  |
| AvailabilityAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\Availability\Communication\Plugin\AclMerchantPortal                                      |
| CategoryAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\Category\Communication\Plugin\AclMerchantPortal                                          |
| CategoryImageAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\CategoryImage\Communication\Plugin\AclMerchantPortal                                     |
| CmsBlockAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\CmsBlock\Communication\Plugin\AclMerchantPortal                                          |
| CommentAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\Comment\Communication\Plugin\AclMerchantPortal                                           |
| CompanyBusinessUnitAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\CompanyBusinessUnit\Communication\Plugin\AclMerchantPortal                               |
| CountryAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\Country\Communication\Plugin\AclMerchantPortal                                           |
| CurrencyAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\Currency\Communication\Plugin\AclMerchantPortal                                          |
| DiscountAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\Discount\Communication\Plugin\AclMerchantPortal                                          |
| DiscountPromotionAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\DiscountPromotion\Communication\Plugin\AclMerchantPortal                                 |
| GiftCardAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\GiftCard\Communication\Plugin\AclMerchantPortal                                          |
| GlossaryAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\Glossary\Communication\Plugin\AclMerchantPortal                                          |
| LocaleAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\Locale\Communication\Plugin\AclMerchantPortal                                            |
| MerchantAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\Merchant\Communication\Plugin\AclMerchantPortal                                          |
| MerchantCategoryAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\MerchantCategory\Communication\Plugin\AclMerchantPortal                                  |
| MerchantProductAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\MerchantProduct\Communication\Plugin\AclMerchantPortal                                   |
| MerchantProductOfferAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\MerchantProductOffer\Communication\Plugin\AclMerchantPortal                              |
| MerchantProfileAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\MerchantProfile\Communication\Plugin\AclMerchantPortal                                   |
| MerchantRelationshipAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\MerchantRelationship\Communication\Plugin\AclMerchantPortal                              |
| MerchantSalesOrderAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\MerchantSalesOrder\Communication\Plugin\AclMerchantPortal                                |
| MerchantStockAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\MerchantStock\Communication\Plugin\AclMerchantPortal                                     |
| MerchantUserAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\MerchantUser\Communication\Plugin\AclMerchantPortal                                      |
| OmsAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\Oms\Communication\Plugin\AclMerchantPortal                                               |
| OmsProductOfferReservationAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\OmsProductOfferReservation\Communication\Plugin\AclMerchantPortal                        |
| PaymentAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\Payment\Communication\Plugin\AclMerchantPortal                                           |
| PriceProductAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\PriceProduct\Communication\Plugin\AclMerchantPortal                                      |
| PriceProductMerchantRelationshipAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\PriceProductMerchantRelationship\Communication\Plugin\AclMerchantPortal                  |
| PriceProductOfferAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\PriceProductOffer\Communication\Plugin\AclMerchantPortal                                 |
| ProductAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\Product\Communication\Plugin\AclMerchantPortal                                           |
| ProductAttributeAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\ProductAttribute\Communication\Plugin\AclMerchantPortal                                  |
| ProductCategoryAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\ProductCategory\Communication\Plugin\AclMerchantPortal                                   |
| ProductImageAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\ProductImage\Communication\Plugin\AclMerchantPortal                                      |
| ProductOfferAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\ProductOffer\Communication\Plugin\AclMerchantPortal                                      |
| ProductOfferStockAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\ProductOfferStock\Communication\Plugin\AclMerchantPortal                                 |
| ProductOfferValidityAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\ProductOfferValidity\Communication\Plugin\AclMerchantPortal                              |
| ProductOptionAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\ProductOption\Communication\Plugin\AclMerchantPortal                                     |
| ProductSearchAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\ProductSearch\Communication\Plugin\AclMerchantPortal                                     |
| ProductValidityAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\ProductValidity\Communication\Plugin\AclMerchantPortal                                   |
| RefundAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\Refund\Communication\Plugin\AclMerchantPortal                                            |
| SalesAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\Sales\Communication\Plugin\AclMerchantPortal                                             |
| SalesInvoiceAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\SalesInvoice\Communication\Plugin\AclMerchantPortal                                      |
| SalesOrderThresholdAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\SalesOrderThreshold\Communication\Plugin\AclMerchantPortal                               |
| ShipmentAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\Shipment\Communication\Plugin\AclMerchantPortal                                          |
| StateMachineAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\StateMachine\Communication\Plugin\AclMerchantPortal                                      |
| StockAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\Stock\Communication\Plugin\AclMerchantPortal                                             |
| StoreAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\Store\Communication\Plugin\AclMerchantPortal                                             |
| TaxAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\Tax\Communication\Plugin\AclMerchantPortal                                               |
| UrlAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\Url\Communication\Plugin\AclMerchantPortal                                               |
| UserPasswordResetAclEntityConfigurationExpanderPlugin | Expands the provided `AclEntityMetadataConfig` transfer object with composite data.       | None          | Spryker\Zed\UserPasswordReset\Communication\Plugin\AclMerchantPortal                                 |

<details><summary markdown='span'>src/Pyz/Zed/AclMerchantPortal/AclMerchantPortalDependencyProvider.php</summary>

```php
<?php

namespace Pyz\Zed\AclMerchantPortal;

use Spryker\Zed\Acl\Communication\Plugin\AclMerchantPortal\AclEntityConfigurationExpanderPlugin;
use Spryker\Zed\AclEntity\Communication\Plugin\AclMerchantPortal\AclEntityAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\AclMerchantPortal\AclMerchantPortalDependencyProvider as SprykerAclMerchantPortalDependencyProvider;
use Spryker\Zed\Availability\Communication\Plugin\AclMerchantPortal\AvailabilityAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\Category\Communication\Plugin\AclMerchantPortal\CategoryAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\CategoryImage\Communication\Plugin\AclMerchantPortal\CategoryImageAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\CmsBlock\Communication\Plugin\AclMerchantPortal\CmsBlockAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\Comment\Communication\Plugin\AclMerchantPortal\CommentAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\CompanyBusinessUnit\Communication\Plugin\AclMerchantPortal\CompanyBusinessUnitAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\Country\Communication\Plugin\AclMerchantPortal\CountryAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\Country\Communication\Plugin\AclMerchantPortal\CountryMerchantAclEntityRuleExpanderPlugin;
use Spryker\Zed\Currency\Communication\Plugin\AclMerchantPortal\CurrencyAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\Currency\Communication\Plugin\AclMerchantPortal\CurrencyMerchantAclEntityRuleExpanderPlugin;
use Spryker\Zed\Customer\Communication\Plugin\AclMerchantPortal\CustomerMerchantAclEntityRuleExpanderPlugin;
use Spryker\Zed\DashboardMerchantPortalGui\Communication\Plugin\AclMerchantPortal\DashboardMerchantPortalGuiMerchantAclRuleExpanderPlugin;
use Spryker\Zed\Discount\Communication\Plugin\AclMerchantPortal\DiscountAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\Discount\Communication\Plugin\AclMerchantPortal\DiscountMerchantAclEntityRuleExpanderPlugin;
use Spryker\Zed\DiscountPromotion\Communication\Plugin\AclMerchantPortal\DiscountPromotionAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\DiscountPromotion\Communication\Plugin\AclMerchantPortal\DiscountPromotionMerchantAclEntityRuleExpanderPlugin;
use Spryker\Zed\DummyMerchantPortalGui\Communication\Plugin\AclMerchantPortal\DummyMerchantPortalGuiMerchantAclRuleExpanderPlugin;
use Spryker\Zed\GiftCard\Communication\Plugin\AclMerchantPortal\GiftCardAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\Glossary\Communication\Plugin\AclMerchantPortal\GlossaryAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\Locale\Communication\Plugin\AclMerchantPortal\LocaleAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\Locale\Communication\Plugin\AclMerchantPortal\LocaleMerchantAclEntityRuleExpanderPlugin;
use Spryker\Zed\Merchant\Communication\Plugin\AclMerchantPortal\MerchantAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\Merchant\Communication\Plugin\AclMerchantPortal\MerchantMerchantAclEntityRuleExpanderPlugin;
use Spryker\Zed\MerchantCategory\Communication\Plugin\AclMerchantPortal\MerchantCategoryAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\MerchantProduct\Communication\Plugin\AclMerchantPortal\MerchantProductAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\MerchantProduct\Communication\Plugin\AclMerchantPortal\MerchantProductMerchantAclEntityRuleExpanderPlugin;
use Spryker\Zed\MerchantProductOffer\Communication\Plugin\AclMerchantPortal\MerchantProductOfferAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\MerchantProfile\Communication\Plugin\AclMerchantPortal\MerchantProfileAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\MerchantProfileMerchantPortalGui\Communication\Plugin\AclMerchantPortal\MerchantProfileMerchantPortalGuiMerchantAclRuleExpanderPlugin;
use Spryker\Zed\MerchantRelationship\Communication\Plugin\AclMerchantPortal\MerchantRelationshipAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\MerchantSalesOrder\Communication\Plugin\AclMerchantPortal\MerchantSalesOrderAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\MerchantSalesOrder\Communication\Plugin\AclMerchantPortal\MerchantSalesOrderMerchantAclEntityRuleExpanderPlugin;
use Spryker\Zed\MerchantStock\Communication\Plugin\AclMerchantPortal\MerchantStockAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\MerchantUser\Communication\Plugin\AclMerchantPortal\MerchantUserAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\MerchantUser\Communication\Plugin\AclMerchantPortal\MerchantUserMerchantUserAclEntityRuleExpanderPlugin;
use Spryker\Zed\Oms\Communication\Plugin\AclMerchantPortal\OmsAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\Oms\Communication\Plugin\AclMerchantPortal\OmsMerchantAclEntityRuleExpanderPlugin;
use Spryker\Zed\OmsProductOfferReservation\Communication\Plugin\AclMerchantPortal\OmsProductOfferReservationAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\OmsProductOfferReservation\Communication\Plugin\AclMerchantPortal\OmsProductOfferReservationMerchantAclEntityRuleExpanderPlugin;
use Spryker\Zed\Payment\Communication\Plugin\AclMerchantPortal\PaymentAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\PriceProduct\Communication\Plugin\AclMerchantPortal\PriceProductAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\PriceProduct\Communication\Plugin\AclMerchantPortal\PriceProductMerchantAclEntityRuleExpanderPlugin;
use Spryker\Zed\PriceProductMerchantRelationship\Communication\Plugin\AclMerchantPortal\PriceProductMerchantRelationshipAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\PriceProductMerchantRelationshipMerchantPortalGui\Communication\Plugin\AclMerchantPortal\PriceProductMerchantRelationshipMerchantPortalGuiMerchantAclRuleExpanderPlugin;
use Spryker\Zed\PriceProductOffer\Communication\Plugin\AclMerchantPortal\PriceProductOfferAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\Product\Communication\Plugin\AclMerchantPortal\ProductAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\Product\Communication\Plugin\AclMerchantPortal\ProductMerchantAclEntityRuleExpanderPlugin;
use Spryker\Zed\ProductAttribute\Communication\Plugin\AclMerchantPortal\ProductAttributeAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\ProductCategory\Communication\Plugin\AclMerchantPortal\ProductCategoryAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\ProductImage\Communication\Plugin\AclMerchantPortal\ProductImageAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\ProductImage\Communication\Plugin\AclMerchantPortal\ProductImageMerchantAclEntityRuleExpanderPlugin;
use Spryker\Zed\ProductMerchantPortalGui\Communication\Plugin\AclMerchantPortal\ProductMerchantPortalGuiMerchantAclRuleExpanderPlugin;
use Spryker\Zed\ProductOffer\Communication\Plugin\AclMerchantPortal\ProductOfferAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\ProductOffer\Communication\Plugin\AclMerchantPortal\ProductOfferMerchantAclEntityRuleExpanderPlugin;
use Spryker\Zed\ProductOfferMerchantPortalGui\Communication\Plugin\AclMerchantPortal\ProductOfferMerchantPortalGuiMerchantAclRuleExpanderPlugin;
use Spryker\Zed\ProductOfferStock\Communication\Plugin\AclMerchantPortal\ProductOfferStockAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\ProductOfferValidity\Communication\Plugin\AclMerchantPortal\ProductOfferValidityAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\ProductOption\Communication\Plugin\AclMerchantPortal\ProductOptionAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\ProductSearch\Communication\Plugin\AclMerchantPortal\ProductSearchAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\ProductValidity\Communication\Plugin\AclMerchantPortal\ProductValidityAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\Refund\Communication\Plugin\AclMerchantPortal\RefundAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\Refund\Communication\Plugin\AclMerchantPortal\RefundMerchantAclEntityRuleExpanderPlugin;
use Spryker\Zed\Sales\Communication\Plugin\AclMerchantPortal\SalesAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\Sales\Communication\Plugin\AclMerchantPortal\SalesMerchantAclEntityRuleExpanderPlugin;
use Spryker\Zed\SalesInvoice\Communication\Plugin\AclMerchantPortal\SalesInvoiceAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\SalesMerchantPortalGui\Communication\Plugin\AclMerchantPortal\SalesMerchantPortalGuiMerchantAclRuleExpanderPlugin;
use Spryker\Zed\SalesOrderThreshold\Communication\Plugin\AclMerchantPortal\SalesOrderThresholdAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\SecurityMerchantPortalGui\Communication\Plugin\AclMerchantPortal\SecurityMerchantPortalGuiMerchantUserAclRuleExpanderPlugin;
use Spryker\Zed\Shipment\Communication\Plugin\AclMerchantPortal\ShipmentAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\StateMachine\Communication\Plugin\AclMerchantPortal\StateMachineAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\Stock\Communication\Plugin\AclMerchantPortal\StockAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\Store\Communication\Plugin\AclMerchantPortal\StoreAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\Store\Communication\Plugin\AclMerchantPortal\StoreMerchantAclEntityRuleExpanderPlugin;
use Spryker\Zed\Tax\Communication\Plugin\AclMerchantPortal\TaxAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\Url\Communication\Plugin\AclMerchantPortal\UrlAclEntityConfigurationExpanderPlugin;
use Spryker\Zed\Url\Communication\Plugin\AclMerchantPortal\UrlMerchantAclEntityRuleExpanderPlugin;
use Spryker\Zed\UserMerchantPortalGui\Communication\Plugin\AclMerchantPortal\UserMerchantPortalGuiMerchantUserAclRuleExpanderPlugin;
use Spryker\Zed\UserPasswordReset\Communication\Plugin\AclMerchantPortal\UserPasswordResetAclEntityConfigurationExpanderPlugin;

class AclMerchantPortalDependencyProvider extends SprykerAclMerchantPortalDependencyProvider
{
    /**
     * @return list<\Spryker\Zed\AclMerchantPortalExtension\Dependency\Plugin\MerchantAclRuleExpanderPluginInterface>
     */
    protected function getMerchantAclRuleExpanderPlugins(): array
    {
        return [
            new DashboardMerchantPortalGuiMerchantAclRuleExpanderPlugin(),
            new MerchantProfileMerchantPortalGuiMerchantAclRuleExpanderPlugin(),
            new ProductOfferMerchantPortalGuiMerchantAclRuleExpanderPlugin(),
            new ProductMerchantPortalGuiMerchantAclRuleExpanderPlugin(),
            new SalesMerchantPortalGuiMerchantAclRuleExpanderPlugin(),
            new DummyMerchantPortalGuiMerchantAclRuleExpanderPlugin(),
            new PriceProductMerchantRelationshipMerchantPortalGuiMerchantAclRuleExpanderPlugin(),
        ];
    }

    /**
     * @return list<\Spryker\Zed\AclMerchantPortalExtension\Dependency\Plugin\MerchantAclEntityRuleExpanderPluginInterface>
     */
    protected function getMerchantAclEntityRuleExpanderPlugins(): array
    {
        return [
            new ProductOfferMerchantAclEntityRuleExpanderPlugin(),
            new ProductMerchantAclEntityRuleExpanderPlugin(),
            new MerchantMerchantAclEntityRuleExpanderPlugin(),
            new MerchantSalesOrderMerchantAclEntityRuleExpanderPlugin(),
            new MerchantProductMerchantAclEntityRuleExpanderPlugin(),
            new CurrencyMerchantAclEntityRuleExpanderPlugin(),
            new CountryMerchantAclEntityRuleExpanderPlugin(),
            new StoreMerchantAclEntityRuleExpanderPlugin(),
            new LocaleMerchantAclEntityRuleExpanderPlugin(),
            new SalesMerchantAclEntityRuleExpanderPlugin(),
            new PriceProductMerchantAclEntityRuleExpanderPlugin(),
            new ProductImageMerchantAclEntityRuleExpanderPlugin(),
            new OmsProductOfferReservationMerchantAclEntityRuleExpanderPlugin(),
            new OmsMerchantAclEntityRuleExpanderPlugin(),
            new UrlMerchantAclEntityRuleExpanderPlugin(),
            new RefundMerchantAclEntityRuleExpanderPlugin(),
            new CustomerMerchantAclEntityRuleExpanderPlugin(),
            new DiscountMerchantAclEntityRuleExpanderPlugin(),
            new DiscountPromotionMerchantAclEntityRuleExpanderPlugin(),
        ];
    }

    /**
     * @return list<\Spryker\Zed\AclMerchantPortalExtension\Dependency\Plugin\MerchantUserAclRuleExpanderPluginInterface>
     */
    protected function getMerchantUserAclRuleExpanderPlugins(): array
    {
        return [
            new SecurityMerchantPortalGuiMerchantUserAclRuleExpanderPlugin(),
            new UserMerchantPortalGuiMerchantUserAclRuleExpanderPlugin(),
        ];
    }

    /**
     * @return list<\Spryker\Zed\AclMerchantPortalExtension\Dependency\Plugin\MerchantUserAclEntityRuleExpanderPluginInterface>
     */
    protected function getMerchantUserAclEntityRuleExpanderPlugins(): array
    {
        return [
            new MerchantUserMerchantUserAclEntityRuleExpanderPlugin(),
        ];
    }

    /**
     * @return list<\Spryker\Zed\AclMerchantPortalExtension\Dependency\Plugin\AclEntityConfigurationExpanderPluginInterface>
     */
    protected function getAclEntityConfigurationExpanderPlugins(): array
    {
        return [
            new AclEntityAclEntityConfigurationExpanderPlugin(),
            new AclEntityConfigurationExpanderPlugin(),
            new AvailabilityAclEntityConfigurationExpanderPlugin(),
            new CategoryAclEntityConfigurationExpanderPlugin(),
            new CategoryImageAclEntityConfigurationExpanderPlugin(),
            new CmsBlockAclEntityConfigurationExpanderPlugin(),
            new CommentAclEntityConfigurationExpanderPlugin(),
            new CompanyBusinessUnitAclEntityConfigurationExpanderPlugin(),
            new CountryAclEntityConfigurationExpanderPlugin(),
            new CurrencyAclEntityConfigurationExpanderPlugin(),
            new DiscountAclEntityConfigurationExpanderPlugin(),
            new DiscountPromotionAclEntityConfigurationExpanderPlugin(),
            new GiftCardAclEntityConfigurationExpanderPlugin(),
            new GlossaryAclEntityConfigurationExpanderPlugin(),
            new LocaleAclEntityConfigurationExpanderPlugin(),
            new MerchantAclEntityConfigurationExpanderPlugin(),
            new MerchantCategoryAclEntityConfigurationExpanderPlugin(),
            new MerchantProductAclEntityConfigurationExpanderPlugin(),
            new MerchantProductOfferAclEntityConfigurationExpanderPlugin(),
            new MerchantProfileAclEntityConfigurationExpanderPlugin(),
            new MerchantRelationshipAclEntityConfigurationExpanderPlugin(),
            new MerchantSalesOrderAclEntityConfigurationExpanderPlugin(),
            new MerchantStockAclEntityConfigurationExpanderPlugin(),
            new MerchantUserAclEntityConfigurationExpanderPlugin(),
            new OmsAclEntityConfigurationExpanderPlugin(),
            new OmsProductOfferReservationAclEntityConfigurationExpanderPlugin(),
            new PaymentAclEntityConfigurationExpanderPlugin(),
            new PriceProductAclEntityConfigurationExpanderPlugin(),
            new PriceProductMerchantRelationshipAclEntityConfigurationExpanderPlugin(),
            new PriceProductOfferAclEntityConfigurationExpanderPlugin(),
            new ProductAclEntityConfigurationExpanderPlugin(),
            new ProductAttributeAclEntityConfigurationExpanderPlugin(),
            new ProductCategoryAclEntityConfigurationExpanderPlugin(),
            new ProductImageAclEntityConfigurationExpanderPlugin(),
            new ProductOfferAclEntityConfigurationExpanderPlugin(),
            new ProductOfferStockAclEntityConfigurationExpanderPlugin(),
            new ProductOfferValidityAclEntityConfigurationExpanderPlugin(),
            new ProductOptionAclEntityConfigurationExpanderPlugin(),
            new ProductSearchAclEntityConfigurationExpanderPlugin(),
            new ProductValidityAclEntityConfigurationExpanderPlugin(),
            new RefundAclEntityConfigurationExpanderPlugin(),
            new SalesAclEntityConfigurationExpanderPlugin(),
            new SalesInvoiceAclEntityConfigurationExpanderPlugin(),
            new SalesOrderThresholdAclEntityConfigurationExpanderPlugin(),
            new ShipmentAclEntityConfigurationExpanderPlugin(),
            new StateMachineAclEntityConfigurationExpanderPlugin(),
            new StockAclEntityConfigurationExpanderPlugin(),
            new StoreAclEntityConfigurationExpanderPlugin(),
            new TaxAclEntityConfigurationExpanderPlugin(),
            new UrlAclEntityConfigurationExpanderPlugin(),
            new UserPasswordResetAclEntityConfigurationExpanderPlugin(),
        ];
    }
}
```
</details>

{% info_block warningBox "Verification" %}

Make sure that all propel-related entities with the merchant have the allowed/restricted access level according to the ACL configuration.

{% endinfo_block %}

## Adjust environment infrastructure

It is not safe to expose MerchantPortal next to the Back Office—MerchantPortal *must not have* OS, DNS name, VirtualHost settings, FileSystem, and service credentials shared with Zed.

### 1) Set up a new virtual machine/docker container dedicated to MerchantPortal

MerchantPortal *must be* placed into its own private subnet.

MerchantPortal *must have* access to the following:

- Primary Database
- Message broker

MerchantPortal *must not have* access to the following:

- Search and Storage
- Gateway
- Scheduler

**deploy.dev.yml**
```yaml
...
groups:
  EU:
    region: EU
    applications:
      merchant_portal_eu:
        application: merchant-portal
        endpoints:
          mp.de.spryker.local:
            entry-point: MerchantPortal
            store: DE
            primal: true
            services:
              session:
                namespace: 7
          mp.at.spryker.local:
            entry-point: MerchantPortal
            store: AT
            services:
              session:
                namespace: 8
  US:
    region: US
    applications:
      merchant_portal_us:
        application: merchant-portal
        endpoints:
          mp.us.spryker.local:
            entry-point: MerchantPortal
            store: US
            services:
              session:
                namespace: 9
```

### 2) Create a dedicated database user

Grant only default CRUD operations—`INSERT`, `DELETE`, `UPDATE`, and `SELECT`. Do not grant `ALL PRIVILEGES`, `GRANT OPTION`, `DROP`, `CREATE`, and other admin-related grants.

The following code snippet example is for MySQL:

```mysql
CREATE USER 'merchantportal'@'localhost' IDENTIFIED BY '{your_merchantportal_password}'; // YOU MUST CHANGE THE PASSWORD.
GRANT SELECT, INSERT, UPDATE, DELETE ON your_app_schema.* TO 'merchantportal'@'localhost';
FLUSH PRIVILEGES;
```

### 3) Create a new Nginx web server configuration

The following is an example of an Nginx configuration:

**/etc/nginx/merchant-portal.conf**

```nginx
server {
    # { Your virtual host settings }

    # Allow /assets/js/mp assets to be served only
    location ~ (/assets/js/mp|/favicon.ico|/robots.txt) {
        access_log        off;
        expires           30d;
        add_header Pragma public;
        add_header Cache-Control "public, must-revalidate, proxy-revalidate";
        try_files $uri =404;
    }

   # Allow /marchant-portal-gui pages to be served only
   location ~ ^/[a-z-]+-merchant-portal-gui {
        add_header X-Server $hostname;
        fastcgi_pass { YOUR_FASTCGI_PASS };
        fastcgi_index index.php;
        include /etc/nginx/fastcgi_params;
        fastcgi_param SCRIPT_NAME /index.php;
        fastcgi_param APPLICATION_ENV $application_env;
        fastcgi_param APPLICATION_STORE $application_store;
        fastcgi_param SCRIPT_FILENAME  $document_root/index.php;

        # Credentials of the newly created DB user.
        fastcgi_param SPRYKER_DB_USERNAME merchantportal;
        fastcgi_param SPRYKER_DB_PASSWORD '{your_merchantportal_password}';


        more_clear_headers 'X-Powered-By' 'X-Store' 'X-Locale' 'X-Env' 'Server';
    }
}
```

After modifying the Nginx config, apply the new `config:f`.

```bash
sudo service nginx reload
```

{% info_block warningBox "Verification" %}

Make sure to use environment variables in `config-default.php`:

**config/Shared/config_default.php**

```php
<?php

// other code

$config[PropelConstants::ZED_DB_USERNAME] = getenv('SPRYKER_DB_USERNAME');
$config[PropelConstants::ZED_DB_PASSWORD] = getenv('SPRYKER_DB_PASSWORD');
```

{% endinfo_block %}

The following page now shows the login page for MerchantPortal: `https://your-merchant-portal.domain/security-merchant-portal-gui/login`.

{% info_block warningBox "Verification" %}

Make sure the following pages do not open: `https://your-merchant-portal.domain/security-gui/login`, `https://your-merchant-portal.domain/`.

{% endinfo_block %}

### 4) Register modules in ACL

Add new modules to installer rules:

**src/Pyz/Zed/Acl/AclConfig.php**

```php
<?php

namespace Pyz\Zed\Acl;

use Spryker\Shared\Acl\AclConstants;
use Spryker\Zed\Acl\AclConfig as SprykerAclConfig;

class AclConfig extends SprykerAclConfig
{
    /**
     * @param array<array<string>> $installerRules
     *
     * @return array<array<string>>
     */
    protected function addMerchantPortalInstallerRules(array $installerRules): array
    {
        $bundleNames = [
            'user-merchant-portal-gui',
            'dashboard-merchant-portal-gui',
            'security-merchant-portal-gui',
        ];

        foreach ($bundleNames as $bundleName) {
            $array<installerRules> = [
                'bundle' => $bundleName,
                'controller' => AclConstants::VALIDATOR_WILDCARD,
                'action' => AclConstants::VALIDATOR_WILDCARD,
                'type' => static::RULE_TYPE_DENY,
                'role' => AclConstants::ROOT_ROLE,
            ];
        }

        return $installerRules;
    }
}

```

{% info_block warningBox "Verification" %}

Make sure that after executing `console setup:init-db`, the `user-merchant-portal-gui` rule is present in the `spy_acl_rule` table.

{% endinfo_block %}

### 5) Update navigation

1. Add the `My Account` and `Logout` sections to `navigation-secondary.xml`:

**config/Zed/navigation-secondary.xml**

```xml
<?xml version="1.0"?>
<config>
    <my-account>
        <label>My Account</label>
        <title>My Account</title>
        <bundle>user-merchant-portal-gui</bundle>
        <controller>my-account</controller>
        <action>index</action>
    </my-account>
    <logout>
        <label>Logout</label>
        <title>Logout</title>
        <bundle>security-merchant-portal-gui</bundle>
        <controller>logout</controller>
        <action>index</action>
        <type>danger</type>
    </logout>
</config>
```

2. Update navigation catche

```bash
console navigation:build-cache
```

{% info_block warningBox "Verification" %}

Log in to the Merchant Portal and make sure that when clicking on the profile picture, the **My Account** and **Logout** buttons are visible in the overlay of the secondary navigation.

{% endinfo_block %}

## Related features

Integrate the following related features:

| FEATURE | REQUIRED FOR THE CURRENT FEATURE | INTEGRATION GUIDE |
| - | - | -|
| Merchant Portal | &check;  |  [Merchant Portal feature integration ](/docs/marketplace/dev/feature-integration-guides/{{page.version}}/merchant-portal-feature-integration.html) |

