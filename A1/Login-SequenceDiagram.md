# Login/Register
```mermaid
%%{init: {
  "theme": "dark",
  "themeVariables": {
    "textColor": "#ffffff",
    
    "signalColor": "#a0a0a0",
    "signalTextColor": "#ffffff",
    "loopTextColor": "#ffffff",
    "actorLineColor": "#787878",
    "noteTextColor": "#ffffff",
    
    "activationBkgColor": "#404040",
    "activationBorderColor": "#5d5d5d"
  }
}}%%
sequenceDiagram
    actor U as User / Browser
    participant W as Web Server
    participant A as Application Server
    participant DB as Database Server

    U->>W: HTTPS GET /login?redirectUrl=<addr>
    W-->>U: Login page in HTML
    alt User has an account
        U->>U: Enter username<br>and password
        U->>W: POST /login
        W->>A: Forward request
        note over A, DB: Authenticate Existing User
        A-->>W: HTTP Response
    else User has no account
        U->>U: Click register button
        U->>W: HTTPS GET /register
        W-->>U: Register page in HTML
        U->>U: Enter new username<br> and password
        U->>W: POST /register
        W->>A: Forward request
        note over A, DB: Register New User
        A-->>W: HTTP Response
    end
    W->>U: Forward HTTP response
```
## Authenticate Existing User
UserAuthenticator requires configuration: token validity duration

SessionTokenValidityDurationSeconds: 143243290
```mermaid
%%{init: {
  "theme": "dark",
  "themeVariables": {

    "textColor": "#ffffff",
    "actorLineColor": "#787878",
    "signalColor": "#a0a0a0",
    "signalTextColor": "#ffffff",
    "loopTextColor": "#ffffff",
    "noteTextColor": "#ffffff",
    
    "activationBkgColor": "#404040",
    "activationBorderColor": "#5d5d5d"
  }
}}%%
sequenceDiagram
    participant H as :HttpHandler<br> <<App. Server>>
    participant P as :RequestParser<br> <<App. Server>>
    participant L as :LoginPageCreator<br> <<App. Server>>
    participant A as :UserAuthenticator<br> <<App. Server>>
    participant DB as Database<br><<DB Server>>

    activate H
    H->>P: parseLoginReq(req)
    activate P

    alt Invalid request
        P->>H: Request is invalid
        deactivate P
        H->>L: buildLoginErrorPage()
        activate L
        L-->>H: HTML of the login page with an error
        
        deactivate L
        
    else Valid request
        activate P
        P-->>H: Username, password,<br>redirect_addr
        deactivate P

        H->>A: authenticateUser(username, password)
        activate A

            activate A
            A->>A: hashPassword(password)
            deactivate A

            activate A
            A->>A: formulateLoginQuery(username, hashed_password)
            deactivate A

        A->>DB: Login query
        activate DB
            activate DB
            DB->>DB: Check if username<br>exists and if so, if the <br>hashed_password <br>matches DB entry
            deactivate DB
        DB-->>A: Login query response
        deactivate DB

        activate A
        A->>A: createLoginResult(db_resp)
        deactivate A

        A-->>H: Login result
        deactivate A
            
        alt Username not found
            H->>L: buildUsernameNotFoundLoginPage()
            activate L
            L-->>H: HTML of the username not found page
            deactivate L
        else Incorrect password
            H->>L: buildIncorrectPasswordLoginPage()
            activate L
            L-->>H: HTML of the incorrect password page
            deactivate L
        else Login successful
            H->>A: createSessionToken(username)
            activate A
            Note over A, DB: Generate Session Token
            A-->>H: Session token
            deactivate A
            H->>H: createRedirectHttpResp(<br>redirect_addr, session_token)
        end

        
    end

    activate H
    H->>H: Write HTTP response<br>to connection
    deactivate H

    deactivate H
```

## Register New User
```mermaid
%%{init: {
  "theme": "dark",
  "themeVariables": {

    "textColor": "#ffffff",
    "actorLineColor": "#787878",
    "signalColor": "#a0a0a0",
    "signalTextColor": "#ffffff",
    "loopTextColor": "#ffffff",
    "noteTextColor": "#ffffff",
    
    "activationBkgColor": "#404040",
    "activationBorderColor": "#5d5d5d"
  }
}}%%
sequenceDiagram
    participant H as :HttpHandler<br> <<App. Server>>
    participant P as :RequestParser<br> <<App. Server>>
    participant R as :RegisterPageCreator<br> <<App. Server>>
    participant A as :UserAuthenicator<br> <<App. Server>>
    participant DB as Database<br><<DB Server>>

    activate H
    H->>P: parseRegisterReq(req)
    activate P

    alt Invalid request
        P->>H: Request is invalid
        deactivate P
        H->>R: buildRegisterErrorPage()
        activate R
        R-->>H: HTML of the regiser<br>page with an error
        deactivate R
        
    else Valid request
        activate P
        P-->>H: Username, password,<br>redirect address
        deactivate P
        H->>A: createAccount(username, password)
        activate A
            activate A
            A->>A: hashPassword(password)
            deactivate A
            activate A
            A->>A: formulateRegisterQuery(<br>username, passwordHash)
            deactivate A
        A->>DB: Register query
        activate DB
            activate DB
            DB->>DB: Check username uniqueness,<br>register user if unique
            deactivate DB
        DB-->>A: Register query response
        deactivate DB
            activate A
            A->>A: createRegisterResponse(db_resp)
            deactivate A
        A-->>H: Register response
        deactivate A
        
        alt Username Unavailable
            H->>R: buildUsernameUnavailablePage()
            activate R
            R-->>H: HTML of the username unavailable page
            deactivate R
        else Registration Successful
            H->>A: createSessionToken(username)
            activate A
            Note over A, DB: Generate Session Token
            A-->>H: Session token
            deactivate A
            H->>H: createRedirectHttpResp(<br>redirect_addr, session_token)
        end
    end

    activate H
    H->>H: Write HTTP response<br>to connection
    deactivate H

    deactivate H
```

## Generate Session Token
```mermaid
%%{init: {
  "theme": "dark",
  "themeVariables": {

    "textColor": "#ffffff",
    "actorLineColor": "#787878",
    "signalColor": "#a0a0a0",
    "signalTextColor": "#ffffff",
    "loopTextColor": "#ffffff",
    
    "activationBkgColor": "#404040",
    "activationBorderColor": "#5d5d5d"
  }
}}%%
sequenceDiagram
    participant A as :UserAuthenicator<br> <<App. Server>>
    participant DB as Database<br><<DB Server>>

    activate A
        activate A
        A->>A: generateSessionToken(username)
        deactivate A
        activate A
        A->>A: formulateStoreTokenQuery(username, <br>token, expiry_date)
        deactivate A
    A->>DB: Store token query

    activate DB
    DB->>DB: Store username <br>and token pair
    deactivate DB
    
    deactivate A
```