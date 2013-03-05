##Jersey Quickstart##

This quickstart shows how you can use Jersey to write RESTful web services on OpenShift. Jersey does not work out of the box on OpenShift due to following reasons :

* OpenShift JBoss AS7 comes with RESTEasy implementation so you have to tell OpenShift JBoss AS7 server to not use RESTEasy scanning.This is done by adding following context param in web.xml

```
<?xml version="1.0" encoding="UTF-8"?>

<web-app version="3.0" xmlns="http://java.sun.com/xml/ns/javaee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
	metadata-complete="false">


	<context-param>
		<param-name>resteasy.deployer</param-name>
		<param-value>false</param-value>
	</context-param>
	
	<context-param>
		<param-name>resteasy.scan</param-name>
		<param-value>false</param-value>
	</context-param>
	<context-param>
		<param-name>resteasy.scan.providers</param-name>
		<param-value>false</param-value>
	</context-param>
	<context-param>
		<param-name>resteasy.scan.resources</param-name>
		<param-value>false</param-value>
	</context-param>
</web-app>
```
* You need to add following Jersey dependencies in pom.xml

```
<dependency>
	<groupId>com.sun.jersey</groupId>
	<artifactId>jersey-server</artifactId>
	<version>1.17</version>
</dependency>
<dependency>
	<groupId>com.sun.jersey.contribs</groupId>
	<artifactId>jersey-multipart</artifactId>
	<version>1.17</version>
</dependency>
<dependency>
	<groupId>com.sun.jersey</groupId>
	<artifactId>jersey-servlet</artifactId>
	<version>1.17</version>
</dependency>
```

* In .openshift/config/standalone.xml you have to disable JAXRS as shown below. Comment the following lines

```
<extension module="org.jboss.as.jaxrs"/>
<subsystem xmlns="urn:jboss:domain:jaxrs:1.0"/>
```

* Now after making these configuration changes, you can start working on writing RESTful web services using Jersey. First write a JaxrsActivator which will define the base url for rest web services as shown below.

```
package com.openshift.examples;

import java.util.HashSet;
import java.util.Set;

import javax.ws.rs.ApplicationPath;
import javax.ws.rs.core.Application;

@ApplicationPath("/rest")
public class JaxrsActivator extends Application {
	
     public Set<Class<?>> getClasses() {
         Set<Class<?>> s = new HashSet<Class<?>>();
         s.add(FileUploadRestService.class);
         return s;
     }
}
```

* Now write your file upload servlet as shown below.

```
package com.openshift.examples;

import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;

import javax.ws.rs.Consumes;
import javax.ws.rs.POST;
import javax.ws.rs.Path;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.Response;

import com.sun.jersey.core.header.FormDataContentDisposition;
import com.sun.jersey.multipart.FormDataParam;

@Path("/file")
public class FileUploadRestService {
	
	
	@POST
	@Path("/upload")
	@Consumes(MediaType.MULTIPART_FORM_DATA)
	public Response uploadFile(
			@FormDataParam("file") InputStream uploadedInputStream,
			@FormDataParam("file") FormDataContentDisposition fileDetail) {

		String uploadedFileLocation = System.getenv("OPENSHIFT_DATA_DIR");
		// save it
		writeToFile(uploadedInputStream, uploadedFileLocation,fileDetail);

		String output = "File uploaded to : " + uploadedFileLocation;

		return Response.status(200).entity(output).build();

	}

	private void writeToFile(InputStream uploadedInputStream,
			String uploadedFileLocation, FormDataContentDisposition fileDetail) {

		try {
			String filePath = uploadedFileLocation + fileDetail.getFileName();
			OutputStream out = new FileOutputStream(new File(
					filePath));
			int read = 0;
			byte[] bytes = new byte[1024];

			out = new FileOutputStream(new File(filePath));
			while ((read = uploadedInputStream.read(bytes)) != -1) {
				out.write(bytes, 0, read);
			}
			out.flush();
			out.close();
		} catch (IOException e) {

			e.printStackTrace();
		}

	}

}
```

* Finally push the code to OpenShift server using git push.
 