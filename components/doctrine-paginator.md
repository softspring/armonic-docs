---
title: "Doctrine Paginator"
description: "Paginate Doctrine query builders, keep filters and sorting together, and render listing pages with a reusable pagination object."
---

# Doctrine Paginator {#doctrine-paginator}

`softspring/doctrine-paginator` helps you build classic listing pages on top of Doctrine ORM.

It is a good fit when you already have a `QueryBuilder` and you want to add:

- page slicing
- total results
- sorting
- filters
- reusable page and sort URLs

It works especially well in admin lists, backoffice screens, and report pages.

## Main Idea {#main-idea}

This component revolves around three pieces:

- `Paginator::queryPage()` to paginate a `QueryBuilder`
- `PaginatedCollection` to hold both results and pagination metadata
- `PaginatorForm` to keep GET filters, ordering, and page state together

That means you can keep one flow from request to query to template, instead of manually managing:

- the current page
- results per page
- sorting field and direction
- filter array
- pagination links

## Installation {#installation}

```bash
composer require softspring/doctrine-paginator:^6.0
```

This package builds on top of Doctrine ORM and is usually used together with `softspring/doctrine-query-filters`.

## Quick Start {#quick-start}

If you already have a query builder, the quickest usage is:

```php
<?php

use Softspring\Component\DoctrinePaginator\Paginator;

$qb = $entityManager
    ->getRepository(User::class)
    ->createQueryBuilder('u');

$pagination = Paginator::queryPage(
    $qb,
    page: 1,
    rpp: 20,
    filters: ['name__like' => 'john'],
    orderBy: ['name' => 'asc'],
);
```

`$pagination` is a `PaginatedCollection`.

You can iterate it like a Doctrine collection and also ask for:

- current page
- total pages
- total results
- next and previous page
- sort URLs

## Paginate A Query Builder {#paginate-a-query-builder}

The base API is `Paginator::queryPage()`:

```php
<?php

$pagination = Paginator::queryPage(
    $qb,
    page: 2,
    rpp: 50,
);
```

This is useful when page and page size are already normalized by your controller or application service.

The paginator clones the query builder internally, so it can:

- count the total number of matching rows
- fetch only the current page slice
- return both results and metadata in one object

## Add Filters And Sorting {#add-filters-and-sorting}

The component uses `softspring/doctrine-query-filters` for filter syntax.

Example:

```php
<?php

$filters = [
    'name__like' => 'john',
    'country__in' => ['ES', 'FR'],
];

$orderBy = [
    'createdAt' => 'desc',
];

$pagination = Paginator::queryPage($qb, 1, 20, $filters, $orderBy);
```

This is the recommended way to sort listing pages.

You can still call `orderBy()` directly on the `QueryBuilder`, but that is not the usual usage here. The component is designed to receive sorting through its own `$orderBy` argument so that the same information can also be reflected in URLs and forms.

## Change Filter Mode {#change-filter-mode}

If your filters should be combined in a different way, you can change the filter mode:

```php
<?php

use Softspring\Component\DoctrineQueryFilters\Filters;

$pagination = Paginator::queryPage(
    $qb,
    page: 1,
    rpp: 20,
    filters: $filters,
    orderBy: $orderBy,
    filtersMode: Filters::MODE_OR,
);
```

This matters when your list screen needs broader matching rules than the default `AND` behavior.

## Query Aggregates On The Same Filtered Query {#query-aggregates-on-the-same-filtered-query}

A listing page often needs more than rows. It may also need totals or summary values.

Use `Paginator::queryAggregate()` when those values should follow the same filters:

```php
<?php

$totals = Paginator::queryAggregate($qb, [
    'totalAmount' => 'SUM(i.amount)',
    'avgAmount' => 'AVG(i.amount)',
], [
    'status' => 'paid',
]);
```

This is useful for screens such as:

- invoice lists with total billed amount
- order lists with revenue summaries
- report pages with one headline metric and one paginated table

## Work With PaginatedCollection {#work-with-paginatedcollection}

`PaginatedCollection` behaves like a Doctrine collection, but adds pagination data and helpers.

Useful metadata methods:

- `getPage()`
- `getRpp()`
- `getTotal()`
- `getPages()`
- `getFirstPage()`
- `getLastPage()`
- `getNextPage()`
- `getPrevPage()`
- `isFirstPage()`
- `isLastPage()`

That means the same object can move cleanly from controller to template without building a separate pagination array.

## Build Pagination Controls {#build-pagination-controls}

`PaginatedCollection` can generate a collapsed page list:

```php
<?php

$pages = $pagination->collapsedPages(7, true);
```

This is useful when you want a compact paginator with ellipsis-like gaps.

Typical output shape:

```php
[1, null, 4, 5, 6, null, 10]
```

You can render `null` as `...` in Twig.

## Generate URLs For Sorting And Page Links {#generate-urls-for-sorting-and-page-links}

The collection also includes helpers that reuse the current request query string:

- `getPageUrl()`
- `getFirstPageUrl()`
- `getLastPageUrl()`
- `getNextPageUrl()`
- `getPrevPageUrl()`
- `getSortUrl()`
- `getSortToggleUrl()`

