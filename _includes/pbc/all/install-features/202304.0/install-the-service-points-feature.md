

This document describes how to integrate the Service Points feature into a Spryker project.

## Install feature core

Follow the steps below to install the Service Points feature core.

### Prerequisites

To start feature integration, integrate the required features:

| NAME         | VERSION          | INTEGRATION GUIDE                                                                                                                    |
|--------------|------------------|--------------------------------------------------------------------------------------------------------------------------------------|
| Spryker Core | {{page.version}} | [Spryker Core feature integration](/docs/pbc/all/miscellaneous/{{page.version}}/install-and-upgrade/install-features/install-the-spryker-core-feature.html) |  |

### 1) Install the required modules using Composer

```bash
composer require spryker-feature/service-points: "{{page.version}}" --update-with-dependencies
```

{% info_block warningBox "Verification" %}

Make sure that the following modules have been installed:

| MODULE                             | EXPECTED DIRECTORY                                     |
|------------------------------------|--------------------------------------------------------|
| ProductOfferServicePoint           | vendor/spryker/product-offer-service-point             |
| ProductOfferServicePointDataImport | vendor/spryker/product-offer-service-point-data-import |
| ServicePoint                       | vendor/spryker/service-point                           |
| ServicePointDataImport             | vendor/spryker/service-point-data-import               |
| ServicePointsBackendApi            | vendor/spryker/service-points-backend-api              |
| ServicePointSearch                 | vendor/spryker/service-point-search                    |
| ServicePointStorage                | vendor/spryker/service-point-storage                   |

{% endinfo_block %}

## 2) Set up database schema and transfer objects

1. Adjust the schema definition so entity changes trigger events.

| AFFECTED ENTITY               | TRIGGERED EVENTS                                                                                                              |
|-------------------------------|-------------------------------------------------------------------------------------------------------------------------------|
| spy_service_point             | Entity.spy_service_point.create<br>Entity.spy_service_point.update<br>Entity.spy_service_point.delete                         |
| spy_service_point_address     | Entity.spy_service_point_address.create<br>Entity.spy_service_point_address.update<br>Entity.spy_service_point_address.delete |
| spy_service_point_store       | Entity.spy_service_point_store.create<br>Entity.spy_service_point_store.update<br>Entity.spy_service_point_store.delete       |

**src/Pyz/Zed/ServicePoint/Persistence/Propel/Schema/spy_service_point.schema.xml**

```xml
<?xml version="1.0"?>
<database xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" name="zed" xsi:noNamespaceSchemaLocation="http://static.spryker.com/schema-01.xsd" namespace="Orm\Zed\ServicePoint\Persistence" package="src.Orm.Zed.ServicePoint.Persistence">

   <table name="spy_service_point">
      <behavior name="event">
         <parameter name="spy_service_point_all" column="*"/>
      </behavior>
   </table>

   <table name="spy_service_point_address">
      <behavior name="event">
         <parameter name="spy_service_point_address_all" column="*"/>
      </behavior>
   </table>

   <table name="spy_service_point_store">
      <behavior name="event">
         <parameter name="spy_service_point_store_all" column="*"/>
      </behavior>
   </table>

   <table name="spy_service">
      <behavior name="event">
         <parameter name="spy_service_all" column="*"/>
      </behavior>
   </table>

</database>
```

2. Apply database changes and generate transfer changes:

```bash
console transfer:generate
console propel:install
console transfer:entity:generate
console frontend:zed:build
```

{% info_block warningBox "Verification" %}

Make sure that the following changes have been applied in the database:

| DATABASE ENTITY           | TYPE   | EVENT   |
|---------------------------|--------|---------|
| spy_service_point         | table  | created |
| spy_service_point_address | table  | created |
| spy_service               | table  | created |
| spy_service_point_store   | table  | created |
| spy_service_point_search  | table  | created |
| spy_service_point_storage | table  | created |
| spy_service_type          | table  | created |
| spy_product_offer_service | table  | created |
| spy_region.uuid           | column | created |

Make sure that propel entities have been generated successfully by checking their existence. Also, make generated entity classes extending respective Spryker core classes.

| CLASS NAMESPACE                                                      | EXTENDS                                                                                 |
|----------------------------------------------------------------------|-----------------------------------------------------------------------------------------|
| \Orm\Zed\ServicePoint\Persistence\SpyServicePoint                    | \Spryker\Zed\ServicePoint\Persistence\Propel\AbstractSpyServicePoint                    |
| \Orm\Zed\ServicePoint\Persistence\SpyServicePointAddress             | \Spryker\Zed\ServicePoint\Persistence\Propel\AbstractSpyServicePointAddress             |
| \Orm\Zed\ServicePoint\Persistence\SpyServicePointAddressQuery        | \Spryker\Zed\ServicePoint\Persistence\Propel\AbstractSpyServicePointAddressQuery        |
| \Orm\Zed\ServicePoint\Persistence\SpyServicePointQuery               | \Spryker\Zed\ServicePoint\Persistence\Propel\AbstractSpyServicePointQuery               |
| \Orm\Zed\ServicePoint\Persistence\SpyService                         | \Spryker\Zed\ServicePoint\Persistence\Propel\AbstractSpyService                         |
| \Orm\Zed\ServicePoint\Persistence\SpyServiceQuery                    | \Spryker\Zed\ServicePoint\Persistence\Propel\AbstractSpyServiceQuery                    |
| \Orm\Zed\ServicePoint\Persistence\SpyServicePointAddressQuery        | \Spryker\Zed\ServicePoint\Persistence\Propel\AbstractSpyServicePointAddressQuery        |
| \Orm\Zed\ServicePoint\Persistence\SpyServicePointStore               | \Spryker\Zed\ServicePoint\Persistence\Propel\AbstractSpyServicePointStore               |
| \Orm\Zed\ServicePoint\Persistence\SpyServicePointStoreQuery          | \Spryker\Zed\ServicePoint\Persistence\Propel\AbstractSpyServicePointStoreQuery          |
| \Orm\Zed\ServicePoint\Persistence\SpyServiceType                     | \Spryker\Zed\ServicePoint\Persistence\Propel\AbstractSpyServiceType                     |
| \Orm\Zed\ServicePoint\Persistence\SpyServiceTypeQuery                | \Spryker\Zed\ServicePoint\Persistence\Propel\AbstractSpyServiceTypeQuery                |
| \Orm\Zed\ServicePointSearch\Persistence\SpyServicePointSearch        | \Spryker\Zed\ServicePointSearch\Persistence\Propel\AbstractSpyServicePointSearch        |
| \Orm\Zed\ServicePointSearch\Persistence\SpyServicePointSearchQuery   | \Spryker\Zed\ServicePointSearch\Persistence\Propel\AbstractSpyServicePointSearchQuery   |
| \Orm\Zed\ServicePointStorage\Persistence\SpyServicePointStorage      | \Spryker\Zed\ServicePointStorage\Persistence\Propel\AbstractSpyServicePointStorage      |
| \Orm\Zed\ServicePointStorage\Persistence\SpyServicePointStorageQuery | \Spryker\Zed\ServicePointStorage\Persistence\Propel\AbstractSpyServicePointStorageQuery |
| \Orm\Zed\ProductOfferServicePoint\Persistence\SpyProductOfferService      | \Spryker\Zed\ProductOfferServicePoint\Persistence\Propel\AbstractSpyProductOfferService      |
| \Orm\Zed\ProductOfferServicePoint\Persistence\SpyProductOfferServiceQuery | \Spryker\Zed\ProductOfferServicePoint\Persistence\Propel\AbstractSpyProductOfferServiceQuery |

Make sure that the following changes have been applied in transfer objects:

