---
name: Key Authentication
publisher: Kong Inc.
version: 2.3.x (internal 2.4.0)
desc: Add key authentication to your Services
description: |
  Add key authentication (also sometimes referred to as an _API key_) to a Service or a Route.
  Consumers then add their API key either in a query string parameter, a header, or a
  request body to authenticate their requests.

  This plugin can be used for authentication in conjunction with the
  [Application Registration](/hub/kong-inc/application-registration) plugin.

  **Tip:** The Kong Gateway [Key Authentication Encrypted](/hub/kong-inc/key-auth-enc/)
    plugin provides the ability to encrypt keys. Keys are encrypted at rest in the API gateway datastore.
type: plugin
categories:
  - authentication
kong_version_compatibility:
  community_edition:
    compatible:
      - 2.8.x
      - 2.7.x
      - 2.6.x
      - 2.5.x
      - 2.4.x
      - 2.3.x
      - 2.2.x
      - 2.1.x
      - 2.0.x
      - 1.5.x
      - 1.4.x
      - 1.3.x
      - 1.2.x
      - 1.1.x
      - 1.0.x
      - 0.14.x
      - 0.13.x
      - 0.12.x
      - 0.11.x
      - 0.10.x
      - 0.9.x
      - 0.8.x
      - 0.7.x
      - 0.6.x
      - 0.5.x
      - 0.4.x
      - 0.3.x
      - 0.2.x
  enterprise_edition:
    compatible:
      - 2.8.x
      - 2.7.x
      - 2.6.x
      - 2.5.x
      - 2.4.x
      - 2.3.x
      - 2.2.x
      - 2.1.x
      - 1.5.x
      - 1.3-x
      - 0.36-x
params:
  name: key-auth
  service_id: true
  route_id: true
  consumer_id: false
  protocols:
    - http
    - https
    - grpc
    - grpcs
  dbless_compatible: partially
  dbless_explanation: |
    Consumers and Credentials can be created with declarative configuration.

    Admin API endpoints that do POST, PUT, PATCH, or DELETE on Credentials are not available on DB-less mode.
  config:
    - name: key_names
      required: true
      default: '[`apikey`]'
      value_in_examples:
        - apikey
      datatype: array of strings
      description: |
        Describes an array of parameter names where the plugin will look for a key. The client must send the
        authentication key in one of those key names, and the plugin will try to read the credential from a
        header, request body, or query string parameter with the same name.
        <br>**Note**: The key names may only contain [a-z], [A-Z], [0-9], [_] underscore, and [-] hyphen.
    - name: key_in_body
      required: true
      default: '`false`'
      datatype: boolean
      description: |
        If enabled, the plugin reads the request body (if said request has one and its MIME type is supported) and tries to find the key in it. Supported MIME types: `application/www-form-urlencoded`, `application/json`, and `multipart/form-data`.
    - name: key_in_header
      required: true
      default: '`true`'
      datatype: boolean
      description: |
        If enabled (default), the plugin reads the request header and tries to find the key in it.
    - name: key_in_query
      required: true
      default: '`true`'
      datatype: boolean
      description: |
        If enabled (default), the plugin reads the query parameter in the request and tries to find the key in it.
    - name: hide_credentials
      required: true
      default: '`false`'
      datatype: boolean
      description: |
        An optional boolean value telling the plugin to show or hide the credential from the upstream service. If `true`,
        the plugin strips the credential from the request (i.e., the header, query string, or request body containing the key) before proxying it.
    - name: anonymous
      required: false
      default: null
      datatype: string
      description: |
        An optional string (Consumer UUID) value to use as an anonymous Consumer if authentication fails.
        If empty (default), the request will fail with an authentication failure `4xx`. Note that this value
        must refer to the Consumer `id` attribute that is internal to Kong, and **not** its `custom_id`.
    - name: run_on_preflight
      required: true
      default: '`true`'
      datatype: boolean
      description: |
        A boolean value that indicates whether the plugin should run (and try to authenticate) on `OPTIONS` preflight requests.
        If set to `false`, then `OPTIONS` requests are always allowed.
  extra: |

    ## Case sensitivity

    Note that, according to their respective specifications, HTTP header names are treated as
    case _insensitive_, while HTTP query string parameter names are treated as case _sensitive_.
    Kong follows these specifications as designed, meaning that the `key_names`
    configuration values are treated differently when searching the request header fields versus
    searching the query string. As a best practice, administrators are advised against defining
    case-sensitive `key_names` values when expecting the authorization keys to be sent in the request headers.

    Once applied, any user with a valid credential can access the Service.
    To restrict usage to certain authenticated users, also add the
    [ACL](/plugins/acl/) plugin (not covered here) and create allowed or
    denied groups of users.
