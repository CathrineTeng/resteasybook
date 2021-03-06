AJAX Client {#AJAX_Client}
===========

RESTEasy resources can be accessed in JavaScript using AJAX using a
proxy API generated by RESTEasy.

Generated JavaScript API {#javascript_api}
========================

RESTEasy can generate a JavaScript API that uses AJAX calls to invoke
JAX-RS operations.

Let's take a simple JAX-RS API:

    @Path("orders")
    public interface Orders {
     @Path("{id}")
     @GET
     public String getOrder(@PathParam("id") String id){
      return "Hello "+id;
     }
    }

The preceding API would be accessible using the following JavaScript
code:

``` {.javascript}
var order = Orders.getOrder({id: 23});
```

JavaScript API servlet {#javascript_api_servlet}
----------------------

In order to enable the JavaScript API servlet you must configure it in
your web.xml file as such:

``` {.xml}
<servlet>
 <servlet-name>RESTEasy JSAPI</servlet-name>
 <servlet-class>org.jboss.resteasy.jsapi.JSAPIServlet</servlet-class>
</servlet>

<servlet-mapping>
 <servlet-name>RESTEasy JSAPI</servlet-name>
 <url-pattern>/rest-js</url-pattern>
</servlet-mapping>
```

JavaScript API usage {#javascript_api_usage}
--------------------

Each JAX-RS resource class will generate a JavaScript object of the same
name as the declaring class (or interface), which will contain every
JAX-RS method as properties.

For example, if the JAX-RS resource X defines methods Y and Z:

``` {.java}
@Path("/")
public interface X{
 @GET
 public String Y();
 @PUT
 public void Z(String entity);
}
```

Then the JavaScript API will define the following functions:

``` {.javascript}
var X = {
 Y : function(params){…},
 Z : function(params){…}
};
```

Each JavaScript API method takes an optional object as single parameter
where each property is a cookie, header, path, query or form parameter
as identified by their name, or the following special parameters:

> **Warning**
>
> The following special parameter names are subject to change.

  Property name   Default                                       Description
  --------------- --------------------------------------------- -------------------------------------------------------------------------------------------------------------------------------------------------
  \$entity                                                      The entity to send as a PUT, POST request.
  \$contentType   As determined by @Consumes.                   The MIME type of the body entity sent as the Content-Type header.
  \$accepts       Determined by @Provides, defaults to \*/\*.   The accepted MIME types sent as the Accept header.
  \$callback                                                    Set to a function(httpCode, xmlHttpRequest, value) for an asynchronous call. If not present, the call will be synchronous and return the value.
  \$apiURL        Determined by container                       Set to the base URI of your JAX-RS endpoint, not including the last slash.
  \$username                                                    If username and password are set, they will be used for credentials for the request.
  \$password                                                    If username and password are set, they will be used for credentials for the request.

  : API parameter properties

Here is an example of JAX-RS API:

``` {.java}
@Path("foo")
public class Foo{
 @Path("{id}")
 @GET
 public String get(@QueryParam("order") String order, @HeaderParam("X-Foo") String header,
                   @MatrixParam("colour") String colour, @CookieParam("Foo-Cookie") String cookie){
  …
 }
 @POST
 public void post(String text){
 }
}
```

We can use the previous JAX-RS API in JavaScript using the following
code:

``` {.javascript}
var text = Foo.get({order: 'desc', 'X-Foo': 'hello',
                    colour: 'blue', 'Foo-Cookie': 123987235444});
Foo.put({$entity: text});
```

Work with @Form {#Form}
---------------

@Form is a RESTEasy specific annotation that allows you to re-use any
@\*Param annotation within an injected class. The generated JavaScript
API will expand the parameters for use automatically. Support we have
the following form:

``` {.java}
public class MyForm {
    @FormParam("stuff")
    private String stuff;

    @FormParam("number")
    private int number;

    @HeaderParam("myHeader")
    private String header;
}
```

And the resource is like:

``` {.java}
@Path("/")
public class MyResource {

    @POST
    public String postForm(@Form MyForm myForm) {...}

}
```

Then we could call the method from JavaScript API like following:

``` {.java}
MyResource.postForm({stuff:"A", myHeader:"B", number:1});
```

Also, @Form supports prefix mappings for lists and maps:

``` {.java}
public static class Person {
    @Form(prefix="telephoneNumbers") List<TelephoneNumber> telephoneNumbers;
    @Form(prefix="address") Map<String, Address> addresses;
}

public static class TelephoneNumber {
    @FormParam("countryCode") private String countryCode;
    @FormParam("number") private String number;
}

public static class Address {
    @FormParam("street") private String street;
    @FormParam("houseNumber") private String houseNumber;
}

@Path("person")
public static class MyResource {
    @POST
    public void postForm(@Form Person p) {...} 
}
```

From JavaScript we could call the API like this:

    MyResource.postForm({
        telephoneNumbers:[
            {"telephoneNumbers[0].countryCode":31},
            {"telephoneNumbers[0].number":12345678},
            {"telephoneNumbers[1].countryCode":91},
            {"telephoneNumbers[1].number":9717738723}
        ],
        address:[
            {"address[INVOICE].street":"Main Street"},
            {"address[INVOICE].houseNumber":2},
            {"address[SHIPPING].street":"Square One"},
            {"address[SHIPPING].houseNumber":13}
        ]
    });

MIME types and unmarshalling. {#mime_unmarshalling}
-----------------------------

The Accept header sent by any client JavaScript function is controlled
by the \$accepts parameter, which overrides the @Produces annotation on
the JAX-RS endpoint. The returned value however is controlled by the
Content-Type header sent in the response as follows:

  MIME                                          Description
  --------------------------------------------- ------------------------------------------------------------------------------------------------------------
  text/xml,application/xml,application/\*+xml   The response entity is parsed as XML before being returned. The return value is thus a DOM Document.
  application/json                              The response entity is parsed as JSON before being returned. The return value is thus a JavaScript Object.
  Anything else                                 The response entity is returned raw.

  : Return values by MIME type

The RESTEasy JavaScript client API can automatically unmarshall JSON and
XML:

    @Path("orders")
    public interface Orders {

     @XmlRootElement
     public static class Order {
      @XmlElement
      private String id;
      
      public Order(){}
      
      public Order(String id){
       this.id = id;
      }
     }

     @Path("{id}/xml")
     @GET
     @Produces("application/xml")
     public Order getOrderXML(@PathParam("id") String id){
      return new Order(id);
     }

     @Path("{id}/json")
     @GET
     @Produces("application/json")
     public Order getOrderJSON(@PathParam("id") String id){
      return new Order(id);
     }
    }

Let us look at what the preceding JAX-RS API would give us on the client
side:

``` {.javascript}
// this returns a JSON object
var orderJSON = Orders.getOrderJSON({id: "23"});
orderJSON.id == "23";

// this one returns a DOM Document whose root element is the order, with one child (id)
// whose child is the text node value
var orderXML = Orders.getOrderXML({id: "23"});
orderXML.documentElement.childNodes[0].childNodes[0].nodeValue == "23";
```

MIME types and marshalling. {#mime_marshalling}
---------------------------

The Content-Type header sent in the request is controlled by the
\$contentType parameter which overrides the @Consumes annotation on the
JAX-RS endpoint. The value passed as entity body using the \$entity
parameter is marshalled according to both its type and content type:

  Type                       MIME                                                   Description
  -------------------------- ------------------------------------------------------ -------------------------------------------------------------------
  DOM Element                Empty or text/xml,application/xml,application/\*+xml   The DOM Element is marshalled to XML before being sent.
  JavaScript Object (JSON)   Empty or application/json                              The JSON object is marshalled to a JSON string before being sent.
  Anything else              Anything else                                          The entity is sent as is.

  : Controlling sent entities

The RESTEasy JavaScript client API can automatically marshall JSON and
XML:

    @Path("orders")
    public interface Orders {

     @XmlRootElement
     public static class Order {
      @XmlElement
      private String id;
      
      public Order(){}
      
      public Order(String id){
       this.id = id;
      }
     }

     @Path("{id}/xml")
     @PUT
     @Consumes("application/xml")
     public void putOrderXML(Order order){
      // store order
     }

     @Path("{id}/json")
     @PUT
     @Consumes("application/json")
     public void putOrderJSON(Order order){
      // store order
     }
    }

Let us look at what the preceding JAX-RS API would give us on the client
side:

``` {.javascript}
// this saves a JSON object
Orders.putOrderJSON({$entity: {id: "23"}});

// It is a bit more work with XML
var order = document.createElement("order");
var id = document.createElement("id");
order.appendChild(id);
id.appendChild(document.createTextNode("23"));
Orders.putOrderXML({$entity: order});
```

Using the JavaScript API to build AJAX queries {#javascript_ajax}
==============================================

The RESTEasy JavaScript API can also be used to manually construct your
requests.

The REST object {#REST_object}
---------------

The REST object contains the following read-write properties:

  Property   Description
  ---------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  apiURL     Set by default to the JAX-RS root URL, used by every JavaScript client API functions when constructing the requests.
  log        Set to a function(string) in order to receive RESTEasy client API logs. This is useful if you want to debug your client API and place the logs where you can see them.

  : The REST object

The REST object can be used to override RESTEasy JavaScript API client
behaviour:

``` {.javascript}
// Change the base URL used by the API:
REST.apiURL = "http://api.service.com";

// log everything in a div element
REST.log = function(text){
 jQuery("#log-div").append(text);
};
```

The REST.Request class {#REST_request}
----------------------

The REST.Request class is used to build custom requests. It has the
following members:

  Member                               Description
  ------------------------------------ ----------------------------------------------------------------------------------------------------------------------------------------------------
  execute(callback)                    Executes the request with all the information set in the current object. The value is never returned but passed to the optional argument callback.
  setAccepts(acceptHeader)             Sets the Accept request header. Defaults to \*/\*.
  setCredentials(username, password)   Sets the request credentials.
  setEntity(entity)                    Sets the request entity.
  setContentType(contentTypeHeader)    Sets the Content-Type request header.
  setURI(uri)                          Sets the request URI. This should be an absolute URI.
  setMethod(method)                    Sets the request method. Defaults to GET.
  setAsync(async)                      Controls whether the request should be asynchronous. Defaults to true.
  addCookie(name, value)               Sets the given cookie in the current document when executing the request. Beware that this will be persistent in your browser.
  addQueryParameter(name, value)       Adds a query parameter to the URI query part.
  addMatrixParameter(name, value)      Adds a matrix parameter (path parameter) to the last path segment of the request URI.
  addHeader(name, value)               Adds a request header.

  : The REST.Request class

The REST.Request class can be used to build custom requests:

``` {.javascript}
var r = new REST.Request();
r.setURI("http://api.service.com/orders/23/json");
r.setMethod("PUT");
r.setContentType("application/json");
r.setEntity({id: "23"});
r.addMatrixParameter("JSESSIONID", "12309812378123");
r.execute(function(status, request, entity){
 log("Response is "+status);
});
```

Caching Features {#caching_features}
================

RESTEasy AJAX Client works well with server side caching features. But
the buggy browsers cache will always prevent the function to work
properly. If you'd like to use RESTEasy's caching feature with its AJAX
client, you can enable 'antiBrowserCache' option:

``` {.javascript}
REST.antiBrowserCache = true;
```

The above setting should be set once before you call any APIs.