| TRANSFER                               | TYPE  | EVENT   | PATH                                                                        |
|----------------------------------------|-------|---------|-----------------------------------------------------------------------------|
| ServicePoint                           | class | created | src/Generated/Shared/Transfer/ServicePointTransfer                          |
| ServicePointCollection                 | class | created | src/Generated/Shared/Transfer/ServicePointCollectionTransfer                |
| ServicePointCollectionRequest          | class | created | src/Generated/Shared/Transfer/ServicePointCollectionRequestTransfer         |
| ServicePointCollectionResponse         | class | created | src/Generated/Shared/Transfer/ServicePointCollectionResponseTransfer        |
| ServicePointCriteria                   | class | created | src/Generated/Shared/Transfer/ServicePointCriteriaTransfer                  |
| ServicePointConditions                 | class | created | src/Generated/Shared/Transfer/ServicePointConditionsTransfer                |
| ApiServicePointsAttributes             | class | created | src/Generated/Shared/Transfer/ApiServicePointsAttributesTransfer            |
| ApiServiceTypesAttributes              | class | created | src/Generated/Shared/Transfer/ApiServiceTypesAttributesTransfer             |
| ApiServicesAttributes                  | class | created | src/Generated/Shared/Transfer/ApiServicesAttributesTransfer                 |
| ApiServicesRequestAttributes           | class | created | src/Generated/Shared/Transfer/ApiServicesRequestAttributesTransfer          |
| ApiServicePointAddressesAttributes     | class | created | src/Generated/Shared/Transfer/ApiServicePointAddressesAttributesTransfer    |
| StoreRelation                          | class | created | src/Generated/Shared/Transfer/StoreRelationTransfer                         |
| Store                                  | class | created | src/Generated/Shared/Transfer/StoreTransfer                                 |
| Error                                  | class | created | src/Generated/Shared/Transfer/ErrorTransfer                                 |
| Sort                                   | class | created | src/Generated/Shared/Transfer/SortTransfer                                  |
| Pagination                             | class | created | src/Generated/Shared/Transfer/PaginationTransfer                            |
| ErrorCollection                        | class | created | src/Generated/Shared/Transfer/ErrorCollectionTransfer                       |
| DataImporterConfiguration              | class | created | src/Generated/Shared/Transfer/DataImporterConfigurationTransfer             |
| DataImporterReport                     | class | created | src/Generated/Shared/Transfer/DataImporterReportTransfer                    |
| CountryCriteria                        | class | created | src/Generated/Shared/Transfer/CountryCriteriaTransfer                       |
| CountryConditions                      | class | created | src/Generated/Shared/Transfer/CountryConditionsTransfer                     |
| Country                                | class | created | src/Generated/Shared/Transfer/CountryTransfer                               |
| CountryCollection                      | class | created | src/Generated/Shared/Transfer/CountryCollectionTransfer                     |
| Region                                 | class | created | src/Generated/Shared/Transfer/RegionTransfer                                |
| ServicePointAddressCollection          | class | created | src/Generated/Shared/Transfer/ServicePointAddressCollectionTransfer         |
| ServicePointAddressCollectionRequest   | class | created | src/Generated/Shared/Transfer/ServicePointAddressCollectionRequestTransfer  |
| ServicePointAddressCollectionResponse  | class | created | src/Generated/Shared/Transfer/ServicePointAddressCollectionResponseTransfer |
| ServicePointAddressCriteria            | class | created | src/Generated/Shared/Transfer/ServicePointAddressCriteriaTransfer           |
| ServicePointAddressConditions          | class | created | src/Generated/Shared/Transfer/ServicePointAddressConditionsTransfer         |
| ServicePointAddress                    | class | created | src/Generated/Shared/Transfer/ServicePointAddressTransfer                   |
| GlueRelationship                       | class | created | src/Generated/Shared/Transfer/GlueRelationshipTransfer                      |
| ServicePointSearchCollection           | class | created | src/Generated/Shared/Transfer/ServicePointSearchCollectionTransfer          |
| ServicePointSearch                     | class | created | src/Generated/Shared/Transfer/ServicePointSearchTransfer                    |
| ServicePointSearchRequest              | class | created | src/Generated/Shared/Transfer/ServicePointSearchRequestTransfer             |
| ServiceCollectionRequest               | class | created | src/Generated/Shared/Transfer/ServiceCollectionRequestTransfer              |
| ServiceCollectionResponse              | class | created | src/Generated/Shared/Transfer/ServiceCollectionResponseTransfer             |
| ServiceCollection                      | class | created | src/Generated/Shared/Transfer/ServiceCollectionTransfer                     |
| ServiceConditions                      | class | created | src/Generated/Shared/Transfer/ServiceConditionsTransfer                     |
| ServiceCriteria                        | class | created | src/Generated/Shared/Transfer/ServiceCriteriaTransfer                       |
| Service                                | class | created | src/Generated/Shared/Transfer/ServiceTransfer                               |
| ServiceTypeCollectionRequest           | class | created | src/Generated/Shared/Transfer/ServiceTypeCollectionRequestTransfer          |
| ServiceTypeCollectionResponse          | class | created | src/Generated/Shared/Transfer/ServiceTypeCollectionResponseTransfer         |
| ServiceTypeCollection                  | class | created | src/Generated/Shared/Transfer/ServiceTypeCollectionTransfer                 |
| ServiceTypeConditions                  | class | created | src/Generated/Shared/Transfer/ServiceTypeConditionsTransfer                 |
| ServiceTypeCriteria                    | class | created | src/Generated/Shared/Transfer/ServiceTypeCriteriaTransfer                   |
| ServiceType                            | class | created | src/Generated/Shared/Transfer/ServiceTypeTransfer                           |
| ServicePointStorage                    | class | created | src/Generated/Shared/Transfer/ServicePointStorageTransfer                   |
| ServicePointAddressStorage             | class | created | src/Generated/Shared/Transfer/ServicePointAddressStorageTransfer            |
| ProductOfferService                    | class | created | src/Generated/Shared/Transfer/ProductOfferServiceTransfer                   |
| ProductOfferServiceCollection          | class | created | src/Generated/Shared/Transfer/ProductOfferServiceCollectionTransfer         |
| ProductOfferServiceCollectionRequest   | class | created | src/Generated/Shared/Transfer/ProductOfferServiceCollectionRequestTransfer  |
| ProductOfferServiceCollectionResponse  | class | created | src/Generated/Shared/Transfer/ProductOfferServiceCollectionResponseTransfer |
| CountryStorage                         | class | created | src/Generated/Shared/Transfer/CountryStorageTransfer                        |
| RegionStorage                          | class | created | src/Generated/Shared/Transfer/RegionStorageTransfer                         |
| ServicePointStorageCollection          | class | created | src/Generated/Shared/Transfer/ServicePointStorageCollectionTransfer         |
| ServicePointStorageCriteria            | class | created | src/Generated/Shared/Transfer/ServicePointStorageCriteriaTransfer           |
| ServicePointStorageConditions          | class | created | src/Generated/Shared/Transfer/ServicePointStorageConditionsTransfer         |
| SynchronizationData                    | class | created | src/Generated/Shared/Transfer/SynchronizationDataTransfer                   |
| Filter                                 | class | created | src/Generated/Shared/Transfer/FilterTransfer                                |

{% endinfo_block %}

### 3) Set up configuration

To make the `service-points`, `service-point-addresses`, `services`, and `service-types` resources protected, adjust the protected paths configuration:

**src/Pyz/Shared/GlueBackendApiApplicationAuthorizationConnector/GlueBackendApiApplicationAuthorizationConnectorConfig.php**

```php
<?php

namespace Pyz\Shared\GlueBackendApiApplicationAuthorizationConnector;

use Spryker\Shared\GlueBackendApiApplicationAuthorizationConnector\GlueBackendApiApplicationAuthorizationConnectorConfig as SprykerGlueBackendApiApplicationAuthorizationConnectorConfig;

class GlueBackendApiApplicationAuthorizationConnectorConfig extends SprykerGlueBackendApiApplicationAuthorizationConnectorConfig
{
    /**
     * @return array<string, mixed>
     */
    public function getProtectedPaths(): array
    {
        return [
            // ...
            '/\/service-points.*/' => [
                'isRegularExpression' => true,
            ],
            '/\/services.*/' => [
                'isRegularExpression' => true,
            ],
            '/\/service-types.*/' => [
                'isRegularExpression' => true,
            ],     
        ];
    }
}
```

### 4) Import service points

1. Prepare your data according to your requirements using our demo data:

**data/import/common/common/service_point.csv**

```csv
key,name,is_active
sp1,Spryker Main Store,1
sp2,Spryker Berlin Store,1
```

| COLUMN    | REQUIRED? | DATA TYPE | DATA EXAMPLE       | DATA EXPLANATION                        |
|-----------|-----------|-----------|--------------------|-----------------------------------------|
| key       | mandatory | string    | sp1                | Unique key of the service point.        |
| name      | mandatory | string    | Spryker Main Store | Name of the service point.              |
| is_active | mandatory | bool      | 0                  | Defines if the service point is active. |

**data/import/common/{{store}}/service_point_store.csv**

```csv
service_point_key,store_name
sp1,DE
sp2,DE
```

