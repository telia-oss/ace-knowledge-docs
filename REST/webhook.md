> This documentation is valid for widget V1-V4

# Contact Method WebHook

This described how to use the Web Service contact method to pass information from a form to a system external to Humany and display information from this service to users using Humany widgets. Developers are the intended audience of this documentation.

## Background

When creating a new contact method, you have the option of choosing a type called Web service. This is a special kind of contact method that transfers the end user's request to an external HTTP REST web service of your choosing. The receiving web service is then responsible to take any action appropriate, based on that request. It can also choose to optionally respond back to Humany with a dynamic answer that describes the outcome of the action. This dynamic answer will then be injected into a special placeholder and displayed to the end user.

The intent of this type of contact method is to provide a way to immediately solve more complex, dynamic end user issues than would have been possible using just simple static text.

## Web service requirements

When executing a contact method of type Web service, Humany will act as a middle-man between the end-user and the external service. The web service is expected to follow a certain contract. More exactly,

- Humany will send a HTTP request with verb POST to the URL specified by the editor under URL.
- The request body will be one of the Content-Types specified by the editor under MIME FORMAT (see section Receiving Content-Type).
- The web service now has a maximum of 15 seconds to complete whatever task it undertakes before responding, or there will be a timeout error on Humany's side.
- Humany expects that the external web service honor the Accept header of the request, making text/html or text/plain the 2 only valid response formats of your service.
- Humany receives a properly formatted response from the web service and injects any possible response text into the special placeholder specified by the editor under the section Text displayed after sending.
- The dynamic answer is displayed to the user.

## Receiving Content-Type

Humany will transfer the current state and context of the end user request to the external web service in the formats listed below. The context includes any possible form fields filled-out by the end user prior to sending the contact method form, aswell as Humanys current list of parameters associated with the current user request.

### Example application/json

```javascript
{
	Context:
	{
		Parameters: [
			{ Name = "FirstName", Value = "Mikael" },
			{ Name = "LastName", Value = "Robinson" },
			{ Name = "ProblemDescription", Value = "Lorem ipsum dolor sit amet" },
			{ Name = "UserId", Value = "1992883-37746GX-F00" },
			{ Name = "Site", Value = "http://demobolaget.humany.net/test-interface#humany-test-interface=/contact/1362" },
			{ Name = "Perspective", Value = "Default" }
		]
	}
}
```

### Example application/xml

```xml
<context>
	<parameters>
	<parameter name="FirstName">Mikael</parameter>
	<parameter name="LastName">Robinson</parameter>
	<parameter name="ProblemDescription">Lorem ipsum dolor sit amet</parameter>
	<parameter name="UserId">1992883-37746GX-F00</parameter>
	<parameter name="Site">http://demobolaget.humany.net/test-interface#humany-test-interface=/contact/1362</parameter>
	<parameter name="Perspective">Default</parameter>
	</parameters>
</context>
```

### Example x-www-form-urlencoded

```
FirstName=Mikael&LastName=Robinson&ProblemDescription=Lorem+ipsum+dolor+sit+amet&UserId=1992883-37746GX-F00&
Site=http%3a%2f%2fdemobolaget.humany.net%2ftest-interface%23humany-test-interface%3d%2fcontact%2f1362&Perspective=Default
```

## Custom HTTP headers

Humany allows the editor to specify custom HTTP headers that will be included in the request to the external web service. These headers can be used to provide additional context or metadata about the request. Headers can be secret (e.g. API keys) or non-secret. Secret headers will be stored encrypted and only decrypted when sending the request to the external web service.

## HTTP request signing

Humany supports HTTP request signing using JWT tokens. This optional feature can be enabled by the editor when creating or editing a contact method of type Web Service by checking `Use request signing` and providing a secret key under `Receiving web service` section.

![alt text](/screenshots/contact-method-service.png)


 If enabled, Humany will sign the HTTP request using a JWT token and include this token in the header `X-ACE-Signature` of the request. The token is signed using the HMAC SHA256 algorithm and a secret key specified by the editor. Token includes the following claims:
 
 * `iss`: The issuer of the token, set to "ace-knowledge".
 * `aud`: The audience of the token, set to receiving endpoint url.
 * `exp`: The token expiration time, this is set to 1 minute after the issued at time.
 * `iat`: The issued at time, represented as a Unix timestamp.
 * `content_hash`: A SHA256 hash of the request body, encoded as a base64 string.
 * `method`: The HTTP method used (e.g., POST).
 * `path`: The request path (e.g., /api/v1/contact).
 * `custom headers`: Any custom headers specified by the editor will be included as claims in the token as well.

The external web service can then verify the signature of the token using the same secret key and validate the claims to ensure the integrity and authenticity of the request.

### Example pseudocode to verify the JWT token and validate the content hash

```
function verifySignature(request, secretKey) {

    // Step 1: Extract signature from request headers
    token = request.headers["X-ACE-Signature"]
    if token is null or empty:
        return false
    
    try:
        // Step 2: Verify and decode the JWT token
        payload = JWT.verify(token, secretKey, algorithm="HS256")
        
        // Step 3: Get request body content
        content = request.body.toString() OR empty_string_if_null
        
        // Step 4: Compute SHA256 hash of the request body
        computed_hash = SHA256(content).encode_as_base64()
        
        // Step 5: Compare the computed hash with the hash in JWT payload
        if payload.content_hash != computed_hash:
            log_error("content_hash mismatch:", payload.content_hash, "!=", computed_hash)
            return false
        
        // Step 6: All checks passed
        return true
        
    catch (JWT_verification_error OR any_other_error):
        return false
}
```

### Security Considerations

When implementing JWT signature verification:

- **Store the secret key securely** - Never expose it in client-side code or logs. The secret key should be at least 32 characters long for adequate security.
- **Use HTTPS only** - All webhook communications should use encrypted connections (new web service contact methods will enforce HTTPS)
- **Implement rate limiting** - Protect your endpoint from excessive requests
- **Log security events** - Monitor for signature verification failures

## Troubleshooting

Any error that occurs in the communication between Humany and the external web service will be reported as a 500 Internal Server Error on the original request to Humany, along with an error message for debugging purposes.