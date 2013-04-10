Service IDL
===========

This Interface Definition Language specification describes the interfaces and
methods that a service implementation will expose to an extension's templates,
blocks, controllers and web services.

The data structure used in the payloads is defined in an XSD. Metadata about a
service call, such as version, required permissions, and locations of XSDs, are
carried by PHP annotations. Data structure and metadata are also specified.

Some advantages of using an IDL to define service implementations:

-   Allows for well-defined and versioned interfaces that the services and
    visual components depend on. This makes it simpler to control compatibility
    issues during upgrades.

-   An IDL can support more than one version of the domain/business logic
    running in the Magento instance at the same time, increasing backward
    compatibility.

-   To support REST and SOAP Web APIs, WADL and WSDL will be auto-generated from
    the IDL.
    

PHP Annotations
---------------



<table style="border:1px solid black;border-collapse:collapse;">
<tr>
<th style="border:1px solid black;"> Annotation </th>
<th style="border:1px solid black;"> Scope </th>
<th style="border:1px solid black;"> Values </th>
<th style="border:1px solid black;"> Required </th>
<th style="border:1px solid black;"> Definition </th>
</tr>
<tr>
<td style="border:1px solid black;"> @Service </td>
<td style="border:1px solid black;"> service </td>
<td style="border:1px solid black;"> string </td>
<td style="border:1px solid black;"> X </td>
<td style="border:1px solid black;"> Identifies the service being implemented </td>
</tr>
<tr>
<td style="border:1px solid black;"> @Version </td>
<td style="border:1px solid black;"> service </td>
<td style="border:1px solid black;"> N.M. </td>
<td style="border:1px solid black;"> X </td>
<td style="border:1px solid black;"> Service interface version </td>
</tr>
<tr>
<td style="border:1px solid black;"> @Consumes </td>
<td style="border:1px solid black;"> Call </td>
<td style="border:1px solid black;"> String </td>
<td style="border:1px solid black;"> X </td>
<td style="border:1px solid black;"> Identifies the location of the XSD that defines the structure of input parameters and the root element of the data structure. </td>
</tr>
<tr>
<td style="border:1px solid black;"> @Produces </td>
<td style="border:1px solid black;"> Call </td>
<td style="border:1px solid black;"> String </td>
<td style="border:1px solid black;"> X </td>
<td style="border:1px solid black;"> Identifies the location of the XSD that defines the structure of output parameters and the root element of the data structure. </td>
</tr>
</table>



Service Calls
---

A service operation is implemented as a class method. A class is mapped to a
service because all operations from a service are grouped into a single class.

**Example of a service call**

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

/**
 * @Service product
 * @Version 1.0
 * @Path /products
 */
class Mage_Catalog_Service_Product
{
	/**
     * Returns info about one particular product.
     *
     * @Type call
     * @Method GET
     * @Path /:id
     * @Bindings [REST]
     * @Consumes (schema="etc/resources/product/item/input.xsd", element="id")
     * @Produces (schema="etc/resources/product/item/output.xsd", element="product")
     * @param int $id
     * @return array
     */
    public function item($id)
    {
        $this->_getObject($id);
        $data = $this->getProduct($id);
        $data = array_merge($data, $this->getAdditionalDetails($id));
        return $data;
    }
}

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Here, the input data structure is identified in the @Consumes annotation as
belonging to the <id\> element in [input.xsd][1], and the output data structure
is identified in the @Produces annotation as belonging to the <product\>
element in [output.xsd][2]. The shipping/consumerux.xsd file is shown below.

[1]: <https://github.com/magento/magento2/blob/master/app/code/Mage/Catalog/etc/resources/product/item/input.xsd>
[2]: <https://github.com/magento/magento2/blob/master/app/code/Mage/Catalog/etc/resources/product/item/output.xsd>

The PHP function receives an associative array ($id function parameter)
reflecting the structure described by the element <id\>. It returns another
associative array reflecting the structure described by the element <product\>.

XSD
---

The XSD is used to describe the data structure passed in during the service
method call as well as the returned data. During runtime data is stored into a
associative array.

**[etc/resources/product/item/input.xsd][1]**

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema">
    <xs:element name="id" type="xs:integer"/>
</xs:schema>

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**[etc/resources/product/item/output.xsd][2]**

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
           xmlns:mage="http://www.magento.com">
    <xs:complexType name="product">
        <xs:sequence>
            <xs:element name="entityId" type="xs:integer" />
            <xs:element name="sku">
                <xs:simpleType>
                    <xs:restriction base="xs:string">
                        <xs:maxLength value="64"/>
                    </xs:restriction>
                </xs:simpleType>
            </xs:element>
            <xs:element name="name" type="xs:string"/>
            <xs:element name="description" type="xs:string"/>
            <xs:element name="shortDescription" type="xs:string"/>
            <xs:element name="price" type="mage:price"/>
        </xs:sequence>
        ...
        ...
        ...
    </xs:complexType>
</xs:schema>

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The sample output associative array returned by the service method is shown below. The input is an integer id (e.g: 1234).


