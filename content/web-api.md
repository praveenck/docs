Creating Web APIs
=================

[[*steal and summarize from service IDL and service implementation summaries*]]

    You can expose your service over the web to REST and SOAP interfaces by
    making simple web API declarations in the form of annotations to a
    configuration file.

Highlights
----------

[[*maybe the first point below is rather generic, and the second is already
explicit in the introductory sentences above--for the second thing, possibly
break up the info by reducing the intro sentences to a teaser with some
generalizations about what a service is. For the first thing, maybe sharpen the
statement with details--e.g., what type of auth n auth?  ]]*

-   Authentication and Authorization functionality is handled by the API
    framework, allowing for developers to focus on building business logic

-   Your service's business logic can be available to both the Magento web
    application and other native applications (iOS, Android, etc. apps)

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

[[*do we also use the declaration in webapi.xml for REST? does the declaration
or where it's made differ for REST?]*

Since REST doesn't pass all inputs in its payload, its payload does not reflect
the description in the XSD. Some inputs are passed as part of the path and query
parameters. These inputs are then missing from the payload. Deserializing the
payload would not result in the expected input associative array.

To work around this problem, after deserialization, we insert those inputs into
the input array. To map query-params and path-params, we use the parameter name
(e.g., shipment-id, filter) for a matching key in the associative array. We use
dotted notation to indicate levels in the array.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

level1.level2.sort
('level1' => array (
    'level2' => array(
        'sort' => ....
    )
))

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

How to Add a New Service
------------------------

1.  Put the business logic PHP code into a class that extends MageService.
    [[*example*?]]

2.  Annotate the new class with @Service and @Version (explained at [Service
    IDL][1].

[1]: <http://praveenck.github.io/docs/service-idl/>

1.  For each method, make sure it takes a single param as an associative array
    and returns another array

2.  Create a new XSD file under the module /etc directory and describe the
    structure of the arrays.

3.  To expose the service through the Web API, add or edit the webapi.xml under
    the module /etc directory and provide REST routing info.

[[*these steps are very high level and would be way better with examples.
without examples, the sequence is not clear enough to me that I can improve
it]]*

