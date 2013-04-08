Creating Web APIs
=================

Once you've created a service, you can expose its functionality over the web to
REST and SOAP interfaces by making simple web API declarations in the form of
annotations to a configuration file.

Note that in the current Alpha-1 release, this is a proposal only, and not yet
implemented. Please leave comments on this proposal on https://getsatisfaction.com/magento-2-prerelease/topics

To find out about creating a service, see [Service Implementation][1].

[1]: <http://praveenck.github.io/docs/service-impl/>

Some advantages of converting your service into a web API:

-   Authentication and authorization will be handled by the API framework,
    allowing you to focus on building business logic

-   Your service's business logic becomes available to the Magento web
    application and also to other native applications (e.g., iOS and Android
    apps).

Web API Declaration
-------------------

We use a configuration file to expose services that use the IDL to REST and SOAP
interfaces.

For SOAP, we only need to declare the location of WSDL and the method. We can
default the WSDL location to the service name (e.g., shipping) and the SOAP
method to the PHP method (e.g., getCarriers).

**webapi.xml**

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

<?xml version="1.0"?>
<webapi>
    <service name="shipping">
        <rest-route method="GET" path="/shipping/carriers" call="getCarriers">
        <rest-route method="GET" path="/shipping/{shipment-id}" call="getOne">
        <rest-route method="GET" path="/shipping" call="getMany" query-params="sort,filter">
    </service>
</webapi>

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To expose your service as a REST API, there is an additional step. Because REST
doesn't pass all inputs in its payload, its payload does not reflect the
description in the XSD. Some inputs are passed as part of the path and query
parameters, and are thus missing from the payload. Deserializing the payload
would not result in the expected input associative array.

To form the expected array, after deserialization, we insert the missing inputs
into the array. We use the parameter name (e.g., shipment-id, filter) as a
matching key to map to query-params and path-params in the array, and we use
dotted notation to indicate levels in the array, as follows:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

level1.level2.sort

('level1' => array (

    'level2' => array(

        'sort' => ....

    )

))

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
