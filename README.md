# 1 Introduction

APISIX is an API management tool and in ODALA it's been designed and configured to work together with Keycloak. This access control layer can replace the Umbrella/Keyrock combination.

# 2 Configurations

Deployment is via GitLab CI/CD pipeline. 

Deployment follows similar pattern like other components, Operator with Helm charts and substituting values to template files.

APISIX has an admin API. Using that API we setup predefined routes and auth plugin (scorpio-client).


# 2.1 Intergration with Keycloak

APISIX is configured to use [authz-keycloak](https://apisix.apache.org/docs/apisix/2.15/plugins/authz-keycloak/) plugin. This allows us to user Keycloak as and IDM with APISIX. 

# 2.2 Fetching a token to access data

In order to get an access token, Keycloak provides a token endpoint:

```
curl --location --request POST 'https://keycloak.staging.odala.kiel.de/realms/apisix-realm/protocol/openid-connect/token' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'client_id=scorpio-client' \
--data-urlencode 'username=scorpiowriter' \
--data-urlencode 'password=xxx' \
--data-urlencode 'grant_type=password' \
--data-urlencode 'client_secret=xxx'
```
After that you can make a request towards Scorpio context broker:

```
curl --location --request GET 'https://scorpio-apisix.staging.odala.kiel.de/ngsi-ld/v1/entities/?type=TrafficFlowObserved' \
--header 'Authorization: Bearer eyJ...3A' \
--header 'Link: https://schema.lab.fiware.org/ld/context; rel=http://www.w3.org/ns/json-ld#context; type=application/ld+json'
```
and receive a response like:

```
[
    {
        "id": "urn:ngsi-ld:TrafficFlowObserved:61bb15be345c06d77f32e528",
        "type": "TrafficFlowObserved",
        "description": {
            "type": "Property",
            "value": "MQ13"
        },
        "address": {
            "type": "Property",
            "value": {
                "type": "PostalAddress",
                "streetAddress": "Alte Lu
                .
                .
                .
```
As an admin, you have access to the client application which holds the secrets needed for the requests. They also need to be in the CI/CD pipeline inorder the deployment to work.

# 3 Preconfigured APIs

There are few APIs created in the CI/CD pipeline. There are two type; one for the protected access and handful of others to provide open access to selected data.

# 4 Configuration model

Crash course to APISIX concepts:

Route: A route is a routing path to upstream targets. In Apache APISIX, routes are responsible for matching client's requests based on defined rules, loading and executing the corresponding plugins, as well as forwarding requests to the specified upstream services. In APISIX, a simple route can be set up with a path-matching URI and a corresponding upstream address.

Upstream: An upstream is a set of target nodes with the same work. It defines a virtual host abstraction that performs load balancing on a given set of service nodes according to the configured rules.

If any changes need to be done to APISIX configuration, we recommend doing it via CI/CD pipe. This way the changes are under version control and can be rolled back as needed. 

The model we found useful was to develop by hand in the API and once configuration was complete, copy that to the CI/CD pipeline and run it from there to the deployment.

# 5 Known issues / to be developed

There is a never [version](https://apisix.apache.org/docs/apisix/getting-started/README/) 3.3 at the time of writing. It is recommended to update to that version. Please note that there are [breaking changes](https://apisix.apache.org/docs/apisix/next/upgrade-guide-from-2.15.x-to-3.0.0/).

Grafana analytics have not been enabled.

Components- APISIX

for the ODALA project.

© 2022 Contrasec Oy

License EUPL 1.2

![](https://ec.europa.eu/inea/sites/default/files/ceflogos/en_horizontal_cef_logo_2.png)
