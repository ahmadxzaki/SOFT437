## theme
```
%%{init: {
  "theme":"base",
  "themeVariables": {
    "primaryColor": "#ffffff",
    "background": "#ffffff",
    
    "actorBkg": "#2b9f52",
    "actorBorder": "#2999ef",
    "actorTextColor": "#ffffff",
    "actorLineColor": "#ef920f",

    "signalColor": "#00ff00",
    "signalTextColor": "#ee1111",
    
    "noteBkgColor": "rgba(176, 16, 16, 0.7)",
    "noteTextColor": "#bf11e2",

    "activationBkgColor": "#ffff00",
    "activationBorderColor": "#000000",

    "loopLineColor": "#ff00ff",
    "loopTextColor": "#0000ff",
    "labelBoxBkgColor": "#e6e6e6",
    "labelBoxBorderColor": "#ff00ff"
  }
}}%%
```

%%{init: {
  "theme": "default",
  "themeVariables": {
    "actorBkg": "#6DB1FF",
    "actorBorder": "#f2a7c6",
    "primaryColor": "#e6f0ff",
    "primaryBorderColor": "#f2a7c6",
    "activationBkgColor": "#ced4db",
    "activationBorderColor": "#1BC8B1",
    "lineColor": "#000000",
    "textColor": "#ffffff",
    "backgroundColor": "#ffffff"
  }
}}%%


## server side rendering moment
```nginx
root /var/www/html/public; # Your static file directory

location / {
    # 1. Try to find the exact file ($uri)
    # 2. Try to find a directory with that name ($uri/)
    # 3. If neither exists, send to the named location @app
    try_files $uri $uri/ @app;
}

location @app {
    proxy_pass http://127.0.0.1:8080;
    proxy_set_header Host $host;
}
```

## High level architecture
```mermaid
%%{init: {
  "theme": "dark",
  "themeVariables": {
    "textColor": "#ffffff",
    
    "signalColor": "#a0a0a0",
    "signalTextColor": "#ffffff",
    "loopTextColor": "#ffffff",
    
    "activationBkgColor": "#404040",
    "activationBorderColor": "#5d5d5d"
  }
}}%%
sequenceDiagram
    actor U as User / Browser
    participant W as Web Server
    participant A as Application Server

    U->>W: some HTTPS req
    activate W
    alt Static p=0.56
        W-->>U: relevant .html file<br>from disk
    else Dynamic p=0.44
        W->>A: Forward request
        activate A
        A-->>W: constructed .html file
        deactivate A
        W-->>U: constructed .html file
    end
    deactivate W
```

## Search
```mermaid
sequenceDiagram
    actor U as User / Browser
    participant W as Web Server
    participant H as Application Server
    participant DB as Database Server

    U->>W: HTTPS GET /search
    activate W
    W->>H: Forward request
    activate H
    note over H, DB: Create Search Page
    H-->>W: HTTP response <br> with HTML of the search page
    deactivate H
    W-->>U: HTML of the search page
    deactivate W
```

## Create Search Page
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
    participant H as :HttpHandler<br> <<App. Server>>
    participant P as :RequestParser<br> <<App. Server>>
    participant S as :SearchPageCreator<br> <<App. Server>>
    participant Repo as :BookRepository<br> <<App. Server>>
    participant DB as Database<br><<DB Server>>

    activate H
    H->>P: parseSearchReq(req)
    activate P

    alt Invalid request
        P->>H: Request is invalid
        deactivate P
        H->>S: buildSearchErrorPage()
        activate S
        S-->>H: HTML of the search page with an error pageHtml := buildSearchErrorPage()
        deactivate S

    else Valid request
        activate P
        P-->>H: Search query <br>and filters
        deactivate P
        H->>S: buildSearchPage(query, filters)
        activate S

        S->>Repo: getBooks(query, filters)
        activate Repo

        activate Repo
        Repo->>Repo: formulateBookSearchQuery(query, filters)
        deactivate Repo

        Repo->>DB: Book search query
        activate DB
            activate DB
            DB->>DB: Retrive all books<br> matching the query<br> and filters
            deactivate DB
        DB-->>Repo: Book search query response
        deactivate DB

        activate Repo
        Repo->>Repo: createBookList(db_resp)
        deactivate Repo

        Repo-->>S: List of books
        deactivate Repo

        alt len(book list) > 0
                activate S
                S->>S: generateSearchResultsHtml(books)
                deactivate S
            S-->>H: HTML of the search results page
            deactivate S
        else len(book list) == 0
            activate S
                activate S
                S->>S: generateNoResultsFoundHtml()
                deactivate S
            S-->>H: HTML of the no search results found page
        end
        deactivate S
    end
    
    activate H
    H->>H: Write HTTP response<br>to connection
    deactivate H

    deactivate H
```