Example:

```php
<?php

$nextUrl = $pagination->getNextPageUrl($request);
$nameSortUrl = $pagination->getSortToggleUrl($request, 'name');
```

This is one of the most useful parts of the package for classic server-rendered UIs.

## Twig Example {#twig-example}

You can iterate the collection like any other collection:

```twig
<table>
    <thead>
        <tr>
            <th>
                <a href="{{ pagination.sortToggleUrl(app.request, 'name') }}">
                    Name
                </a>
            </th>
            <th>
                <a href="{{ pagination.sortToggleUrl(app.request, 'email') }}">
                    Email
                </a>
            </th>
        </tr>
    </thead>
    <tbody>
    {% for user in pagination %}
        <tr>
            <td>{{ user.name }}</td>
            <td>{{ user.email }}</td>
        </tr>
    {% endfor %}
    </tbody>
</table>
```

You can also render page metadata:

```twig
<p>
    Page {{ pagination.page }} of {{ pagination.pages }}
    - {{ pagination.total }} results
</p>
```

And a basic paginator:

```twig
<nav>
    <ul class="pagination">
        {% for page in pagination.collapsedPages(7, true) %}
            {% if page is null %}
                <li>...</li>
            {% else %}
                <li class="{{ page == pagination.page ? 'active' : '' }}">
                    <a href="{{ pagination.pageUrl(app.request, page) }}">{{ page }}</a>
                </li>
            {% endif %}
        {% endfor %}
    </ul>
</nav>
```

## Use PaginatorForm For GET Listing Forms {#use-paginatorform-for-get-listing-forms}

`PaginatorForm` is the most practical entry point when a listing page is driven by a Symfony form.

It extends `FiltersForm` from `doctrine-query-filters` and adds the hidden fields used for:

- current page
- results per page
- order field
- sort direction

That makes it a good base type for admin filters or search forms submitted through `GET`.

## Create A Listing Form {#create-a-listing-form}

Example:

```php
<?php

namespace App\Form;

use App\Entity\User;
use Softspring\Component\DoctrinePaginator\Form\PaginatorForm;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class UsersListFilterForm extends PaginatorForm
{
    public function configureOptions(OptionsResolver $resolver): void
    {
        parent::configureOptions($resolver);

        $resolver->setDefaults([
            'class' => User::class,
            'rpp_valid_values' => [20, 50, 100],
            'rpp_default_value' => 20,
            'order_valid_fields' => ['id', 'name', 'email'],
            'order_default_value' => 'name',
        ]);
    }

    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        parent::buildForm($builder, $options);

        $builder->add('search', TextType::class, [
            'property_path' => '[name__like___or___email__like]',
            'required' => false,
        ]);
    }
}
```

Two details matter here:

- always call `parent::configureOptions()`
- always call `parent::buildForm()`

That is what adds the paginator fields and base form behavior.

## Process The Form In A Controller {#process-the-form-in-a-controller}

The common controller flow is:

```php
<?php

public function list(Request $request): Response
{
    $form = $this->createForm(UsersListFilterForm::class)->handleRequest($request);
    $pagination = Paginator::queryPaginatedFilterForm($form, $request);

    return $this->render('users/list.html.twig', [
        'form' => $form->createView(),
        'pagination' => $pagination,
    ]);
}
```

This is the main value of `PaginatorForm`: you stop wiring page, rpp, order, direction, and filters by hand in every listing action.

## Important PaginatorForm Options {#important-paginatorform-options}

The options you will use most often are:

- `class`
- `query_builder`
- `rpp_valid_values`
- `rpp_default_value`
- `order_valid_fields`
- `order_default_value`

Field name options are also available:

- `page_field_name`
- `rpp_field_name`
- `order_field_name`
- `order_direction_field_name`

Those are useful when your frontend already uses a different query-string convention.

## Let The Form Adjust The QueryBuilder {#let-the-form-adjust-the-querybuilder}

If a listing form needs to adjust the query builder before pagination runs, implement `QueryBuilderProcessorInterface`.

This is useful when:

- a filter requires a conditional join
- sorting depends on a custom select
- the base query changes depending on submitted filters

That keeps query preparation close to the form type that owns the listing behavior.

## Recommended Usage Pattern {#recommended-usage-pattern}

This component works best when:

- your application already uses server-rendered listing pages
- filters are submitted through GET
- sorting is explicit and limited to allowed fields
- the same screen needs both query results and pagination helpers

Typical examples:

- admin user listings
- invoices and orders backoffice screens
- filtered report pages
- CRUDL index pages

## Practical Limits {#practical-limits}

Keep these limits in mind:

- it targets Doctrine ORM query builders, not generic data sources
- it is designed for classic page-based pagination, not cursor pagination
- URL helpers are intentionally simple and based on the current request path and query string
- filter semantics come from `doctrine-query-filters`, so complex filter behavior belongs there

## Summary {#summary}

Choose `doctrine-paginator` when you want classic Symfony listing pages to stay simple: one query builder, one pagination object, one GET form, and reusable helpers for sorting and page links.
