# Image model {#image-model}

WebCmsModule provides an asset representing an image file.  The main entity is the `WebCmsImage` and you can use the `WebCmsImageRepository` to query the image entities.

WebCmsModule does not actually store the physical image data.  It routes all interactions with the physical files through a `WebCmsImageConnector` that is responsible for the connection with an image repository.  The image repository is also expected to serve as CDN for rendering images in a browser.

The administration UI allows you to upload and select images in components.  Different utility functions are available to generate urls to the images for use in templates.

## Connecting to an image repository

Out of the box, WebCmsModule has support for connecting to either [Cloudinary](http://cloudinary.com/) or a [Foreach ImageServer](https://repository.foreach.be/projects/image-server/4.0.0.RELEASE/reference/) image repository.

### Cloudinary connector

To connect to Cloudinary for your images you should add the Cloudinary dependency to your application and add the properties with the correct credentials.

> WebCmsModule will only store the image assets in your Cloudinary account and will generate optimized urls for rendering an image.  Previously existing assets will not be read from your Cloudinary account.

##### pom.xml

```xml
<dependency>
    <groupId>com.cloudinary</groupId>
    <artifactId>cloudinary-http44</artifactId>
    <version>1.14.0</version>
</dependency>
```

##### application.yml

```yaml
webCmsModule:
  images:
    cloudinary:
      enabled: true
      cloudName: yourCloudName
      apiKey: 123456789
      apiSecret: secret
      # Optionally specify additional settings
      settings:
        secure: true
```

When setting **webCmsModule.images.cloudinary.enabled** to `true`, WebCmsModule will create its own `Cloudinary` instance with the specified values.  If your application creates a `Cloudinary` bean manually however, there is no need to specify the properties as WebCmsModule will automatically detect it and create the corresponding `WebCmsImageConnector`.

> When you delete a `WebCmsImage`, the corresponding image asset in your Cloudinary account will be deleted as well.  If you do not want this, you can manually create a `CloudinaryWebCmsImageConnector` with different settings.

### ImageServer connector

To connect to a remote ImageServer for your images you should add the ImageServer Client dependency to your application and add the properties with the correct credentials.

> WebCmsModule will only store the image assets in your ImageServer account and will generate optimized urls for rendering an image.  Previously existing assets will not be read from ImageServer.

##### pom.xml

```xml
<dependency>
    <groupId>com.foreach.imageserver</groupId>
    <artifactId>imageserver-client</artifactId>
    <version>4.0.0.RELEASE</version>
</dependency>
```

##### application.yml

```yaml
webCmsModule:
  images:
    imageServer:
      enabled: true
      url: "http://remote.imageserver.domain/resources/images"
      hashToken: optionalHashToken
      accessToken: secureApiAccessToken
```

When not using a **hashToken**, retrieving the resolution **0x0** should be allowed in order to retrieve the original image.

When setting **webCmsModule.images.imageServer.enabled** to `true`, WebCmsModule will create its own `ImageServerClient` instance with the specified values.  If your application creates an `ImageServerClient` bean manually however, there is no need to specify the properties as WebCmsModule will automatically detect it and create the corresponding `WebCmsImageConnector`.

If your application is also the ImageServer \(by running ImageServerCoreModule\), there is no need to add the client dependency seperately or to specify any property credentials.  If you ensure the ImageServerCoreModule is configured to create a local client, it will be picked up automatically.

> When you delete a `WebCmsImage`, the corresponding image asset in your ImageServer will be deleted as well.  If you do not want this, you can manually create a `ImageServerWebCmsImageConnector` with different settings.

### Using a custom connector

You can easily create a connector for a different type of image store.  Simply provide a bean called **webCmsImageConnector** that implements `WebCmsImageConnector`.  The bean will be automatically detected by WebCmsModule and will be integrated with the `WebCmsImage` infrastructure.

## Rendering images

Rendering images is done by generating an url to the image from the image repository.  This can be done directly through `WebCmsImageConnector` or by using the `WebCmsRenderUtilityService`.

The [Thymeleaf dialect](/thymeleaf-dialect.adoc) has the **\#wcm** expression object to easily generate the url for a `WebCmsImage` in your Thymeleaf template.

## Importing images

You can easily import images by providing a resource location to the actual image file.

Any property of `WebCmsImage` can either refer the **objectId** \(starting with _wcm:asset:image_\) or be a resource location representing the image file.  If the latter, an object id will be generated and if no `WebCmsImage` with that object id can be found, the resource file will be uploaded.

Only if the resource location changes between import runs would a new image be uploaded, otherwise it is perfectly safe to re-run the import.

> The ability to import images depends on the presence of the `WebCmsImageConnector` bean.  Because it is possible that a `WebCmsImageConnector` is only initialized late during the application bootstrap, data installers are best set to execute in the `AfterContextBootstrap` phase.

##### Example image import yaml

```yaml
#
# Example of importing image web components.
# The actual WebCmsImage of the components is specified by setting the
# image property to a resource location of the image file.
#
assets:
  component:
    custom-component:
      title: Custom container with Image member
      componentType: custom-type-with-image-member
      wcm:components:
        image:
          image: "file:./src/main/test-data/test-image.jpg"
    sample-image:
      title: Sample image of a deer
      componentType: image
      image: "classpath:installers/test-data/deer.jpg"
    sample-image-from-url:
      title: Sample image fetched from URL
      componentType: image
      image: "http://images.freeimages.com/images/large-previews/afa/black-jaguar-1402097.jpg"
```

> All Spring resource locations are supported.  Take into account that URL resources using HTTPS will only work if the executing Java runtime has the required SSL certificates to connect to the remote host.



