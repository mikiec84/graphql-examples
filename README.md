# GraphQL Examples

[![Build Status](https://travis-ci.org/uniiverse/graphql-examples.svg?branch=master)](https://travis-ci.org/uniiverse/graphql-examples)

In this repository you'll find some examples of using our [GraphQL API](https://developers.universe.com/page/graphql-explorer).

## Contents

* [Authentication](#authentication)
* [Queries](#queries)
  * [Viewer](#viewer)
  * [List of Events](#list-of-events)
  * [Event](#event)
  * [Orders](#orders)
* [Mutatations](#mutations)

## Authentication

1. Create a [new application](https://www.universe.com/oauth/applications).
2. Get an authorization code after signing in with the application.
3. Get an access token with the authorization code.

<details><summary>Details</summary>
<p>

```sh
export APPLICATION_ID=...
export APPLICATION_SECRET=...
export REDIRECT_URI=...
export AUTHORIZATION_CODE=...

curl --url 'https://www.universe.com/oauth/token' \
  --header 'content-type: application/json' \
  --data "{\"grant_type\":\"authorization_code\", \"client_id\":\"$APPLICATION_ID\", \"client_secret\":\"$APPLICATION_SECRET\", \"redirect_uri\":\"$REDIRECT_URI\", \"code\":\"$AUTHORIZATION_CODE\"}"
```

Here is a quick demo which demonstrates how to get an access token at Universe in 1 minute:

<a href="https://www.youtube.com/watch?v=BafbpoahNJU">
  <img src="./images/thumbnail.png" alt="demo" style="max-width:100%;" height="441px" width="560px">
</a>

For more information about OAuth 2.0, please see our [guide](https://developers.universe.com/docs/authorizing-with-oauth).

</p>
</details>

## Queries

### Viewer

`Viewer` is the user who makes the current GraphQL request. To see the account information:

<details name="curl/queries_viewer.sh"><summary>cURL example</summary>
<p>

```sh
export ACCESS_TOKEN=your_access_token

curl https://www.universe.com/graphql/beta \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d @- << EOF
{
  "query": "query GraphqlExample {
    viewer {
      id
      firstName
      lastName
    }
  }"
}
EOF
# => {
#   "data": {
#     "viewer": {
#       "id": "58d98e52bbdebd003804e065",
#       "firstName": "Evgeny",
#       "lastName": "Li"
#     }
#   }
# }
```

</p>

</details>

<details name="ruby/queries_viewer.rb"><summary>Ruby example</summary>
<p>

```rb
require 'net/http'
require 'uri'
require 'json'

def graphql(query, access_token)
  uri = URI.parse('https://www.universe.com/graphql/beta')
  http = Net::HTTP.new(uri.host, uri.port).tap { |h| h.use_ssl = true }
  headers = {'Authorization' => "Bearer #{access_token}", 'Content-Type' => 'application/json'}
  request = Net::HTTP::Post.new(uri.path, headers)
  request.body = {query: query}.to_json
  http.request(request).body
end

access_token = 'your_access_token'

puts graphql(
  "query GraphqlExample {
    viewer {
      id
      firstName
      lastName
    }
  }",
  access_token
)
# => {
#   "data": {
#     "viewer": {
#       "id": "58d98e52bbdebd003804e065",
#       "firstName": "Evgeny",
#       "lastName": "Li"
#     }
#   }
# }
```

</p>

</details>

### List of Events

With the `Host.id`, it's possible to get the list of all `Events`:

<details name="curl/queries_list_of_events.sh"><summary>cURL example</summary>
<p>

```sh
export HOST_ID=4f5e06cb2078f9730c000014

curl https://www.universe.com/graphql/beta \
  -H "Content-Type: application/json" \
  -d @- << EOF
{
  "query": "query GraphqlExample {
    host(id: \"$HOST_ID\") {
      name
      events {
        totalCount
        nodes {
          id
          title
        }
      }
    }
  }"
}
EOF
# => {
#   "data": {
#     "host": {
#       "name": "Joshua Kelly",
#       "events": {
#         "totalCount": 1,
#         "nodes": [
#           {
#             "id": "5879ad8f6672e70036d58ba5",
#             "title": "End of Unix Time"
#           }
#         ]
#       }
#     }
#   }
# }
```

</p>

</details>

<details name="ruby/queries_list_of_events.rb"><summary>Ruby example</summary>
<p>

```rb
require 'net/http'
require 'uri'
require 'json'

def graphql(query)
  uri = URI.parse('https://www.universe.com/graphql/beta')
  http = Net::HTTP.new(uri.host, uri.port).tap { |h| h.use_ssl = true }
  request = Net::HTTP::Post.new(uri.path, {'Content-Type' => 'application/json'})
  request.body = {query: query}.to_json
  http.request(request).body
end

host_id = '4f5e06cb2078f9730c000014'

puts graphql(
  "query GraphqlExample {
    host(id: \"#{host_id}\") {
      name
      events {
        totalCount
        nodes {
          id
          title
        }
      }
    }
  }"
)
# => {
#   "data": {
#     "host": {
#       "name": "Joshua Kelly",
#       "events": {
#         "totalCount": 1,
#         "nodes": [
#           {
#             "id": "5879ad8f6672e70036d58ba5",
#             "title": "End of Unix Time"
#           }
#         ]
#       }
#     }
#   }
# }
```

</p>

</details>

### Event

With the `Event.id`, it's possible to get `Event` information with the `TimeSlots`:

<details name="curl/queries_event.sh"><summary>cURL example</summary>
<p>

```sh
export ACCESS_TOKEN=your_access_token
export EVENT_ID=5879ad8f6672e70036d58ba5

curl https://www.universe.com/graphql/beta \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -d @- << EOF
{
  "query": "query GraphqlExample {
    event(id: \"$EVENT_ID\") {
      title
      state
      privacy
      address
      currency
      discounts {
        totalCount
        nodes(limit: 1) {
          code
          fixed
        }
      }
      rates {
        nodes(limit: 1) {
          price
          maxQuantity
        }
      }
      timeSlots {
        nodes(limit: 1) {
          startAt
          endAt
        }
      }
    }
  }"
}
EOF
# => {
#   "data": {
#     "event": {
#       "title": "End of Unix Time",
#       "state": "POSTED",
#       "privacy": "PUBLIC",
#       "address": "17 Phoebe St, Toronto, ON M5T 1A8, Canada",
#       "currency": "ERN",
#       "discounts": {
#         "totalCount": 2,
#         "nodes": [
#           {
#             "code": "1234",
#             "fixed": 5
#           }
#         ]
#       },
#       "rates": {
#         "nodes": [
#           {
#             "price": 10,
#             "maxQuantity": 100
#           }
#         ]
#       },
#       "timeSlots": {
#         "nodes": [
#           {
#             "startAt": "2038-01-19T00:00:00-05:00",
#             "endAt": "2038-01-19T01:00:00-05:00"
#           }
#         ]
#       }
#     }
#   }
# }
```

</p>

</details>

<details name="ruby/queries_event.rb"><summary>Ruby example</summary>
<p>

```rb
require 'net/http'
require 'uri'
require 'json'

def graphql(query, access_token)
  uri = URI.parse('https://www.universe.com/graphql/beta')
  http = Net::HTTP.new(uri.host, uri.port).tap { |h| h.use_ssl = true }
  headers = {'Authorization' => "Bearer #{access_token}", 'Content-Type' => 'application/json'}
  request = Net::HTTP::Post.new(uri.path, headers)
  request.body = {query: query}.to_json
  http.request(request).body
end

access_token = 'your_access_token'
event_id = '5879ad8f6672e70036d58ba5'

puts graphql(
  "query GraphqlExample {
    event(id: \"#{event_id}\") {
      title
      state
      privacy
      address
      currency
      discounts {
        totalCount
        nodes(limit: 1) {
          code
          fixed
        }
      }
      rates {
        nodes(limit: 1) {
          price
          maxQuantity
        }
      }
      timeSlots {
        nodes(limit: 1) {
          startAt
          endAt
        }
      }
    }
  }",
  access_token
)
# => {
#   "data": {
#     "event": {
#       "title": "End of Unix Time",
#       "state": "POSTED",
#       "privacy": "PUBLIC",
#       "address": "17 Phoebe St, Toronto, ON M5T 1A8, Canada",
#       "currency": "ERN",
#       "discounts": {
#         "totalCount": 2,
#         "nodes": [
#           {
#             "code": "1234",
#             "fixed": 5
#           }
#         ]
#       },
#       "rates": {
#         "nodes": [
#           {
#             "price": 10,
#             "maxQuantity": 100
#           }
#         ]
#       },
#       "timeSlots": {
#         "nodes": [
#           {
#             "startAt": "2038-01-19T00:00:00-05:00",
#             "endAt": "2038-01-19T01:00:00-05:00"
#           }
#         ]
#       }
#     }
#   }
# }
```

</p>

</details>

### Orders

<details name="curl/queries_orders.sh"><summary>cURL example</summary>
<p>

```sh
export ACCESS_TOKEN=your_access_token
export EVENT_ID=5879ad8f6672e70036d58ba5

curl https://www.universe.com/graphql/beta \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -d @- << EOF
{
  "query": "query GraphqlExample {
    event(id: \"$EVENT_ID\") {
      orders {
        totalCount
        nodes(limit: 2) {
          id
          state
          createdAt
          orderItems {
            totalCount
          }
          timeSlot {
            startAt
            endAt
          }
          buyer {
            name
            email
          }
        }
      }
    }
  }"
}
EOF
# => {
#   "data": {
#     "event": {
#       "orders": {
#         "totalCount": 14,
#         "nodes": [
#           {
#             "id": "587e95e60ae44d002a1dfd26",
#             "state": "PAID",
#             "createdAt": "2017-01-17T22:08:38Z",
#             "orderItems": {
#               "totalCount": 1
#             },
#             "timeSlot": {
#               "startAt": "2038-01-19T00:00:00-05:00",
#               "endAt": "2038-01-19T01:00:00-05:00"
#             },
#             "buyer": {
#               "name": "Joshua Kelly",
#               "email": "jdjkelly@gmail.com"
#             }
#           },
#           {
#             "id": "587e96a39b577e002a4a39f5",
#             "state": "PAID",
#             "createdAt": "2017-01-17T22:11:48Z",
#             "orderItems": {
#               "totalCount": 2
#             },
#             "timeSlot": {
#               "startAt": "2038-01-19T00:00:00-05:00",
#               "endAt": "2038-01-19T01:00:00-05:00"
#             },
#             "buyer": {
#               "name": "Joshua Kelly",
#               "email": "jdjkelly@gmail.com"
#             }
#           }
#         ]
#       }
#     }
#   }
# }
```

</p>

</details>

<details name="ruby/queries_orders.rb"><summary>Ruby example</summary>
<p>

```rb
require 'net/http'
require 'uri'
require 'json'

def graphql(query, access_token)
  uri = URI.parse('https://www.universe.com/graphql/beta')
  http = Net::HTTP.new(uri.host, uri.port).tap { |h| h.use_ssl = true }
  headers = {'Authorization' => "Bearer #{access_token}", 'Content-Type' => 'application/json'}
  request = Net::HTTP::Post.new(uri.path, headers)
  request.body = {query: query}.to_json
  http.request(request).body
end

access_token = 'your_access_token'
event_id = '5879ad8f6672e70036d58ba5'

puts graphql(
  "query GraphqlExample {
    event(id: \"#{event_id}\") {
      orders {
        totalCount
        nodes(limit: 2) {
          id
          state
          createdAt
          orderItems {
            totalCount
          }
          timeSlot {
            startAt
            endAt
          }
          buyer {
            name
            email
          }
        }
      }
    }
  }",
  access_token
)
# => {
#   "data": {
#     "event": {
#       "orders": {
#         "totalCount": 14,
#         "nodes": [
#           {
#             "id": "587e95e60ae44d002a1dfd26",
#             "state": "PAID",
#             "createdAt": "2017-01-17T22:08:38Z",
#             "orderItems": {
#               "totalCount": 1
#             },
#             "timeSlot": {
#               "startAt": "2038-01-19T00:00:00-05:00",
#               "endAt": "2038-01-19T01:00:00-05:00"
#             },
#             "buyer": {
#               "name": "Joshua Kelly",
#               "email": "jdjkelly@gmail.com"
#             }
#           },
#           {
#             "id": "587e96a39b577e002a4a39f5",
#             "state": "PAID",
#             "createdAt": "2017-01-17T22:11:48Z",
#             "orderItems": {
#               "totalCount": 2
#             },
#             "timeSlot": {
#               "startAt": "2038-01-19T00:00:00-05:00",
#               "endAt": "2038-01-19T01:00:00-05:00"
#             },
#             "buyer": {
#               "name": "Joshua Kelly",
#               "email": "jdjkelly@gmail.com"
#             }
#           }
#         ]
#       }
#     }
#   }
# }
```

</p>

</details>

## Mutatations

Coming soon

## Contributing

Please check out the [CONTRIBUTING.md](./CONTRIBUTING.md) guide.