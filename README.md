# PHTAL - Profiled HyperText Application Language
## An opinionated hypermedia type
* Author: Jose Montoya
* Created: 2018-07-03

## Summary
PHTAL is an opinionated evolution from the [HAL media type](http://stateless.co/hal_specification.html) that adds elements for in-band instructions and defines a profile structure to breach the gap to design time description languages.

## General Description 
PHTAL representations provide in-band instructions for interacting with resources. This allows agents to evaluate the alternatives provided and submit the appropriate data or select the appropriate link to get it to the next state.

### The Link element
The link element is very much based on the HAL specification in that it will provide the relation type name and its corresponding URI, the main contribution of PHTAL is the link's  Instructions and Partial elements.

```xml
<catalog xmlns="http://localhost/schemas/xsd/Catalog" xmlns:phtal="http://localhost/schemas/xsd/Phtal">
    <name>summerCatalog</name>
    <items>1</items>
    <phtal:links rel="item" href="http://localhost/catalog/items/23-1289">
</catalog>
```

### The Instructions element
The `instructions` elements aims to inform the agent how to follow a particular link. The instructions content is defined by the protocol that would be used to follow that link. For example in order to follow a link using the HTTP protocol the agent at the very least needs to know what method to use, so HTTP instructions define a `method` element. Likewise there are other elements that can help guide agents in interacting with a resource such as what data to send as input `requestContent`, what type of data may be returned as part of a successful interaction `responseContent`, what security requirements are imposed by that resource`security`. Most of the language used by the http instructions are borrowed from the OpenAPI specification.

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

A link may have more than one instructions element as long as they are of different protocols, this allows the URIs to be used for resource identification only and not tie them to communication protocols.

### The Partial element
The `partial` element is based on the `embedded` elements defined by HAL but explicitly defines that they should not be considered full representations even if their contents are the same. Partial representations should provide just enough information for agents to be able to discern which link they want to follow not as mechanism to batch interactions. The `profile` attribute identifies the profile that extends the processing model for its content.

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
