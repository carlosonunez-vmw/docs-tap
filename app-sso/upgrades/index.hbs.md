# Upgrade Application Single Sign-On

This topic tells you how to upgrade Application Single Sign-On (commonly called AppSSO) 
outside of a Tanzu Application Platform profile installation.
If you installed Tanzu Application Platform through a profile, see
[Upgrade Tanzu Application Platform](../../upgrading.hbs.md) instead.

For help on migrating your resources in between versions, see the [migration guides](#migration-guides).

If you installed the `AppSSO` package on its own, and not as part of `TAP`, you can upgrade it individually by running:

```console
tanzu package installed update PACKAGE_INSTALLATION_NAME -p sso.apps.tanzu.vmware.com -v 3.0.0 --values-file PATH_TO_YOUR_VALUES_YAML -n YOUR_INSTALL_NAMESPACE
```

>**Note** You can also upgrade AppSSO as part of upgrading Tanzu Application Platform as a whole. See [Upgrading Tanzu Application Platform](../../upgrading.hbs.md) for more information.

## <a id="migration-guides"></a>Migration guides

### <a id="v2-to-v3">`v2.0.0` to `v3.0.0`

VMware recommends that you recreate your `AuthServers` after upgrading your AppSSO to `v3.0.0`
with the following changes:

- Migrate field `.spec.tls.disabled` to `.spec.tls.deactivated`.

### <a id="v1-to-v2">`v1.0.0` to `v2.0.0`

VMware recommends that you recreate your `AuthServers` after upgrading your AppSSO to `v2.0.0`
with the following changes:

- Migrate from `.spec.issuerURI` to `.spec.tls`:

    >**Note** AppSSO templates your issuer URI and enables TLS. When using the newer `.spec.tls`,
    a custom `Service` and an ingress resource are no longer required.

    >**Note** It is not recommended to continue using `.spec.issuerURI` in AppSSO v2.0.0. 
    To use `.spec.issuerURI` in AppSSO v2.0.0, you must provide a `Service` and an ingress resource as in AppSSO v1.0.0.

    1. Configure one of `.spec.tls.{issuerRef, certificateRef, secretRef}`. See [Issuer URI & TLS](../service-operators/issuer-uri-and-tls.md) for more information.
    1. (Optional) Disable TLS with `.spec.tls.disabled`.
    1. Remove `.spec.issuerURI`.
    1. Delete your `AuthServer`-specific `Service` and ingress resources.
    1. Apply your `AuthServer`. You can find its issuer URI in `.status.issuerURI`.
    1. Update the redirect URIs in your upstream identity providers.

- If you use the `internalUnsafe` identity provider to migrate existing users by replacing the bcrypt hash through the
plain-text equivalent. You can still use existing bcrypt passwords by prefixing them with `{bcrypt}`:

    ```yaml
    ---
    apiVersion: sso.apps.tanzu.vmware.com/v1alpha1
    kind: AuthServer
    metadata:
      # ...
    spec:
      identityProviders:
        - name: internal
          internalUnsafe:
            users:
              # v1.0
              - username: test-user-1
                password: $2a$10$201z9o/tHlocFsHFTo0plukh03ApBYe4dRiXcqeyRQH6CNNtS8jWK # bcrypt-encoded "password"
                # ...
              # v2.0
              - username: "test-user-1"
                password: "{bcrypt}$2a$10$201z9o/tHlocFsHFTo0plukh03ApBYe4dRiXcqeyRQH6CNNtS8jWK" # same bcrypt hash, with {bcrypt} prefix
              - username: "test-user-2"
                password: "password" # plain text
      # ...
    ```