| COLUMN            | REQUIRED? | DATA TYPE | DATA EXAMPLE | DATA EXPLANATION                        |
|-------------------|-----------|-----------|--------------|-----------------------------------------|
| service_point_key | mandatory | string    | sp1          | Unique key of the service point.        |
| store_name        | mandatory | string    | DE           | Name of the store to make relation for. |

**data/import/common/common/service_point_address.csv**

```csv
service_point_key,region_iso2_code,country_iso2_code,address1,address2,address3,city,zip_code
sp1,,DE,Caroline-Michaelis-Straße,8,,Berlin,10115
sp2,,DE,Julie-Wolfthorn-Straße,1,,Berlin,10115
```

| COLUMN            | REQUIRED? | DATA TYPE | DATA EXAMPLE              | DATA EXPLANATION                 |
|-------------------|-----------|-----------|---------------------------|----------------------------------|
| service_point_key | mandatory | string    | sp1                       | Unique key of the service point. |
| region_iso2_code  | optional  | string    | DE-BE                     | Region ISO2 code                 |
| country_iso2_code | mandatory | string    | DE                        | Country ISO2 code                |
| address1          | mandatory | string    | Caroline-Michaelis-Straße | First line of address            |
| address2          | mandatory | string    | 8a                        | Second line of address           |
| address3          | optional  | string    | 12/1                      | Third line of address            |
| city              | mandatory | string    | Berlin                    | City                             |
| zip_code          | mandatory | string    | 10115                     | Zip code                         |

**data/import/common/common/service_type.csv**

```csv
name,key
Pickup,pickup
```

| COLUMN | REQUIRED? | DATA TYPE | DATA EXAMPLE | DATA EXPLANATION                  |
|--------|-----------|-----------|--------------|-----------------------------------|
| name   | mandatory | string    | Pickup       | Unique key of the service type.   |
| key    | mandatory | string    | pickup       | Unique name of the service type.  |

**data/import/common/common/service.csv**

```csv
key,service_point_key,service_type_key,is_active
s1,sp1,pickup,1
s2,sp2,pickup,1
```

| COLUMN            | REQUIRED? | DATA TYPE | DATA EXAMPLE      | DATA EXPLANATION                  |
|-------------------|-----------|-----------|-------------------|-----------------------------------|
| key               | mandatory | string    | sps1              | Unique key of the service.        |
| service_point_key | mandatory | string    | sp1               | Unique key of the service point.  |
| service_type_key  | mandatory | string    | pickup            | Unique key of the service type.   |
| is_active         | mandatory | bool      | 0                 | Defines if the service is active. |

**data/import/common/common/product_offer_service.csv**

```csv
product_offer_reference,service_key
offer419,s1
offer420,s1
offer421,s1
offer422,s1
offer423,s1
offer424,s1
```

| COLUMN                  | REQUIRED? | DATA TYPE | DATA EXAMPLE | DATA EXPLANATION                       |
|-------------------------|-----------|-----------|--------------|----------------------------------------|
| product_offer_reference | mandatory | string    | offer419     | Unique reference of the product offer. |
| service_key             | mandatory | string    | s1           | Unique key of the service.             |

2. Enable data imports at your configuration file—for example:

**data/import/local/full_EU.yml**

```yml
    - data_entity: service-point
      source: data/import/common/common/service_point.csv
    - data_entity: service-point-store
      source: data/import/common/{{store}}/service_point_store.csv
    - data_entity: service-point-address
      source: data/import/common/common/service_point_address.csv
    - data_entity: service-type
      source: data/import/common/common/service_type.csv
    - data_entity: service
      source: data/import/common/common/service.csv
    - data_entity: product-offer-service
      source: data/import/common/common/marketplace/product_offer_service.csv
```

3. Register the following data import plugins:

| PLUGIN                              | SPECIFICATION                                                 | PREREQUISITES | NAMESPACE                                                                       |
|-------------------------------------|---------------------------------------------------------------|---------------|---------------------------------------------------------------------------------|
| ServicePointDataImportPlugin        | Imports service points data into the database.                | None          | \Spryker\Zed\ServicePointDataImport\Communication\Plugin\DataImport             |
| ServicePointStoreDataImportPlugin   | Imports service point store relations data into the database. | None          | \Spryker\Zed\ServicePointDataImport\Communication\Plugin\DataImport             |
| ServicePointAddressDataImportPlugin | Imports service point addresses into the database.            | None          | \Spryker\Zed\ServicePointDataImport\Communication\Plugin\DataImport             |
| ServiceTypeDataImportPlugin         | Imports service types into the database.                      | None          | \Spryker\Zed\ServicePointDataImport\Communication\Plugin\DataImport             |
| ServiceDataImportPlugin             | Imports services into the database.                           | None          | \Spryker\Zed\ServicePointDataImport\Communication\Plugin\DataImport             |
| ProductOfferServiceDataImportPlugin | Imports product offer services into the database.             | None          | \Spryker\Zed\ProductOfferServicePointDataImport\Communication\Plugin\DataImport |

**src/Pyz/Zed/DataImport/DataImportDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\DataImport;

use Spryker\Zed\DataImport\DataImportDependencyProvider as SprykerDataImportDependencyProvider;
use Spryker\Zed\ProductOfferServicePointDataImport\Communication\Plugin\DataImport\ProductOfferServiceDataImportPlugin;
use Spryker\Zed\ServicePointDataImport\Communication\Plugin\DataImport\ServicePointDataImportPlugin;
use Spryker\Zed\ServicePointDataImport\Communication\Plugin\DataImport\ServicePointStoreDataImportPlugin;
use Spryker\Zed\ServicePointDataImport\Communication\Plugin\DataImport\ServicePointAddressDataImportPlugin;

class DataImportDependencyProvider extends SprykerDataImportDependencyProvider
{
    /**
     * @return array<\Spryker\Zed\DataImport\Dependency\Plugin\DataImportPluginInterface>
     */
    protected function getDataImporterPlugins(): array
    {
        return [
            new ServicePointDataImportPlugin(),
            new ServicePointStoreDataImportPlugin(),
            new ServicePointAddressDataImportPlugin(),
            new ServiceTypeDataImportPlugin(),
            new ServiceDataImportPlugin(),
            new ProductOfferServiceDataImportPlugin(),
        ];
    }
}
```

4. Enable the behaviors by registering the console commands:

**src/Pyz/Zed/Console/ConsoleDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\Console;

use Spryker\Zed\Kernel\Container;
use Spryker\Zed\Console\ConsoleDependencyProvider as SprykerConsoleDependencyProvider;
use Spryker\Zed\DataImport\Communication\Console\DataImportConsole;
use Spryker\Zed\ProductOfferServicePointDataImport\ProductOfferServicePointDataImportConfig;
use Spryker\Zed\ServicePointDataImport\ServicePointDataImportConfig;

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
            // ...
            new DataImportConsole(DataImportConsole::DEFAULT_NAME . static::COMMAND_SEPARATOR . ServicePointDataImportConfig::IMPORT_TYPE_SERVICE_POINT),
            new DataImportConsole(DataImportConsole::DEFAULT_NAME . static::COMMAND_SEPARATOR . ServicePointDataImportConfig::IMPORT_TYPE_SERVICE_POINT_STORE),
            new DataImportConsole(DataImportConsole::DEFAULT_NAME . static::COMMAND_SEPARATOR . ServicePointDataImportConfig::IMPORT_TYPE_SERVICE_POINT_ADDRESS),
            new DataImportConsole(DataImportConsole::DEFAULT_NAME . static::COMMAND_SEPARATOR . ServicePointDataImportConfig::IMPORT_TYPE_SERVICE_TYPE),
            new DataImportConsole(DataImportConsole::DEFAULT_NAME . static::COMMAND_SEPARATOR . ServicePointDataImportConfig::IMPORT_TYPE_SERVICE),
            new DataImportConsole(DataImportConsole::DEFAULT_NAME . static::COMMAND_SEPARATOR . ProductOfferServicePointDataImportConfig::IMPORT_TYPE_PRODUCT_OFFER_SERVICE),
        ];

        return $commands;
    }
}
```

5. Import data:

```bash
console data:import service-point
console data:import service-point-address
console data:import:service
console data:import service-point-store
console data:import:service-type
console data:import:product-offer-service
```

{% info_block warningBox “Verification” %}

Make sure that entities were imported to the following database tables respectively:

- `spy_service_point`
- `spy_service_point_store`
- `spy_service_point_address`
- `spy_service_type`
- `spy_service`
- `spy_product_offer_service`

{% endinfo_block %}

### 5) Add translations

1. Append the glossary according to your configuration:

