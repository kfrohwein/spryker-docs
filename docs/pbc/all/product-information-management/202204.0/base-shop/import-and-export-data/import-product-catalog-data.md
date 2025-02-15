---
title: Import product catalog data
last_updated: Jun 16, 2021
template: data-import-template
originalLink: https://documentation.spryker.com/2021080/docs/catalog-setup
originalArticleId: 16830216-0c33-4009-86e0-f9995eef7eed
redirect_from:
  - /2021080/docs/catalog-setup
  - /2021080/docs/en/catalog-setup
  - /docs/catalog-setup
  - /docs/en/catalog-setup
  - /docs/scos/dev/data-import/202204.0/data-import-categories/catalog-setup/catalog-setup.html
---

*Catalog Setup* contains data required to sell products and build their main structure.

{% info_block warningBox "Important" %}

We recommend setting up the Catalog after having done the [Commerce Setup](/docs/scos/dev/data-import/{{page.version}}/data-import-categories/commerce-setup/commerce-setup.html), which provides the overall structure of the store.

{% endinfo_block %}

This section helps you import the necessary product-related data to be able to sell products and services in your online store. We have structured this section into four main categories focusing on the following topics:

* [Categories](/docs/pbc/all/product-information-management/{{page.version}}/base-shop/import-and-export-data/categories-data-import/categories-data-import.html)
* [Products](/docs/scos/dev/data-import/{{page.version}}/data-import-categories/catalog-setup/products/products.html)
* [Pricing](/docs/pbc/all/price-management/{{site.version}}/base-shop/import-and-export-data/import-of-prices.html)
* [Stocks](/docs/scos/dev/data-import/{{page.version}}/data-import-categories/catalog-setup/stocks/stocks.html)

Within the [Categories](/docs/pbc/all/product-information-management/{{page.version}}/base-shop/import-and-export-data/categories-data-import/categories-data-import.html) section, you can find all information about the data imports required to set up categories that can be used in your online store as well as whether they are active, searchable in the menu, etc.

The [Products](/docs/scos/dev/data-import/{{page.version}}/data-import-categories/catalog-setup/products/products.html) section helps you import all data defining the products' properties such as the abstract products, the concrete products, product images, and any type of related attributes which describe the products' properties—for example, their specifications, colors, and sizes.

In the [Pricing](/docs/pbc/all/price-management/{{site.version}}/base-shop/import-and-export-data/import-of-prices.html) section, you will be able to import the data necessary to set up prices for all the products you would like to sell in your online store, including advanced pricing such as scheduled prices (e.g., for special sales campaigns like Black Friday).

In the [Stocks](/docs/scos/dev/data-import/{{page.version}}/data-import-categories/catalog-setup/stocks/stocks.html) section, we describe the data import containing the number of product units stored in your warehouses as well as any type of products and services which are never out of stock—for example, software downloads.


{% info_block warningBox "Import order" %}

By default, most of the product data is stored in a separate subfolder in `data/import/icecat_biz_data`. The order the files are imported in is *very strict*:

1. Any product-related entities such as categories, attributes, and tax sets must be imported before the actual products.
2. [product_abstract.csv](/docs/pbc/all/product-information-management/{{page.version}}/base-shop/import-and-export-data/products-data-import/file-details-product-abstract.csv.html) and for multi-store setups [product_abstract_store.csv](/docs/pbc/all/product-information-management/{{page.version}}/base-shop/import-and-export-data/products-data-import/file-details-product-abstract-store.csv.html).
3. [product_concrete.csv](/docs/pbc/all/product-information-management/{{page.version}}/base-shop/import-and-export-data/products-data-import/file-details-product-concrete.csv.html).
4. Other product data such as images, product sets, etc. in any order.

{% endinfo_block %}