---

## Usage

### Create a Consumer

You need to associate a credential to an existing [Consumer][consumer-object] object.
A Consumer can have many credentials.

{% navtabs %}
{% navtab With a database %}

To create a Consumer, make the following request:

```bash
curl -d "username=user123&custom_id=SOME_CUSTOM_ID" http://kong:8001/consumers/
```
{% endnavtab %}

{% navtab Without a database %}

Your declarative configuration file will need to have one or more Consumers. You can create them
on the `consumers:` yaml section:

``` yaml
consumers:
- username: user123
  custom_id: SOME_CUSTOM_ID
```

{% endnavtab %}
{% endnavtabs %}

In both cases, the parameters are as described below:

parameter                       | description
---                             | ---
`username`<br>*semi-optional*   | The username of the Consumer. Either this field or `custom_id` must be specified.
`custom_id`<br>*semi-optional*  | A custom identifier used to map the Consumer to another database. Either this field or `username` must be specified.

If you are also using the [ACL](/plugins/acl/) plugin and allow lists with this
service, you must add the new Consumer to the allowed group. See
[ACL: Associating Consumers][acl-associating] for details.

### Create a Key

<div class="alert alert-warning">
  <strong>Note:</strong> It is recommended to let the API gateway autogenerate the key. Only specify it yourself if
  you are migrating an existing system to {{site.base_gateway}}. You must reuse your keys to make the
  migration to {{site.base_gateway}} transparent to your Consumers.
</div>

{% navtabs %}
{% navtab With a database %}

Provision new credentials by making the following HTTP request:

```bash
$ curl -X POST http://kong:8001/consumers/{consumer}/key-auth -d
```

Response:

```
HTTP/1.1 201 Created

{
    "consumer": { "id": "876bf719-8f18-4ce5-cc9f-5b5af6c36007" },
    "created_at": 1443371053000,
    "id": "62a7d3b7-b995-49f9-c9c8-bac4d781fb59",
    "key": "62eb165c070a41d5c1b58d9d3d725ca1"
}
```
{% endnavtab %}
{% navtab Without a database %}

Add credentials to your declarative config file in the `keyauth_credentials` YAML
entry:

```yaml
keyauth_credentials:
- consumer: {consumer}
```
{% endnavtab %}
{% endnavtabs %}

In both cases, the fields/parameters work as follows:

field/parameter     | description
---                 | ---
`{consumer}`        | The `id` or `username` property of the [Consumer][consumer-object] entity to associate the credentials to.
`ttl`<br>*optional* | The number of seconds the key is going to be valid. If missing or set to zero, the `ttl` of the key is unlimited. If present, the value must be an integer between 0 and 100000000.
`key`<br>*optional* | You can optionally set your own unique `key` to authenticate the client. If missing, the plugin will generate one.

### Make a Request with the Key

Make a request with the key as a query string parameter:

```bash
$ curl http://kong:8000/{proxy path}?apikey=<some_key>
```

**Note:** The `key_in_query` parameter must be set to `true` (default).

Make a request with the key in the body:

```bash
$ curl http://kong:8000/{proxy path} \
    --data 'apikey: <some_key>'
```

**Note:** The `key_in_body` parameter must be set to `true` (default is `false`).

Make a request with the key in a header:

```bash
$ curl http://kong:8000/{proxy path} \
    -H 'apikey: <some_key>'
```

**Note:** The `key_in_header` parameter must be set to `true` (default).

gRPC clients are supported too:

