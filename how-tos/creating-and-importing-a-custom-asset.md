# Creating a custom WebCmsAsset

To create a custom `WebCmsAsset`, we create a class which extends `WebCmsAsset`. `WebCmsAsset`has two superclasses which require a method to be implemented:

* `WebCmsObjectInheritanceSuperClass`which defines `getObjectType()`
* `WebCmsObjectSuperClass`which defines `getObjectCollectionId()` with which every object id of the class will be prefixed with.

Aside of these two methods, WebCmsAsset defines an abstract method `getAssetType()`, which also needs to be implemented.

`WebCmsAsset`uses joined inheritance, so don't forget to add an `@DiscriminatorValue` to the class, which typically is the same as the return value of `getObjectType()`.

`getObjectCollectionId()` returns the prefix of the asset that defines to which collection it belongs. For example, the collection id for a `WebCmsPage`is `wcm:asset:page`. A typical collection id starts with _wcm:asset:_ followed by the object type.

Often, we create a `WebCmsAssetType`for our asset, which our asset will hold and which will be the return value of `getAssetType()`. However, it is not required to have an explicit asset type \(unless you want to be able to import your custom asset through yaml imports and create several components by default from a template\), in which case the method may return `null`.

## Creating a WebCmsAssetType

A `WebCmsAssetType`extends `WebCmsTypeSpecifier`- which in itself implements the two superclasses we've talked about earlier. This means that once again we have to implement a `getObjectType()` and `getObjectCollectionId()` method. By convention and to keep a clear overview, we advise you to use the same object type for both the custom asset and the custom asset type. The collection id is usually slightly different, as it represents a type, so it should not be in the same collection as our assets. If we once again take a look at `WebCmsPage`and its `WebCmsPageType`, we see that the collection id for `WebCmsPageType`is `wcm:type:page`. Typically the collection id for a WebCmsPage starts with _wcm:type:_ followed by the collection id.

Note: the prefix _wcm:_ is mostly used by objects provided by WebCmsModule, so please check that there's no WebCmsAsset defined with the same collection type. It would definitely be a good idea to have your own prefix within your project.

An example `AuthorAssetType`:

```java
@NotThreadSafe
@Entity
@DiscriminatorValue(OBJECT_TYPE)
@Table(name = "example_author_asset_type")
@NoArgsConstructor
@Getter
@Setter
public class AuthorAssetType extends WebCmsAssetType<AuthorAssetType>
{

    public static final String OBJECT_TYPE = "author";
    public static final String COLLECTION_ID = "example:type:author";

    @Override
    public String getObjectType() {
        return OBJECT_TYPE;
    }

    @Override
    protected String getObjectCollectionId() {
        return COLLECTION_ID;
    }
}
```

For WebCmsModule to be able to use the WebCmsAssetType, we also need to register it to the WebCmsTypeRegistry. This usually happens as follows within an @Configuration class:

```java
@Autowired
public void registerDefaultTypes( WebCmsTypeRegistry typeRegistry ) {
    typeRegistry.register( AuthorAssetType.OBJECT_TYPE, AuthorAssetType.class, AuthorAssetType::new );
}
```

An example `AuthorAsset`:

```java
@NotThreadSafe
@Entity
@DiscriminatorValue(OBJECT_TYPE)
@Table(name = "example_author_asset")
@Getter
@Setter
@NoArgsConstructor
public class AuthorAsset extends WebCmsAsset<AuthorAsset>
{

    /**
     * Object type name (discriminator value).
     */
    public static final String OBJECT_TYPE = "author";

    /**
     * Prefix that all asset ids for an author should have.
     */
    public static final String COLLECTION_ID = "example:asset:author";

    /**
     * Name of the author.
     */
    @NotBlank
    @Length(max = 255)
    @Column(name = "name")
    private String name;

    /**
     * The job title of the author.
     */
    @NotBlank
    @Length(max = 255)
    @Column(name = "job_title")
    private String jobTitle;

    @ManyToOne
    @JoinColumn(name = "author_asset_type_id")
    @NotNull
    private AuthorAssetType authorType;

    @Override
    public WebCmsAssetType getAssetType() {
        return authorType;
    }

    @Override
    public String getObjectType() {
        return OBJECT_TYPE;
    }

    @Override
    protected String getObjectCollectionId() {
        return COLLECTION_ID;
    }

}
```

