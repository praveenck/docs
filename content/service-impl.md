Service Implementation
======================

This is the proposed model for implementing Magento services. Using a service
layer with interfaces described by service IDLs provides the following
advantages for developing and maintaining Magento extensions:

-   A service encapsulates domain/business logic in a single component.

-   Multiple implementations of a service can be supported in the same Magento
    instance.

-   A service's owner controls  the quality of the service component, because
    other components are restricted from directly modifying the implementation.

-   Data access becomes a private concern of the service implementation, which
    increases predictability and compatibility.

-   A service can be exposed as a Web API. To see how this is done, check out
    [Creating Web APIs][1].

[1]: <http://magento.github.io/magento2-developer-docs/web-api/>

-   The data are defined declaratively in schemas, rather than in PHP classes.

Implementing a Service
----------------------

The first step in creating a service component is to define the service using
the service IDL.

Once you have defined a service IDL, you implement the service using a PHP
class. You make the public methods of this PHP class available to your
extension's presentation layer, which uses the methods to access data.

As an example, the documentation for the Product service IDL [here][2] describes
how to define a service using the service IDL.

[2]: <http://magento.github.io/magento2-developer-docs/service-idl/>

The source code for Product service is [here][4]

[4]: <https://github.com/magento/magento2/blob/master/app/code/Mage/Catalog/Service/Product.php>

The service has a method, `item` (line 117</i>). The method takes a single
parameter, which is the product ID, and retrieves product data in an array
defined by the response schema.

The `@Consumes` annotation names the schema for the request, and `@Produces`
annotation names the schema for the response defined in the IDL. (By the way,
`@Version` refers to the version of the service, not the individual call.)

The business logic for the service method is implemented using models and
resource models.

Once you have implemented the service as a PHP class, you make the public
methods of the service available to the presentation tier components by
declaring the service-calls in [service-calls.xml][5].

[5]: <https://github.com/magento/magento2/blob/master/app/code/Mage/Catalog/etc/service-calls.xml>

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

<service-calls>
    <service-call name="selectedProductDetails" service="Mage_Catalog_Service_Product" method="item">
        <arg name="productId">{{request.params.id}}</arg>
    </service-call>

    <service-call name="selectedProductOptions" service="Mage_Catalog_Service_Product" method="getOptions">
        <arg name="productId">{{request.params.id}}</arg>
    </service-call>


    <service-call name="relatedProducts" service="Mage_Catalog_Service_Product" method="getRelatedProducts">
        <arg name="productId">{{request.params.id}}</arg>
    </service-call>
</service-calls>

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Once the service is implemented and its service methods are declared as
service-calls, you invoke the service-call by including the following
declaration in layout.xml

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

<block type="Mage_Catalog_Block_Product_Twig" module="Mage_Catalog" name="product.info"
                   template="Mage_Catalog::product/view.twig">
	<data service-call="selectedProductDetails" alias="product" />
	...
	...
</block>

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Product service-calls are invoked in the Catalog Product layout when
rendering the templates for the Product View page. For details, you can refer to
the [layout.xml][6] for Catalog Product.

[6]: <https://github.com/magento/magento2/blob/master/app/code/Mage/Catalog/view/frontend/layout.xml>

Below are details of the input and output data schema for Product service item
method.

### Input Schema 

This schema defines the service's single request parameter.

<table style="border:1px solid black;border-collapse:collapse;">
<tr>
<th style="border:1px solid black;"> Name </th>
<th style="border:1px solid black;"> Type </th>
<th style="border:1px solid black;"> Description </th>
<th style="border:1px solid black;"> Sample </th>
</tr>
<tr>
<td style="border:1px solid black;"> <b>id</b> </td>
<td style="border:1px solid black;"> xs:integer </td>
<td style="border:1px solid black;"> The product ID number </td>
<td style="border:1px solid black;"> 1234 </td>
</tr>
</table>

### Output Schema

This schema defines the array to be returned  by the product service item
method.

<table style="border:1px solid black;border-collapse:collapse;">
<tr>
<th style="border:1px solid black;"> Name </th>
<th style="border:1px solid black;"> Type </th>
<th style="border:1px solid black;"> Description </th>
<th style="border:1px solid black;"> Sample </th>
</tr>

