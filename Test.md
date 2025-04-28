Got it — you want detailed steps to integrate Entra ID (Azure Active Directory) to implement SSO using SAML 2.0 in your Java web application, and you want:

Multiple Country and Role claims per user (multi-valued claims)

Application claim (static or dynamic depending on use case)

A sequential diagram showing from User enters URL → Landing page

A sample SAML 2.0 payload showing how claims would look.


I'll cover everything very clearly.


---

1. High-Level SSO Flow with SAML 2.0

User enters the app URL

App redirects to Entra ID login (SAML Request)

User authenticates with Entra ID

Entra ID issues a SAML Response with user info (claims)

App validates the SAML Response

App extracts Country, Application, Role claims

App authorizes and shows the landing page



---

2. Detailed Step-by-Step Integration Guide


---

Step 1: Prepare Azure Entra ID (Azure Active Directory)

1.1. Create an Enterprise Application

Go to Azure portal → Entra ID → Enterprise applications → New Application

Select Create your own application

Choose Integrate any other application you don’t find → SAML (Single sign-on option)



---

1.2. Configure Basic SAML Settings

Identifier (Entity ID): Your application’s Entity ID (e.g., https://yourapp.com/saml/metadata)

Reply URL (Assertion Consumer Service URL): Your SAML ACS URL (e.g., https://yourapp.com/saml/acs)

Sign-on URL (optional): https://yourapp.com/ (entry point)



---

1.3. Configure User Attributes and Claims

Go to Single Sign-on > Attributes & Claims

Edit and Add claims:


You can map multiple claims by using expressions like:

Country → user.extensionattribute1 (if you are using extension attributes)

Application → Static string

Roles → user.assignedroles or based on group membership



---

1.4. Download Federation Metadata XML

From SAML Signing Certificate section, download the Federation Metadata XML.

You'll need it in your app.



---

Step 2: Prepare Java Web Application


---

2.1. Choose SAML 2.0 Library

Use a mature SAML library, like:

Spring Security SAML (Spring apps)

Pac4j + SAML (Java Web apps)

OpenSAML (low level, harder)


For a normal web app, I recommend Spring Security SAML 2.0 Service Provider.


---

2.2. Add Maven Dependencies

Example for Spring Boot:

<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-saml2-service-provider</artifactId>
</dependency>


---

2.3. Configure SAML Settings

SP Entity ID = same as registered in Entra ID

ACS URL = your app’s endpoint for receiving SAML Response

IDP Metadata URL = the Federation Metadata XML URL or file downloaded


Example Spring Security config:

@Bean
public RelyingPartyRegistrationRepository relyingPartyRegistrationRepository() {
    RelyingPartyRegistration registration = RelyingPartyRegistrations
        .fromMetadataLocation("classpath:/metadata/entra-id-metadata.xml")
        .registrationId("azure-saml")
        .entityId("https://yourapp.com/saml/metadata")
        .assertionConsumerServiceLocation("https://yourapp.com/saml/acs")
        .build();
    return new InMemoryRelyingPartyRegistrationRepository(registration);
}


---

2.4. Handle Claims Extraction

When a user logs in, Spring Security will make the claims available in the Authentication object.

Example:

Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
Saml2Authentication samlAuth = (Saml2Authentication) authentication;
Saml2AuthenticatedPrincipal principal = samlAuth.getPrincipal();

// Access attributes
List<String> countries = principal.getAttribute("country");
List<String> roles = principal.getAttribute("role");
String application = principal.getFirstAttribute("application");


---

2.5. Landing Page Controller

After successful authentication, your app redirects to a secured landing page:

@GetMapping("/landing")
public String landingPage(Model model) {
    Saml2AuthenticatedPrincipal principal = (Saml2AuthenticatedPrincipal) SecurityContextHolder.getContext().getAuthentication().getPrincipal();
    model.addAttribute("countries", principal.getAttribute("country"));
    model.addAttribute("roles", principal.getAttribute("role"));
    model.addAttribute("application", principal.getFirstAttribute("application"));
    return "landing";
}


---

Step 3: SAML 2.0 Payload Example (SAML Response)

Here’s how a typical decoded SAML Response would look:

<samlp:Response>
  <saml:Assertion>
    <saml:Subject>
      <saml:NameID>user@example.com</saml:NameID>
    </saml:Subject>
    <saml:AttributeStatement>
      <saml:Attribute Name="country">
        <saml:AttributeValue>India</saml:AttributeValue>
        <saml:AttributeValue>Singapore</saml:AttributeValue>
      </saml:Attribute>
      <saml:Attribute Name="application">
        <saml:AttributeValue>MyApp</saml:AttributeValue>
      </saml:Attribute>
      <saml:Attribute Name="role">
        <saml:AttributeValue>Admin</saml:AttributeValue>
        <saml:AttributeValue>Viewer</saml:AttributeValue>
      </saml:Attribute>
    </saml:AttributeStatement>
  </saml:Assertion>
</samlp:Response>


---

3. Sequential Diagram

Here's the end-to-end sequence in Mermaid format (I'll render it as text):
``` mermaid
sequenceDiagram
    participant User
    participant Browser
    participant JavaApp
    participant EntraID (Azure)

    User ->> Browser: Enter app URL
    Browser ->> JavaApp: HTTP Request
    JavaApp ->> Browser: Redirect to Entra ID (SAML AuthnRequest)
    Browser ->> EntraID (Azure): SAML AuthnRequest
    EntraID (Azure) ->> Browser: Login Page
    User ->> EntraID (Azure): Enter Username & Password
    EntraID (Azure) ->> Browser: SAML Response (with Country, Application, Role claims)
    Browser ->> JavaApp: POST SAML Response (to /saml/acs)
    JavaApp ->> JavaApp: Validate Signature, Extract Claims
    JavaApp ->> JavaApp: Create User Session
    JavaApp ->> Browser: Redirect to Landing Page
    Browser ->> JavaApp: GET /landing
    JavaApp ->> Browser: Landing Page with User info
```

---

4. Special Notes for Your Requirements

Multiple Countries and Roles: Make sure claims in Azure allow multi-valued attributes.

Session Storage: After authentication, you may store extracted claims in the session or use them via the security context.

Logout: Implement SAML Single Logout (optional but recommended).



---

Summary


---

Would you also like me to provide a full ready-to-use Spring Boot project template with this setup (SAML integrated, login, landing page)?
It would save you at least 2-3 days of manual work. Let me know!

