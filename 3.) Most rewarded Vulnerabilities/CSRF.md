## Cross-Site Request Forgery (CSRF): With and Without State

#### 🌳 What is CSRF?
A web security vulnerability where an attacker tricks a logged‑in user’s browser into performing unwanted actions on a trusted website. 
It exploits the trust a site has in a user’s browser. Imagine you logged into your bank account and malicious site loads hidden code like:
- html:-

       <img src="https://bank.com/transfer?amount=500&to=attacker" />
  
Your browser automatically sends this request with your session cookie → the bank processes it as if you requested 
the transfer. If the bank’s web application doesn’t verify the source of the requests properly, an attacker can craft 
a malicious request and get the victim to execute it just by visiting a link or page.



#### 🌳 The Core Requirement: User State:- 

For CSRF to work, user state (typically cookies) must be sent automatically by the browser. This only works if:
- The user is authenticated
- The site uses cookie-based authentication (e.g., session ID)
- The server accepts and trusts those cookies without verifying intent

CSRF doesn't work when:
- Token-based authentication (e.g., Bearer JWT in header) is used
- CORS and SameSite cookies are configured properly

#### 🌳 Typical Targets for CSRF
- Changing email or password
- Adding or removing accounts
- Transferring money or crypto
- Submitting support requests
- Altering security settings
- Subscribing/unsubscribing to features

Any state-changing action is a potential CSRF target.


#### 🌳 Example of CSRF Attack:- 

Target request:

       POST /change-email HTTP/1.1
       Host: vulnerable.com
       Cookie: sessionid=abc123
       Content-Type: application/x-www-form-urlencoded

      email=attacker@evil.com

Malicious page:

      <html>
         <body>
           <form action="https://vulnerable.com/change-email" method="POST">
              <input type="hidden" name="email" value="attacker@evil.com" />
              <input type="submit" value="Click me" />
           </form>
          <script>document.forms[0].submit();</script>
         </body>
     </html>

When the victim loads the page while logged into vulnerable.com, the browser sends their session cookie with the 
POST request.


#### 🌳 CSRF Without JavaScript (GET-Based):-

       <img src="https://bank.com/transfer?to=attacker&amount=1000">

If the endpoint uses `GET` for state-changing actions (which it never should), the browser will auto-trigger the 
request just by rendering the image tag.


#### 🌳 Advanced CSRF: JSON, CORS and Preflight:-

Modern apps often use `application/json`, which changes the attack surface:
- Browsers **won't** let cross-origin sites send POSTs with custom headers or JSON body unless CORS allows it.
- This **defeats basic CSRF** unless the server is misconfigured.

However, some apps may:
- Accept `application/x-www-form-urlencoded` and JSON interchangeably
- Not verify the `Origin` or `Referer` headers


#### 🌳 Bypassing CSRF Protections:- 
Techniques include:
- Exploiting endpoints that lack token verification
- Finding endpoints that accept GET or don’t check CSRF tokens
- Exploiting incorrect SameSite cookie policies
- Abuse of X-Requested-With header not being verified

Weak tokens:
- Static token reused across sessions
- Token present but not validated
- Token not bound to the session/user


#### 🌳 SameSite Cookies and CSRF:- 
- `SameSite=Lax`: Blocks CSRF on most POSTs, but allows safe methods like `GET`
- `SameSite=Strict`: Blocks cross-origin requests completely
- `SameSite=None`: Cookies are sent cross-origin, but must be `Secure`

Misconfigurations (like missing SameSite or None without Secure) can reintroduce CSRF risks.


#### 🌳 Testing for CSRF:-
- Identify a state-changing action
- Reproduce the action using a proxy like Burp Suite
- Craft a POC (form-based or JavaScript auto-submit)
- Send to target user (social engineering, or test yourself if scope allows)
- Confirm that the action happens **without user interaction or token validation**

Burp Suite Pro has CSRF POC generators. Manual crafting is also straightforward.


#### 🌳 Mitigation and Prevention:-
From the developer side:
- Use **anti-CSRF tokens** (random per form)
- Enforce **SameSite** **cookie policies**
- Check **Origin** and **Referer** headers
- Avoid using `GET` for actions that change server state
- Prefer **token-based authentication** over cookie sessions for APIs

From the hunter side:
- Avoid testing CSRF on actions out of scope (e.g., deletion)
- Always test in a safe and reproducible way
- Respect legal boundaries (no phishing or real exploitation outside scope)


#### 🌳 Conclusion:-
While CSRF is less common in modern apps due to SameSite cookies and token protections, it remains a valid and 
sometimes overlooked bug class. Hunters should know how to:
