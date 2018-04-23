---
layout: default
group: mftf
title: Page structure
version: 2.2
github_link: magento-functional-testing-framework/release-2/page.md
functional_areas:
 - Testing
mftf-release: 2.0.2
---

_This topic was updated due to the {{page.mftf-release}} MFTF release._
{: style="text-align: right"}

The MFTF uses a modified concept of [PageObjects](https://github.com/SeleniumHQ/selenium/wiki/PageObjects), which models the testing areas of your page as objects within the code. This reduces occurrences of duplicated code and enables you to fix things quickly, in one place, when things change. You define the contents of a page, for reference in a [`<test>`](./test.html), at both the [`<page>`](#page-tag) and [`<section>`](#section-tag) level.

The `pageObject` lists the URL of the `page` and the `sections` that it contains. You can reuse `sections` to define the elements on a page that are exercisable, while also ensuring a single source of truth to help maintain consistency.

<div class="bs-callout bs-callout-tip" markdown="1">
Avoid hard-coded location selectors from tests to increase the maintainability and readability of the tests, as well as improve the test execution output and logging.
</div>

Two types of pages are available:
 {%raw%}
 * Page with `url` declared as a constant string or [explicit page](#explicit-page) - In a test it is called in a format like `{{NameOfPage.url}}`, where `NameOfPage` is a value of `name` in the corresponding page declaration `*.xml` file.
 * Page with `url` declared as a string with one or more variables or [parameterized page](#parameterized-page) - In a test it is called using a format like `{{NameOfPage.url(var1, var2, ...)}}`, where `var1, var2` etc. are parameters that will be substituted in the `url` of the corresponding `<page>` declaration.
{%endraw%}
The following diagram shows the XML structure of an MFTF page:

{% include_relative img/page-dia.svg %}

## Format

The format of `<page>` is:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<pages xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="../../../../../../vendor/magento/magento2-functional-testing-framework/src/Magento/FunctionalTestingFramework/Page/etc/PageObject.xsd">
    <page name="" url="" module="" area="">
        <section name=""/>
        <section name=""/>
    </page>
</pages>
```

## Principles

The following conventions apply to MFTF pages:

* `<page>` name is the same as the file name.
* `<page>` name must be alphanumeric.
* `*Page.xml` is stored in the _Section_ directory of a module.
* The name format is `{Admin|Storefront}{PageDescription}Page.xml`.

## Page examples

These examples demonstrate explicit and parameterized pages and include informative explanations.

### Explicit page

Example (`Catalog/Page/AdminCategoryPage.xml` file):

```xml
<?xml version="1.0" encoding="UTF-8"?>

<pages xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="../../../../../../vendor/magento/magento2-functional-testing-framework/src/Magento/FunctionalTestingFramework/Page/etc/PageObject.xsd">
    <page name="AdminCategoryPage" url="catalog/category/" module="Magento_Catalog" area="admin">
        <section name="AdminCategorySidebarActionSection"/>
        <section name="AdminCategorySidebarTreeSection"/>
        <section name="AdminCategoryBasicFieldSection"/>
        <section name="AdminCategorySEOSection"/>
    </page>
</pages>
```

In this example, the `Catalog/Page/AdminCategoryPage.xml` file declares a page with the name `AdminCategoryPage`. It will be merged with the other `AdminCategoryPage` pages from other modules.

The corresponding web page is generated by the Magento Catalog module and is called by the `baseUrl` + `backendName` + `catalog/category/` URl.

The `AdminCategoryPage` declares four [sections](./section.html):

 * `AdminCategorySidebarActionSection` - located in the `Catalog/Section/AdminCategorySidebarActionSection.xml` file
 * `AdminCategorySidebarTreeSection` - located in the `Catalog/Section/AdminCategorySidebarTreeSection.xml` file
 * `AdminCategoryBasicFieldSection` - located in the `Catalog/Section/AdminCategoryBasicFieldSection.xml` file
 * `AdminCategorySEOSection` - located in the `Catalog/Section/AdminCategorySEOSection.xml` file


The following is an example of a call in test:
{%raw%}
```xml
<amOnPage url="{{AdminCategoryPage.url}}" stepKey="navigateToAdminCategory"/>
```

### Parameterized page

Example (`Catalog/Page/StorefrontCategoryPage.xml` file):

```xml
<?xml version="1.0" encoding="UTF-8"?>

<pages xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="../../../../../../vendor/magento/magento2-functional-testing-framework/src/Magento/FunctionalTestingFramework/Page/etc/PageObject.xsd">
    <page name="StorefrontCategoryPage" url="/{{var1}}.html" module="Magento_Catalog" parameterized="true" area="storefront">
        <section name="StorefrontCategoryMainSection"/>
    </page>
</pages>
```

This example shows the page with the name `StorefrontCategoryPage`. It will be merged with the other `StorefrontCategoryPage` pages from other modules.

The following is an example of a call in test:

```xml
<amOnPage url="{{StorefrontCategoryPage.url($$createPreReqCategory.name$$)}}" stepKey="navigateToCategoryPage"/>
```

The `StorefrontCategoryPage` page is declared as parameterized, where the `url` contains a `{{var1}}` parameter.

The corresponding web page is generated by the Magento Catalog module and is called by the `baseUrl`+`/$$createPreReqCategory.name$$.html` URl.

`{{var1}}` is substituted with the `name` of the previously created category in the `createPreReqCategory` action.

See also [`<createData>`](./test/actions.html#createdata).

****

The `StorefrontCategoryPage` page declares only the `StorefrontCategoryMainSection` [section](./section.html), which is located in the `Catalog/Section/StorefrontCategoryMainSection.xml` file.

## Elements reference

There are several XML elements that are used in `<page>` in the MFTF.

### pages {#pages-tag}

`<pages>` are elements that point to the corresponding XML Schema location. It contains one or more `<page>` elements.

### page {#page-tag}

`<page>` contains a sequence of UI sections in a page.

Attributes|Type|Use|Description
---|---|---|---
`name`|string|required|Unique page name identifier.
`url`|string|required|URL path (excluding the base URL) for the page. Use parameterized notation (`{{var1}}`) for replaceable parameters, such as the edit page for a persisted entity that is based on an ID or a name.
`module`|string|required|The name of the module to which the page belongs. Example: `"Magento_Catalog"`.
`area`|string|required|The area where this page lives. Three possible values: `admin` prepends `BACKEND_NAME` to `url`, `storefront` does not prepend anything to `url`, `external` flags the page for use with `amOnUrl`. The `url` provided must be a full URL, such as `http://myFullUrl.com/`, instead of the URL for a Magento page.
`parameterized`|boolean |optional|Include and set to `"true"` if the `url` for this page has parameters that need to be replaced for proper use.
`remove`|boolean|optional|The default value is `"false"`. Set to `"true"` to remove this element during parsing.

`<page>` may contain several [<section>](#section-tag) elements.

### section {#section-tag}

`<section>` contains the sequence of UI elements. A section is a reusable piece or part of a page.

Attributes|Type|Use|Description
---|---|---|---
`name`|string|required|Unique section name identifier.
`remove`|boolean|optional|The default value is `"false"`. Set to `"true"` to remove this element during parsing.

{%endraw%}