**Sample output**

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Array
(
    [StoreId] => 1
    [EntityId] => 1
    [EntityTypeId] => 4
    [AttributeSetId] => 4
    [TypeId] => simple
    [Sku] => Simple1
    [HasOptions] => 0
    [RequiredOptions] => 0
    [CreatedAt] => 2013-03-20 16:35:27
    [UpdatedAt] => 2013-04-04 18:31:26
    [Description] => 
    [ShortDescription] => 
    [MetaKeyword] => Simple1,Simple1
    [CustomLayoutUpdate] => 
    [Status] => 1
    [IsRecurring] => 0
    [Visibility] => 4
    [QuantityAndStockStatus] => Array
        (
            [is_in_stock] => 1
            [qty] => 999.0000
        )
 
    [TaxClassId] => 2
    [EnableGooglecheckout] => 1
    [Name] => Simple1
    [MetaTitle] => Simple1
    [MetaDescription] => Simple1 
    [Image] => /e/b/ebay.png
    [SmallImage] => /e/b/ebay.png
    [Thumbnail] => /e/b/ebay.png
    [MediaGallery] => Array
        (
            [images] => Array
                (
                    [0] => Array
                        (
                            [value_id] => 1
                            [file] => /e/b/ebay.png
                            [label] => ebay.png
                            [position] => 1
                            [disabled] => 0
                            [label_default] => ebay.png
                            [position_default] => 1
                            [disabled_default] => 0
                        )
 
                )
 
            [values] => Array
                (
                )
 
        )
 
    [UrlKey] => simple1
    [UrlPath] => simple1.html
    [CustomDesign] => 
    [PageLayout] => 
    [OptionsContainer] => container2
    [ImageLabel] => ebay.png
    [SmallImageLabel] => ebay.png
    [ThumbnailLabel] => ebay.png
    [CountryOfManufacture] => 
    [MsrpEnabled] => 2
    [MsrpDisplayActualPriceType] => 4
    [GiftMessageAvailable] => 
    [GiftWrappingAvailable] => 
    [IsReturnable] => 2
    [Price] => 12.0000
    [SpecialPrice] => 
    [Weight] => 
    [TierPrice] => Array
        (
            [32000-2] => Array
                (
                    [price_id] => 1
                    [website_id] => 1
                    [all_groups] => 1
                    [cust_group] => 32000
                    [price] => 11
                    [price_qty] => 2.0000
                    [website_price] => 11
                )
 
        )
 
    [Msrp] => 
    [GiftWrappingPrice] => 
    [SpecialFromDate] => 
    [SpecialToDate] => 
    [NewsFromDate] => 
    [NewsToDate] => 
    [CustomDesignFrom] => 
    [CustomDesignTo] => 
    [GroupPrice] => Array
        (
        )
 
    [GroupPriceChanged] => 0
    [TierPriceChanged] => 0
    [CategoryIds] => Array
        (
        )
 
    [IsInStock] => 1
    [IsSalable] => 1
    [WebsiteIds] => Array
        (
            [0] => 1
        )
 
    [name] => Simple1
    [sku] => Simple1
    [description] => 
    [shortDescription] => 
    [price] => Array
        (
            [amount] => 12.0000
            [currencyCode] => USD
            [formattedPrice] => $12.00
        )
 
    [weight] => 
    [metaTitle] => Simple1
    [metaKeyword] => Simple1,Simple1
    [metaDescription] => Simple1 
    [images] => Array
        (
            [image] => /e/b/ebay.png
            [smallImage] => /e/b/ebay.png
            [thumbnail] => /e/b/ebay.png
        )
 
    [mediaGallery] => Array
        (
            [images] => Array
                (
                    [0] => Array
                        (
                            [image] => Array
                                (
                                    [valueId] => 1
                                    [file] => /e/b/ebay.png
                                    [label] => ebay.png
                                    [position] => 1
                                    [isDisabled] => 
                                    [labelDefault] => ebay.png
                                    [positionDefault] => 1
                                    [isDisabledDefault] => 
                                    [url] => http://vagrant.vm/api/pub/media/catalog/product/e/b/ebay.png
                                    [id] => 1
                                    [path] => /var/www/api/pub/media/catalog/product/e/b/ebay.png
                                )
 
                        )
 
                )
 
        )
 
    [groupPrice] => Array
        (
            [amount] => 12.0000
            [currencyCode] => USD
            [formattedPrice] => $12.00
        )
 
    [tierPrice] => Array
        (
            [0] => Array
                (
                    [tierPrice] => Array
                        (
                            [priceId] => 1
                            [websiteId] => 1
                            [allGroups] => 1
                            [customerGroup] => 32000
                            [price] => Array
                                (
                                    [amount] => 11
                                    [currencyCode] => USD
                                    [formattedPrice] => $11.00
                                )
 
                            [priceQty] => 2.0000
                            [websitePrice] => Array
                                (
                                    [amount] => 11
                                    [currencyCode] => USD
                                    [formattedPrice] => $11.00
                                )
 
                        )
 
                )
 
        )
 
    [status] => 1
    [urlKey] => simple1
    [urlPath] => simple1.html
    [isRecurring] => 0
    [visibility] => 4
    [categoryIds] => Array
        (
        )
 
    [optionsContainer] => container2
    [requiredOptions] => 0
    [hasOptions] => 0
    [createdAt] => 2013-03-20 16:35:27
    [updatedAt] => 2013-04-04 18:31:26
    [msrpEnabled] => 2
    [msrpDisplayActualPriceType] => 4
    [taxClassId] => 2
    [isReturnable] => 2
    [isInStock] => 1
    [qty] => 999.0000
    [websiteIds] => Array
        (
            [0] => Array
                (
                    [id] => 1
                )
 
        )
 
)

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

