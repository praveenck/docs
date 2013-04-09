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

The sample input associative array that is passed to the service method and the
sample output associative array returned by the service method are shown below.

**Sample input**

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~



~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Sample output**

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~



~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

