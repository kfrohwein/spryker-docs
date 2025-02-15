---
title: File details - product_alternative.csv
last_updated: Jun 16, 2021
template: data-import-template
originalLink: https://documentation.spryker.com/2021080/docs/file-details-product-alternativecsv
originalArticleId: cda20a75-66a5-4c07-a130-74725e5e5da0
redirect_from:
  - /2021080/docs/file-details-product-alternativecsv
  - /2021080/docs/en/file-details-product-alternativecsv
  - /docs/file-details-product-alternativecsv
  - /docs/en/file-details-product-alternativecsv
related:
  - title: Execution order of data importers in Demo Shop
    link: docs/scos/dev/data-import/page.version/demo-shop-data-import/execution-order-of-data-importers-in-demo-shop.html
---

This document describes the `product_alternative.csv` file to configure [Alternative Product](/docs/scos/user/features/{{page.version}}/alternative-products-feature-overview.html) information in your Spryker Demo Shop.

To import the file, run:

```bash
data:import:product-alternative
```

## Import file parameters

The file should have the following parameters:

| PARAMETER | REQUIRED | TYPE | REQUIREMENTS OR COMMENTS | DESCRIPTION |
| --- | --- | --- | --- | --- |
| concrete_sku | &check; | String |N/A* | SKU of the concrete product to which this alternative is applied. |
| alternative_product_concrete_sku | &check; (*if the `alternative_product_abstract_sku` is empty*) | String |  | SKU of the alternative concrete product. |
| alternative_product_abstract_sku | &check; (*if `alternative_product_concrete_sku` is empty*) | String |  | SKU of the alternative abstract product. |

## Import file dependencies

This file has the following dependencies:

* [product_concrete.csv](/docs/scos/dev/data-import/{{page.version}}/data-import-categories/catalog-setup/products/file-details-product-concrete.csv.html)
* [product_abstract.csv](/docs/scos/dev/data-import/{{page.version}}/data-import-categories/catalog-setup/products/file-details-product-abstract.csv.html)

## Additional information

It does not exist on by default on the project level. It can be created in order to override the CSV file from module: 

* `vendor/spryker/product-alternative-data-import/data/import/product_alternative.csv`

## Import template file and content example

Find the template and an example of the file below:

| FILE | DESCRIPTION |
| --- | --- |
| [product_alternative.csv template](https://spryker.s3.eu-central-1.amazonaws.com/docs/Developer+Guide/Back-End/Data+Manipulation/Data+Ingestion/Data+Import/Data+Import+Categories/Merchandising+Setup/Product+Merchandising/Template+product_alternative.csv) | Exemplary import file with headers only. |
| [product_alternative.csv](https://spryker.s3.eu-central-1.amazonaws.com/docs/Developer+Guide/Back-End/Data+Manipulation/Data+Ingestion/Data+Import/Data+Import+Categories/Merchandising+Setup/Product+Merchandising/product_alternative.csv) | Exemplary import file with Demo Shop data. |