```csv
service_point.validation.service_point_key_exists,A service point with the same key already exists.,en_US
service_point.validation.service_point_key_exists,Es existiert bereits eine Servicestelle mit dem gleichen Schlüssel.,de_DE
service_point.validation.service_point_key_is_not_unique,A service point with the same key already exists in request.,en_US
service_point.validation.service_point_key_is_not_unique,Es existiert bereits eine Servicestelle mit dem gleichen Schlüssel in einer Abfrage.,de_DE
service_point.validation.service_point_key_wrong_length,A service point key must have length from %min% to %max% characters.,en_US
service_point.validation.service_point_key_wrong_length,Ein Servicestellen-Schlüssel muss eine Länge von %min% bis %max% Zeichen haben.,de_DE
service_point.validation.service_point_name_wrong_length,A service point name must have length from %min% to %max% characters.,en_US
service_point.validation.service_point_name_wrong_length,Ein Servicestellen-Name muss eine Länge von %min% bis %max% Zeichen haben.,de_DE
service_point.validation.store_does_not_exist,Store with name '%name%' does not exist.,en_US
service_point.validation.store_does_not_exist,Store mit dem Namen '%name%' existiert nicht.,de_DE
service_point.validation.service_point_entity_not_found,Service point entity was not found.,en_US
service_point.validation.service_point_entity_not_found,Servicestelle wurde nicht gefunden.,de_DE
service_point.validation.wrong_request_body,Wrong request body.,en_US
service_point.validation.wrong_request_body,Falscher Anforderungstext.,de_DE
service_point.validation.country_entity_not_found,Country with iso2 code '%iso2Code%' does not exist.,en_US
service_point.validation.country_entity_not_found,Das Land mit dem iso2-Code '%iso2Code%' existiert nicht.,de_DE
service_point.validation.region_entity_not_found,Region with uuid '%uuid%' does not exist for country with iso2 code '%countryIso2Code%'.,en_US
service_point.validation.region_entity_not_found,Region mit uuid '%uuid%' existiert nicht für Land mit iso2-Code '%countryIso2Code%',de_DE
service_point.validation.service_point_address_address1_wrong_length,Service Point Address Input address1 must have a length of %min% to %max% characters.,en_US
service_point.validation.service_point_address_address1_wrong_length,Service Point Adresse Input address1 muss eine Länge von %min% bis %max% Zeichen haben.,de_DE
service_point.validation.service_point_address_address2_wrong_length,Service Point Address Input address2 must have a length of %min% to %max% characters.,en_US
service_point.validation.service_point_address_address2_wrong_length,Service Point Adresse Input address2 muss eine Länge von %min% bis %max% Zeichen haben.,de_DE
service_point.validation.service_point_address_address3_wrong_length,Service Point Address Input address3 must have a length of %min% to %max% characters.,en_US
service_point.validation.service_point_address_address3_wrong_length,Service Point Adresse Input address3 muss eine Länge von %min% bis %max% Zeichen haben.,de_DE
service_point.validation.service_point_address_city_wrong_length,A service point address city must have length from %min% to %max% characters.,en_US
service_point.validation.service_point_address_city_wrong_length,Eine Service Point Adresse Stadt muss eine Länge von %min% bis %max% Zeichen haben.,de_DE
service_point.validation.service_point_address_entity_not_found,Service point address entity was not found.,en_US
service_point.validation.service_point_address_entity_not_found,Die Entität Service Point Adresse wurde nicht gefunden.,de_DE
service_point.validation.service_point_address_zip_code_wrong_length,A service point address zip code must have length from %min% to %max% characters.,en_US
service_point.validation.service_point_address_zip_code_wrong_length,Die Postleitzahl einer Service Point Adresse muss eine Länge von %min% bis %max% Zeichen haben.,de_DE
service_point.validation.service_point_address_already_exists,A service point address for the service point already exists.,en_US
service_point.validation.service_point_address_already_exists,Es existiert bereits eine Service Point Adresse für den Service Point.,de_DE
service_point.validation.service_point_uuid_is_not_unique,A service point with the same uuid already exists in request.,en_US
service_point.validation.service_point_uuid_is_not_unique,Ein Service Point mit der gleichen uuid existiert bereits in der Anfrage.,de_DE
service_point.validation.service_type_key_exists,A service type with the same key already exists.,en_US
service_point.validation.service_type_key_exists,Ein Service-Typ mit demselben Schlüssel existiert bereits.,de_DE
service_point.validation.service_type_key_wrong_length,A service type key must have length from %min% to %max% characters.,en_US
service_point.validation.service_type_key_wrong_length,Ein Service-Typ-Schlüssel muss eine Länge von %min% bis %max% Zeichen haben.,de_DE
service_point.validation.service_type_key_is_not_unique,A service type with the same key already exists in request.,en_US
service_point.validation.service_type_key_is_not_unique,Ein Service-Typ mit demselben Schlüssel existiert bereits in der Anfrage.,de_DE
service_point.validation.service_type_name_exists,A service type with the same name already exists.,en_US
service_point.validation.service_type_name_exists,Ein Service-Typ mit demselben Namen existiert bereits.,de_DE
service_point.validation.service_type_name_wrong_length,A service type name must have length from %min% to %max% characters.,en_US
service_point.validation.service_type_name_wrong_length,Ein Service-Typ-Name muss eine Länge von %min% bis %max% Zeichen haben.,de_DE
service_point.validation.service_type_name_is_not_unique,A service type with the same name already exists in request.,en_US
service_point.validation.service_type_name_is_not_unique,Ein Service-Typ mit demselben Namen existiert bereits in der Anfrage.,de_DE
service_point.validation.service_type_entity_not_found,The service type entity was not found.,en_US
service_point.validation.service_type_entity_not_found,Die Service-Typ-Entität wurde nicht gefunden.,de_DE
service_point.validation.service_poinst_service_key_exists,A service with the same key already exists.,en_US
service_point.validation.service_poinst_service_key_exists,Ein Service mit demselben Schlüssel existiert bereits.,de_DE
service_point.validation.service_key_wrong_length,A service key must have length from %min% to %max% characters.,en_US
service_point.validation.service_key_wrong_length,Ein Service-Schlüssel muss eine Länge von %min% bis %max% Zeichen haben.,de_DE
service_point.validation.service_key_is_not_unique,A service with the same key already exists in request.,en_US
service_point.validation.service_key_is_not_unique,Ein Service mit demselben Schlüssel existiert bereits in der Anfrage.,de_DE
service_point.validation.service_type_relation_already_exists,A service with defined relation of service point and service type already exists.,en_US
service_point.validation.service_type_relation_already_exists,Ein Service mit einer definierten Beziehung von Servicepunkt und Service-Typ existiert bereits.,de_DE
service_point.validation.service_type_relation_is_not_unique,A service with defined relation of service pint and service type already exists in request.,en_US
service_point.validation.service_type_relation_is_not_unique,Ein Service mit definierter Beziehung von Servicepunkt und Service-Typ existiert bereits in der Anfrage.,de_DE
service_point.validation.service_entity_not_found,The service entity was not found.,en_US
service_point.validation.service_entity_not_found,Die Service-Entität wurde nicht gefunden.,de_DE
service_point.validation.service_key_immutability,The service key is immutable.,en_US
service_point.validation.service_key_immutability,Der Service-Schlüssel ist unveränderlich.,de_DE
service_point.validation.service_type_key_immutability,The service type key is immutable.,en_US
service_point.validation.service_type_key_immutability,Der Service-Typ-Schlüssel ist unveränderlich.,de_DE
service_point.validation.service_key_exists,A service with the same key already exists.,en_US
service_point.validation.service_key_exists,Ein Service mit demselben Schlüssel existiert bereits.,de_DE
product_offer_service_point.validation.product_offer_reference_not_found,Product offer '%product_offer_reference%' not found.,en_US
product_offer_service_point.validation.product_offer_reference_not_found,Product offer '%product_offer_reference%' nicht gefunden.,de_DE
product_offer_service_point.validation.product_offer_has_multiple_service_points,Product offer '%product_offer_reference%' can have only one service point.,en_US
product_offer_service_point.validation.product_offer_has_multiple_service_points,Das Product Offer '%product_offer_reference%' kann nur einen Service Point haben.,de_DE
product_offer_service_point.validation.service_uuid_not_found,Services with uuids '%service_uuids%' not found.,en_US
product_offer_service_point.validation.service_uuid_not_found,Services mit den uuids '%service_uuids%' wurde nicht gefunden.,de_DE
product_offer_service_point.validation.service_not_unique,A service for product offer '%product_offer_reference%' with the same uuid already exists in request.,de_DE
product_offer_service_point.validation.service_not_unique,Ein Service für Product Offer '%product_offer_reference%' mit derselben UUID ist bereits in der Anfrage vorhanden.,de_DE
product_offer_service_point.validation.product_offer_not_unique,A product offer with the same reference already exists in request.,de_DE
product_offer_service_point.validation.product_offer_not_unique,Ein Product Offer mit der gleichen Referenz liegt bereits in der Anfrage vor.,de_DE
```

