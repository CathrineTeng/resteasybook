@PathParam {#_PathParam}
==========

@PathParam is a parameter annotation which allows you to map variable
URI path fragments into your method call.

    @Path("/library")
    public class Library {

       @GET
       @Path("/book/{isbn}")
       public String getBook(@PathParam("isbn") String id) {
          // search my database and get a string representation and return it
       }
    }

What this allows you to do is embed variable identification within the
URIs of your resources. In the above example, an isbn URI parameter is
used to pass information about the book we want to access. The parameter
type you inject into can be any primitive type, a String, or any Java
object that has a constructor that takes a String parameter, or a static
valueOf method that takes a String as a parameter. For example, lets say
we wanted isbn to be a real object. We could do:

    @GET
    @Path("/book/{isbn}")
    public String getBook(@PathParam("isbn") ISBN id) {...}

    public class ISBN {
       public ISBN(String str) {...}
    }

Or instead of a public String constructors, have a valueOf method:

    public class ISBN {
        public static ISBN valueOf(String isbn) {...}
    }

Advanced @PathParam and Regular Expressions {#Advanced__PathParam_and_Regular_Expressions}
===========================================

There are a few more complicated uses of @PathParams not discussed in
the previous section.

You are allowed to specify one or more path params embedded in one URI
segment. Here are some examples:

    1. @Path("/aaa{param}bbb")
    2. @Path("/{name}-{zip}")
    3. @Path("/foo{name}-{zip}bar")

So, a URI of "/aaa111bbb" would match \#1. "/bill-02115" would match
\#2. "foobill-02115bar" would match \#3.

We discussed before how you can use regular expression patterns within
@Path values.

    @GET
    @Path("/aaa{param:b+}/{many:.*}/stuff")
    public String getIt(@PathParam("param") String bs, @PathParam("many") String many) {...}

For the following requests, lets see what the values of the "param" and
"many" @PathParams would be:

  Request                    param   many
  -------------------------- ------- ----------
  GET /aaabb/some/stuff      bb      some
  GET /aaab/a/lot/of/stuff   b       a/lot/of

@PathParam and PathSegment {#_PathParam_and_PathSegment}
==========================

The specification has a very simple abstraction for examining a fragment
of the URI path being invoked on javax.ws.rs.core.PathSegment:

    public interface PathSegment {

        /**
         * Get the path segment.
         * <p>
         * @return the path segment
         */
        String getPath();
        
        /**
         * Get a map of the matrix parameters associated with the path segment
         * @return the map of matrix parameters
         */
        MultivaluedMap<String, String> getMatrixParameters();

    }

You can have Resteasy inject a PathSegment instead of a value with your
@PathParam.

    @GET
    @Path("/book/{id}")
    public String getBook(@PathParam("id") PathSegment id) {...}

This is very useful if you have a bunch of @PathParams that use matrix
parameters. The idea of matrix parameters is that they are an arbitrary
set of name-value pairs embedded in a uri path segment. The PathSegment
object gives you access to these parameters. See also MatrixParam.

A matrix parameter example is:

GET http://host.com/library/book;name=EJB 3.0;author=Bill Burke

The basic idea of matrix parameters is that it represents resources that
are addressable by their attributes as well as their raw id.
