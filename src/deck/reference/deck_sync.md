---
title: deck sync
source_url: https://github.com/Kong/deck/tree/main/cmd
---

The sync command reads the state file and performs operation on Kong
to get Kong's state in sync with the input state.

## Syntax

```
deck sync [command-specific flags] [global flags]
```

## Flags

`--db-update-propagation-delay`
:  artificial delay (in seconds) that is injected between insert operations 
for related entities (usually for Cassandra deployments).
See `db_update_propagation` in kong.conf. (Default: `0`)

`-h`, `--help`
:  help for sync (Default: `false`)

`--parallelism`
:  Maximum number of concurrent operations. (Default: `10`)

`--rbac-resources-only`
:  diff only the RBAC resources (Kong Enterprise only). (Default: `false`)

`--select-tag`
:  only entities matching tags specified via this flag are synced.
When this setting has multiple tag values, entities must match every tag.

`--silence-events`
:  disable printing events to stdout (Default: `false`)

`--skip-ca-certificates`
:  do not sync CA certificates. (Default: `false`)

`--skip-consumers`
:  do not sync consumers or any plugins associated with consumers. (Default: `false`)

`-s`, `--state`
:  file(s) containing Kong's configuration.
This flag can be specified multiple times for multiple files.
Use `-` to read from stdin. (Default: `[kong.yaml]`)

`--workspace`
:  Sync configuration to a specific workspace (Kong Enterprise only).
This takes precedence over _workspace fields in state files.



## Global flags

`--analytics`
:  Share anonymized data to help improve decK.
Use `--analytics=false` to disable this. (Default: `true`)

`--ca-cert`
:  Custom CA certificate (raw contents) to use to verify Kong's Admin TLS certificate.
This value can also be set using DECK_CA_CERT environment variable.
This takes precedence over `--ca-cert-file` flag.

`--ca-cert-file`
:  Path to a custom CA certificate to use to verify Kong's Admin TLS certificate.
This value can also be set using DECK_CA_CERT_FILE environment variable.

`--config`
:  Config file (default is $HOME/.deck.yaml).

`--headers`
:  HTTP headers (key:value) to inject in all requests to Kong's Admin API.
This flag can be specified multiple times to inject multiple headers.

`--kong-addr`
:  HTTP address of Kong's Admin API.
This value can also be set using the environment variable DECK_KONG_ADDR
 environment variable. (Default: `"http://localhost:8001"`)

`--kong-cookie-jar-path`
:  Absolute path to a cookie-jar file in the Netscape cookie format for auth with Admin Server.
You may also need to pass in as header the User-Agent that was used to create the cookie-jar.

`--konnect-addr`
:  Address of the Konnect endpoint. (Default: `"https://us.api.konghq.com"`)

`--konnect-email`
:  Email address associated with your Konnect account.

`--konnect-password`
:  Password associated with your Konnect account, this takes precedence over `--konnect-password-file` flag.

`--konnect-password-file`
:  File containing the password to your Konnect account.

`--konnect-runtime-group-name`
:  Konnect Runtime group name.

`--no-color`
:  Disable colorized output (Default: `false`)

`--skip-workspace-crud`
:  Skip API calls related to Workspaces (Kong Enterprise only). (Default: `false`)

`--timeout`
:  Set a request timeout for the client to connect with Kong (in seconds). (Default: `10`)

`--tls-client-cert`
:  PEM-encoded TLS client certificate to use for authentication with Kong's Admin API.
This value can also be set using DECK_TLS_CLIENT_CERT environment variable. Must be used in conjunction with tls-client-key

`--tls-client-cert-file`
:  Path to the file containing TLS client certificate to use for authentication with Kong's Admin API.
This value can also be set using DECK_TLS_CLIENT_CERT_FILE environment variable. Must be used in conjunction with tls-client-key-file

`--tls-client-key`
:  PEM-encoded private key for the corresponding client certificate .
This value can also be set using DECK_TLS_CLIENT_KEY environment variable. Must be used in conjunction with tls-client-cert

`--tls-client-key-file`
:  Path to file containing the private key for the corresponding client certificate.
This value can also be set using DECK_TLS_CLIENT_KEY_FILE environment variable. Must be used in conjunction with tls-client-cert-file

`--tls-server-name`
:  Name to use to verify the hostname in Kong's Admin TLS certificate.
This value can also be set using DECK_TLS_SERVER_NAME environment variable.

`--tls-skip-verify`
:  Disable verification of Kong's Admin TLS certificate.
This value can also be set using DECK_TLS_SKIP_VERIFY environment variable. (Default: `false`)

`--verbose`
:  Enable verbose logging levels
Setting this value to 2 outputs all HTTP requests/responses
between decK and Kong. (Default: `0`)



## See also

* [deck](/deck/{{page.kong_version}}/reference/deck)	 - Administer your Kong clusters declaratively

