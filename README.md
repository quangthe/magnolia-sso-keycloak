# Magnolia SSO with Keycloak

## Installation steps

### Keycloak

Version: `23.0.3`

Spin up Keycloak server with pre-configured realm/client/users for Magnolia SSO

```
docker-compose up
```

- Keycloak URL: [http://localhost:8180](http://localhost:8180)
- Discovery Url: [http://localhost:8180/realms/magnolia-sso/.well-known/openid-configuration](http://localhost:8180/realms/magnolia-sso/.well-known/openid-configuration)
- Admin user: `admin/admin`
- Realm: `magnolia-sso`
  * Demo user: `ssouser/ssouser`

### Magnolia 

Tested with:
- Magnolia CMS: [`6.2.39`](https://docs.magnolia-cms.com/product-docs/6.2/Releases/Release-notes-for-Magnolia-CMS-6.2.39.html)
- magnolia-sso module: [`3.1.7`](https://docs.magnolia-cms.com/magnolia-sso/3.1.7/index.html)

Edit magnolia-sso config.yaml file: `<light-modules-dir>/magnolia-sso/config.yaml`

```yaml
path: /.magnolia/admincentral
callbackUrl: http://localhost:8080/.auth
postLogoutRedirectUri: http://localhost:8080/.magnolia/admincentral
authorizationGenerators:
  - name: groupsAuthorization
    groups:
      mappings:
        - name: /superuser
          targetRoles:
            - superuser
clients:
  oidc.id: mgnl-client
  oidc.secret: 763c0d26-a075-4242-b386-946a3ddccbff
  oidc.clientAuthenticationMethod: client_secret_basic
  oidc.scope: openid profile email
  oidc.discoveryUri:  http://localhost:8180/realms/magnolia-sso/.well-known/openid-configuration
  oidc.preferredJwsAlgorithm: RS256
  oidc.authorizationGenerators: groupsAuthorization

userFieldMappings:
  name: preferred_username
  removeEmailDomainFromUserName: true
  removeSpecialCharactersFromUserName: false
  fullName: name
  email: email
  language: locale
```

`jaas.config` file: `WEB-INF/config/jaas.config`

```
sso-authentication {
  info.magnolia.sso.jaas.SsoAuthenticationModule requisite;
  info.magnolia.jaas.sp.jcr.JCRAuthorizationModule required;
};
```

Then start Magnolia instance (ensure to configure `magnolia.resources.dir` point to `<light-modules-dir>`), navigating to admin central should redirect you to Keycloak login page.

## Clean up 

Stop keycloak `docker-compose down -v`
