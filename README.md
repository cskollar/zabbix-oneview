# HPE OneView template for Zabbix

This is a Zabbix template for HPE OneView (especially for Synergy frames). It uses native Zabbix features and functions and doesn't require any external scripts or plugins, therefore the footprint is very small. Queries are implemented through the official OneView REST API.

#### Features

- get OneView active and locked alerts with lld
- enclosure discovery with lld (enclosure hosts created dynamically)
- server blades discovery with lld (server hosts created dynamically)

#### Work in progress

- interconnect bays discovery
- SAN/storage discovery
- tags & applications support
- Zabbix 6.0 support

...PRs are welcomed!

#### Compatiblity

Zabbix: 4.4 - 5.4

OneView: tested with OneView 6.2 (rest api version 3000) but it works with newest versions as well. The api version "3000" is hardcoded in the template. For older versions (<6.2) try rewriting the "req.AddHeader('x-api-version: 3000')" lines in the javascripts in master items.

## How to use

1. Import the template file (it contains 3 templates)
2. Set the proper host group in the main template ("HPE OneView" -> enclosure AND server discovery -> host prototype -> groups)
3. Create a host for the OneView appliance (eg.: composer1.local.tld)
4. Link the main template ("HPE OneView") to the host
5. Assign user macros to the host (see below)

The other two tepmlates ("HPE OneView Enclosure", "HPE OneView Server") are automatically assigned to the created hosts by llds.

#### These following settings are required on the OneView host by user macros:

- {$ONEVIEW_HOST} - host or IP address of the composer (OneView appliance)
- {$ONEVIEW_USER} - OneView username
- {$ONEVIEW_PASS} - OneView password

## How it works

#### Authentication mechanism
The authentiaciton process of OneView rest api based on login-sessions (also known as "bearer tokens"). Zabbix must request a token first by username/password pair then it can launch queries authenticated by this token. This process is handled by the template completely with built-in Zabbix javascript objects, no external solution or configuration needed (scripts, services, etc..).

Each query requests a new token from the api and the received token will be deleted (call a DELETE method on the api) after the query completed. This is necessary because the tokens expire in 24 hours.

#### Performance
Fortunately, the api is really fast so it may be able to high frequency monitoring (not tested yet). The templates only launch a small number of queries using master items<->dependent LLDs structure. Each master item holds all the necessary data (in a huge json response), so OneView doesn't need to filtering data or processing complicated queries, it's processed on Zabbix side. Therefore, the average time of a query (include the token request/delete method) is between 0.05 and 0.2 seconds only.