2. Import data:

```bash
console data:import glossary
```

### 5) Configure export to Elasticsearch

1. In `SearchElasticsearchConfig`, adjust the Elasicsearch config:

**src/Pyz/Shared/SearchElasticsearch/SearchElasticsearchConfig.php**

```php
<?php

namespace Pyz\Shared\SearchElasticsearch;

use Spryker\Shared\SearchElasticsearch\SearchElasticsearchConfig as SprykerSearchElasticsearchConfig;

class SearchElasticsearchConfig extends SprykerSearchElasticsearchConfig
{
    protected const SUPPORTED_SOURCE_IDENTIFIERS = [
        'service_point',
    ];
}
```

2. Set up a new source for Service Points:

```bash
console search:setup:source-map
```


3. In `src/Pyz/Client/RabbitMq/RabbitMqConfig.php`, adjust the `RabbitMq` module's configuration:

**src/Pyz/Client/RabbitMq/RabbitMqConfig.php**

```php
<?php

namespace Pyz\Client\RabbitMq;

use Spryker\Client\RabbitMq\RabbitMqConfig as SprykerRabbitMqConfig;
use Spryker\Shared\ServicePointSearch\ServicePointSearchConfig;

class RabbitMqConfig extends SprykerRabbitMqConfig
{
    /**
     * @return array<mixed>
     */
    protected function getQueueConfiguration(): array
    {
        return [
            ServicePointSearchConfig::QUEUE_NAME_SYNC_SEARCH_SERVICE_POINT,
        ];
    }
}
```

4. Register the new queue message processor:

**src/Pyz/Zed/Queue/QueueDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\Queue;

use Spryker\Shared\ServicePointSearch\ServicePointSearchConfig;
use Spryker\Zed\Kernel\Container;
use Spryker\Zed\Queue\QueueDependencyProvider as SprykerDependencyProvider;
use Spryker\Zed\Synchronization\Communication\Plugin\Queue\SynchronizationSearchQueueMessageProcessorPlugin;

class QueueDependencyProvider extends SprykerDependencyProvider
{
    /**
     * @param \Spryker\Zed\Kernel\Container $container
     *
     * @return array<\Spryker\Zed\Queue\Dependency\Plugin\QueueMessageProcessorPluginInterface>
     */
    protected function getProcessorMessagePlugins(Container $container): array
    {
        return [
            ServicePointSearchConfig::QUEUE_NAME_SYNC_SEARCH_SERVICE_POINT => new SynchronizationSearchQueueMessageProcessorPlugin(),
        ];
    }
}
```

5. Configure the synchronization pool:

**src/Pyz/Zed/ServicePointSearch/ServicePointSearchConfig.php**

```php
<?php

namespace Pyz\Zed\ServicePointSearch;

use Pyz\Zed\Synchronization\SynchronizationConfig;
use Spryker\Zed\ServicePointSearch\ServicePointSearchConfig as SprykerServicePointSearchConfig;

class ServicePointSearchConfig extends SprykerServicePointSearchConfig
{
    /**
     * @return string|null
     */
    public function getServicePointSearchSynchronizationPoolName(): ?string
    {
        return SynchronizationConfig::DEFAULT_SYNCHRONIZATION_POOL_NAME;
    }
}
```

#### Set up regenerate and resync features

| PLUGIN                                              | SPECIFICATION                                                                                          | PREREQUISITES | NAMESPACE                                                           |
|-----------------------------------------------------|--------------------------------------------------------------------------------------------------------|---------------|---------------------------------------------------------------------|
| ServicePointSynchronizationDataBulkRepositoryPlugin | Allows synchronizing the service point search table content into Elasticsearch.                        | None          | Spryker\Zed\ServicePointSearch\Communication\Plugin\Synchronization |
| ServicePointPublisherTriggerPlugin                  | Allows populating service point search table with data and triggering further export to Elasticsearch. | None          | Spryker\Zed\ServicePointSearch\Communication\Plugin\Publisher       |

**src/Pyz/Zed/Synchronization/SynchronizationDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\Synchronization;

use Spryker\Zed\ServicePointSearch\Communication\Plugin\Synchronization\ServicePointSynchronizationDataBulkRepositoryPlugin;
use Spryker\Zed\Synchronization\SynchronizationDependencyProvider as SprykerSynchronizationDependencyProvider;

class SynchronizationDependencyProvider extends SprykerSynchronizationDependencyProvider
{
    /**
     * @return array<\Spryker\Zed\SynchronizationExtension\Dependency\Plugin\SynchronizationDataPluginInterface>
     */
    protected function getSynchronizationDataPlugins(): array
    {
        return [
            new ServicePointSynchronizationDataBulkRepositoryPlugin(),
        ];
    }
}
```

**src/Pyz/Zed/Publisher/PublisherDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\Publisher;

use Spryker\Zed\Publisher\PublisherDependencyProvider as SprykerPublisherDependencyProvider;
use Spryker\Zed\ServicePointSearch\Communication\Plugin\Publisher\ServicePointPublisherTriggerPlugin;

class PublisherDependencyProvider extends SprykerPublisherDependencyProvider
{
    /**
     * @return array<\Spryker\Zed\PublisherExtension\Dependency\Plugin\PublisherTriggerPluginInterface>
     */
    protected function getPublisherTriggerPlugins(): array
    {
        return [
            new ServicePointPublisherTriggerPlugin(),
        ];
    }
}
```

#### Register publisher plugins

| PLUGIN                                  | SPECIFICATION                                             | PREREQUISITES | NAMESPACE                                                                         |
|-----------------------------------------|-----------------------------------------------------------|---------------|-----------------------------------------------------------------------------------|
| ServicePointWritePublisherPlugin        | Listens for events and publishes respective data.         | None          | Spryker\Zed\ServicePointSearch\Communication\Plugin\Publisher\ServicePoint        |
| ServicePointDeletePublisherPlugin       | Listens for events and unpublishes respective data.       | None          | Spryker\Zed\ServicePointSearch\Communication\Plugin\Publisher\ServicePoint        |
| ServicePointAddressWritePublisherPlugin | Listens for events and publishes respective data.         | None          | Spryker\Zed\ServicePointSearch\Communication\Plugin\Publisher\ServicePointAddress |
| ServicePointStoreWritePublisherPlugin   | Listens for events and publishes respective data.         | None          | Spryker\Zed\ServicePointSearch\Communication\Plugin\Publisher\ServicePointStore   |
| ServiceWritePublisherPlugin             | Listens for service events and publishes respective data. | None          | Spryker\Zed\ServicePointSearch\Communication\Plugin\Publisher\Service             |

**src/Pyz/Zed/Publisher/PublisherDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\Publisher;

use Spryker\Zed\Publisher\PublisherDependencyProvider as SprykerPublisherDependencyProvider;
use Spryker\Zed\ServicePointSearch\Communication\Plugin\Publisher\Service\ServiceWritePublisherPlugin;
use Spryker\Zed\ServicePointSearch\Communication\Plugin\Publisher\ServicePoint\ServicePointDeletePublisherPlugin;
use Spryker\Zed\ServicePointSearch\Communication\Plugin\Publisher\ServicePoint\ServicePointWritePublisherPlugin;
use Spryker\Zed\ServicePointSearch\Communication\Plugin\Publisher\ServicePointAddress\ServicePointAddressWritePublisherPlugin;
use Spryker\Zed\ServicePointSearch\Communication\Plugin\Publisher\ServicePointStore\ServicePointStoreWritePublisherPlugin;

class PublisherDependencyProvider extends SprykerPublisherDependencyProvider
{
    /**
     * @return array<\Spryker\Zed\PublisherExtension\Dependency\Plugin\PublisherPluginInterface>
     */
    protected function getPublisherPlugins(): array
    {
        return [
            new ServicePointWritePublisherPlugin(),
            new ServicePointDeletePublisherPlugin(),
            new ServicePointAddressWritePublisherPlugin(),
            new ServicePointStoreWritePublisherPlugin(),
            new ServiceWritePublisherPlugin(),
        ];
    }
}
```

