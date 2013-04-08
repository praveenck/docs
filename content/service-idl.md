Service IDL
===========

The proposed new architecture consolidates all of an extension's business into a
service layer. Presented here is an Interface Definition Language specification
that describes the services exposed through the service layer. An IDL formally
describes the functionality a service provides (methods, inputs and outputs of
methods) without exposing the details of how that functionality is implemented.

Some advantages of using an IDL:

-   Allows for well-defined and versioned interfaces that the services and
    visual components depend on. This makes it simpler to control compatibility
    issues during upgrades.

-   An IDL can support more than one version of the domain/business logic
    running in the Magento instance at the same time, increasing backward
    compatibility.

Overview
--------

This document specifies the Interface Definition Language (IDL) that describes
the operations permitted by the Service layer. To support REST and SOAP Web
APIs, WADL and WSDL will be auto-generated from the IDL.

The data structure used in the payloads is defined in an XSD. The metadata about
a service call, such as version, required permissions, and locations of XSDs,
are carried by PHP annotations. Data structure and metadata are described in
what follows.

PHP Annotations
---------------

### 

<div>
<table>
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
<div>

###

### Service Calls

A service operation is implemented as a class method. The class maps to the
service via the requirement that operations from a service be grouped into a
single class.

**Example of a service call**

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

/**
@Service shipping
 * @Version 2
*/

class ShippingService extends MageService 
{

    /**
     * @Permissions [shipping, inventory]
     * @Consumes (schema="shipping.xsd", element="getCarriersRequest")
     * @Produces (schema="shipping.xsd", element="getCarriersResponse")
     */

    public function getCarriers ($params)
    {

    ...

    }
}

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Here, the input data structure is identified in the @Consumes annotation as
belonging to the <getCarriersRequest> element in shipping/consumerux.xsd, and the
output data structure is identified in the @Produces annotation as belonging to
the <getCarriersResponse> element in shipping/consumerux.xsd. The shipping/consumerux.xsd file is
shown below.

The PHP function receives an associative array ($params function parameter)
reflecting the structure described by the element <getCarriersRequest>. It returns
another associative array reflecting the structure described by the element
<getCarriersResponse>.

XSD
---

The XSD is used to describe the data structure passed in during the API call as
well as the returned data. During runtime data is stored into a associative
array.

**shipping/consumerux.xsd**

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

<?xml version="1.0"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema">

    <xs:complexType name="address_type">
        <xs:sequence>
            <xs:element name="street" type="xs:string"/>
            <xs:element name="city" type="xs:string"/>
            <xs:element name="region_id" type="xs:integer"/>
            <xs:element name="country_id" type="xs:string"/>
            <xs:element name="postcode" type="xs:integer"/>
            <xs:element name="meta_data" minOccurs="0">
                <xs:complexType>
                    <xs:sequence>
                        <xs:element name="lift_gate_required" type="xs:boolean" minOccurs="0"/>
                    </xs:sequence>
                </xs:complexType>
            </xs:element>
        </xs:sequence>
    </xs:complexType>

    <xs:complexType name="item_type">
        <xs:sequence>
            <xs:element name="sku" type="xs:integer"/>
            <xs:element name="name" type="xs:string"/>
            <xs:element name="value" type="xs:string"/>
            <xs:element name="weight" type="xs:string"/>
            <xs:element name="height" type="xs:string"/>
            <xs:element name="width" type="xs:string"/>
            <xs:element name="depth" type="xs:string"/>
            <xs:element name="qty" type="xs:integer"/>
            <xs:element name="free_shipping" type="xs:boolean"/>
            <xs:element name="meta_data">
                <xs:complexType>
                    <xs:sequence>
                        <xs:element name="hazard" type="xs:boolean" minOccurs="0"/>
                    </xs:sequence>
                </xs:complexType>
            </xs:element>
            <xs:element name="parent" minOccurs="0">
                <xs:complexType>
                    <xs:sequence>
                        <xs:element name="item" type="item_type"/>
                    </xs:sequence>
                </xs:complexType>
            </xs:element>
            <xs:element name="children" minOccurs="0">
                <xs:complexType>
                    <xs:sequence>
                        <xs:element name="item" type="item_type" />
                    </xs:sequence>
                </xs:complexType>
            </xs:element>
        </xs:sequence>
    </xs:complexType>

    <xs:element name="getCarriersRequest">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="rate_request">
                    <xs:complexType>
                        <xs:sequence>
                            <xs:element name="orig" type="address_type"/>
                            <xs:element name="dest" type="address_type"/>
                            <xs:element name="list_of_items">
                                <xs:complexType>
                                    <xs:sequence maxOccurs="unbounded">
                                        <xs:element name="item" type="item_type" />
                                    </xs:sequence>
                                </xs:complexType>
                            </xs:element>
                            <xs:element name="carrier_configuration" maxOccurs="1">
                                <xs:complexType>
                                    <xs:sequence>
                                        <xs:element name="insurance" type="xs:boolean"/>
                                        <xs:element name="handling" type="xs:string"/>
                                        <xs:element name="pickup" type="xs:string"/>
                                    </xs:sequence>
                                </xs:complexType>
                            </xs:element>
                        </xs:sequence>
                    </xs:complexType>
                </xs:element>
            </xs:sequence>
        </xs:complexType>
    </xs:element>

    <xs:complexType name="method_type">
        <xs:sequence>
            <xs:element name="carrier" type="xs:string"/>
            <xs:element name="carrier_title" type="xs:string"/>
            <xs:element name="method" type="xs:string"/>
            <xs:element name="method_title" type="xs:string"/>
            <xs:element name="price" type="xs:string"/>
        </xs:sequence>
    </xs:complexType>

    <xs:element name="getCarriersResponse">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="shipping_methods">
                    <xs:complexType>
                        <xs:sequence maxOccurs="unbounded">
                            <xs:element name="method" type="method_type" />
                        </xs:sequence>
                    </xs:complexType>
                </xs:element>
            </xs:sequence>
        </xs:complexType>
    </xs:element>
</xs:schema>

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

