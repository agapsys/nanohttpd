## NanoHTTPD – a tiny web server in Java

*NanoHTTPD* is a light-weight HTTP server designed for embedding in other applications, released under a Modified BSD licence.

It is being developed at Github and uses Apache Maven for builds & unit testing

## Quickstart

### Custom web app

Let's raise the bar and build a custom web application next:

    mvn archetype:generate -DgroupId=com.example -DartifactId=myHellopApp -DinteractiveMode=false
    cd myHellopApp
    
Edit `pom.xml`, and add this between \<dependencies\>:
 
	<dependency>
		<groupId>com.agapsys</groupId>
		<artifactId>nanohttpd-core</artifactId>
		<version>0.1.0</version>
	</dependency>
	
Edit `src/main/java/com/example/App.java` and replace it with:
```java
    package com.example;
    
    import java.io.IOException;
    import java.util.Map;
    
    import fi.iki.elonen.NanoHTTPD;
    
    public class App extends NanoHTTPD {
    
        public App() throws IOException {
            super(8080);
            start(NanoHTTPD.SOCKET_READ_TIMEOUT, false);
            System.out.println("\nRunning! Point your browsers to http://localhost:8080/ \n");
        }
    
        public static void main(String[] args) {
            try {
                new App();
            } catch (IOException ioe) {
                System.err.println("Couldn't start server:\n" + ioe);
            }
        }
    
        @Override
        public Response serve(IHTTPSession session) {
            String msg = "<html><body><h1>Hello server</h1>\n";
            Map<String, String> parms = session.getParms();
            if (parms.get("username") == null) {
                msg += "<form action='?' method='get'>\n  <p>Your name: <input type='text' name='username'></p>\n" + "</form>\n";
            } else {
                msg += "<p>Hello, " + parms.get("username") + "!</p>";
            }
            return newFixedLengthResponse(msg + "</body></html>\n");
        }
    }
```

Compile and run the server:
 
    mvn compile
    mvn exec:java -Dexec.mainClass="com.example.App"
    
If it started ok, point your browser at <http://localhost:8080/> and enjoy a web server that asks your name and replies with a greeting. 

## Features
### Core
* Only one Java file, providing HTTP 1.1 support.
* No fixed config files, logging, authorization etc. (Implement by yourself if you need them. Errors are passed to java.util.logging, though.)
* Support for HTTPS (SSL).
* Basic support for cookies.
* Supports parameter parsing of GET and POST methods.
* Some built-in support for HEAD, POST and DELETE requests. You can easily implement/customize any HTTP method, though.
* Supports file upload. Uses memory for small uploads, temp files for large ones.
* Never caches anything.
* Does not limit bandwidth, request time or simultaneous connections by default.
* All header names are converted to lower case so they don't vary between browsers/clients.
* Persistent connections (Connection "keep-alive") support allowing multiple requests to be served over a single socket connection.

### Develop your own specialized HTTP service

For a specialized HTTP (HTTPS) service you can use the module with artifactId *nanohttpd-core*.

		<dependency>
			<groupId>com.agapsys</groupId>
			<artifactId>nanohttpd-core</artifactId>
			<version>CURRENT_VERSION</version>
		</dependency>
		
Here you write your own subclass of *fi.iki.elonen.NanoHTTPD* to configure and to serve the requests.
  
### generating an self signed SSL certificate

Just a hint how to generate a certificate for localhost.

	keytool -genkey -keyalg RSA -alias selfsigned -keystore keystore.jks -storepass password -validity 360 -keysize 2048 -ext SAN=DNS:localhost,IP:127.0.0.1  -validity 9999

This will generate a keystore file named 'keystore.jks' with a self signed certificate for a host named localhost with the IP address 127.0.0.1 . Now
you can use:

	server.makeSecure(NanoHTTPD.makeSSLSocketFactory("/keystore.jks", "password".toCharArray()), null);

Before you start the server to make NanoHTTPD serve HTTPS connections, when you make sure 'keystore.jks' is in your classpath.
 
-----

*Thank you to everyone who has reported bugs and suggested fixes.*