#### Register query expander and result formatter plugins

| PLUGIN                                            | SPECIFICATION                                      | PREREQUISITES | NAMESPACE                                                              |
|---------------------------------------------------|----------------------------------------------------|---------------|------------------------------------------------------------------------|
| ServicePointSearchResultFormatterPlugin           | Maps raw Elasticsearch results to a transfer.      | None          | Spryker\Client\ServicePointSearch\Plugin\Elasticsearch\ResultFormatter |
| SortedServicePointSearchQueryExpanderPlugin       | Adds sorting to a search query.                    | None          | Spryker\Client\ServicePointSearch\Plugin\Elasticsearch\Query           |
| PaginatedServicePointSearchQueryExpanderPlugin    | Adds pagination to a search query.                 | None          | Spryker\Client\ServicePointSearch\Plugin\Elasticsearch\Query           |
| StoreServicePointSearchQueryExpanderPlugin        | Adds filtering by locale to a search query.        | None          | Spryker\Client\ServicePointSearch\Plugin\Elasticsearch\Query           |
| ServiceTypesServicePointSearchQueryExpanderPlugin | Adds filtering by service types to a search query. | None          | Spryker\Client\ServicePointSearch\Plugin\Elasticsearch\Query           |

**src/Pyz/Client/ServicePointSearch/ServicePointSearchDependencyProvider.php**

```php
<?php

namespace Pyz\Client\ServicePointSearch;

use Spryker\Client\ServicePointSearch\Plugin\Elasticsearch\Query\PaginatedServicePointSearchQueryExpanderPlugin;
use Spryker\Client\ServicePointSearch\Plugin\Elasticsearch\Query\SortedServicePointSearchQueryExpanderPlugin;
use Spryker\Client\ServicePointSearch\Plugin\Elasticsearch\Query\StoreServicePointSearchQueryExpanderPlugin;
use Spryker\Client\ServicePointSearch\Plugin\Elasticsearch\ResultFormatter\ServicePointSearchResultFormatterPlugin;
use Spryker\Client\ServicePointSearch\ServicePointSearchDependencyProvider as SprykerServicePointSearchDependencyProvider;

class ServicePointSearchDependencyProvider extends SprykerServicePointSearchDependencyProvider
{
    /**
     * @return list<\Spryker\Client\SearchExtension\Dependency\Plugin\ResultFormatterPluginInterface>
     */
    protected function getServicePointSearchResultFormatterPlugins(): array
    {
        return [
            new ServicePointSearchResultFormatterPlugin(),
        ];
    }

    /**
     * @return list<\Spryker\Client\SearchExtension\Dependency\Plugin\QueryExpanderPluginInterface>
     */
    protected function getServicePointSearchQueryExpanderPlugins(): array
    {
        return [
            new StoreServicePointSearchQueryExpanderPlugin(),
            new SortedServicePointSearchQueryExpanderPlugin(),
            new PaginatedServicePointSearchQueryExpanderPlugin(),
            new ServiceTypesServicePointSearchQueryExpanderPlugin(),
        ];
    }
}
```

{% info_block warningBox "Verification" %}

1. Fill the `spy_service_point` table with some data and run `console publish:trigger-events -r service_point`.
2. Make sure that the `spy_service_point_search` table is filled with respective data per store.
3. Check Elasticearch documents and make sure data is structured in the following format:

```yaml
{
   "store":"DE",
   "type":"service_point",
   "search-result-data":{
      "idServicePoint":123,
      "uuid":"40320bdf-c2af-4dd8-8d09-4550ece4287d",
      "name":"Service Point Name #1",
      "key":"service-point-name-1",
      "address":{
         "idServicePointAddress":44,
         "uuid":"2f02b327-0165-46ea-88df-0190d9a1c242",
         "address1":"Seeburger Str.",
         "address2":"270",
         "address3":"Block B",
         "city":"Berlin",
         "zipCode":"10115",
         "country":{
            "iso2Code":"DE",
            "name":"Germany"
         },
         "region":{
            "name":"Saxony"
         }
      }
   },
   "full-text-boosted":[
      "Service Point Name #1"
   ],
   "full-text":[
      "Service Point Name #1",
      "Seeburger Str. 270 Block B",
      "Berlin",
      "10115",
      "Germany",
      "Saxony"
   ],
   "suggestion-terms":[
      "Service Point Name #1"
   ],
   "completion-terms":[
      "Service Point Name #1"
   ],
   "string-sort":{
      "city":"Berlin"
   }
}
```

4. In the `spy_service_point_search` table, change some records and run `console sync:data service_point`.
5. Make sure that your changes have been synced to the respective Elasticsearch document.

{% endinfo_block %}

### 6) Configure export to Redis

Configure tables to be published and synchronized to the Storage on create, edit, and delete changes.

1. In `src/Pyz/Client/RabbitMq/RabbitMqConfig.php`, adjust the `RabbitMq` module's configuration:

**src/Pyz/Client/RabbitMq/RabbitMqConfig.php**

```php
<?php

namespace Pyz\Client\RabbitMq;

use Spryker\Client\RabbitMq\RabbitMqConfig as SprykerRabbitMqConfig;
use Spryker\Shared\ServicePointStorage\ServicePointStorageConfig;

class RabbitMqConfig extends SprykerRabbitMqConfig
{
    /**
     * @return array<mixed>
     */
    protected function getSynchronizationQueueConfiguration(): array
    {
        return [
            ServicePointStorageConfig::QUEUE_NAME_SYNC_STORAGE_SERVICE_POINT,
        ];
    }
}
```

2. Register new queue message processor:

**src/Pyz/Zed/Queue/QueueDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\Queue;

use Spryker\Shared\ServicePointStorage\ServicePointStorageConfig;
use Spryker\Zed\Queue\QueueDependencyProvider as SprykerDependencyProvider;

class QueueDependencyProvider extends SprykerDependencyProvider
{
    /**
     * @param \Spryker\Zed\Kernel\Container $container
     *
     * @return array<\Spryker\Zed\Queue\Dependency\Plugin\QueueMessageProcessorPluginInterface>
     */
    protected function getProcessorMessagePlugins(Container $container): array
    {
        return [
            ServicePointStorageConfig::QUEUE_NAME_SYNC_STORAGE_SERVICE_POINT => new SynchronizationStorageQueueMessageProcessorPlugin(),
        ];
    }
}
```

3. Configure synchronization pool and event queue name:

**src/Pyz/Zed/ServicePointStorage/ServicePointStorageConfig.php**

```php
<?php

namespace Pyz\Zed\ServicePointStorage;

use Pyz\Zed\Synchronization\SynchronizationConfig;
use Spryker\Shared\Publisher\PublisherConfig;
use Spryker\Zed\ServicePointStorage\ServicePointStorageConfig as SprykerServicePointStorageConfig;

class ServicePointStorageConfig extends SprykerServicePointStorageConfig
{
    /**
     * @return string|null
     */
    public function getServicePointStorageSynchronizationPoolName(): ?string
    {
        return SynchronizationConfig::DEFAULT_SYNCHRONIZATION_POOL_NAME;
    }