```bash
$ grpcurl -H 'apikey: <some_key>' ...
```
### About API Key Locations in a Request

{% include /md/plugins-hub/api-key-locations.md %}

### Disable a Key Location

This example disables using a key in a query parameter:

```bash
curl -X POST http://<admin-hostname>:8001/routes/<route>/plugins \
    --data "name=key-auth"  \
    --data "config.key_names=apikey" \
    --data "config.key_in_query=false"
```

### Delete a Key

Delete an API Key by making the following request:

```bash
$ curl -X DELETE http://kong:8001/consumers/{consumer}/key-auth/{id}
```

Response:

```bash
HTTP/1.1 204 No Content
```

* `consumer`: The `id` or `username` property of the [Consumer][consumer-object] entity to associate the credentials to.
* `id`: The `id` attribute of the key credential object.

### Upstream Headers

When a client has been authenticated, the plugin appends some headers to the request before
proxying it to the upstream service so that you can identify the Consumer in your code:

* `X-Consumer-ID`, the ID of the Consumer on Kong
* `X-Consumer-Custom-ID`, the `custom_id` of the Consumer (if set)
* `X-Consumer-Username`, the `username` of the Consumer (if set)
* `X-Credential-Identifier`, the identifier of the Credential (only if the consumer is not the 'anonymous' consumer)
* `X-Anonymous-Consumer`, will be set to `true` when authentication failed, and the 'anonymous' consumer was set instead.

You can use this information on your side to implement additional logic. You can use the `X-Consumer-ID` value to query the Kong Admin API and retrieve more information about the Consumer.

### Paginate through keys

Paginate through the API keys for all Consumers by making the following
request:

```bash
$ curl -X GET http://kong:8001/key-auths
```

Response:

```bash
...
{
   "data":[
      {
         "id":"17ab4e95-9598-424f-a99a-ffa9f413a821",
         "created_at":1507941267000,
         "key":"Qslaip2ruiwcusuSUdhXPv4SORZrfj4L",
         "consumer": { "id": "c0d92ba9-8306-482a-b60d-0cfdd2f0e880" }
      },
      {
         "id":"6cb76501-c970-4e12-97c6-3afbbba3b454",
         "created_at":1507936652000,
         "key":"nCztu5Jrz18YAWmkwOGJkQe9T8lB99l4",
         "consumer": { "id": "c0d92ba9-8306-482a-b60d-0cfdd2f0e880" }
      },
      {
         "id":"b1d87b08-7eb6-4320-8069-efd85a4a8d89",
         "created_at":1507941307000,
         "key":"26WUW1VEsmwT1ORBFsJmLHZLDNAxh09l",
         "consumer": { "id": "3c2c8fc1-7245-4fbb-b48b-e5947e1ce941" }
      }
   ]
   "next":null,
}
```

Filter the list by Consumer by using a different endpoint:

```bash
$ curl -X GET http://kong:8001/consumers/{username or id}/key-auth
```

`username or id`: The username or id of the Consumer whose credentials need to be listed.

Response:

```bash
...
{
    "data": [
       {
         "id":"6cb76501-c970-4e12-97c6-3afbbba3b454",
         "created_at":1507936652000,
         "key":"nCztu5Jrz18YAWmkwOGJkQe9T8lB99l4",
         "consumer": { "id": "c0d92ba9-8306-482a-b60d-0cfdd2f0e880" }
       }
    ]
    "next":null,
}
```

### Retrieve the Consumer associated with a key

Retrieve a [Consumer][consumer-object] associated with an API
key by making the following request:

```bash
curl -X GET http://kong:8001/key-auths/{key or id}/consumer
```

`key or id`: The `id` or `key` property of the API key for which to get the
associated Consumer.

Response:

```bash
{
   "created_at":1507936639000,
   "username":"foo",
   "id":"c0d92ba9-8306-482a-b60d-0cfdd2f0e880"
}
```

[configuration]: /gateway/latest/reference/configuration
[consumer-object]: /gateway/latest/admin-api/#consumer-object
[acl-associating]: /plugins/acl/#associating-consumers