<tr>
<td style="border:1px solid black;"> <b>product</b> </td>
<td style="border:1px solid black;"> <b>mage:product</b> </td>
<td style="border:1px solid black;"> <b></b> </td>
<td style="border:1px solid black;"> <b></b> </td>
</tr>

<tr>
<td style="border:1px solid black;"> product.<b>data</b></td> 
<td style="border:1px solid black;"> <b></b> </td>
<td style="border:1px solid black;"> <b></b> </td>
<td style="border:1px solid black;"> <b></b> </td>
</tr>

<tr>
<td style="border:1px solid black;"> product.<b>relatedData</b></td>
<td style="border:1px solid black;"> <b></b> </td>
<td style="border:1px solid black;"> <b></b> </td>
<td style="border:1px solid black;"> <b></b> </td>
</tr>

<tr>
<td style="border:1px solid black;"> <b>dictionary</b></td>
<td style="border:1px solid black;"> Complex type </td>
<td style="border:1px solid black;"> <b></b> </td>
<td style="border:1px solid black;"> <b></b> </td>
</tr>

<tr>
<td style="border:1px solid black;"> dictionary.<b>urlInStore</b></td>
<td style="border:1px solid black;"> xs:anyURI </td>
<td style="border:1px solid black;"> <b></b> </td>
<td style="border:1px solid black;"> <b></b> </td>
</tr>

<tr>
<td style="border:1px solid black;"> dictionary.<b>urlPath</b></td>
<td style="border:1px solid black;"> xs:string </td>
<td style="border:1px solid black;"> <b></b> </td>
<td style="border:1px solid black;"> <b></b> </td>
</tr>

<tr>
<td style="border:1px solid black;"> dictionary<strong>url</strong></td>
<td style="border:1px solid black;">  xs.anyURI</td>
<td style="border:1px solid black;"> <b></b> </td>
<td style="border:1px solid black;"> <b></b> </td>
</tr>

<tr>
<td style="border:1px solid black;"> dictionary.<strong>submitUrl</strong></td>  
<td style="border:1px solid black;">xs.anyURI</td>
<td style="border:1px solid black;"> <b></b> </td>
<td style="border:1px solid black;"> <b></b> </td>
</tr>

<tr>
<td style="border:1px solid black;"> dictionary.<strong>compareURL</strong></td>
<td style="border:1px solid black;"> xs:anyURI</td>
<td style="border:1px solid black;"> <b></b> </td>
<td style="border:1px solid black;"> <b></b> </td>
</tr>

<tr>
<td style="border:1px solid black;"> dictionary.<strong>compareURL</strong></td>
<td style="border:1px solid black;"> xs:anyURI</td>
<td style="border:1px solid black;"> <b></b> </td>
<td style="border:1px solid black;"> <b></b> </td>
</tr>

<tr>
<td style="border:1px solid black;"> dictionary.<strong>fileViewURL</strong></td>
<td style="border:1px solid black;"> xs:anyURI</td>
<td style="border:1px solid black;"> <b></b> </td>
<td style="border:1px solid black;"> <b></b> </td>
</tr>

<tr>
<td style="border:1px solid black;"> dictionary.<strong>emailToFriendURL</strong></td>
<td style="border:1px solid black;">xs:anyURI</td>
<td style="border:1px solid black;"> <b></b> </td>
<td style="border:1px solid black;"> <b></b> </td>
</tr>

<tr>
<td style="border:1px solid black;"> dictionary.<strong>canEmailToFriend</strong></td>
<td style="border:1px solid black;"> xs:anyURI</td>
<td style="border:1px solid black;"> <b></b> </td>
<td style="border:1px solid black;"> <b></b> </td>
</tr>

<tr>
<td style="border:1px solid black;"> dictionary.<strong>hasOptions</strong></td>
<td style="border:1px solid black;"> xs:boolean</td>
<td style="border:1px solid black;"> <b></b> </td>
<td style="border:1px solid black;"> <b></b> </td>
</tr>

<tr>
<td style="border:1px solid black;"> dictionary.<strong>wishListDataAllowed</strong></td>
<td style="border:1px solid black;">xs:boolean</td>
<td style="border:1px solid black;"> <b></b> </td>
<td style="border:1px solid black;"> <b></b> </td>
</tr>

<tr>
<td style="border:1px solid black;"> dictionary.<strong>optionsContainer</strong></td>
<td style="border:1px solid black;"> xs:string</td>
<td style="border:1px solid black;"> <b></b> </td>
<td style="border:1px solid black;"> <b></b> </td>
</tr>

