---
title: Integrating Symfony 6
description: Learn about the main changes in the new Symfony version 6
last_updated: Dec 7, 2022
template: howto-guide-template
originalLink: https://documentation.spryker.com/2021080/docs/symfony-6-integration
originalArticleId: d5e96c3b-3ed6-49ed-982c-aa641e09b559
---

Spryker primarily supports Symfony 6 that was released in December 2022. Backwards compatibility remains for all three major Symfony versions, but support for Symfony 4 was partially dropped in October 2022.

{% info_block warningBox "Old Symfony versions" %}

Although Spryker still supports older versions of Symfony,you should avoid installing them in your project. This is because any other packages you or Spryker use have different requirements. Always try to keep your dependencies updated.

{% endinfo_block %}

<a name="changes"></a>

## Main changes in Symfony 6

Symfony 6 has a new cycle of innovations. When it starts, one cycle lasts two years, on a modernized codebase that has been cleaned up from the dead weight of the past.
The major change included in Symfony 6 is PHP 8.0 being the minimum required version of PHP.
The code of Symfony 6 has been updated. You can take advantage of all the new features in PHP.
For example, the code includes PHP 8 [attributes](https://www.php.net/manual/fr/language.attributes.overview.php), more expressive and rigorous type declarations, and more.

Read more at [CHANGELOG-6.0](https://github.com/symfony/symfony/blob/6.0/CHANGELOG-6.0.md). 

## Integration

To make your project compatible with Symfony 6, update the [Symfony](https://github.com/spryker/symfony) module and all modules that use it:

```bash
composer require spryker/symfony:"^3.11.0"
```

If you can’t install the required version, run the following command to see what else you need to update:

```bash
composer why-not spryker/symfony:3.11.0
```

This gives you a list of modules that require the latest `spryker/symfony` module and that need to be updated as well.
