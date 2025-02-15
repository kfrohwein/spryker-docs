---
title: File details- product_search_attribute.csv
last_updated: Sep 14, 2020
template: data-import-template
originalLink: https://documentation.spryker.com/v5/docs/file-details-product-search-attributecsv
originalArticleId: 257d522c-7afb-4235-b5fc-c1ae6d90d374
redirect_from:
  - /v5/docs/file-details-product-search-attributecsv
  - /v5/docs/en/file-details-product-search-attributecsv
---

This article contains content of the **product_search_attribute.csv** file to configure Product Search Attribute information on your Spryker Demo Shop.

## Import file parameters 
These are the header fields to be included in the .csv file:

| Field Name | Mandatory | Type | Other Requirements/Comments | Description |
| --- | --- | --- | --- | --- |
| **key** | Yes | String |N/A* | Key identifier string of the product search attribute. |
| **filter_type** | No | String |N/A | Type of search filter, Elasticsearch-specific. |
| **position** | No | Number |N/A | Position of the product search attribute, Elasticsearch specific. |
| **key.{ANY_LOCALE_NAME}** **<br>Example value: *key.en_US*  | Yes | String |N/A | Key identifier string of the product search attribute, translated in the specified locale US for our example). |
*N/A: Not applicable.
**ANY_LOCALE_NAME: Locale date is dynamic in data importers. It means that ANY_LOCALE_NAME postifx can be changed, removed, and any number of columns with different locales can be added to the .csv files.

## Dependencies

This file has the following dependency:
*    [product_attribute_key.csv](/docs/scos/dev/data-import/{{page.version}}/data-import-categories/catalog-setup/products/file-details-product-attribute-key.csv.html)

## Recommendations & other information
The attribute key is previously loaded from *productattributekey.csv*, which can be translated in key.* fields.

## Import template file and content example
A template and an example of the *product_search_attribute.csv*  file can be downloaded here:

| File | Description |
| --- | --- |
| [product_search_attribute.csv template](https://spryker.s3.eu-central-1.amazonaws.com/docs/Developer+Guide/Back-End/Data+Manipulation/Data+Ingestion/Data+Import/Data+Import+Categories/Merchandising+Setup/Product+Merchandising/Template+product_search_attribute.csv) | Product Search Attribute .csv template file (empty content, contains headers only). |
| [product_search_attribute.csv](https://spryker.s3.eu-central-1.amazonaws.com/docs/Developer+Guide/Back-End/Data+Manipulation/Data+Ingestion/Data+Import/Data+Import+Categories/Merchandising+Setup/Product+Merchandising/product_search_attribute.csv) | Product Search Attribute .csv file containing a Demo Shop data sample. |