Aside of creating our custom `WebCmsAsset`and its `WebCmsAssetType`, do not forget to create their repositories, which should extends `WebCmsObjectEntityRepository`.

Now we've created a custom `WebCmsAsset`and `WebCmsAssetType`which can be managed from the backend, should EntityModule and AdminWebModule be present. However, managing the related components will not be possible from the administration interface. We simply need to enable the option:

```java
@Autowired
public void enableComponentViews( WebCmsObjectComponentViewsConfiguration componentViewsConfiguration ){
    componentViewsConfiguration.enable(AuthorAsset.class);
}
```

## Importing a custom WebCmsAsset

To be able to import the custom asset we have just created we will require to create a custom importer, and should we want to create templates for our custom WebCmsAsset, we will also require an interceptor to copy the template to our asset.

Let's start by creating an importer for our `AuthorAsset`. We'll create a class that extends `AbstractWebCmsAssetImporter`, which is a base class for importing a single asset type. `AbstractWebCmsDataImporter`extends `AbstractWebCmsDataImporter`, which is the base class that supports the imports of `WebCmsObject`s. Several methods required by AbstractWebCmsDataImporter have a default implementation provided by AbstractWebCmsAssetImporter, which can be used to further customize the imports of your assets.  There is a single method left that has no default implementation, being `createDto( WebCmsDataEntry data, <WebCmsAsset> existing, WebCmsDataAction action, Map<String, Object> values )`.  All `createDto( ... )` is required to do is create a DTO object which can afterwards be used to set the imported properties, and be persisted to the database.

By default, the unique key to find a WebCmsObject is its objectId, found in the top level of the data entry we are importing. Should you wish to use a different identifier for your asset in your imports, you can implement `getExistingEntity( String entryKey, WebCmsDataEntry entryData, WebCmsDomain domain )` to specify which keys can be allowed and what the entry key represents. \(e.g. the canonical path can be used as an entrykey in the case of a `WebCmsPage`\).

Aside of that, we also need to provide a constructor, matching the one of `AbstractWebCmsAssetImporter`to where we define the data key that we want to use under one of the root keys. Several data keys already exist, so once again, double check whether your key is unique.

Our base `AuthorAssetImporter`would require nothing more than the following:

```java
@Component
public class AuthorAssetImporter extends AbstractWebCmsAssetImporter<AuthorAsset>
{
    private AuthorAssetRepository authorRepository;

    public AuthorAssetImporter() {
        super( "author", AuthorAsset.class );
    }

    @Override
    protected Author createDto( WebCmsDataEntry data,
                                Author existing,
                                WebCmsDataAction action,
                                Map<String, Object> dataValues ) {
        return existing != null ? existing.toDto() : new Author();
    }

    @Autowired
    void setAuthorAssetRepository( AuthorAssetRepository authorAssetRepository ) {
        this.authorAssetRepository = authorAssetRepository;
    }
}
```

Next up is our interceptor. When importing WebCmsObjects, we have several custom properties that can be imported, like _wcm:components_, _wcm:types_, ... We can also define a type \(hence why we made `AuthorAssetType`\) which defines a template for our asset. If we then import an asset with that type, the template will be copied over to our asset and the components will be created. For this we need an interceptor, which extends `EntityInterceptorAdaptor`, and where we tell the `WebCmsDefaultComponentsService `to copy the template after our asset has been created.

```java
@Component
@RequiredArgsConstructor
public class AuthorInterceptor extends EntityInterceptorAdapter<Author>
{
	private final WebCmsDefaultComponentsService webCmsDefaultComponentsService;

	@Override
	public boolean handles( Class<?> entityClass ) {
		return Author.class.isAssignableFrom( entityClass );
	}

	@Override
	public void afterCreate( Author entity ) {
		webCmsDefaultComponentsService.createDefaultComponents( entity, new HashMap<>() );
	}
}
```