    /**
     * @return string|null
     */
    public function getEventQueueName(): ?string
    {
        return PublisherConfig::PUBLISH_QUEUE;
    }
}
```

4. Set up publisher plugins:

| PLUGIN                                  | SPECIFICATION                                                                                 | PREREQUISITES | NAMESPACE                                                                          |
|-----------------------------------------|-----------------------------------------------------------------------------------------------|---------------|------------------------------------------------------------------------------------|
| ServicePointWritePublisherPlugin        | Publishes service point data by `SpyServicePoint` entity events.                              |               | Spryker\Zed\ServicePointStorage\Communication\Plugin\Publisher\ServicePoint        |
| ServicePointAddressWritePublisherPlugin | Publishes service point data by `SpyServicePointAddress` entity events.                       |               | Spryker\Zed\ServicePointStorage\Communication\Plugin\Publisher\ServicePointAddress |
| ServicePointStoreWritePublisherPlugin   | Publishes service point data by service point store entity events.                            |               | Spryker\Zed\ServicePointStorage\Communication\Plugin\Publisher\ServicePointStore   |
| ServicePointPublisherTriggerPlugin      | Allows to populate service point storage table with data and trigger further export to Redis. |               | Spryker\Zed\ServicePointStorage\Communication\Plugin\Publisher                     |

**src/Pyz/Zed/Publisher/PublisherDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\Publisher;

use Spryker\Zed\Publisher\PublisherDependencyProvider as SprykerPublisherDependencyProvider;
use Spryker\Zed\ServicePointStorage\Communication\Plugin\Publisher\ServicePoint\ServicePointWritePublisherPlugin as ServicePointStorageWritePublisherPlugin;
use Spryker\Zed\ServicePointStorage\Communication\Plugin\Publisher\ServicePointAddress\ServicePointAddressWritePublisherPlugin as ServicePointStorageAddressWritePublisherPlugin;
use Spryker\Zed\ServicePointStorage\Communication\Plugin\Publisher\ServicePointPublisherTriggerPlugin as ServicePointStoragePublisherTriggerPlugin;
use Spryker\Zed\ServicePointStorage\Communication\Plugin\Publisher\ServicePointStore\ServicePointStoreWritePublisherPlugin as ServicePointStorageStoreWritePublisherPlugin;

class PublisherDependencyProvider extends SprykerPublisherDependencyProvider
{
    /**
     * @return array
     */
    protected function getPublisherPlugins(): array
    {
        return array_merge(
            $this->getServicePointStoragePlugins(),
        );
    }

    /**
     * @return array<\Spryker\Zed\PublisherExtension\Dependency\Plugin\PublisherTriggerPluginInterface>
     */
    protected function getPublisherTriggerPlugins(): array
    {
        return [
            new ServicePointStoragePublisherTriggerPlugin(),
        ];
    }

    /**
     * @return list<\Spryker\Zed\PublisherExtension\Dependency\Plugin\PublisherPluginInterface>
     */
    protected function getServicePointStoragePlugins(): array
    {
        return [
            new ServicePointStorageWritePublisherPlugin(),
            new ServicePointStorageAddressWritePublisherPlugin(),
            new ServicePointStorageStoreWritePublisherPlugin(),
        ];
    }
}
```

5. Set up synchronization plugins:

| PLUGIN                                              | SPECIFICATION                                                            | PREREQUISITES | NAMESPACE                                                            |
|-----------------------------------------------------|--------------------------------------------------------------------------|---------------|----------------------------------------------------------------------|
| ServicePointSynchronizationDataBulkRepositoryPlugin | Allows synchronizing the service point storage table content into Redis. |               | Spryker\Zed\ServicePointStorage\Communication\Plugin\Synchronization |

**src/Pyz/Zed/Synchronization/SynchronizationDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\Synchronization;

use Spryker\Zed\ServicePointStorage\Communication\Plugin\Synchronization\ServicePointSynchronizationDataBulkRepositoryPlugin as ServicePointStorageSynchronizationDataBulkRepositoryPlugin;
use Spryker\Zed\Synchronization\SynchronizationDependencyProvider as SprykerSynchronizationDependencyProvider;

class SynchronizationDependencyProvider extends SprykerSynchronizationDependencyProvider
{
    /**
     * @return array<\Spryker\Zed\SynchronizationExtension\Dependency\Plugin\SynchronizationDataPluginInterface>
     */
    protected function getSynchronizationDataPlugins(): array
    {
        return [
            new ServicePointStorageSynchronizationDataBulkRepositoryPlugin(),
        ];
    }
}
```

{% info_block warningBox "Verification" %}

Make sure that the `service-point` trigger plugin works correctly:

1. Fill the `spy_service_point`, `spy_service_point_store`, and `spy_servoce_point_address` tables with data.
2. Run the `console publish:trigger-events -r service_point` command.
3. Make sure that the `spy_service_point_storage` table has been filled with respective data.
4. Make sure that, in your system, storage entries are displayed with `kv:service_point:{store}:{service_point_id}` mask.

Make sure that `service-point` synchronization plugin works correctly:

1. Fill the `spy_service_point_storage` table with some data.
2. Run the `console sync:data -r service_point` command.
3. Make sure that, in your system, storage entries are displayed with the `kv:service_point:{store}:{service_point_id}` mask.

Make sure that when a service point is created or edited through BAPI, it is exported to Redis accordingly.

Make sure that, in Redis, data is displayed in the following format:
```yaml
{
   "id_service_point": 1,
   "uuid": "262feb9d-33a7-5c55-9b04-45b1fd22067e",
   "name": "Spryker Main Store",
   "key": "sp1",
   "is_active": true,
   "address": {
      "id_service_point_address": 1,
      "uuid": "74768ee9-e7dd-5e3c-bafd-b654e7946c54",
      "address1": "Caroline-Michaelis-Stra\u00dfe",
      "address2": "8",
      "address3": null,
      "zip_code": "10115",
      "city": "Berlin",
      "country": {
         "iso2_code": "DE",
         "id_country": 60
      },
      "region": {
         "uuid": "2f02b327-0165-46ea-88df-0190d9a1c242",
         "id_region": 1,
         "name": "Berlin"
      }
   },
   "_timestamp": 1683216744.8334839
}
```
{% endinfo_block %}

### 7) Set up behavior

1. To expand product offers with services, register the plugins:

| PLUGIN                                    | SPECIFICATION                               | PREREQUISITES | NAMESPACE                                                               |
|-------------------------------------------|---------------------------------------------|---------------|-------------------------------------------------------------------------|
| ServiceProductOfferPostCreatePlugin       | Creates the product offer service entities. |               | \Spryker\Zed\ProductOfferServicePoint\Communication\Plugin\ProductOffer |
| ServiceProductOfferPostUpdatePlugin       | Updates the product offer service entities. |               | Spryker\Zed\ProductOfferServicePoint\Communication\Plugin\ProductOffer  |
| ServiceProductOfferExpanderPlugin         | Expands product offer with services.        |               | \Spryker\Zed\ProductOfferServicePoint\Communication\Plugin\ProductOffer |

**src/Pyz/Zed/ProductOffer/ProductOfferDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\ProductOffer;

use Spryker\Zed\ProductOffer\ProductOfferDependencyProvider as SprykerProductOfferDependencyProvider;
use Spryker\Zed\ProductOfferServicePoint\Communication\Plugin\ProductOffer\ServiceProductOfferExpanderPlugin;
use Spryker\Zed\ProductOfferServicePoint\Communication\Plugin\ProductOffer\ServiceProductOfferPostCreatePlugin;
use Spryker\Zed\ProductOfferServicePoint\Communication\Plugin\ProductOffer\ServiceProductOfferPostUpdatePlugin;

class ProductOfferDependencyProvider extends SprykerProductOfferDependencyProvider
{
        /**
     * @return array<\Spryker\Zed\ProductOfferExtension\Dependency\Plugin\ProductOfferPostCreatePluginInterface>
     */
    protected function getProductOfferPostCreatePlugins(): array
    {
        return [
            ...
            new ServiceProductOfferPostCreatePlugin(),
        ];
    }

    /**
     * @return array<\Spryker\Zed\ProductOfferExtension\Dependency\Plugin\ProductOfferPostUpdatePluginInterface>
     */
    protected function getProductOfferPostUpdatePlugins(): array
    {
        return [
            ...
            new ServiceProductOfferPostUpdatePlugin(),
        ];
    }

    /**
     * @return array<\Spryker\Zed\ProductOfferExtension\Dependency\Plugin\ProductOfferExpanderPluginInterface>
     */
    protected function getProductOfferExpanderPlugins(): array
    {
        return [
            ...
            new ServiceProductOfferExpanderPlugin(),
        ];
    }
}

```

2. To enable the Backend API, register the plugins:

| PLUGIN                                     | SPECIFICATION                                     | PREREQUISITES | NAMESPACE                                                                                            |
|--------------------------------------------|---------------------------------------------------|---------------|------------------------------------------------------------------------------------------------------|
| ServicePointsBackendResourcePlugin         | Registers the `service-points` resource.          |               | \Spryker\Glue\ServicePointsBackendApi\Plugin\GlueBackendApiApplicationGlueJsonApiConventionConnector |
| ServicePointAddressesBackendResourcePlugin | Registers the `service-point-addresses` resource. |               | \Spryker\Glue\ServicePointsBackendApi\Plugin\GlueBackendApiApplicationGlueJsonApiConventionConnector |

**src/Pyz/Glue/GlueBackendApiApplication/GlueBackendApiApplicationDependencyProvider.php**

