# Internet Threat models
- Different to other threat models; operates under other different constraints than network-level attacks discussed previously (except DoS DNS and some other cases)
- The attack does not have control of the network, and cannot passively sniff traffic, decrypt it, or inject packets at the IP layer. 
- The attack surface is the web application layer itself, that is, the interaction between the browser and server over HTTP.
	- The browser is the attacker's tool whilst also being the victim's environment.
	- The attacker's goal is typically to get the browser to execute code of make requests it shouldn't, abusing trust relationships the browser maintains with legitimate servers.

## HTTP and Cookies
- HTTP is a stateless request-response protocol - each transaction is entirely independent
	- From perspective of the protocol, every request is from an anonymous client
- This is a problem, because almost everything users do on the web is stateful
- Some mechanism is therefore needed to associate a series of HTTP requests with a particular user session - this mechanism is called a **cookie**.
- When a server wants to establish state with a client, it includes a `Set-Cookie` header inside a HTTP response e.g., 
```
	HTTP/1.1 200 OK
	Content-Type: text/html
	Set-Cookie: theme=light
	Set-Cookie: sessionid=abc123; Expires=Wed, 09 Jun 2026
	10:18:14 GMT
```
- Browser stores these key-value pairs and associates them with the domain that set them.
- On every subsequent request to that domain, the browser automatically includes these cookies in the `Cookie` request header
- The server reads the `sessionid` value, looks it up in its session store, and identifies which authenticated user this request belongs to. 
	- Stateful sessions have attributes stored server  side
- Thus, whoever presents the cookie header in a get request with the matching `sessionid` and other attributes, is treated as an authenticated user.

### Types of Cookies
- **Session**
	- Have no `Expires` or `Max-Age` attribute. The browser discards them when the session ends, this is usually when the browser closes.
		- May be killed server side, just not stored explicitly on the cookie header
	- Used for short-lived authentication. 
- **Persistent**
	- Include an `Expires` or `Max-Age` attribute specifying when they should be deleted
	- Survive restarts. E.g., the keep me logged in checkbox typically sets a persistent cookie with a long expiry
		- E.g., login via `POST /login ...credentials`, then server creates a persistent cookie.
- **Secure**
	- Include a `Secure` flag, meaning the browser will only transmit this cookie over HTTPS connections, to prevent the cookie being intercepted by a network-level eavesdropper over an unencrypted connection. 
- **HTTPOnly**
	- Security attribute on cookies that controls who is permitted to read the cookie
	- Means that JavaScript cannot read the cookie or modify it, which it can under normal circumstances.

### Third-Party cookies and tracking
- Cookies are scoped to the domain that set them - amazon-com cookies don't go to google.com
- However, a website can, and often does, embed resources from other domains (images, iframes, scripts).
- When the browser loads such a site, and it includes an resource with an attribute (`src`) pointing to another domain, the browser makes a HTTP request to load that resource, and the resource provider is then able to set a cookie on your browser in the header of the response.
- Next time you visit any site containing resources from that provider, the browser resends the cookie, so the server knows that both sites were visited.
	- This is the mechanism behind cross-site tracking. 
### Cookie Vulnerabilities
- A session cookie is a *bearer token* - if an attacker obtains it as mentioned prior, they can present it to the server in the header of their own requests and the server will treat them as an authenticated user
- No cryptography needs to be broken, the attacker simply needs the token. 
- On plain HTTP, cookies are transmitted plaintext.
	- A network-level attacker - someone on the same network, an evil twin AP, a compromised router, can passively read every cookie. 
	- This is why the `Secure` flag matters and why HTTPS is non-negotiable for any authenticated session. 
- HTTPS prevents network-level interception of cookies, but does not permit the injection attack we discuss below - XSS, which can exfiltrate cookies via JavaScript - and also DNS poisoning, which will result in cookies being sent to the attacker IP instead (but HTTPS + DNSSEC mitigate).

## Cross-Site Scripting (XSS)
- Injection attack where an attacker injects malicious code into content that is then delivered to and executed by victim's browsers
- The injection itself is HTML markup via `<script>` tags; when the browser receives a HTTP response containing HTML, it parses it into a DOM, representing every element on the page. 
	- As it builds the DOM, it executes any JavaScript it encounters, regardless of origin. 
	- The browser can't distinguish between JavaScript written by the web developer and JavaScript injected by an attacker through an XSS vulnerability.
- Injected script runs in the origin of the vulnerable site, and so has access to that site's cookies provided they are not `HTTPOnly`.
	- Can install keylogger, access local storage, session storage, full DOM access/rewrite , can make authenticated requests to that site's APIs via use of the session cookie, usually target `document.cookie` which is the cookie associated with the document.
- Usually exfiltrate by constructing an image component in the injected JavaScript, whose source attribute points to an attacker controlled machine, and attaches the cookie/data in the GET request to 'load' the image.

### Reflected XSS
- Reflected XSS is a vulnerability that occurs when user-supplied(really attacker supplied) input is immediately included in the server's HTTP response without sanitisation.
- Take a 404 page that naively reflects the requested URL back to the user. 
- If the server template for such a response is like so:
```
return f"<p>Cannot find resource {request.path}</p>"
```

