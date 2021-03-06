Server-side Mock Framework {#RESTEasy_Server-side_Mock_Framework}
==========================

Although RESTEasy has an Embeddable Container, you may not be
comfortable with the idea of starting and stopping a web server within
unit tests (in reality, the embedded container starts in milli seconds),
or you might not like the idea of using Apache HTTP Client or
java.net.URL to test your code. RESTEasy provides a mock framework so
that you can invoke on your resource directly.

    import org.jboss.resteasy.mock.*;
    ...

          Dispatcher dispatcher = MockDispatcherFactory.createDispatcher();

          POJOResourceFactory noDefaults = new POJOResourceFactory(LocatingResource.class);
          dispatcher.getRegistry().addResourceFactory(noDefaults);

          {
             MockHttpRequest request = MockHttpRequest.get("/locating/basic");
             MockHttpResponse response = new MockHttpResponse();

             dispatcher.invoke(request, response);


             Assert.assertEquals(HttpServletResponse.SC_OK, response.getStatus());
             Assert.assertEquals("basic", response.getContentAsString());
          }

See the RESTEasy Javadoc for all the ease-of-use methods associated with
MockHttpRequest, and MockHttpResponse.
