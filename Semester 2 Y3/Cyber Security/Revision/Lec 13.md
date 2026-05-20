## Internet threat models
- Different to other threat models; operate under different constraints than network-level attacks (except DoS DNS and some other cases)
- Attacker has no control over network, cannot passively sniff traffic, decrypt it, or inject packets
- **Attack surface = web application layer**, that is, interaction between the **browser** and the **server** over HTTP
	- The browser is the attacker's tool whilst also being the victim's environment
	- Attacker's goal is to get browser to execute code and make requests it shouldn't abusing trust relationships browser maintains with legit servers. 

### HTTP and Cookies
- HTTP is a **stateless request-response** protocol - each transaction is entirely independent
	- From perspective of protocol on server end, every req, comes from, anonymous client
- Problem, everything users do on web, typically stateful
- Need mechanism, associate HTTP requests, with user session, this is called a **cookie**
- When server wants to establish state with a client, includes a `Set-Cookie` header inside HTTP response
	- Browser stores KVPairs returned and associates them with domain that set them
- On every subsequent request, automatically include cookie in the `Cookie` header in request. 
- Server will read `sessionid` value, look up in session store, and identify which authenticated user the request belongs to.
	- Stateful sessions have attributes stored server side e.g., role
- Thus, whoever presents cookie header with session id and other attributes, is treated as an authenticated user. 


### Types of Cookies
- **Session**
	- Have no expiration/max age attribute
	- Discarded when session ends, usually when browser closed.
		- May be killed server side, just not stored on cookie header
	- Used for short-lived auth
- **Persistent**
	- Includes expiration/max age, specifying when should be deleted, acquire new cookie
	- Survives restarts though, the keep me logged in checkbox typically sets persistent cookie with long expiry.
- **Secure**
	- Include a `Secure` flag, meaning browser only transmit cookie over HTTPS connection to prevent hijack by network-lvl eavesdropper
- **HTTPOnly**
	- Security attribute on cookies that controls who can read the cookie
	- Means that JS can't read/modify cookie, which it can under normal circumstances


### Third-party cookies and tracking
- Cookies are scoped to the domain that set them
- Website can, often does, embed resources from other domains
- When browser load site, include resource with src attribute pointing to another domain, browser makes HTTP request, resource provider then sets cookie on browser in header of response
- Next time visit any site containing resources from that provider, browser resends cookie
	- Cross-site tracking - know visited both

### Cookie Vulnerabilities
- A **session cookie** is a **bearer token** - if an attacker obtains it as mentioned prior, can present to server in header of HTTP request to authenticate to it
- No cryptography needs to be broken
- Plain HTTP, cookies transmitted plaintext
	- Network-level attacker, evil twin AP, someone on same network, compromised router etc, can passively read every cookie
	- This is why the `Secure` flag matters, and why HTTPs is non-negotiable for authenticated sessions
- HTTPs prevents network-level interception of cookies, but does not stop the injection attack discussed below:
	- XSS, exfiltrate cookies via JS
	- DNS poisoning, which will result in cookies being sent to attacker IP instead.
		- DNNSEC mitigates

## Cross-Site Scripting
- General term for **Injection attack**
	- Inject malicious code into content delivered to victim's browser
- Injection = HTML markup, `script` tags; when browser receives HTTP response containing HTML, parses into DOM:
	- As it builds DOM, executes JS encountered
	- Browser can't distinguish between JS written by web dev and JS written by an attacker
- Injected script runs in origin of vulnerable site and so has access to that site's cookies, provided they are not `HTTPOnly`
	- Can install keylogger, access local+session storage, rewrite DOM, make authenticated requests via use of the session cookie
- Exfiltrate by constructing an image component in injected JS that points to attacker controlled machine, attach cookie/data in GET request to 'load' the image

## Reflected XSS
- Reflected XSS is a vulnerability that occurs when supplied input is immediately included in the HTTP response without sanitation
- 404 page that reflects requested URL back to user via HTTP response
- If can get user to click on fake link, that has script tag in it, with payload, and ability to exfiltrate or make request, then will make HTTP request to server, will be 'reflected' back to the user, and the JS will be executed as the browser parses the DOM. 
- Requires user interaction, user must click the crafted link.

## Persistent/Stored XSS
- Reflected XSS requires victims to click crafted link
- **Stored XSS** is more dangerous; this is not required
- The payload is **stored on the server** and served to every user
- Any feature that accepts user input and displays it to users, with minimal sanitation, is a potential vector:
	- Comment sections, product reviews, chat messages
	- If the application stores the raw input, an attacker can post a comment containing a script tag
	- Every visitor then loads page, making http request, returned in response, parsed, and executed. 


### Preventing XSS
- Instinctive fix is to search user input at application and request layer, for dangerous strings 
	- 'Blocklist sanitisation' fails, browsers are permissive HTML parsers, and attackers have hundreds of ways to achieve script execution without triggering obvious patterns
	- It is not possible to enumerate all possible representations of dangerous input.
- The correct approach is **output encoding.**
	- Encode HTML elements submitted by users so browsers can distinguish between developer markup and user-supplied content
	- Malicious input is encoded then so it never executes
		- This encoding may be at diff levels e.g., if it appears in URL, then encoded on server before response returned; if appears in comment, then encoded locally before messafe is sent to server
	- The input still shows up as normal, just not treated as a real element. 


### Cross-site Request Forgery (XSRF)
- Browser attaches cookies to reqs, based on destination domain
- When browser makes HTTP request to a site, it looks up stored cookies for that site, attaches attributes for cookie header automatically
- Server receives that request with valid session cookie, treats as authenticated request, **no way of determining whether the user conciously sent the cookie.**
- XSRF requires no code injection, no cookie theft, no password. 
	- Just need victim to make specific request to target site while victim has valid session cookie
	- E.g., visit fake site, site contains element with an image component and src attribute
	- src attribute contains link to authenticated host with malicious HTTP request, that only authenticated user could make
		![](Pasted%20image%2020260322021551.png)
- Attacker cannot see result of this attack, cannot exfiltrate data


### XSRF in POST
- Above used GET request
- Recognising server-side that state-changing opeations should use POST rather than GET is correct, doesn't do much to prevent XSRF
- Usually host hidden form; no restrict on which domain a form can submit to
	- Build POST request as normal, auto submit form, browser POSTs with the session cookie, making the malicious request to the legitimate site
	- Can even make the form invisible, execute payload, and then
	do brief redirect.


### Preventing XSRF
- Prevent XSS first. XSS attacks are a superset of XSRF attacks, as the payload of XSS attack could be a request forgery/POST request to legit site triggered immediately, so stopping XSRF amounts to also stopping XSS
- Prove that request originated from a page that is implicitly trusted:
	- Mechanism:
		- Generate pseudorandom token, associate with session (or gen one for each request), embed in every form as hidden field.
			- Validate on every req
		- Reqs without this are treated as invalid; and only the true client can attach them to requests
		- These are called **synchroniser** tokens
			- Strongest = new one for every req, prevents replay attacks.