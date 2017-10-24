# Reversed relations specification for Site API

## RelationService

Methods specified here return all available \Netgen\EzPlatformSiteApi\API\Values\Content objects by some criteria.

#### Specification `loadReverseFieldRelations` method

```php
/**
 * Load all content which has relation to given contentId in one of given fieldDefIdentifiers.
 * Optionally, array of content type identifiers can be provided, to filter by content type.
 * Also, a limit can be provided. Otherwise, default one will be used.
 *
 * @param int|string $contentId
 * @param array $fieldDefIdentifiers
 * @param array $contentTypeIdentifiers
 * @param int $limit MAYBE TO USE DEFAULT REPOSITOR LIMIT??
 *
 * @return \Netgen\EzPlatformSiteApi\API\Values\Content[]
 */
public function loadReverseFieldRelations($contentId, array $fieldDefIdentifiers, array $contentTypeIdentifiers = array(), $limit = 25);
```

DIGRESSION: would it make more sense to pair fieldDefIdentifiers with contentTypeIdentifiers?

#### Specification `loadReverseFieldRelation` method

This one doesn't make sense since there is always a possibility of multiple relations. Even if we have, for example, an Article which has single relation to Author, when we fetch articles for an author, there will be more than one articles.

## Content

All specified methods return \Netgen\EzPlatformSiteApi\API\Values\Content objects that can be safely displayed on frontend.

#### Specification `getReverseFieldRelations` method

```php
/**
 * Load all content which has relation to current content in one of given fieldDefIdentifiers.
 * Also, a limit can be provided. Otherwise, default one will be used.
 *
 * @param int|string $contentId
 * @param array $fieldDefIdentifiers
 * @param int $limit MAYBE TO USE DEFAULT REPOSITOR LIMIT??
 */
public function getReverseFieldRelations(array $fieldDefIdentifiers, $limit = 25);
```

Usage in Controller for example:
```php
$relatedContents = $content->getReverseFieldRelations('my_related_articles');
```
Usage in templates:
```jinja
{% for related_content in content.getReverseFieldRelations('my_related_articles') %}
	{{ related_content.name }}
{% endfor %}
```

#### Specification `filterReverseFieldRelations` method

```php
/**
 * Load all content which has relation to current content in one of given fieldDefIdentifiers.
 * Optionally, array of content type identifiers can be provided, to filter by content type.
 * Also, a limit can be provided. Otherwise, default one will be used.
 *
 * @param int|string $contentId
 * @param array $fieldDefIdentifiers
 * @param array $contentTypeIdentifiers
 * @param int $limit MAYBE TO USE DEFAULT REPOSITOR LIMIT??
 */
public function filterReverseFieldRelations(array $fieldDefIdentifiers, array $contentTypeIdentifiers, $limit = 25);
```

Usage in Controller for example:
```php
$relatedContents = $content->filterReverseFieldRelations('my_related_articles', ['ng_article']);
```
Usage in templates:
```jinja
{% for related_content in content.filterReverseFieldRelations('my_related_articles', 'ng_article']) %}
	{{ related_content.name }}
{% endfor %}
```

## Implementation Notes

Example for reverse related content search:
```php
$query = new Query();

$criteria = array(
    new Criterion\LogicalOr(array(
        new Criterion\FieldRelation(
            'bpost',
            null,
            array($view->getContent()->id)
        ),
    ))
);

/* if filtering by content identifiers is set */
if (!empty($contentTypeIdentifiers)) {
	$criteria[] = new Criterion\ContentTypeIdentifier($contentIdentifier);
}

$query->filter = new Criterion\LogicalAnd($criteria);

$findService->findContent($query);
```

I've tested this on current ngmore stack with different use cases and I think it's the best to use array() for value in FieldRelation. It works both for ObjectRelations and ObjectRelation.

## Example use case

On Frydenbo, we have car brands (such as Volvo, Nissan etc.). Each car brand has relations to stores which sell cars from this brand. We need to display all brands which a store sells, for each store, which means that we have to fetch all brands which have relations to this store. So, we need a reverse related content (brands) for this store.