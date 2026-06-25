# Web Proxies Fundamentals

## What is a Proxy?

### Core Concept: Forward vs. Reverse Proxies

The term "proxy" gets thrown around a lot, but context matters.

- Forward Proxy: Acts on behalf of the client (you). When you use a corporate proxy to reach the internet, you are using a forward proxy.
- Reverse Proxy: Acts on behalf of the server. When Nginx sits in front of a Node.js app, it's a reverse proxy.

Key takeaway: A forward proxy acts on behalf of the client while a reverse proxy acts on behalf of the server.

### Technical Overview: Intercepting Proxies vs. DevTools

Field Fact: During a pentest, why is an intercepting proxy considered essential rather than just using browser DevTools? It captures all traffic including requests the browser hides from DevTools, such as silent AJAX calls and API requests. Furthermore, an intercepting proxy allows you to halt the request mid-flight, alter the raw bytes, and release it—something DevTools struggles with.

### Analytical Workflow: Intercepting HTTPS

If you set your system proxy to Burp Suite and visit https://google.com, you will immediately get a massive certificate warning. This is because the proxy is trying to present its own certificate for a domain it doesn't own.

Scenario: When testing through a proxy, what must you do first to intercept HTTPS traffic without certificate errors? Verdict: Install the proxy's CA certificate in your browser's trust store. Once your browser trusts the proxy's root CA, the proxy can seamlessly decrypt, inspect, and re-encrypt the traffic without throwing warnings.

## Burp Suite Basics

### Core Concept: Community vs Pro

Burp Suite comes in a few flavors. Community is free and fully functional for manual testing. But what is the main practical difference between Burp Suite Community and Pro for daily pentest work?

Answer: Pro includes an automated vulnerability scanner and fast Intruder without rate limiting. Community Edition deliberately throttles the Intruder tool, making large-scale brute force impractical.

### Technical Overview: Configuring Your Browser

You can route traffic to Burp by setting your proxy to 127.0.0.1:8080. However, you should never do this on your main, personal browser.

Analyst Trap - The Accidental Leak: Why should you use a dedicated Firefox profile instead of your regular browser when testing through Burp? To avoid leaking personal cookies, session data, and browsing history into the target engagement. Imagine testing a client's site while Burp silently captures and logs your background Gmail or social media requests. Keep your testing environment strictly isolated!

### Analytical Workflow: The Proxy Tab

The Proxy tab has multiple sub-tabs, but the two most important are Intercept and HTTP history.

Scenario: In Burp's Proxy tab, what is the difference between Intercept and HTTP history? Verdict: Intercept pauses individual requests in flight while HTTP history logs all traffic passively. Most of the time, you will leave Intercept off so your browsing is smooth, and review the traffic passively in the HTTP history tab. You only turn Intercept on when you want to catch and alter a specific request right before it leaves your machine.

## Intercepting Requests

### Core Concept: Intercept On vs. Off

Burp's Proxy tab allows you to toggle "Intercept is on" and "Intercept is off".

Field Fact: Why do experienced testers leave Intercept off during most of the testing workflow? Leaving Intercept on pauses every request and makes normal browsing slow and impractical. Instead, testers browse normally to log traffic passively in the HTTP History tab, and only toggle Intercept on when targeting a specific request.

### Technical Overview: Mass Assignment Vulnerability

Mass assignment happens when a web framework automatically binds client-side request parameters directly to database models without validation.

How to find it:
- Intercept a profile update request (POST /api/profile).
- Inject extra fields to the JSON body: e.g., "is_admin": true or "role": "admin".
- Forward the request and check if your permissions changed. Definition: Adding extra fields like is_admin to a request that the server processes and stores without validation.

### Analytical Workflow: Analyzing Session Tokens

Sometimes the vulnerability is not in the request, but in the server's response.

Analyst Trap - Client-side Session Handling: If you intercept a login response and see the session token decodes to user:admin in Base64, what does that indicate? The application stores user identity in a client-side cookie that can be decoded and modified. This represents a critical authentication bypass vulnerability, as any user can change the Base64 value to spoof another identity.

## Repeater & Decoder

### Core Concept: The Manual Testing Cycle

When manually testing an application, you should follow a structured sequence of capture, analysis, modification, and execution.

Field Fact: What is the correct order of the manual testing cycle using Repeater and Decoder? Capture the request in Repeater, decode interesting values in Decoder, modify, re-encode, and send. This ensures you understand what the client sends before you attempt payload variations.

### Technical Overview: Bypassing Filters via Double URL Encoding

Web application firewalls (WAFs) inspect incoming HTTP requests. If they see characters like ' (%27), they block the request.

How to bypass it:

- Double URL encode the payload. For example, encode % (%25) so that %27 becomes %2527.
- The WAF decodes %2527 to %27 and assumes it is safe since no active quote is executed.
- The backend application decodes %27 to ' and processes it. Definition: Encoding the percent signs themselves so the server decodes twice and the payload reaches the backend intact.

### Analytical Workflow: Repeater for Blind SQLi

When testing for blind vulnerabilities (where the web page does not print database errors), you must look for structural changes in the server's responses.

Analyst Trap - Blind Verification: When testing for blind SQL injection using Repeater, what is the key technique to confirm the vulnerability? Send requests with true and false conditions and compare the response differences. For example, sending AND 1=1 (true) versus AND 1=2 (false) and noting differences in response length or response time.

## Proxy Chaining & VPNs

### Core Concept: Proxy Chaining

Proxy chaining involves routing your traffic through multiple intermediate proxy servers before it reaches the final destination target.

Field Fact: Why is proxy chaining useful during a pentest when a single proxy is not enough? Each proxy in the chain only knows the previous hop, so the target cannot trace traffic back to you. This provides strong network anonymity and makes it harder for target defenders to find your real origin.

### Technical Overview: Tor Exit Node Security Risks

Tor (The Onion Router) is a popular proxy network. However, using Tor exit nodes for sensitive penetration tests introduces critical risks.

The Risk:

- The connection between the exit node and the target server is unencrypted (unless HTTPS is enforced).
- Exit node operators can capture, inspect, and modify the raw payload. Takeaway: Exit node operators can see all unencrypted traffic including HTTP requests and credentials. Never send sensitive client data over Tor without end-to-end encryption.   

### Analytical Workflow: Pre-engagement Verification

Before executing any active scans or exploits, you must verify your routing path to ensure your setup is functioning correctly.

Analyst Action Plan: What should you always do before starting a pentest session when using a proxy chain for anonymity? Test your anonymity setup by checking your visible IP against what you expect it to be. Open a browser routed through the proxy chain and visit an external service (like ifconfig.me or icanhazip.com) to confirm your real IP is fully hidden.