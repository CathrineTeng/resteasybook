Filters and Interceptors {#Interceptors}
========================

JAX-RS 2.0 has two different concepts for interceptions: Filters and
Interceptors. Filters are mainly used to modify or process incoming and
outgoing request headers or response headers. They execute before and
after request and response processing.

Server Side Filters {#server-side}
===================

On the server-side you have two different types of filters.
ContainerRequestFilters run before your JAX-RS resource method is
invoked. ContainerResponseFilters run after your JAX-RS resource method
is invoked. As an added caveat, ContainerRequestFilters come in two
flavors: pre-match and post-matching. Pre-matching
ContainerRequestFilters are designated with the @PreMatching annotation
and will execute before the JAX-RS resource method is matched with the
incoming HTTP request. Pre-matching filters often are used to modify
request attributes to change how it matches to a specific resource
method (i.e. strip .xml and add an Accept header).
ContainerRequestFilters can abort the request by calling
ContainerRequestContext.abortWith(Response). A filter might want to
abort if it implements a custom authentication protocol.

After the resource class method is executed, JAX-RS will run all
ContainerResponseFilters. These filters allow you to modify the outgoing
response before it is marshalling and sent to the client. So given all
that, here's some pseudo code to give some understanding of how things
work.

            // execute pre match filters
            for (ContainerRequestFilter filter : preMatchFilters) {
                filter.filter(requestContext);
                if (isAborted(requestContext)) {
                   sendAbortionToClient(requestContext);
                   return;
                }
            }
            // match the HTTP request to a resource class and method
            JaxrsMethod method = matchMethod(requestContext);

            // Execute post match filters
            for (ContainerRequestFilter filter : postMatchFilters) {
               filter.filter(requestContext);
               if (isAborted(requestContext)) {
                  sendAbortionToClient(requestContext);
                  return;
               }
            }

            // execute resource class method
            method.execute(request);

            // execute response filters
            for (ContainerResponseFilter filter : responseFilters) {
               filter.filter(requestContext, responseContext);
            }
        

Client Side Filters {#client-side}
===================

On the client side you also have two types of filters:
ClientRequestFilter and ClientResponseFilter. ClientRequestFilters run
before your HTTP request is sent over the wire to the server.
ClientResponseFilters run after a response is received from the server,
but before the response body is unmarshalled. ClientRequestFilters are
also allowed to abort the execute of the request and provide a canned
response without going over the wire to the server.
ClientResponseFilters can modfiy the Response object before it is handed
back to application code. Here's some pseudo code to illustrate things.

                // execute request filters
                for (ClientRequestFilter filter : requestFilters) {
                   filter.filter(requestContext);
                   if (isAborted(requestContext)) {
                      return requestContext.getAbortedResponseObject();
                   }
                }

                // send request over the wire
                response = sendRequest(request);

                // execute response filters
                for (ClientResponseFilter filter : responseFilters) {
                   filter.filter(requestContext, responseContext);
                }
            

Reader and Writer Interceptors {#reader-writer}
==============================

While filters modify request or response headers, interceptors deal with
message bodies. Interceptors are executed in the same call stack as
their corresponding reader or writer. ReaderInterceptors wrap around the
execution of MessageBodyReaders. WriterInterceptors wrap around the
execution of MessageBodyWriters. They can be used to implement a
specific content-encoding. They can be used to generate digital
signatures or to post or pre-process a Java object model before or after
it is marshalled.

Per Resource Method Filters and Interceptors {#per-resource}
============================================

Sometimes you want a filter or interceptor to only run for a specific
resource method. You can do this in two different ways: register an
implementation of DynamicFeature or use the @NameBinding annotation. The
DynamicFeature interface is executed at deployment time for each
resource method. You just use the Configurable interface to register the
filters and interceptors you want for the specific resource method.
@NameBinding works a lot like CDI interceptors. You annotate a custom
annotation with @NameBinding and then apply that custom annotation to
your filter and resource method

                @NameBinding
                public @interface DoIt {}

                @DoIt
                public class MyFilter implements ContainerRequestFilter {...}

                @Path("/root")
                public class MyResource {

                   @GET
                   @DoIt
                   public String get() {...}
                }
            

Ordering {#ordering}
========

Ordering is accomplished by using the @BindingPriority annotation on
your filter or interceptor class.
