# Healthchecks with OIDC/Oauth2 in Docker/Kubernetes
This is a short summary of our setup for OIDC login for Healthchecks in Kubernetes. This should work in Docker as well, haven't really tested it though.
Haven't had the time to put this in a helm chart - if you wanna do that feel free to!

## Background
We wanted to add federated login to Healthchecks, which doesn't support OpenID Connect. Y
ou could probably add it on the Django side if you wanted to but this method is more of a quick fix.

## How
Healthchecks supports the REMOTE_USER_HEADER parameter which is combined with Oauth2-Proxy to enable oidc/oauth2 login

## Usage
### The docker image
Build the custom docker image, or retrieve it from cloudyne/healthchecks:latest-proxyfix. The only difference between this and the healthchecks/healthchecks image is a setting un uwsgi.ini extending the size of the proxy headers. This would probably work with the bitnami image out of the box.

### Configuring Oauth2 Proxy
Included are some example variables for configuring oauth2-proxy for Keycloak, more information can be found in the oauth2-proxy documentation.
You need client ID, client secret and URLs for your Oauth2/OIDC IDP to configure the container.

### Kubernetes
A short warning, the example resources in this repository have not been tested for security and should be individually reviewed before applying them in any cluster. This is an example of a setup, not a complete solution (yet anyways).

- Configmap: Settings for healthchecks and oauth2-proxy for brokering the identity. It would be wise to put some of the values in a secret instead, they're in a config map here for visibility/readability.
- Deployment: The proxy and healthchecks application. PVCs/external databases should be used for healthchecks persistence.
- Service: Two ports, one for the pings directly to healthchecks and the other to the oauth2-proxy
- Ingress: Route everything to the oauth2-proxy except for the /ping prefix, which will be unauthenticated and used to send healthchecks from applications.