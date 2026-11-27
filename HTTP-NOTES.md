# 🌍 HTTP (HyperText Transfer Protocol) Notes

**What is HTTP?**
It is the foundation of data communication on the World Wide Web. It dictates how web browsers (clients) and web servers communicate.

## 📊 Client-Server Communication Model
```text
[Client (Browser)] ────────(HTTP Request)───────▶ [Web Server]
   (You)                                           (TryHackMe)
                   ◀───────(HTTP Response)───────
📦 Structure of an HTTP Request
text
GET /blog?id=1 HTTP/1.1          <-- Method, Path/Query, Version
Host: tryhackme.com              <-- Header (Destination)
User-Agent: Mozilla/5.0          <-- Header (Browser info)
Accept: text/html                <-- Header (What client accepts)
                                 <-- Blank Line (End of headers)
(username=admin&pass=123)        <-- Body (Data sent, usually in POST/PUT)
⚙️ HTTP Methods (Verbs)
GET: Retrieve data from the server (e.g., loading a webpage).

POST: Submit data to the server (e.g., submitting a login form).

PUT: Update existing data on the server.

DELETE: Remove data from the server.

🔍 URL Parameters vs. Query Strings
Path: The resource location (e.g., /blog).

Query String: Data passed in the URL, starting with ? (e.g., /blog?id=1&author=pankaj).

Body Parameters: Data sent hidden inside the request body (not visible in URL), mostly used in POST/PUT.

📡 HTTP Headers
Request Headers: Sent by client (e.g., Host, User-Agent, Cookie).

Response Headers: Sent by server (e.g., Server, Set-Cookie, Content-Type).

🔢 HTTP Status Codes
2xx (Success): 200 OK (Request successful).

3xx (Redirection): 301 Moved Permanently.

4xx (Client Error): 404 Not Found, 403 Forbidden.

5xx (Server Error): 500 Internal Server Error.