<tr>
<td style="border:1px solid black;"> dictionary.<strong>jsonConfig</strong></td>
<td style="border:1px solid black;"> xs:string</td>
<td style="border:1px solid black;"> <b></b> </td>
<td style="border:1px solid black;"> <b></b> </td>
</tr>

<tr>
<td style="border:1px solid black;"> dictionary.<strong>isSaleable</strong></td>
<td style="border:1px solid black;"> xs:boolean</td>
<td style="border:1px solid black;"> <b></b> </td>
<td style="border:1px solid black;"> <b></b> </td>
</tr>

<tr>
<td style="border:1px solid black;"> dictionary.<strong>formattedPrice</strong></td>
<td style="border:1px solid black;"> xs:string</td>
<td style="border:1px solid black;"> <b></b> </td>
<td style="border:1px solid black;"> <b></b> </td>
</tr>
</table>

<p> The table below documents complex type Product from the above schema </p>

<table style="border:1px solid black;border-collapse:collapse;">
<tr>
<th style="border:1px solid black;"> Name </th>
<th style="border:1px solid black;"> Type </th>
<th style="border:1px solid black;"> Description </th>
<th style="border:1px solid black;"> Sample </th>
</tr>

<tr>
<td style="border:1px solid black;"> name </td>
<td style="border:1px solid black;"> string </td>
<td style="border:1px solid black;"> Product name. <b>Required</b>. </td>
<td style="border:1px solid black;"> New product </td>
</tr>
<tr>
<td style="border:1px solid black;"> sku </td>
<td style="border:1px solid black;"> string </td>
<td style="border:1px solid black;"> Product SKU. Constrains: maxLength value = 64. <b>Required</b>. </td>
<td style="border:1px solid black;"> new_product </td>
</tr>
<tr>
<td style="border:1px solid black;"> description </td>
<td style="border:1px solid black;"> string </td>
<td style="border:1px solid black;"> Product description. <b>Required</b>.  </td>
<td style="border:1px solid black;"> This is a new product.  </td>
</tr>
<tr>
<td style="border:1px solid black;"> shortDescription </td>
<td style="border:1px solid black;"> string </td>
<td style="border:1px solid black;"> Product short description. <b>Required</b>. </td>
<td style="border:1px solid black;"> A new product.  </td>
</tr>
<tr>
<td style="border:1px solid black;"> price </td>
<td style="border:1px solid black;"> mage:price </td>
<td style="border:1px solid black;"> Product price. <b>Required</b>.  </td>
<td style="border:1px solid black;"> 200 </td>
</tr>
<tr>
<td style="border:1px solid black;"> <b>specialPrice</b> </td>
<td style="border:1px solid black;"> complex type </td>
<td style="border:1px solid black;">   </td>
<td style="border:1px solid black;">  </td>
</tr>
<tr>
<td style="border:1px solid black;"> specialPrice.<b>specialPrice</b> </td>
<td style="border:1px solid black;"> mage:price </td>
<td style="border:1px solid black;"> Product special price. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 150 </td>
</tr>
<tr>
<td style="border:1px solid black;"> specialPrice.<b>specialFromDate</b> </td>
<td style="border:1px solid black;"> dateTime </td>
<td style="border:1px solid black;"> Date starting from which the special price will be applied to the product. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 2005-08-15T15:52:01+0000 </td>
</tr>
<tr>
<td style="border:1px solid black;"> specialPrice.<b>specialToDate</b> </td>
<td style="border:1px solid black;"> dateTime </td>
<td style="border:1px solid black;"> Date till which the special price will be applied to the product. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 2005-08-15T15:52:01+0000  </td>
</tr>
<tr>
<td style="border:1px solid black;"> cost </td>
<td style="border:1px solid black;"> mage:price </td>
<td style="border:1px solid black;"> Product cost. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 100 </td>
</tr>
<tr>
<td style="border:1px solid black;"> weight </td>
<td style="border:1px solid black;"> decimal </td>
<td style="border:1px solid black;"> Product weight. <b>Required</b>. </td>
<td style="border:1px solid black;"> 0.5 </td>
</tr>
<tr>
<td style="border:1px solid black;"> manufacturer </td>
<td style="border:1px solid black;"> integer </td>
<td style="border:1px solid black;"> Product manufacturer. <b>Optional</b>. </td>
<td style="border:1px solid black;"> Test manufacturer </td>
</tr>
<tr>
<td style="border:1px solid black;"> metaTitle </td>
<td style="border:1px solid black;"> string </td>
<td style="border:1px solid black;"> Product meta title. <b>Optional</b>. </td>
<td style="border:1px solid black;"> new product  </td>
</tr>
<tr>
<td style="border:1px solid black;"> metaKeyword </td>
<td style="border:1px solid black;"> string </td>
<td style="border:1px solid black;"> Product meta keywords. <b>Optional</b>. </td>
<td style="border:1px solid black;"> new, product </td>
</tr>
<tr>
<td style="border:1px solid black;"> metaDescription </td>
<td style="border:1px solid black;"> string </td>
<td style="border:1px solid black;"> Product meta description. Constrains: maxLength value = 255. <b>Optional</b>. </td>
<td style="border:1px solid black;"> This is a new product  </td>
</tr>
<tr>
<td style="border:1px solid black;"> <b>images</b>  </td>
<td style="border:1px solid black;"> complex type  </td>
<td style="border:1px solid black;">  </td>
<td style="border:1px solid black;">  </td>
</tr>
<tr>
<td style="border:1px solid black;"> images.<b>image</b> </td>
<td style="border:1px solid black;"> mage:image_type </td>
<td style="border:1px solid black;"> Product image. <b>Optional</b>. </td>
<td style="border:1px solid black;">  </td>
</tr>
<tr>
<td style="border:1px solid black;"> images.<b>smallImage</b> </td>
<td style="border:1px solid black;"> mage:image_type </td>
<td style="border:1px solid black;"> Product small image. <b>Optional</b>. </td>
<td style="border:1px solid black;">  </td>
</tr>
<tr>
<td style="border:1px solid black;"> images.<b>thumbnail</b> </td>
<td style="border:1px solid black;"> mage:image_type </td>
<td style="border:1px solid black;"> Product thumbnail image. <b>Optional</b>. </td>
<td style="border:1px solid black;">  </td>
</tr>
<tr>
<td style="border:1px solid black;"> mediaGallery </td>
<td style="border:1px solid black;"> mage:media_gallery </td>
<td style="border:1px solid black;"> Product images. <b>Optional</b>. </td>
<td style="border:1px solid black;">  </td>
</tr>
<tr>
<td style="border:1px solid black;"> oldId </td>
<td style="border:1px solid black;"> integer </td>
<td style="border:1px solid black;"> Original system ID assigned to the product, if a catalog has been imported from legacy system. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 123 </td>
</tr>
<tr>
<td style="border:1px solid black;"> groupPrice </td>
<td style="border:1px solid black;"> mage:price </td>
<td style="border:1px solid black;"> Product group price. <b>Optional</b>. </td>
<td style="border:1px solid black;"> array of group price </td>
</tr>
<tr>
<td style="border:1px solid black;"> tierPrice </td>
<td style="border:1px solid black;"> mage:price </td>
<td style="border:1px solid black;"> Product tier price. <b>Optional</b>. </td>
<td style="border:1px solid black;"> array of tier price  </td>
</tr>
<tr>
<td style="border:1px solid black;"> color </td>
<td style="border:1px solid black;"> integer </td>
<td style="border:1px solid black;"> Product color. <b>Optional</b>. </td>
<td style="border:1px solid black;">  </td>
</tr>
<tr>
<td style="border:1px solid black;"> newsFromDate </td>
<td style="border:1px solid black;"> dateTime </td>
<td style="border:1px solid black;"> Date starting from which the product is promoted as a new product. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 2005-08-15T15:52:01+0000  </td>
</tr>
<tr>
<td style="border:1px solid black;"> newsToDate </td>
<td style="border:1px solid black;"> dateTime </td>
<td style="border:1px solid black;"> Date till which the product is promoted as a new product. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 2005-08-15T15:52:01+0000 </td>
</tr>
<tr>
<td style="border:1px solid black;"> gallery </td>
<td style="border:1px solid black;"> string </td>
<td style="border:1px solid black;"> <b>Optional</b>. </td>
<td style="border:1px solid black;">  </td>
</tr>
<tr>
<td style="border:1px solid black;"> status </td>
<td style="border:1px solid black;"> mage:integer_label </td>
<td style="border:1px solid black;"> Product status. Can have the following values: 1- Enabled, 2 - Disabled. <b>Required</b>. </td>
<td style="border:1px solid black;"> 1 </td>
</tr>
<tr>
<td style="border:1px solid black;"> urlKey </td>
<td style="border:1px solid black;"> string </td>
<td style="border:1px solid black;"> A friendly URL path for the product. <b>Optional</b>. </td>
<td style="border:1px solid black;"> new-product  </td>
</tr>
<tr>
<td style="border:1px solid black;"> urlPath </td>
<td style="border:1px solid black;"> string </td>
<td style="border:1px solid black;"> <b>Optional</b>. </td>
<td style="border:1px solid black;">  </td>
</tr>
<tr>
<td style="border:1px solid black;"> minimalPrice </td>
<td style="border:1px solid black;"> mage:price </td>
<td style="border:1px solid black;"> Minimal price of the product. <b>Optional</b>. </td>
<td style="border:1px solid black;">  </td>
</tr>
<tr>
<td style="border:1px solid black;"> isRecurring </td>
<td style="border:1px solid black;"> boolean </td>
<td style="border:1px solid black;"> Defines whether the product is recurring. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 1 </td>
</tr>
<tr>
<td style="border:1px solid black;"> recurringProfile </td>
<td style="border:1px solid black;"> string </td>
<td style="border:1px solid black;"> Information about the recurring payments. <b>Optional</b>. </td>
<td style="border:1px solid black;">  </td>
</tr>
<tr>
<td style="border:1px solid black;"> visibility </td>
<td style="border:1px solid black;"> mage:integer_label </td>
<td style="border:1px solid black;"> Product visibility. Can have the following values: 1 - Not Visible Individually, 2 - Catalog, 3 - Search, 4 - Catalog, Search. <b>Required</b>. </td>
<td style="border:1px solid black;"> 1 </td>
</tr>
<tr>
<td style="border:1px solid black;"> customDesign </td>
<td style="border:1px solid black;"> string </td>
<td style="border:1px solid black;"> Custom design applied for the product page. <b>Optional</b>. </td>
<td style="border:1px solid black;"> enterprise/default  </td>
</tr>
<tr>
<td style="border:1px solid black;"> customDesignFrom </td>
<td style="border:1px solid black;"> dateTime </td>
<td style="border:1px solid black;"> Date starting from which the custom design will be applied to the product page. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 2005-08-15T15:52:01+0000 </td>
</tr>
<tr>
<td style="border:1px solid black;"> customDesignTo </td>
<td style="border:1px solid black;"> dateTime </td>
<td style="border:1px solid black;"> Date till which the custom design will be applied to the product page. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 2005-08-15T15:52:01+0000 </td>
</tr>
<tr>
<td style="border:1px solid black;"> customLayoutUpdate </td>
<td style="border:1px solid black;"> string </td>
<td style="border:1px solid black;"> An XML block to alter the page layout. <b>Optional</b>. </td>
<td style="border:1px solid black;"> XML body </td>
</tr>
<tr>
<td style="border:1px solid black;"> pageLayout </td>
<td style="border:1px solid black;"> mage:string_label_array </td>
<td style="border:1px solid black;"> Page template that can be applied to the product page. <b>Optional</b>. </td>
<td style="border:1px solid black;"> one_column </td>
</tr>
<tr>
<td style="border:1px solid black;"> categoryIds </td>
<td style="border:1px solid black;"> mage:string_label_array </td>
<td style="border:1px solid black;"> Contains the category(s) ID (s) to which a product belongs. <b>Optional</b>. </td>
<td style="border:1px solid black;">  </td>
</tr>
<tr>
<td style="border:1px solid black;"> optionsContainer </td>
<td style="border:1px solid black;"> string </td>
<td style="border:1px solid black;"> Defines how the custom options for the product will be displayed. Can have the following values: Block after Info Column or Product Info Column. <b>Optional</b>. </td>
<td style="border:1px solid black;"> container2 </td>
</tr>
<tr>
<td style="border:1px solid black;"> requiredOptions </td>
<td style="border:1px solid black;"> boolean </td>
<td style="border:1px solid black;"> Defines whether specifying the product option is mandatory or not. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 1 </td>
</tr>
<tr>
<td style="border:1px solid black;"> hasOptions </td>
<td style="border:1px solid black;"> boolean </td>
<td style="border:1px solid black;"> Defines whether a product has the custom options or not. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 1 </td>
</tr>
<tr>
<td style="border:1px solid black;"> createdAt </td>
<td style="border:1px solid black;"> dateTime </td>
<td style="border:1px solid black;"> Defines the date of a product creation. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 2005-08-15T15:52:01+0000 </td>
</tr>
<tr>
<td style="border:1px solid black;"> updatedAt </td>
<td style="border:1px solid black;"> dateTime </td>
<td style="border:1px solid black;"> Defines the date of a product update. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 2005-08-15T15:52:01+0000 </td>
</tr>
<tr>
<td style="border:1px solid black;"> countryOfManufacture </td>
<td style="border:1px solid black;"> string </td>
<td style="border:1px solid black;"> Product's country of manufacture. <b>Optional</b>. </td>
<td style="border:1px solid black;"> AD </td>
</tr>
<tr>
<td style="border:1px solid black;"> msrpEnabled </td>
<td style="border:1px solid black;"> mage:integer_label </td>
<td style="border:1px solid black;"> The Apply MAP option. Defines whether the price in the catalog in the frontend is substituted with a Click for price link. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 1 </td>
</tr>
<tr>
<td style="border:1px solid black;"> msrpDisplayActualPriceType </td>
<td style="border:1px solid black;"> mage:integer_label </td>
<td style="border:1px solid black;"> Defines how the price will be displayed in the frontend. Can have the following values: In Cart, Before Order Confirmation, and On Gesture. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 2 </td>
</tr>
<tr>
<td style="border:1px solid black;"> msrp </td>
<td style="border:1px solid black;"> mage:price </td>
<td style="border:1px solid black;"> The Manufacturer's Suggested Retail Price option. The price that a manufacturer suggests to sell the product at. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 140 </td>
</tr>
<tr>
<td style="border:1px solid black;"> <b>quantityAndStockStatus</b> </td>
<td style="border:1px solid black;"> complexType </td>
<td style="border:1px solid black;">  </td>
<td style="border:1px solid black;">  </td>
</tr>
<tr>
<td style="border:1px solid black;"> quantityAndStockStatus.<b>isInStock</b> </td>
<td style="border:1px solid black;"> boolean </td>
<td style="border:1px solid black;"> Defines whether the product is available for selling. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 1 </td>
</tr>
<tr>
<td style="border:1px solid black;"> quantityAndStockStatus.<b>qty</b> </td>
<td style="border:1px solid black;"> decimal </td>
<td style="border:1px solid black;"> Quantity of stock items for the current product. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 99 </td>
</tr>
<tr>
<td style="border:1px solid black;"> <b>bundle</b> </td>
<td style="border:1px solid black;"> complexType </td>
<td style="border:1px solid black;">  </td>
<td style="border:1px solid black;">  </td>
</tr>
<tr>
<td style="border:1px solid black;"> bundle.<b>priceType</b> </td>
<td style="border:1px solid black;"> integer </td>
<td style="border:1px solid black;"> Defines whether the bundle product price is Dynamic or Fixed. <b>Required</b>. </td>
<td style="border:1px solid black;"> 1 </td>
</tr>
<tr>
<td style="border:1px solid black;"> bundle.<b>skuType</b> </td>
<td style="border:1px solid black;"> integer </td>
<td style="border:1px solid black;"> Defined whether the bundle product SKU is Dynamic or Fixed. <b>Required</b>. </td>
<td style="border:1px solid black;"> 1 </td>
</tr>
<tr>
<td style="border:1px solid black;"> bundle.<b>weightType</b> </td>
<td style="border:1px solid black;"> integer </td>
<td style="border:1px solid black;"> Defined whether the bundle product weight is Dynamic or Fixed. <b>Required</b>. </td>
<td style="border:1px solid black;"> 1 </td>
</tr>
<tr>
<td style="border:1px solid black;"> bundle.<b>priceView</b> </td>
<td style="border:1px solid black;"> mage:integer_label </td>
<td style="border:1px solid black;"> Defined whether the bundle product price should display as "Price Range" or "As Low as." <b>Required</b>. </td>
<td style="border:1px solid black;"> 1 </td>
</tr>
<tr>
<td style="border:1px solid black;"> bundle.<b>shipmentType</b> </td>
<td style="border:1px solid black;"> integer </td>
<td style="border:1px solid black;"> Defines whether the bundle items are to be shipped together or separately. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 1 </td>
</tr>
<tr>
<td style="border:1px solid black;"> <b>downloadable</b> </td>
<td style="border:1px solid black;"> complexType </td>
<td style="border:1px solid black;">  </td>
<td style="border:1px solid black;">  </td>
</tr>
<tr>
<td style="border:1px solid black;"> downloadable.<b>linksPurchasedSeparately</b> </td>
<td style="border:1px solid black;"> boolean </td>
<td style="border:1px solid black;"> Defines whether the downloadable product links could be purchased separately. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 1 </td>
</tr>
<tr>
<td style="border:1px solid black;"> downloadable.<b>samplesTitle</b> </td>
<td style="border:1px solid black;"> string </td>
<td style="border:1px solid black;"> Title of the downloadable product sample. <b>Optional</b>. </td>
<td style="border:1px solid black;"> Sample </td>
</tr>
<tr>
<td style="border:1px solid black;"> downloadable.<b>linksTitle</b> </td>
<td style="border:1px solid black;"> string </td>
<td style="border:1px solid black;"> Title of the downloadable product link. <b>Optional</b>. </td>
<td style="border:1px solid black;"> Links </td>
</tr>
<tr>
<td style="border:1px solid black;"> downloadable.<b>linksExist</b> </td>
<td style="border:1px solid black;"> boolean </td>
<td style="border:1px solid black;"> <b>Optional</b>. </td>
<td style="border:1px solid black;"> 1 </td>
</tr>
<tr>
<td style="border:1px solid black;"> giftMessageAvailable </td>
<td style="border:1px solid black;"> boolean </td>
<td style="border:1px solid black;"> Defines whether the gift message is available for the product. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 1 </td>
</tr>
<tr>
<td style="border:1px solid black;"> taxClassId </td>
<td style="border:1px solid black;"> mage:string_label </td>
<td style="border:1px solid black;"> Product tax class. Can have the following values: 0 - None, 2 - taxable Goods, 4 - Shipping, etc., depending on created tax classes. <b>Required</b>. </td>
<td style="border:1px solid black;"> 7 </td>
</tr>
<tr>
<td style="border:1px solid black;"> enableGooglecheckout </td>
<td style="border:1px solid black;"> boolean </td>
<td style="border:1px solid black;"> Defines whether the product can be purchased with the help of the Google Checkout payment service. Can have the following values: Yes and No. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 1 </td>
</tr>
<tr>
<td style="border:1px solid black;"> <b>giftcard</b> </td>
<td style="border:1px solid black;"> complexType </td>
<td style="border:1px solid black;">  </td>
<td style="border:1px solid black;">  </td>
</tr>
<tr>
<td style="border:1px solid black;"> <b>giftcard.giftcardAmounts</b> </td>
<td style="border:1px solid black;"> complexType </td>
<td style="border:1px solid black;"> Defines the available amounts for the gift card product. <b>Required</b>. </td>
<td style="border:1px solid black;">  </td>
</tr>
<tr>
<td style="border:1px solid black;"> giftcard.giftcardAmounts.<b>amount</b> </td>
<td style="border:1px solid black;"> mage:price </td>
<td style="border:1px solid black;"> Defines the amount for a gift card. Constrains: maxOccurs = unbounded. <b>Required</b>. </td>
<td style="border:1px solid black;"> 20 </td>
</tr>
<tr>
<td style="border:1px solid black;"> giftcard.<b>allowOpenAmount</b> </td>
<td style="border:1px solid black;"> boolean </td>
<td style="border:1px solid black;"> Defines whether the open amount is allowed for a gift card. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 1 </td>
</tr>
<tr>
<td style="border:1px solid black;"> giftcard.<b>openAmountMin</b> </td>
<td style="border:1px solid black;"> mage:price </td>
<td style="border:1px solid black;"> Defines the minimal amount of a gift card if the open amount is allowed. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 5 </td>
</tr>
<tr>
<td style="border:1px solid black;"> giftcard.<b>openAmountMax</b> </td>
<td style="border:1px solid black;"> mage:price </td>
<td style="border:1px solid black;"> Defines the maximal amount of a gift card if the open amount is allowed. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 100 </td>
</tr>
<tr>
<td style="border:1px solid black;"> giftcard.<b>giftcardType</b> </td>
<td style="border:1px solid black;"> mage:integer_label </td>
<td style="border:1px solid black;"> Defines the type of a gift card: Virtual, Physical, or Combined. <b>Required</b>. </td>
<td style="border:1px solid black;"> 2 </td>
</tr>
<tr>
<td style="border:1px solid black;"> giftcard.<b>isRedeemable</b> </td>
<td style="border:1px solid black;"> boolean </td>
<td style="border:1px solid black;"> Defines whether the card is redeemable. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 1 </td>
</tr>
<tr>
<td style="border:1px solid black;"> giftcard.<b>useConfigIsRedeemable</b> </td>
<td style="border:1px solid black;"> boolean </td>
<td style="border:1px solid black;"> Defines whether the settings specified in the store's Configuration section should apply for the Is Redeemable option. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 1 </td>
</tr>
<tr>
<td style="border:1px solid black;"> giftcard.<b>lifetime</b> </td>
<td style="border:1px solid black;"> integer </td>
<td style="border:1px solid black;"> Defines for how many days a gift card is valid. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 3 </td>
</tr>
<tr>
<td style="border:1px solid black;"> giftcard.<b>useConfigLifetime</b> </td>
<td style="border:1px solid black;"> boolean </td>
<td style="border:1px solid black;"> Defines whether the settings specified in the store's Configuration section should apply for the Lifetime option. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 1 </td>
</tr>
<tr>
<td style="border:1px solid black;"> giftcard.<b>emailTemplate</b> </td>
<td style="border:1px solid black;"> string </td>
<td style="border:1px solid black;"> Defines which email template should be used for a gift card. <b>Optional</b>. </td>
<td style="border:1px solid black;">  </td>
</tr>
<tr>
<td style="border:1px solid black;"> giftcard.<b>useConfigEmailTemplate</b> </td>
<td style="border:1px solid black;"> boolean </td>
<td style="border:1px solid black;"> Defines whether the settings specified in the store's Configuration section should apply for the Email Template option. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 1 </td>
</tr>
<tr>
<td style="border:1px solid black;"> giftcard.<b>allowMessage</b> </td>
<td style="border:1px solid black;"> boolean </td>
<td style="border:1px solid black;"> Defines whether the accompanying message is allowed for a gift card. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 1 </td>
</tr>
<tr>
<td style="border:1px solid black;"> giftcard.<b>useConfigAllowMessage</b> </td>
<td style="border:1px solid black;"> boolean </td>
<td style="border:1px solid black;"> Defines whether the settings specified in the store's Configuration section should apply for the Allow Message option. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 1 </td>
</tr>
<tr>
<td style="border:1px solid black;"> giftWrappingAvailable </td>
<td style="border:1px solid black;"> boolean </td>
<td style="border:1px solid black;"> Defines whether the gift wrapping is allowed. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 1 </td>
</tr>
<tr>
<td style="border:1px solid black;"> giftWrappingPrice </td>
<td style="border:1px solid black;"> mage:price </td>
<td style="border:1px solid black;"> Defines the price for the gift wrapping of a product. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 5 </td>
</tr>
<tr>
<td style="border:1px solid black;"> isReturnable </td>
<td style="border:1px solid black;"> mage:integer_label </td>
<td style="border:1px solid black;"> Defines whether a customer will be able to return a product. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 1 </td>
</tr>
<tr>
<td style="border:1px solid black;"> <b>targetRules</b> </td>
<td style="border:1px solid black;"> complexType </td>
<td style="border:1px solid black;">  </td>
<td style="border:1px solid black;">  </td>
</tr>
<tr>
<td style="border:1px solid black;"> targetRules.<b>related</b> </td>
<td style="border:1px solid black;"> mage:target_rule </td>
<td style="border:1px solid black;"> Contains the IDs of the related products. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 1 </td>
</tr>
<tr>
<td style="border:1px solid black;"> targetRules.<b>upsell</b> </td>
<td style="border:1px solid black;"> mage:target_rule </td>
<td style="border:1px solid black;"> Contains the IDs of the upsell products. <b>Optional</b>. </td>
<td style="border:1px solid black;"> 1 </td>
</tr>
</table>


