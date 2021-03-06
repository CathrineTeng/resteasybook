Maven and RESTEasy {#Maven_and_RESTEasy}
==================

JBoss's Maven Repository is at:
http://repository.jboss.org/nexus/content/groups/public/

Here's the pom.xml fragment to use. Resteasy is modularized into various
components. Mix and max as you see fit. Please replace 3.1.0-SNAPSHOT
with the current Resteasy version you want to use.


    <repositories>
       <repository>
          <id>jboss</id>
          <url>http://repository.jboss.org/nexus/content/groups/public/</url>
       </repository>
    </repositories>
    <dependencies>
       <!-- core library -->
       <dependency>
          <groupId>org.jboss.resteasy</groupId>
          <artifactId>resteasy-jaxrs</artifactId>
          <version>3.1.0-SNAPSHOT</version>
       </dependency>
       <dependency>
          <groupId>org.jboss.resteasy</groupId>
          <artifactId>resteasy-client</artifactId>
          <version>3.1.0-SNAPSHOT</version>
       </dependency>

       <!-- optional modules -->

       <!-- JAXB support -->
       <dependency>
          <groupId>org.jboss.resteasy</groupId>
          <artifactId>resteasy-jaxb-provider</artifactId>
          <version>3.1.0-SNAPSHOT</version>
       </dependency>
       <!-- multipart/form-data and multipart/mixed support -->
       <dependency>
          <groupId>org.jboss.resteasy</groupId>
          <artifactId>resteasy-multipart-provider</artifactId>
          <version>3.1.0-SNAPSHOT</version>
       </dependency>
       <!-- Resteasy Server Cache -->
       <dependency>
          <groupId>org.jboss.resteasy</groupId>
          <artifactId>resteasy-cache-core</artifactId>
          <version>3.1.0-SNAPSHOT</version>
       </dependency>
       <!-- Ruby YAML support -->
       <dependency>
          <groupId>org.jboss.resteasy</groupId>
          <artifactId>resteasy-yaml-provider</artifactId>
          <version>3.1.0-SNAPSHOT</version>
       </dependency>
       <!-- JAXB + Atom support -->
       <dependency>
          <groupId>org.jboss.resteasy</groupId>
          <artifactId>resteasy-atom-provider</artifactId>
          <version>3.1.0-SNAPSHOT</version>
       </dependency>
       <!-- Spring integration -->
       <dependency>
          <groupId>org.jboss.resteasy</groupId>
          <artifactId>resteasy-spring</artifactId>
          <version>3.1.0-SNAPSHOT</version>
       </dependency>
       <!-- Guice integration -->
       <dependency>
          <groupId>org.jboss.resteasy</groupId>
          <artifactId>resteasy-guice</artifactId>
          <version>3.1.0-SNAPSHOT</version>
       </dependency>

       <!-- Asynchronous HTTP support with Servlet 3.0  -->
       <dependency>
          <groupId>org.jboss.resteasy</groupId>
          <artifactId>async-http-servlet-3.0</artifactId>
          <version>3.1.0-SNAPSHOT</version>
       </dependency>

    </dependencies>

There is also a pom that can be imported so the versions of the
individual modules do not have to be specified. Note that maven 2.0.9 is
required for this.


        <dependencyManagement>
            <dependencies>
                <dependency>
                    <groupId>org.jboss.resteasy</groupId>
                    <artifactId>resteasy-bom</artifactId>
                    <version>3.1.0-SNAPSHOT</version>
                    <type>pom</type>
                    <scope>import</scope>
                </dependency>
            </dependencies>
        </dependencyManagement>

        
