@QueryParam {#_QueryParam}
===========

The @QueryParam annotation allows you to map a URI query string
parameter or url form encoded parameter to your method invocation.

GET /books?num=5

    @GET
    public String getBooks(@QueryParam("num") int num) {
    ...
    }

Currently since Resteasy is built on top of a Servlet, it does not
distinguish between URI query strings or url form encoded parameters.
Like PathParam, your parameter type can be an String, primitive, or
class that has a String constructor or static valueOf() method.
