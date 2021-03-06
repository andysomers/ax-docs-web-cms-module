# EntityModule attributes

This section lists the different attributes that can be configured on EntityModule configuration to influence configuration of the WebCmsModule administration UI.  All attributes described here are constant fields on `WebCmsEntityAttributes`.

| Attribute constant | Use on | Description |
| :--- | :--- | :--- |
| `ALLOW_PER_DOMAIN` | `EntityConfiguration`  | Boolean value that specifies if an entity should be manageable per domain.  This will overrule the default domain-bound or not-domain-bound behaviour.  |
| `DOMAIN_PROPERTY` | `EntityConfiguration`  | String value that contains the name of the property that links to the `WebCmsDomain`. By default only `WebCmsDomainBound` entities support multi-domain, where the implicit value of this attribute would then be _domain_.  <br/><br/>The presence of this attribute would activate multi-domain auto-configuration for entities not implementing `WebCmsDomainBound` directly.  Examples include `WebCmsUrl` and `WebCmsMenuItem` that have their domain linked transitively (resp. via _endpoint.domain_ and _menu.domain_). |

#### Multi-domain auto-configuration

The following attributes influence the multi-domain auto-configuration and are constant fields on `WebCmsEntityAttributes.MultiDomainConfiguration`.  Setting these values indicates that configuration has been performed manually and will make sure the auto-configuration for the corresponding part is skipped. 

| Attribute constant | Use on | Description |
| :--- | :--- | :--- |
| `LIST_VIEW_ADJUSTED` | `EntityConfiguration`<br/>`EntityAssociation`  | Boolean value that indicates the list view has already been adjusted for multi-domain support.  |
| `ENTITY_MODEL_ADJUSTED` | `EntityConfiguration`  | Boolean value that indicates that the `EntityModel` has been modified for multi-domain support. |
| `ALLOWABLE_ACTIONS_ADJUSTED` | `EntityConfiguration`  | Boolean value that indicates that the `EntityConfigurationAllowableActionsBuilder` has been modified for multi-domain support. |
| `OPTIONS_QUERY_ADJUSTED` | `EntityConfiguration`<br/>`EntityPropertyDescriptor`  | Boolean value that indicates that the `EntityAttributes.OPTIONS_QUERY` has been modified for multi-domain support. |
| `FINISHED` | `EntityConfiguration`  | Boolean value that indicates that the entity mulit-domain configuration has been performed manually.  Setting  |








































































