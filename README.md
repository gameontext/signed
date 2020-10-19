# Signed: A library for processing signed JWTs and HMACs in Game On

For client/server consistency, we've created some common libraries to help with
signing and verifying requests that use JWTs or HMACs

## Using the library in your Java projects

We use jitpack to build this library, which means you can direct maven or gradle directly to our github releases to satisfy dependencies.

1. include jitpack.io in your list of repositories:
  * In maven:
  ```
    <repositories>
      <repository>
        <id>jitpack.io</id>
        <url>https://jitpack.io</url>
      </repository>
    </repositories>
  ```
  * In gradle:
  ```
    repositories {
      maven { url "https://jitpack.io" }
    }
  ```
2. Include version of the library in your dependencies:
  * In maven:
  ```
    <dependency>
      <groupId>com.github.gameontext</groupId>
      <artifactId>signed</artifactId>
      <version>v1.0.4</version>
    </dependency>
  ```
  * In gradle:
  ```
    dependencies {
	    compile 'com.github.gameontext:signed:v1.0.4'
    }
  ```
3. Use SignedRequest* utilities to handle request signing

  * SignedRequestFilter with JAX-RS 2.0 client:
  ```
    Client client = ClientBuilder.newClient().register(JsonProvider.class);

    SignedClientRequestFilter apikeyFilter = new SignedClientRequestFilter(userid, secret);
    client.register(apikeyFilter);
  ```

  * Room-side of WebSocket Handshake via ServerEndpointConfig.Configurator
  ```
        public void modifyHandshake(ServerEndpointConfig sec, HandshakeRequest request, HandshakeResponse response) {
            super.modifyHandshake(sec, request, response);

            if ( token == null || token.isEmpty() ) {
                Log.log(Level.FINEST, this, "No token set for room, skipping validation");
            } else {
                Log.log(Level.FINEST, this, "Validating WS handshake");
                SignedRequestHmac wsHmac = new SignedRequestHmac("", token, "", request.getRequestURI().getRawPath());

                try {
                    wsHmac.checkHeaders(new SignedRequestMap.MLS_StringMap(request.getHeaders()))
                            .verifyFullSignature()
                            .wsResignRequest(new SignedRequestMap.MLS_StringMap(response.getHeaders()));

                    Log.log(Level.INFO, this, "validated and resigned", wsHmac);
                } catch(Exception e) {
                    Log.log(Level.WARNING, this, "Failed to validate HMAC, unable to establish connection", e);

                    response.getHeaders().replace(HandshakeResponse.SEC_WEBSOCKET_ACCEPT, Collections.emptyList());
                }
            }
        }
  ```
