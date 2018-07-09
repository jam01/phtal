# PHTAL - Profiled HyperText Application Language
* Author: Jose Montoya
* Created: 2018-05-28

## Summary
In a REST/HTTP application the origin server/user agent pair rely on a shared out-of-band understanding of communication protocols, URI schemes, non-generic media types definitions and typed link relations. In-band, the agent requires instructions from the origin server on how to interact with a given resource.

PHTAL (pronounced fatal) is an opinionated evolution from the [HAL media type](http://stateless.co/hal_specification.html) that draws from various other projects to provide all that is necessary for truly REST/HTTP applications. Those projects include [ALPS](http://alps.io/), [Hyperschema](https://datatracker.ietf.org/doc/draft-handrews-json-schema-hyperschema/), and [  HYDRA](https://www.markus-lanthaler.com/hydra/).

## General Description
Communication protocols and URI schemes are mature and generally well understood elements of a REST/HTTP application, PHTAL makes no contributions here.

### In-band contributions
PHTAL representations provide in-band instructions for interacting with resources. This allows agents to evaluate the alternatives provided and submit the appropriate data or select the appropriate link to get it to the next state.

#### The Link element
The link element is very much based on the HAL specification in that it will provide the relation type name and its corresponding URI, the main contribution of PHTAL is the link's  Instructions and Partial elements.

```xml
<catalog xmlns="http://localhost/schemas/xsd/Catalog" xmlns:phtal="http://localhost/schemas/xsd/Phtal">
    <name>summerCatalog</name>
    <items>1</items>
    <phtal:links rel="item" href="http://localhost/catalog/items/23-1289">
</catalog>
```

##### The Instructions element
The `instructions` elements aims to inform the agent how to follow a particular link. The instructions content is defined by the protocol that would be used to follow that link. For example in order to follow a link using the HTTP protocol the agent at the very least needs to know what method to use, so HTTP instructions define a `method` element. Likewise there are other elements that can help guide agents in interacting with a resource such as what data to send as input `requestContent`, what type of data may be returned as part of a successful interaction `responseContent`, what security requirements are imposed by that resource`security`. Most of the language used by the HTTP instructions content is borrowed from the OpenAPI specification.

```xml
<catalog xmlns="http://localhost/schemas/xsd/Catalog" xmlns:phtal="http://localhost/schemas/xsd/Phtal">
    <name>summerCatalog</name>
    <items>1</items>
    <phtal:links rel="http://localhost/rels/addItem" href="http://localhost/catalog/items">
        <phtal:instructions protocol="http">
            <phtal:method>POST</phtal:method>
            <phtal:security type="apiKey" name="auth-api-key" in="header"/>
            <phtal:requestContentRequired>true</phtal:requestContentRequired>
            <phtal:requestContent>application/phtal+xml;profile="http://localhost/profiles/catalog#addItemRequest"</phtal:requestContent>
            <phtal:responseContent>application/phtal+xml;profile="http://localhost/profiles/catalog#addItemResult"</phtal:responseContent>
        </phtal:instructions>
    </phtal:links>
</catalog>
```

A link may have more than one instructions element as long as they are of different protocols, this allows the URIs to be used for resource identification only and not tie them to communication protocols, ie. `http://localhost/catalog/items/23-1289` specifies that the URI follows the HTTP scheme and not that it should only allow HTTP interactions.

##### The Partial element
The `partial` element is based on the `embedded` elements defined by HAL but explicitly defines that they should not be considered full representations even if their contents happen to be the same. Partial representations should provide just enough information for agents to be able to discern which link they want to follow and not as mechanism to batch interactions. The `profile` attribute identifies the profile that extends the processing model for the partial.

```xml
<catalog xmlns="http://localhost/schemas/xsd/Catalog" xmlns:phtal="http://localhost/schemas/xsd/Phtal">
    <name>summerCatalog</name>
    <items>1</items>
    <phtal:links rel="item" href="http://localhost/catalog/items/23-1289">
        <phtal:instructions protocol="http">
            <phtal:method>GET</phtal:method>
            <phtal:requestContentRequired>false</phtal:requestContentRequired>
            <phtal:responseContent>application/phtal+xml;profile="http://localhost/profiles/catalog#catalogItem"</phtal:responseContent>
        </phtal:instructions>
        <phtal:partial profile="http://localhost/profiles/catalog#catalogItem">
          <item:Item xmlns:item="http://localhost/schemas/xsd/CatalogItem">
            <item:namex>soccerBall</namex>
            <item:price>$45</price>
          </item:Item>
        </phtal:partial>
    </phtal:links>
</catalog>
```

### Out-of-band contributions
The out-of-band contributions are an attempt to bridge the gap between design time languages like RAML and OpenAPI and truly REST/HTTP applications. With that in mind the most of the language used by Profiles and Extension Link Relation Types is based on the OpenAPI specification.

#### Extension Link Relation Types
PHTAL defines a format for describing extension link relation types, it's based on the [Web Linking RFC](https://tools.ietf.org/html/rfc8288) without including any Target Attributes unlike what HAL suggests, instead leaving those details to be provided in-band.
```yaml
name: addItem
description: A service that adds the supplied item to the context catalog
type: unsafe
```

#### Profiles
Media types are currently mostly used to indicate generic data transfer formats, ie. xml or json. The [intent of media type metadata](https://www.w3.org/2001/tag/doc/mime-respect-20130422#media-type) on HTTP interactions is not only to indicate data format but the sender's preferred interpretation of that format, ie. an application specific format. PHTAL attempts to complement generic media types with a profile that extends its semantics to a specific application. This is based on ALPS profiles and HAL's specification of a profile parameter, whereas PHTAL makes the profile parameter required and defines a more concrete take on what a profile should be. The profile element is specifically based on OpenAPI [media type object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md#media-type-object).

```yaml
description: A store's catalog item.
baseType: application/xml
schema:
  type: xsd
  externalValue: http://localhost/jam01/schemas/xsd/CatalogItem
examples:
  productItem:
    summary: A simple example that shows a product item, a soccer ball.
    externalValue: http://localhost/jam01/examples/xml/SoccerBall
  serviceItem:
    description: This item shows all the additional items that could be present on a catalog item for a service, these kind of items will only be available...
    externalValue: http://localhost/jam01/examples/xml/CarWash
```
