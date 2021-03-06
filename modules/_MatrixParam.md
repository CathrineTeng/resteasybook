@MatrixParam {#_MatrixParam}
============

The idea of matrix parameters is that they are an arbitrary set of
name-value pairs embedded in a uri path segment. A matrix parameter
example is:

GET http://host.com/library/book;name=EJB 3.0;author=Bill Burke

The basic idea of matrix parameters is that it represents resources that
are addressable by their attributes as well as their raw id. The
@MatrixParam annotation allows you to inject URI matrix parameters into
your method invocation

    @GET
    public String getBook(@MatrixParam("name") String name, @MatrixParam("author") String author) {...}

There is one big problem with @MatrixParam that the current version of
the specification does not resolve. What if the same MatrixParam exists
twice in different path segments? In this case, right now, its probably
better to use PathParam combined with PathSegment.