- Then the path is inserted directly into HTML. Now consider this URL:
```html
www.web.com/<script>var adr='../evil.php?cookie='+escape(document.cookie);</script>
```
- Then the server generates:
```html
<p>Cannot find resource web.com/<script>
var adr = '../evil.php?cookie=' + escape(document.cookie);
</script></p>
```
- The browser parses this as HTML, builds the DOM, encounters the script tag, and executes the JavaScript, returning all non-`HttpOnly` cookies for *web.com*.
- This requires the user to click on an entirely legitimate, but crafted link, with the malicious payload usually appearing in the URL
- The browser sees the response as coming from the legitimate server and so executes the script without question.

### Persistent (Stored) XSS
- Reflected XSS requires tricking victims into clicking a crafted link
- **Persistent XSS** is considerably more dangerous, in that this is not required.
- The malicious payload is **stored on the server** and served to every visitor automatically.
- Any feature that accepts user input and displays it to users, with minimal sanitisation, is a potential vector:
	- Comment sections, product reviews, chat messages
	- If the application stores the raw input, an attacker can post a comment containing a `<script>` tag.
	- Every subsequent visitor loading that page then receives and executes the script in their browser.
#### Samy Worm
- Exploited persistent XSS vulnerability in MySpace to create a self-propagating worm. 
- MySpace allowed some HTML in profile pages, but attempted to filter dangerous tags
- Used obsfuscation techniques to bypass filters
- The payload did two things when it executed in a victim's browser:
	1. Added Samy as a friend
	2. Copied itself onto the victim's profile.
- Spread exponentially and took down MySpace in 24 hours.


### Preventing XSS
- The instinctive fix is to search user input (both at the application layer, and at the request layer (reflected)) for dangerous strings and remove or block them. 
	- This 'blocklist sanitisation' fails as browsers are permissive HTML parsers, and attackers have hundreds of ways to achieve script execution without triggering obvious patterns, many of which do not even use a `<script>` tag, but still execute JavaScript.
	- It is not possible to enumerate all possible representations of dangerous input.
- The correct approach is **output encoding** - encode HTML elements which delimit tags, attributes, and entities
- It encodes those characters (`<`, `>`, `&`, `"`, `'`) that appear in *user-supplied data* so it can distinguish between the developer's intentional markup, the structure of the page, and the user-supplied content. 
- Malicious input is encoded, such that it never executes; this encoding may occur at different levels e.g., if it appears in URL, then happens on server before response returned, if it is appears in chat message, it is encoded locally before the message is sent to the server.
- The input still shows up as normal; typically encode via HTML entities, which are alternative representations of a character beginning with `&` but are displayed correctly
	- ![](Pasted%20image%2020260322013614.png)
	- But never treated as an element.

### Cross-site Request Forgery (XSRF)
- The browser attaches cookies to requests based on the destination domain. 
- When the browser makes a HTTP request to some arbitrary site, it doesn't matter how that request was triggered, the browser looks up all stored cookies for the arbitrary site and attaches the attributes for the cookie header in the HTTP request automatically.
- The server then receives that request with a valid session cookie and treats it as an authenticated request from the logged-in user, with no way of determining whether the user consciously sent the cookie.
- XSRF requires no code injection, no cookie theft, no password. They simply need the victim's browser to make a specific request to a target site while the victim has a valid session cookie. 
	- E.g., visit a site, and that site contains an element with an image component with a `src` attribute which contains a link to the authenticated host whos subdirectory is actually a malicious HTTP request that only an authenticated user on that site could make.
		![](Pasted%20image%2020260322021551.png)
- The attacker cannot see the result of this attack, and cannot exfiltrate any data.
	- Can only perform state-changing actions the victim is authorised to perform. 
	- Cannot read data, it is entirely an action attack. 

### XSRF in POST
- In the above, used GET request. 
- Recognising server-side that state-changing operations should use POST rather than GET is correct, but does little to prevent CSRF: ![](Pasted%20image%2020260322021832.png)
	- There is little restriction on which domain a form can submit to, the browser will happily submit it. 
	- Build the POST request as normal, with bank.com as the host, and the body as the name attribute with the corresponding value.
	- Auto-submits the form, the browser POSTs with the session cookie, with no user interaction required
	- Can even make the form invisible, execute payload, and then do brief redirect.

### Preventing XSRF
- Prevent XSS first. XSS attacks are a superset of XSRF attacks, as the payload can contain POST requests to mutate state which cannot see the result, so stopping XSS vulnerabilities amounts to preventing the most dangerous form of XSRF.
	![](Pasted%20image%2020260322032435.png)
- Prove that the request originated from a page that is implicitly trusted(which is why preventing XSS is so important)
	- The mechanism by which this is achieved:
		- The server generates a pseudorandom token, and associates it with the session(or generates one for each request), and embeds it in every form as a hidden field, and validates it on **every state-changing request.**
		- Only the client, on that web page, can attach the token to the body of a POST, PUT, DELETE, or PATCH request
			![](Pasted%20image%2020260322032632.png)
		- If there is no matching token, then the page the request comes from was not served by the server.
		- These are called **synchroniser tokens.** 
		- The strongest variant is a fresh one for each form submission, preventing replay attacks.