```php
<?php

namespace Pyz\Glue\GlueBackendApiApplication;

use Spryker\Glue\GlueBackendApiApplication\GlueBackendApiApplicationDependencyProvider as SprykerGlueBackendApiApplicationDependencyProvider;
use \Spryker\Glue\ServicePointsBackendApi\Plugin\GlueBackendApiApplication\ServicePointsBackendResourcePlugin;
use \Spryker\Glue\ServicePointsBackendApi\Plugin\GlueBackendApiApplication\ServicePointAddressesBackendResourcePlugin;
use Spryker\Glue\ServicePointsBackendApi\Plugin\GlueBackendApiApplication\ServicesBackendResourcePlugin;
use Spryker\Glue\ServicePointsBackendApi\Plugin\GlueBackendApiApplication\ServiceTypesBackendResourcePlugin;

class GlueBackendApiApplicationDependencyProvider extends SprykerGlueBackendApiApplicationDependencyProvider
{
    /**
     * @return array<\Spryker\Glue\GlueApplicationExtension\Dependency\Plugin\ResourceInterface>
     */
    protected function getResourcePlugins(): array
    {
        return [
            new ServicePointsBackendResourcePlugin(),
            new ServicePointAddressesBackendResourcePlugin(),
            new ServiceTypesBackendResourcePlugin(),
            new ServicesBackendResourcePlugin(),
        ];
    }
}

```

3. To enable the Backend API relationships, register the plugin:

| PLUGIN                                                                 | SPECIFICATION                                                             | PREREQUISITES | NAMESPACE                                                                                            |
|------------------------------------------------------------------------|---------------------------------------------------------------------------|---------------|------------------------------------------------------------------------------------------------------|
| ServicePointAddressesByServicePointsBackendResourceRelationshipPlugin  | Adds `service-point-addresses` relationship to `service-points` resource. |               | \Spryker\Glue\ServicePointsBackendApi\Plugin\GlueBackendApiApplicationGlueJsonApiConventionConnector |
| ServicesByServicePointsBackendResourceRelationshipPlugin  | Adds `services` relationship to `service-points` resource.                |               | \Spryker\Glue\ServicePointsBackendApi\Plugin\GlueBackendApiApplicationGlueJsonApiConventionConnector |
| ServicePointsByServicesBackendResourceRelationshipPlugin  | Adds `service-points` relationship to `services` resource.                |               | \Spryker\Glue\ServicePointsBackendApi\Plugin\GlueBackendApiApplicationGlueJsonApiConventionConnector |
| ServiceTypesByServicesBackendResourceRelationshipPlugin  | Adds `service-types` relationship to `services` resource.                 |               | \Spryker\Glue\ServicePointsBackendApi\Plugin\GlueBackendApiApplicationGlueJsonApiConventionConnector |

**src/Pyz/Glue/GlueBackendApiApplicationGlueJsonApiConventionConnector/GlueBackendApiApplicationGlueJsonApiConventionConnectorDependencyProvider.php**

```php
<?php

namespace Pyz\Glue\GlueBackendApiApplication;

use Spryker\Glue\GlueBackendApiApplicationGlueJsonApiConventionConnector\GlueBackendApiApplicationGlueJsonApiConventionConnectorDependencyProvider as SprykerGlueBackendApiApplicationGlueJsonApiConventionConnectorDependencyProvider;
use Spryker\Glue\ServicePointsBackendApi\Plugin\GlueBackendApiApplicationGlueJsonApiConventionConnector\ServicePointAddressesByServicePointsBackendResourceRelationshipPlugin;
use Spryker\Glue\ServicePointsBackendApi\Plugin\GlueBackendApiApplicationGlueJsonApiConventionConnector\ServicePointsByServicesBackendResourceRelationshipPlugin;
use Spryker\Glue\ServicePointsBackendApi\Plugin\GlueBackendApiApplicationGlueJsonApiConventionConnector\ServicesByServicePointsBackendResourceRelationshipPlugin;
use Spryker\Glue\ServicePointsBackendApi\Plugin\GlueBackendApiApplicationGlueJsonApiConventionConnector\ServiceTypesByServicesBackendResourceRelationshipPlugin;
use Spryker\Glue\ServicePointsBackendApi\ServicePointsBackendApiConfig;

class GlueBackendApiApplicationGlueJsonApiConventionConnectorDependencyProvider extends SprykerGlueBackendApiApplicationGlueJsonApiConventionConnectorDependencyProvider{
    /**
     * @param \Spryker\Glue\GlueJsonApiConventionExtension\Dependency\Plugin\ResourceRelationshipCollectionInterface $resourceRelationshipCollection
     *
     * @return \Spryker\Glue\GlueJsonApiConventionExtension\Dependency\Plugin\ResourceRelationshipCollectionInterface
     */
    protected function getResourceRelationshipPlugins(
        ResourceRelationshipCollectionInterface $resourceRelationshipCollection,
    ): ResourceRelationshipCollectionInterface {
        ...

        $resourceRelationshipCollection->addRelationship(
            ServicePointsBackendApiConfig::RESOURCE_SERVICE_POINTS,
            new ServicePointAddressesByServicePointsBackendResourceRelationshipPlugin(),
        );

        $resourceRelationshipCollection->addRelationship(
            ServicePointsBackendApiConfig::RESOURCE_SERVICE_POINTS,
            new ServicesByServicePointsBackendResourceRelationshipPlugin(),
        );

        $resourceRelationshipCollection->addRelationship(
            ServicePointsBackendApiConfig::RESOURCE_SERVICES,
            new ServicePointsByServicesBackendResourceRelationshipPlugin(),
        );

        $resourceRelationshipCollection->addRelationship(
            ServicePointsBackendApiConfig::RESOURCE_SERVICES,
            new ServiceTypesByServicesBackendResourceRelationshipPlugin(),
        );

        ...
    }
}

```

{% info_block warningBox "Verification" %}

Make sure that you can send the following requests:

* `POST https://glue-backend.mysprykershop.com/service-points`
   ```json
      {
          "data": {
              "type": "service-points",
              "attributes": {
                  "name": "Some Service Point",
                  "key": "ssp",
                  "isActive": "true",
                  "stores": ["DE", "AT"]
              }
          }
      }
   ```

* `PATCH https://glue-backend.mysprykershop.com/service-points/{{service-point-uuid}}`
    ```json
        {
            "data": {
                "type": "service-points",
                "attributes": {
                    "name": "Another Name"
                }
            }
        }
    ```

* `GET https://glue-backend.mysprykershop.com/service-points/`
* `GET https://glue-backend.mysprykershop.com/service-points/{{service-point-uuid}}`
* `POST https://glue-backend.mysprykershop.com/service-points/{{service-point-uuid}}/service-point-addresses`
   ```json
      {
          "data": {
              "type": "service-point-address",
              "attributes": {
                  "address1": "address1",
                  "address2": "address2",
                  "address3": "address3",
                  "city": "city",
                  "zipCode": "10115",
                  "countryIso2Code": "DE"
              }
          }
      }
   ```

* `PATCH https://glue-backend.mysprykershop.com/service-points/{{service-point-uuid}}/service-point-addresses/{{service-point-address-uuid}}`
  ```json
     {
         "data": {
             "type": "service-point-address",
             "attributes": {
                 "address1": "another address1",
                 "address2": "another address2",
                 "address3": "another address3",
                 "city": "another city",
                 "zipCode": "20115",
                 "countryIso2Code": "AT"
             }
         }
     }
  ```

* `GET https://glue-backend.mysprykershop.com/service-points/{{service-point-uuid}}/service-point-addresses`

* `GET https://glue-backend.mysprykershop.com/service-types/`
* `GET https://glue-backend.mysprykershop.com/service-types/{{service-type-uuid}}`
* `POST https://glue-backend.mysprykershop.com/service-types/`
   ```json
      {
          "data": {
              "type": "service-types",
              "attributes": {
                  "name": "ServiceType",
                  "key": "st-4"
              }
          }
      }
   ```

* `PATCH https://glue-backend.mysprykershop.com/service-types/{{service-type-uuid}}`
  ```json
      {
          "data": {
              "type": "service-types",
              "attributes": {
                  "name": "ServiceType"
              }
          }
      }
  ```

* `GET https://glue-backend.mysprykershop.com/services/`
* `GET https://glue-backend.mysprykershop.com/services/{{service-uuid}}`
* `POST https://glue-backend.mysprykershop.com/services/`
   ```json
      {
          "data": {
              "type": "services",
              "attributes": {
                  "isActive": false,
                  "key": 123,
                  "servicePointUuid": "{{service-point-uuid}}",
                  "serviceTypeUuid": "{{service-type-uuid}}"
              }
          }
      }
   ```

* `PATCH https://glue-backend.mysprykershop.com/services/{{service-uuid}}`
  ```json
      {
          "data": {
              "type": "services",
              "attributes": {
                  "isActive": true
              }
          }
      }
  ```

{% endinfo_block %}
