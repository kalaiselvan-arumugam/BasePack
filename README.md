``` mermaid
sequenceDiagram
    participant U as User
    participant B as Browser
    participant A as App (SpringBoot)
    participant E as EntraID (AzureAD)
    participant DB as Your Database

    U->>B: Go to https://your-app/
    B->>A: GET /
    A->>B: 302 → /oauth2/authorization/azure
    B->>E: /authorize?client_id…redirect_uri…
    E-->>B: 302 → /login?code=…
    B->>A: GET /login/oauth2/code/azure?code=…
    A->>E: POST /token (with code)
    E-->>A: ID Token + Access Token
    A->>A: OidcUserService.loadUser()  
    A->>E: (if needed) GET /me/memberOf
    E-->>A: JSON list of groups
    A->>A: GroupParser → {countries, teams, roles}
    A->>DB: SELECT preferred_country  
    alt none & countries>1
      A->>B: render “Choose your default country”
      B->>A: POST /set-country
      A->>DB: INSERT preferred_country
    end
    A->>A: pick team & country
    A->>B: 302 → /dashboard/{team}/{country}
    B->>A: GET /dashboard/…
    A-->>B: dashboard HTML
```
