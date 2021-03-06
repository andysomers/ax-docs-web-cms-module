# Message codes

WebCmsModule allows you to customize the labels and messages shown with your custom component types in the administration UI.  The initial prefix of component type message codes is **webCmsComponents.**_**yourComponentType**_.

In case of a container component, every member can have its labels customized, the prefix for an individual member is **webCmsComponents.**_**yourComponentType**_**.members.**_**memberName**_.  After the prefix, the same codes can be specified as with a regular component type.

## Default message codes
Default component type admin has 4 possible sections \(tabs in the UI\):

* **content**: contains the custom rendered view elements \(eg. for a text-field type\)
* **members**: contains the member management in case of a container or fixed-container type
* **metadata**: contains the metadata form, the form generated for the metadata type
* **settings**: contains the general component information like name and title

Every section supports the following keys:

* **[description]**: description text rendered on top of the section (before the controls)
* **[additionalDescription]**: description text rendered at the bottom of the section (after the controls)

The *metadata* section represents the custom metadata type registered.  As such all EntityModule message codes are available to customize the generated form.

Examples of actual message codes resolved would be:
* `webCmsComponents.myComponentType.content[description]`
* `webCmsComponents.myComponentType.members[additionalDescription]`
* `webCmsComponents.myComponentType.members.body.metadata[description]`
* `webCmsComponents.myComponentType.members.body.metadata.properties.title`

## Specific message codes
Specific message codes are available depending on the context or the component type.

* **[placholder]**: this key is available on the content section for any component type extending `WebCmsTextComponentModel`


## Example
Example of component type message codes.

```properties
# add some description for the default text-field
webCmsComponents.text-field.content[additionalDescription]=Enter a single line of text.
webCmsComponents.text-field.content[placeholder]=Your text...

# section is a custom component type extending fixed-container
webCmsComponents.section.members[description]=This is a global description for the "members" tab.

# image and body are members of the section container
webCmsComponents.section.members.image.content[description]=Optionally select an image to display.
webCmsComponents.section.members.body.content[additionalDescription]=Body of the section can be <u>rich-text</u>.

# metadata is a specific type, labels can be defined 
webCmsComponents.section.metadata[description]=This is a global description for the "metadata" tab.

# since the metadata is a custom class, property labels can be configured just like EntityModule properties
webCmsComponents.section.metadata.properties.shortTitle[placeholder]=Optional title
webCmsComponents.section.metadata.properties.shortTitle[description]=Leave empty if you do not wish to display a title.
```



