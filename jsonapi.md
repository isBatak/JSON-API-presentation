---
marp: false
theme: default
---

![JSON:API](https://raw.githubusercontent.com/patrickcate/nuxt-jsonapi/main/playground/public/nuxt-jsonapi-logo.svg)

---

## JSON:API

> BAD naming!

Homepage: [jsonapi.org](https://jsonapi.org/)

### It's a Specification!

- A standardized specification
- Minimize the number of requests
- Minimize amount of data transmitted between clients and servers.
- Follows **Internet Engineering Task Force** (IETF) standards.


---

## What is REST?

### Representational State Transfer

> It is not a standard, rather a style describing the act of transferring a state of something by its representation.

---


![REST](https://www.graphiti.dev/assets/img/rest1.gif)

---

## What is HATEOAS?

**Hypermedia As The Engine Of Application State**

> A constraint of REST that requires a server response to include hypermedia links to related resources.

---

![HATEOAS](https://www.graphiti.dev/assets/img/rest2.gif)

---

## What problems does JSON:API solve?

1. Overfetching
2. Uderfetching
3. Waterfalls
4. Normalization
5. URL consistency
6. Uniqueness
7. Discoverability

---

## Content Negotiation

Content Type: `application/vnd.api+json`

---

# CONCEPTS

---

## Document

- ✉️ Response Envelope 
- JSON:API wraps all responses in a **Document**.
- Ensures a consistent response structure.

### Top level Document

- Document **MUST** contain at least one of: `data`, `errors`, or `meta`.
- Document **MAY** contain any of: `jsonapi`, `links`, `included`.

---

Example:

```json
{
  "data": [],
  "errors": [],
  "jsonapi": {},
  "links": {},
  "meta": {}
}
```

---

## Resource Objects

- A **resource** in JSON:API represents an entity or object.
- **MUST** contain at least `type` and `id`.
- `id` can be replaced with `lid` for client-generated IDs.
- **MAY** contain `attributes`, `relationships`, `links`, and `meta`.

---

Example:

```json
{
  "type": "articles",
  "id": "1",
  "attributes": {},
  "relationships": {},
  "links": {},
  "meta": {}
}
```

---

## Resource Identifier Objects

- A **resource identifier** is a unique reference to a resource.
- **MUST** contain `type` and `id`.
- **MAY** have `lid` for client-generated IDs.
- **MAY** contain `meta`.

---

Example:

```json
{
  "type": "articles",
  "id": "1"
}
```

---

## Fields

- A resource object’s `attributes` and its `relationships` are collectively called its `fields`.

---

Plane JSON response with `name` and `address` fields:

```json
[{
  "name": "John Doe",
  "address": {
    "street": "123 Main St",
    "city": "New York",
    "state": "NY"
  }
},
{
  "name": "Jane Doe",
  "address": {
    "street": "123 Main St",
    "city": "New York",
    "state": "NY"
  }
}]
```
---

JSON:API counterpart:

```json
{
  "data": [{
    "type": "people",
    "id": "1",
    "attributes": { "name": "John Doe" },
    "relationships": {
      "address": {
        "data": { "type": "addresses", "id": "1" }
      }
    }
  },
  {
    "type": "people",
    "id": "2",
    "attributes": { "name": "Jane Doe" },
    "relationships": {
      "address": {
        "data": { "type": "addresses", "id": "1" }
      }
    }
  }],
  // ...
```

---

```json
  // ...
  "included": [
    {
      "type": "addresses",
      "id": "1",
      "attributes": { 
        "street": "123 Main St", 
        "city": "New York", 
        "state": "NY"
        }
    }
  ]
}
```

---

## Relationships

---

### Types of Relationships
- **toOne**: A relationship where a resource has one related resource.
- **toMany**: A relationship where a resource is related to multiple resources.

---

**toOne** example: 

```json
{
  "type": "articles",
  "id": "1",
  "attributes": { "title": "JSON:API" },
  "relationships": {
    "author": {
      "data": { "type": "people", "id": "9" }
    }
  },
  "included": [
    {
      "type": "people",
      "id": "9",
      "attributes": { "name": "John Doe" }
    }
  ]
}
```

---

**toMany** example:

```json
{
  "type": "articles",
  "id": "1",
  "attributes": { "title": "JSON:API" },
  "relationships": {
    "comments": {
      "data": [
        { "type": "comments", "id": "5" },
        { "type": "comments", "id": "12" }
      ]
    }
  },
  "included": [
    {
      "type": "comments",
      "id": "5",
      "attributes": { "body": "Great article!" }
    },
    {
      "type": "comments",
      "id": "12",
      "attributes": { "body": "I learned a lot!" }
    }
  ]
}
```

---

### Polymorphic Relationships
- Allows for relationships to multiple types of resources.

> **Note**: Not defined in the standard but can be implemented as a profile.

---

Example `vehicles` relationship with different attributes:

```json
{
  "type": "car-dealers",
  "id": "1",
  "attributes": { "name": "Awesome Cars Dealership" },
  "relationships": {
    "vehicles": {
      "data": [
        { "type": "vehicles.cars", "id": "car-5" },
        { "type": "vehicles.trucks", "id": "truck-12" }
      ]
    }
  },
  // ...
```

---
```json
  // ...
  "included": [
    {
      "type": "vehicles.cars",
      "id": "car-5",
      "attributes": { "make": "Toyota", "model": "Corolla", "type": "sedan" }
    },
    {
      "type": "vehicles.trucks",
      "id": "truck-12",
      // Truck attributes are different then car attributes
      "attributes": { "make": "Ford", "model": "F-150", "capacity": "1 ton" }
    }
  ]
}
```

---

## URL Structure

---

```
/[resource-type]/[id]/relationships?/[related-resource-field-name]/[related-id]
```

- **Collection URL**: `/articles`
- **Single Resource URL**: `/articles/1`
- **Relationship URL**: `/articles/1/relationships/author`
- **Related Resource URL**: `/articles/1/author`

---

## Links in Relationships

- **self**: `/articles/1/relationships/author`
  - A link to the relationship itself.
  - Used for updating or deleting the relationship data on a parent resource.
  - Returns the linkage data for the relationship. 

- **related**: `/articles/1/author`
  - A link to the related resource(s).
  - Used for retrieving the related resource(s) complete representation.

---

Example:

```json
{
  "type": "articles",
  "id": "1",
  "attributes": { "title": "JSON:API" },
  "relationships": {
    "author": {
      "links": {
        "self": "/articles/1/relationships/author",
        "related": "/articles/1/author"
      },
      "data": { "type": "people", "id": "9" }
    }
  }
}
```

---

## Compound Documents

---

### Compound Documents
- A single request can fetch multiple resources and their relationships.
- Reduces the number of requests
- Avoid waterfalls
- `include` parameter in the query string.
- `included` key in the response.

> DDOS incident: If ignored client can potentially DDOS itself if for example when the UI have a big table with many cells and each cell is a separate request. Usually happens in Admin like applications.

---

Example:

```json
GET /articles/1?include=author,comments

{
  "data": {
    "type": "articles",
    "id": "1",
    "attributes": { "title": "JSON:API" },
    "relationships": {
      "author": {
        "data": { "type": "people", "id": "9" }
      },
      "comments": {
        "data": [
          { "type": "comments", "id": "5" },
          { "type": "comments", "id": "12" }
        ]
      }
    }
  },
  // ...
```

---

```json
  // ...
  "included": [
    {
      "type": "people",
      "id": "9",
      "attributes": { "name": "John Doe" }
    },
    {
      "type": "comments",
      "id": "5",
      "attributes": { "body": "Great article!" }
    },
    {
      "type": "comments",
      "id": "12",
      "attributes": { "body": "I learned a lot!" }
    }
  ]
}
```

---

## Sorting

---

### Sorting in JSON:API

- Commonly used query parameter: `sort`.
- Sort resources based on one or more fields.
  - **Multiple fields** can be separated by commas `,`.
- Sort order can be **ascending** or **descending** by adding a `-` prefix.

---

Example:

```json
GET /articles?sort=title,-created

{
  "data": [
    { "type": "articles", "id": "1", "attributes": { "title": "JSON:API" } },
    { "type": "articles", "id": "2", "attributes": { "title": "RESTful APIs with JSON:API" } },
    // ...
  ]
}
```

---

## Filtering

---

### Filtering Data

- Filters are applied via the `filter` query parameter.

- the filter query parameter family includes parameters named: `filter`, `filter[x]`, `filter[]`, `filter[x][]`, `filter[][]`, `filter[x][y]`, `filter[x.y]`, etc.


### Examples
- `filter[author.status]=active`: Filters resources by status.
- `filter[createdAt][gte]=2021-01-01`: Filters resources created after January 1st, 2021.

---

Example:

```json
GET /articles?filter[author.status]=active

{
  "data": [
    { "type": "articles", "id": "1", "attributes": { "title": "JSON:API" } },
    { "type": "articles", "id": "2", "attributes": { "title": "RESTful APIs
    // ...
  ]
}
```

---

## Pagination

---

### Pagination in JSON:API
- Commonly used query parameters:
  - `page[number]`: Page number.
  - `page[size]`: Number of resources per page.
- `meta` can be used to provide additional pagination information.

---

### Examples of Pagination Links
- **first**: URL for the first page.
- **last**: URL for the last page.
- **prev**: URL for the previous page.
- **next**: URL for the next page.

---

Example:

```json
GET /articles?page[number]=2&page[size]=10

{
  "data": [
    { "type": "articles", "id": "11", "attributes": { "title": "JSON:API" } },
    { "type": "articles", "id": "12", "attributes": { "title": "RESTful APIs with JSON:API" } },
    // ...
  ],
  "links": {
    "first": "/articles?page[number]=1&page[size]=10",
    "prev": "/articles?page[number]=1&page[size]=10",
    "next": "/articles?page[number]=3&page[size]=10",
    "last": "/articles?page[number]=10&page[size]=10"
  },
  "meta": {
    "totalPages": 10,
    "totalCount": 100,
    "pageSize": 10
  }
}
```

---

## Sparse Fieldsets

---

### Sparse Fieldsets Overview
- Reduce the amount of data returned by specifying which fields to include.
- Achieved using the `fields` query parameter.

### Examples
- `fields[articles]=title,body`: Only return the `title` and `body` fields for `articles`.
- `fields[people]=name`: Only return the `name` field for `people`.

---

Example:

```json
GET /articles/1?fields[articles]=title,body

{
  "data": {
    "type": "articles",
    "id": "1",
    "attributes": { "title": "JSON:API", "body": "..." }
  }
}
```

---

## Error Objects

---


- **MUST** be returned as an `array` keyed by `errors` in the top level of the response.
- Error Object Structure:
  - **MAY** have the following members, and **MUST** contain at least one of:

---


  - **id**: Unique identifier for this error.
  - **links**: Links related to the error.
    - **about**: URL to a page containing more information about the error.
    - **type**: URL to a document describing the error.
  - **status**: HTTP status code.
  - **code**: Application-specific error code.
  - **title**: Short, human-readable summary of the problem.
  - **detail**: Detailed human-readable explanation.
  - **source**: Where the error originated in the request.
    - **pointer**: JSON Pointer to the location of the error [rfc6901](https://datatracker.ietf.org/doc/html/rfc6901).
    - **parameter**: Query parameter that caused the error.
    - **header**: Header that caused the error.
  - **meta**: Additional information about the error.

---

Example:

```json
{
  "errors": [
    {
     "id": "123",
     "links": {
       "about": "https://example.com/docs/error-123",
        "type": "https://example.com/docs/error-type"
      },
      "status": "404",
      "code": "8890123",
      "title": "Resource not found",
      "detail": "The requested resource could not be found.",
      "source": {
        "pointer": "/data/attributes/id",
        "parameter": "id",
        "header": "Authorization",
      },
      "meta": {
        "requestId": "abc123"
      }
    }
  ]
}
```

---

## Extensions/Profiles

---

### JSON:API Extensions vs. Profiles
- [**Extensions**](): Custom modifications or additions to the standard JSON:API behavior.
  - **MUST** have a namespace separated bg :, e.g. `bulk:data`.
  - **Atomic Operations**: Batch operations like multiple create, update, delete in a single request.
- [**Profiles**](https://jsonapi.org/profiles/ethanresnick/cursor-pagination/): Define optional, non-breaking additions to the base specification. 
  - **Cursor Pagination**: Pagination method using cursors rather than offset-based pagination.
  - **Timestaps**: Standardized way to represent timestamps in JSON:API.

---

### Atomic Operations extension example:
```json
POST /operations HTTP/1.1
Host: example.org
Content-Type: application/vnd.api+json;ext="https://jsonapi.org/ext/atomic"
Accept: application/vnd.api+json;ext="https://jsonapi.org/ext/atomic"

{
  "atomic:operations": [{
    "op": "add",
    "href": "/blogPosts",
    "data": {
      "type": "articles",
      "attributes": {
        "title": "JSON API paints my bikeshed!"
      }
    }
  }]
}
```

---

### Cursor Pagination Profile:

```bash
GET /example-data?page[after]=abcde&page[size]=2
```

```json
{
  "links": {
    "prev": "/example-data?page[before]=yyy&page[size]=2",
    "next": "/example-data?page[after]=zzz&page[size]=2"
  },
  "data": [
    // the pagination item metadata is optional below.
    { "type": "examples", "id": "7", "meta": { "page": { "cursor": "yyy" } } },
    { "type": "examples", "id": "8", "meta": { "page": { "cursor": "zzz" } }  }
  ]
}

```

---

### Timestamps Profile:

```json
{
  "data": {
    "type": "articles",
    "id": "1",
    "attributes": {
      "title": "JSON:API",
      "timestamps": {
        "created": "2021-01-01T12:00:00Z",
        "updated": "2021-01-01T12:00:00Z"
      }
    }
  }
}
```

---

### Custom Extensions

- **Bulk Create**: [JSON:API Bulk Create Extension](https://github.com/jelhan/json-api-bulk-create-extension#jsonapi-bulk-create-extension)
  - Allows creating multiple resources in a single request.
- **Filters**: [JSON API Fancy Filters](https://gist.github.com/e0ipso/efcc4e96ca2aed58e32948e4f70c2460)
  - Allows filtering resources based on custom criteria.
  - [Filter by relationship](https://gist.github.com/e0ipso/1acbced683908ff631e3)
  - [Additional info](https://www.drupal.org/docs/core-modules-and-themes/core-modules/jsonapi-module/filtering)
  - A simpler example [here](https://www.graphiti.dev/quickstart#querying)
- **Relative Fields**: [Relative Fields Extension](https://github.com/ThorstenSuckow/relfield)
  - Allows inclusion and exclusion of fields

---

### Actions (RPC)

- **Actions**: Custom operations that don't fit into the standard CRUD model.
- Laravel JSON:API: [Custom Actions](https://laraveljsonapi.io/docs/1.0/routing/custom-actions.html)
- My attempt: [JSON:API Actions](https://gist.github.com/isBatak/87efcbc665a63725424cdb345845524d)

---

## Resources

### Further Reading and Resources
- JSON:API Discussion Forum: [discuss.jsonapi.org](https://discuss.jsonapi.org/)
- JSON:API Specification: [jsonapi.org](https://jsonapi.org/)
- Why Graphiti: [graphiti.dev/why](https://www.graphiti.dev/guides/why)
- Laravel Implementation: [laraveljsonapi.io](https://laraveljsonapi.io/)
- .NET Implementation: [jsonapi.net](https://www.jsonapi.net/)
- Drupal JSON:API: [jsonapi](https://www.drupal.org/docs/core-modules-and-themes/core-modules/jsonapi-module/jsonapi)