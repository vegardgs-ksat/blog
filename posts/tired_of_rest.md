# Tired of REST

Ever felt uncomfortable when deciding which combination of HTTP method, path segments, path parameters to mount a new
endpoint behind? The current climate when consulting the internet is to assimilate everything into REST - whatever that
actually means. When reading up on the online resource regarding REST, one will encounter some very strong connotations
about _URI_ structure, as well as placing significant purpose behind each HTTP method. This heavily impacts the design
space of your API, if you are to follow REST as described. When applying these principles from the get-go, it may seam
obvious and easy to follow. However, as the surface area of your API grows, and your resources grow sub-resources, and
your operations require additional context or expansions - you find yourself in a complexity jungle of simply finding an
_organized_ expansion of your API.

However, the distance between what
[Roy Fielding's 2000 dissertation](https://ics.uci.edu/~fielding/pubs/dissertation/fielding_dissertation_2up.pdf)
defining REST is, and how it has been cherry-picked into common practices to develop "REST" APIs is vast. For instance,
Fielding does not mention HTTP methods at all when defining REST. The ubiquitous meaning of REST used throughout the
internet in 2025 does not match its 2000 definition. Meaning we're left with misappropriation of the word, which in and
of itself has become the new meaning behind the term. Fielding himself uttered frustration over the misappropriation of
any HTTP-based interface as REST API
[in a 2008 blog post](https://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven). He even clearly states
that a REST API should not be dependent on any single communication protocol - meaning his definition itself is not
limited to HTTP based interface.

To better understand were the definition of REST is coming from, we also need to understand the technical landscape of
the mid-to-late 90s. One of the key factors in the web at this time was addressing. Being able to locate the _data_ that
was requested in a consistent, programmatic manner. An influential technology largely deployed at the time was Common
Gateway Interface (CGI), which are dynamic scripts invoked from a fixed server location. The web was invented to share
research among peers, addressed via hyperlink addresses. These addresses formed the basis of where the data resided, and
this in turn formed the basis of a Uniform Resource Location (URL), which uniquely addresses data on a given server. As
the technology and ecosystems evolved, an idea of a location-independent unique identifiers emerged - one to serve the
parent taxonomy to existing schemes like URL, URN, ISBN, and so forth. This is what we know as Uniform Resource
Identifier (URI). This parent taxonomy was not clearly established in the initial design of URI. It was first clarified
in [W3C Note 21 September 2001](https://www.w3.org/TR/uri-clarification/) about
`URIs, URLs, and URNs: Clarifications and Recommendations 1.0`, which outlines a contemporary view that URI simply form
a superset over `URL` and `URN` - which was further cemented by
[IETF RFC3986](https://datatracker.ietf.org/doc/html/rfc3986#section-1.1.3) from 2005. Since then, there is still much
confusion in the industry, and the term `URL` is more or less adopted as the de facto mechanism - much due to the
prevalence of HTTP based APIs.

Tim Berners-Lee is largely recognized as one of the founders of the web, being the original author of the HyperText
Transfer Protocol specification, and putting it into use in research centers around the western world. After his initial
success, Berners-Lee went on to envision a complete machine-to-machine web, summarized best in his (with James Hendler &
Ora Lassila) 2001 Scientific American paper titled
`The Semantic Web: A New Form of Web Content That is Meaningful to Computers Will Unleash a Revolution of New Possibilities`.
Roy (and others) largely built upon the early HTTP work of Tim Berners-Lee, and rewrote or enhanced his earlier
specifications. His 2001 dissertation is primarily a defense of work already done, and a detailed reasoning behind these
specifications - defining an architectural style that conforms to this specification. This is a natural effect in an
attempt to promote the adoption of ones own work.

The counter-point for the REST architecture is to combat those early days viral Remote Procedure Call (RPC) interfaces
like CGI. Its key observation is that we are largely caring about data, and stateful transformations of such data.

## Present Day

For the remainder of this blog post, we will refer to the modern day incarnation of REST - however incorrect that may
seem from its original definition.

The requirements of modern day APIs are often rater complex, and the set of invariant transformations that can be
applied to a resource is oft large or plentiful. Expressing these within the confines of REST may be extremely
difficult. Here are some common pitfalls:

#### Pitfall: Search operation with many parameters

You have a central resource that you want to expose a generic search operation upon. The REST denoted method verb to use
is `GET`, since the operation is idempotent, and the response should be cache-able based upon the URL. You end up
specifying a set of filtering predicates that you cram into the query string. You have to use the query string, because
the HTTP specification limits request bodies to `PATCH`, `POST`, and `PUT` methods. It starts out not too bad. Maybe you
are filtering on a set of associated resources.

For example, your resource is the set of organizational users. Each user is the member of a single team. Your
organization starts out with a reasonable 20 teams. Performing a search for users accepts some filtering on users
belonging to any of the specified teams. Given that each team is represented with an UUID, your query string starts
growing in length rather quickly. What ends up working just fine when the organization is small-ish with 20 teams, ends
up failing when the organization grew to 200 teams. This ends up exceeding the URL limits, which arbitrarily enforced
throughout the internet ecosystem.

The worst part about this scenario, is that its remains a ticking time-bomb in your API. You _may_ of course attempt to
follow the original HTTP/1.1 specification from 1999 in [RFC 2616](https://datatracker.ietf.org/doc/html/rfc2616) and
return a `414 Request-URI Too Long` - if you are not arbitrarily cut-off by an intermediary proxy or gateway outside of
your control. Whether or not your can adequately document and guarantee this behavior on the modern web is an entirely
different story. The server/infrastructure landscape is entirely different than what was true at the time the RFC was
accepted.

This situation in the HTTP ecosystem could be remedied by the introduction of the
[HTTP `QUERY` method](https://datatracker.ietf.org/doc/draft-ietf-httpbis-safe-method-w-body/12/), which attempts to
navigate a solution space where the search parameters are expressed through the request message body instead. This draft
RFC has been actively worked on for over 10 years.

Roy Fieldings
[early review of revision 11](https://datatracker.ietf.org/doc/review-ietf-httpbis-safe-method-w-body-11-httpdir-early-fielding-2025-06-20/)
starts off rather unpromising:

> the technology being described fails to meet the basic architectural requirements for the Web and HTTP. "All important
> resources are identified by a URI" is the primary design principle of the Web. The entire system depends on it for
> linkability and scale.
>
> Likewise, there is no opportunity to just "move the request content into the cache key" and call that cacheable.
> That's a security vulnerability, not a feature.

This is a rather restrictive view of HTTP and the web, presumably still rooted back in the world where the original RFCs
were written, the Semantic Web was envisioned, and the REST architecture was solidified. It yields no inch to the very
real problems we face in today's age - it dismisses the problem by touting that we're all using the web wrong.

#### Pitfall: Bulk operations

Performing bulk operations over a REST-full API has no natural path structure. Even worse, if your bulk operation
supports providing hundreds of resource identifiers, they cannot fit into the query string for `GET` operations - made
evident by the search operation pitfall above.

Thus, we must also sacrifice the usage of the natural HTTP verbs `GET` and `DELETE`, so that we can employ the request
message body. This itself introduces ambiguity in the API over what methods are employed were, and developer
expectations are intermittently broken.

#### Pitfall: Security

Security concern is not necessarily a pitfall of REST itself, more so for employed HTTP practices in general. It is
neither a concern for the development nor consumption of the API - but rather its deployment infrastructure and request
path traversal towards the API server actually handling the request.

Some deployments of APIs (which were never designed or intended for men-in-green-suits scenario), may end up as part of
security restricted or limited environment - even if it is hosted in the cloud. There is some very real requirements
out there that any intermediate processing chain that process or audit the requests must scrub the logs for sensitive information
prior to persisting the logs. Or at very least before those logs are shared with anyone (i.e., security monitoring
software) outside of the restricted environment.

For HTTP requests where all application level data is only transmitted in the request body, this is a non-issue. However,
when the classified information is communicated over the API in query strings or path parameters, we would be required
to implement such redaction measures. This is because most reverse proxy or intermediate processing software
for an HTTP deployment nominally logs the entire request URI, including the query string,


### Caching

Caching is a large portion of the REST architecture, leaning heavily into the fact that stateless data, and idempotent
operations shall in-and-large be cache-controlled. This is influenced by
[Cache-Control header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Cache-Control). The cache
keys themselves are computed entirely based on the URI and a subset of request headers. This also follows naturally from
the REST architecture that these operations are thus `GET` methods, due to their idempotent nature, and cache
eligibleness.

What type of operations in a modern API that are cacheable is highly contextual, and dependent on the domain of the API
itself. It is also rather dependent on the scale of your operations, and the trade-offs taken when evaluating ones
position on the [CAP theorem](https://en.wikipedia.org/wiki/CAP_theorem). I'll wager that in today's ecosystem, caching
for _most_ API operations are rarely performed on the transport layer. There is a plethora of more controllable cache
mechanisms closer to the data source, whose invalidation mechanisms is reachable for the API owner.

All in all, it seems restrictive to perpetuate the necessity of REST on the basis of cache control.

### Query String complexity

With the evolving set of complexity offered by modern APIs, more and more information is required to be transferred in
the HTTP request. In lieu of unable to provide a request message body for the HTTP methods `GET` and `DELETE`, and in
the spirit of following modern day REST, this complexity has migrated into the HTTP query string, in the form of query
parameters. An effort to standardize these were made in
[RFC 6470](https://datatracker.ietf.org/doc/html/rfc6570#section-3.2.8), and was partially adopted into the
[OpenAPI](https://spec.openapis.org/oas/v3.2.html#style-values) parameter style specification. However, as noted in the
OpenAPI specification, there are also many non-standardized variants - in an attempt to capture naturally evolved
variants that has been implemented and used throughout the last few decades.

My point with this detour is to signal that even modeling vaguely standardized methods of exchanging an array of
identifiers can manifest in a query string in so many ways. Each which oft need to be communicated manually in text
documentation, instead of being obvious based on the application protocol. This is made even worse if using the
`deepObject` style to transfer full data structures (which can include arrays). Consult the
[OpenAPI style example section at your own risk](https://spec.openapis.org/oas/v3.2.html#style-examples).

### Why HTTP

You may rightfully wonder why we then bother designing our APIs on top of HTTP. If the current practices advocate for
designing complex service-to-service APIs atop protocol principles designed for the structured, automatic
machine-to-machine operated, although driven by a human on the other end. Today, we design APIs whose primary purpose is
machine-to-machine operated, and driven by a machine.

We select HTTP because it is ubiquitous. Every switch, router, proxy, and service providers have top-notch support for
HTTP. The basics of the protocol is easy to understand for all and any. You do not need advanced programming experience
to be able to construct a request, send it off, and read & understand the response. We select HTTP not because it is the
best in class, but because it is the common denominator among all.

For APIs with a narrow user-base, or whose audience is advanced users - there is a plethora of other API protocols that
are more suited for the task. Hopefully, any of these will become as approachable as HTTP in the future.

## Embrace RPC over HTTP: a predictable alternative

Based on all the above battle scar, I would like to propose an alternative. For most complex HTTP APIs out there, those
that have many invariants and constrains in their actions, with cascading effects throughout the system, one should
embrace the RPC mechanism Fielding rejected. Let us call this RPC embraced HTTP API something along the lines of
Predictable Transfer of Stateful Data - PTSD-HTTP, for short.

This is an _opinionated_ set of guidelines that attempts to take the ambiguity out of the design space for how to
structure the HTTP API operation. I will summarize them in individual sections.

#### Rule: Method

All operations primarily rely on HTTP Methods: `POST`, `PATCH`, `PUT`.

- `GET` may be used only for operations where we are confident in the cache-ability in the resource, and _must_ set cache
  control headers with a sufficiently large window.
- `GET` may be used for locating a resource by a unique qualifier (whose serialization is a query parameter) ONLY when
  you are unable to convince your colleges who are still stuck in the REST-bubble.

#### Rule: HTTP paths are operation names

All HTTP paths are unique, regardless of HTTP method. This entails no _path parameters_.

Examples of HTTP paths:

- `/users/locate/unit`
- `/users/locate/search`
- `/groups/fetch`

#### Rule: Pagination

All pagination for `application/json` is done through the HTTP message body.

This is motivated by multiple factors. A paginated response is _often_ stale the moment it is issued, and cannot be
reasonably cached. Secondly, some pagination designs may rely on supplying the search predicates on each request, or
provides a huge opaque cursor. By always forcing the pagination into the HTTP request body, we have a consistent rule set
for providing it, regardless of chosen pagination mechanism.

There a multiple pagination mechanisms that can be applicable to ones specific API domain. This rule does not enforce
any specific mechanism.

It does however specify the following structure and field names for some predictability.

Request HTTP message body:

```json
{
  "pagination": {
    "cursor": "..,",
    "limit": 1000
  }
}
```

Response HTTP message body:

```json
{
  "metadata": {
    "pagination_cursor": "..",
    "pagination_size": 1000
  }
}
```

All pagination _responses_ for non-`application/json` is done through HTTP Headers

- `Pagination-Cursor`: For cursor based API
- `Pagination-Size`: The number of elements returned in the response, if applicable.

### Terminology: actions

This is a non-exhaustive list of various words we associate with a particular action, and how we should unequivocally
interpret their meaning when they appear in an operation name.

- `assign` - Set or update a value or resource binding.
- `find` - Lookup a resource by some unique qualifier, returning a single resource. If no resource exists by this
  identifier, an _empty_ successful response is returned.
- `locate` - Lookup a resource by some unique qualifier, returning a single resource. If no resource exists by this
  identifier, an erroneous response is returned.
- `create` - Creates a new resource instance.
- `update` - Update _piece wise_ properties of an existing resource. This operation is suited for resources with
  individual fields.
- `replace` - Update _all_ content of an existing resource. This operation is suited for blob-like resources, e.g., like
  files opaque JSON objects.
- `search` - An operation to retrieve multiple resource instances in a paginated response. There may be filtering
  predicates to limit the resources returned based on properties on or associated with the resource.
- `fetch` - An operated to fetch a fixed or limited number of resources, without any filtering predicates. The response
  _cannot_ be paginated.
- `resolve` - An operation that returns a single resource based on some predicates. If no resource exists, the operation
  fails with an erroneous response.
- `batch` - Process an existing set of resources, usually collected over many previous operations. Does not necessarily
  reference each resource directly. No transactional guarantees can be given for a batch operation.
- `bulk` - Process the provided set of resources. Used to provide a mechanism for the caller to perform many individual
  operations in a single invocation. There may also be transactional guarantees to the operation.

I have yet to land on good definitions for `remove`, `delete`, or `archive` - when those are applicable.

There may be resource specific action names that fit better for that specific resource, and it should take precedence
over these basis terms.

### Operation Names

This is the ideal template of various path segments to construct an operation name that is applicable to most, if not
all, API operations:

- `/{resource}/{action}{/quantifier}`
  - `{resource}`: can be one or more path segments that best articulate the resource, or its variation.
    - e.g., `/users`, `/users/admins`, `/user-admins`
  - `{action}`: An application and resource specification action, scoped to the `{resource}` namespace.
  - `{/quanitfier}`: An optional suffix to another action. May itself be an action.
    - e.g., `/users/create/bulk`, `/users/locate/single`.

Here is the classical OpenAPI petstore example expressed as these operation names:

- `GET /pets` -> `POST /pets/search`
  - `limit` parameter serialized in request body.
  - `next` URI link serialized in response body, `metadata` object.
- `POST /pets` -> `POST /pets/create`
- `GET /pets/{petId}` -> `/pets/locate/single`
  - `petId` parameter serialized in request body.

## Attribution

Some key insight into this topic was gathered from the following
[Hacker News](https://news.ycombinator.com/item?id=23670238) discussion.
