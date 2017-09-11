---
title: API
layout: default
permalink: api/
---

# API

The API adheres to [json:api](http://jsonapi.org) standards. All endpoints are available without throttling.

### Get public data

Prepend all requests with `https://panoptikum.io/jsonapi`.

All instances in "relationships" can be found with a link and a minimal set of attributes in "included".
You can basically explore the whole API world of Panoptikum by starting at /categories and following
links from there recursively.

path | method | purpose | included
--- | --- | ---
`/categories` | GET | tree of categories | children = subcategories
`/categories/:id` | GET | single category | children, (paginated) podcasts, parent ; podcasts ordered by last episode publishing date descending nulls last
`/podcasts` | GET | list of podcast, paginated, ordered by insertion date descending | categories, languages, engagements & contributors (= personas)
`/podcasts/:id` | GET | single podcast | (paginated) episodes, subscription_count, engagements, recommendations, categories, contributors (= personas, follower_count, likes_count, languages, feeds
`/podcasts/most_liked` | GET | 10 most liked podcasts ordered by like count descending | -
`/podcasts/most_subscribed` | GET | 10 most subscribed podcasts ordered by subscription count descending | -
`/podcasts/last_updated` | GET | last updated podcasts | paginated
`/languages` | GET | list of languages | -
`/languages/:id` | GET | single language | -
`/engagements/:id` | GET | engagement | persona, podcast
`/personas/:id` | GET | single persona | redirect, engagements & podcasts, (paginated) gigs & episodes, delegates
`/recommendations` | GET | list of recommendations, paginated, order by insertion date descending | episode, podcast, chapter, user
`/recommendations/:id` | GET | single recommendation | user-name, one of: podcast, episode, chapter |
`/recommendations/random` | GET | random recommendation | episode, podcast, category;<br/> the episode belongs to that podcast, the podcast belongs to that category
`/feeds/:id` | GET | single feed | podcast, alternate_feed
`/alternate_feeds/:id` | GET | single alternate feed | feed
`/episodes` | GET | list of episodes, paginated, ordered by publishing_date descending | podcast, gigs & contributors (= personas)
`/episodes/:id` | GET | single episode | podcast, chapters, enclosures, recommendations, gigs & contributors (= personas), like_count
`/chapters/:id` | GET | single chapter | episode, recommendations, like_count
`/enclosure/:id` | GET | single enclosure | episode
`/gigs/:id` | GET | single gig | persona, episode
{: .table .table-bordered}


### Pagination

Podcasts within a category, gigs for a persona and episodes for a podcast are paginated:

* Example request: `https://panoptikum.io/jsonapi/personas/1?page[number]=1&page[size]=10`
* page[number] ... page number; starts counting at 1, defaults to 1
* page[size] ... number of items per page, defaults to 10
* Links contain self, prev, next, last and first link (if appropriate).


### Full Text Search

path | method | purpose | included
--- | --- | ---
<nobr><code class="highlighter-rouge">/search?filter[:type]=:term</code></nobr> | GET | redirects to the appropriate route below | type can be either `persona`,`category`,`podcast` or `episode`
<nobr><code class="highlighter-rouge">/categories/search?filter=:term</code></nobr> | GET | searches for categories with `:term` | children = subcategories
<nobr><code class="highlighter-rouge">/podcasts/search?filter=:term</code></nobr> | GET | searches for podcasts with `:term` | categories, languages, engagements & contributors (= personas)
<nobr><code class="highlighter-rouge">/episodes/search?filter=:term</code></nobr> | GET | searches for episodes with `:term` | podcast, gigs & contributors (= personas)
<nobr><code class="highlighter-rouge">/personas/search?filter=:term</code></nobr> | GET | searches for personas with `:term` | redirect (= persona) , delegates (= personas), podcasts
{: .table .table-bordered}


### Login

To be able to post data and receive your own private data, it's necessary to post a token along
side. To get a token, post username and password to recieve a token back, that's valid for one hour.

Typically, you would call something a login, though it's only getting you a new token,
we provide both routes as endpoints, take the one with the name you prefer.
There is no session or cookie connected to that, so there is also no logout.

path | method | params | purpose | included
`/login` or `/get_token` | POST | `username`, `password` | get a token | token including validity data and user id
{: .table .table-bordered}

One user can access the application via the api from different devices simuntaneously. They are not
device specific.

#### *Example*

If working with tokens is something new for you, an example might help to set the expectations right.

Request: `curl --data "username=janedoe&password=secret" http://localhost:4000/jsonapi/login`

Response:
```
{"jsonapi":
  {"version":"1.0"},
  "data":{"type":"session-api",
          "id":"6",
          "attributes":{"valid-until":"2017-09-08T16:46:08.393642Z",
                        "valid-for":"1 hour",
                        "token":"SFMyNTY.g3QAAAACdwRk....yNXj3wvSs9a9Ps5wO6yrY",
                        "created-at":"2017-09-08T15:46:08.393630Z"}}}
```

#### *A note on security*

You get the token in a transport encrypted response via https. If you disclose the token, you
basically disclose your password. This is the case because the token owner can change the password
within the next hour, no matter if you recreated another token. So don't store it unencrypted.

### Post data and get private data

These endpoints are currently under development.