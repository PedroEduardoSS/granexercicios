# Introduction to OSINT

## What is OSINT?

### Core Concept: Passive vs. Semi-Active OSINT

OSINT collection strategies are classified by the level of risk and interaction with the target systems.

Field Fact: What distinguishes passive OSINT from semi-active OSINT? Passive OSINT involves no interaction with the target system while semi-active involves some anonymous interaction. For example, querying Shodan for a server is passive; sending an HTTP request directly to the target's server to inspect its public security headers is semi-active.

### Technical Overview: Legal Authorization

Even though OSINT utilizes public resources, collecting intelligence systematically on individuals or corporate entities carries legal risk.

Why authorization matters:

- Stalking, harassment, or unauthorized surveillance laws can apply to aggressive intelligence collection.
- Takeaway: Written authorization is important before conducting OSINT because it protects you legally from potential surveillance or stalking claims even when accessing public data. Never start an assessment without an official Scope of Work (SOW).

### Analytical Workflow: Case Study - MH17

The capabilities of modern OSINT researchers were proven to the world during the investigation of Malaysia Airlines Flight 17 in 2014.

Analyst Trap - The Bellingcat Case Study: What made the Bellingcat MH17 investigation groundbreaking for OSINT? They identified a specific military unit using only publicly available photos, videos, and social media posts. By geolocating dashcam footage, analyzing tank transporter route photos, and identifying soldiers' selfies on social networks, researchers mapped the exact missile unit path without official intelligence leaks.

## The Intelligence Cycle

### Core Concept: Confirmation Bias in Collection

The intelligence cycle consists of: Planning and Direction, Collection, Processing, Analysis, and Dissemination.

Field Fact: Why should you avoid analyzing data during the collection phase of the intelligence cycle? Analyzing during collection leads to confirmation bias where you unconsciously support your initial theories. If you form conclusions during collection, you will naturally look for data that supports your theory while ignoring key evidence that contradicts it. Keep collection and analysis strictly separate!

### Technical Overview: The Processing Phase

Raw collected data (social media dumps, domain registry exports, raw text files) is unstructured and chaotic.

Takeaway: The primary purpose of the processing phase in the intelligence cycle is converting raw collected data into organized, deduplicated, and indexed datasets ready for analysis. It bridges the gap between raw data collection and analytical evaluation.

### Analytical Workflow: An Iterative Cycle

Security investigations are rarely linear processes with a clean beginning and end.

Analyst Trap - The Infinite Loop: Why is the intelligence cycle considered iterative rather than linear? Because dissemination generates new questions that require another round of collection and analysis. When stakeholders receive your intelligence report, they will invariably raise new queries and requests that trigger the cycle all over again.

## OPSEC & Sock Puppets

### Core Concept: Browser Fingerprinting

Hiding your IP address is no longer sufficient to remain anonymous online.

Field Fact: What is browser fingerprinting and why is it a concern for OPSEC? It is a unique combination of browser and system attributes that can identify you even without cookies or IP address. Details like screen resolution, installed fonts, user-agent, and system language settings combine to create a highly unique footprint that trackers use to trace you across sessions.

### Technical Overview: Sock Puppet Lifespans

A sock puppet is a fake online persona created to interact anonymously with targets.

The Golden Rule:

- Pre-aging: Create sock puppet accounts months before they are needed.
- Takeaway: Sock puppet social media accounts should be created months before they are needed because accounts with no posting history and immediate target interaction raise red flags to experienced targets. An empty profile that immediately starts messaging a hacker forum target looks like an obvious investigator.

### Analytical Workflow: VPN + Tor Chaining

For maximum network anonymity, advanced OSINT investigators chain connection protocols.

Analyst Action Plan: What is the advantage of routing traffic through VPN then Tor instead of Tor alone? The VPN hides your real IP from the Tor entry relay while the exit node IP is what the target sees. This guarantees your ISP only sees a VPN connection, the Tor entry relay only sees your VPN's IP, and the target only sees the public Tor exit node IP.

## Google Dorking Mastery

### Core Concept: The site: Operator

Google dorking, or Google hacking, uses advanced search operators to filter index results.

Field Fact: What does the site: operator do in Google dorking? It restricts search results to pages from a specific domain. For example, site:example.com will only return pages matching that exact hostname.

### Technical Overview: The Google Hacking Database (GHDB)

You don't need to reinvent the wheel. Exploit-DB maintains a massive repository of pre-written dork queries.

Takeaway: The Google Hacking Database is useful for OSINT practitioners because it provides a curated collection of proven dork patterns that can be adapted to specific targets. These dorks target exposed backup files, configuration keys, directory indexes, and vulnerable platforms.

### Analytical Workflow: Structured Dorking Workflow

To search effectively without getting overwhelmed by millions of Google hits, you must proceed in a structured sequence.

Analyst Action Plan: What is the recommended workflow for effective Google dorking? Start broad with site operator, add filetype operators, then narrow with intitle or inurl, and finally add intext operators. For example:
- site:example.com
- site:example.com filetype:xls
- site:example.com filetype:xls inurl:budget
- site:example.com filetype:xls inurl:budget intext:"confidential"

## People & Username Recon

### Core Concept: Username Reuse

A single handle chosen by a target during high school can follow them throughout their professional career.

Field Fact: Why is a single username such a powerful starting point for people identification? People reuse usernames across many platforms, and each platform reveals different types of personal information. A target might use the same handle on a gaming forum (revealing hobbies), GitHub (revealing code and real name), and LinkedIn (revealing professional employment).

### Technical Overview: Wayback Machine Investigations

Targets often delete accounts or sanitize their personal information when they realize they are being monitored.

Takeaway: The Wayback Machine is valuable for OSINT investigations because it contains historical snapshots of deleted or modified web content that no longer exists on live sites. You can see the target's profile page as it existed 5 years ago, when they were less security-conscious.

## Analytical Workflow: The Pivot Technique

Reconnaissance is a branching tree. You never stop at a single finding; you use every clue to pivot to another platform.

Analyst Action Plan - The Pivot: What is the pivot technique in username recon? Using each new piece of information like email or real name to search additional platforms and expand the investigation. For example: Username leads to a GitHub repository commit history -> Commit reveals real name and email address -> Real name leads to LinkedIn profile -> Email address leads to breach database records -> Breach records reveal additional linked handles and passwords.

## Email & Breach Data

### Core Concept: Email as a Pivot Point

Email addresses are unique, persistent, and rarely changed by users.

Field Fact: Why is an email address considered the most valuable pivot point in OSINT? Email addresses are unique persistent identifiers that connect a person to accounts across many platforms. Unlike temporary handles or IP addresses, an email address consistently maps a person's digital identity over years.

### Technical Overview: Have I Been Pwned vs. DeHashed

Breach databases are critical for identifying historical leaks and credentials.

- Have I Been Pwned (HIBP): Checks if an email has been leaked in a known breach and reports which platforms were affected.
- DeHashed: A searchable breach archive. Takeaway: DeHashed shows the actual exposed data from breaches while HIBP only shows which breaches an email appeared in. With DeHashed, you can find the actual leaked cleartext passwords, IP addresses, or home addresses.

### Analytical Workflow: Corporate Pattern Harvesting

When conducting OSINT against an enterprise, you need to know how they construct corporate email accounts.

Analyst Action Plan: What is the value of using Hunter.io during a corporate OSINT investigation? It reveals the email naming pattern used by an organization, which helps find other employee email addresses. If Hunter.io reveals the pattern is {first}.{last}@company.com, you can immediately predict and verify the email addresses of the entire executive board.

## Image Intelligence (IMINT)

### Core Concept: EXIF Metadata Survival

Exchangeable Image File Format (EXIF) metadata stores camera details, timestamps, and GPS coordinates inside image files.

Field Fact: Why are photos from direct messages or email attachments more valuable for IMINT than photos from social media? Social media platforms strip EXIF metadata while direct messages and email attachments usually preserve it. When a target uploads a photo to Facebook, the metadata is automatically cleaned. If they email it to you or send it via message attachments, the raw GPS coordinates are often intact.

### Technical Overview: Geolocating Without Metadata

When EXIF data is stripped, you must rely entirely on visual indicators within the image.

Key Visual Cues:

- Local Infrastructure: Street signs, license plates, public transport markings.
- Environment: Architectural styles, local vegetation, sun angle, and language on billboards. Takeaway: Street signs, license plates, building styles, vegetation, sun angle, and language on advertisements are visual cues that can help geolocate a photo when EXIF data has been stripped.

### Analytical Workflow: Video Intelligence

Videos are simply sequences of images. Analyzing a raw video requires extracting and inspecting individual frames.

Analyst Trap - The Hidden Detail: What is the value of extracting individual frames from a video for IMINT analysis? Frame-by-frame analysis reveals background details, moving objects, and metadata not visible in the full video. A single frame might catch a passing reflective truck, a street address label, or a document sitting on a desk.

## Domain & Infrastructure

### Core Concept: Subdomain Discovery via CT Logs

Certificate Transparency (CT) logs are public ledgers that record every SSL/TLS certificate issued by Certificate Authorities (CAs).

Field Fact: Why are Certificate Transparency logs valuable for subdomain discovery? They contain publicly searchable records of all SSL certificates which include the domains and subdomains covered. By querying logs (using tools like crt.sh), you can discover subdomains (like staging.company.com) without sending a single packet to the target.

### Technical Overview: Active vs. Passive Discovery

Mapping out subdomains requires balancing speed against stealth.

- Passive Subdomain Enumeration: Uses third-party databases, search engines, and CT logs. Passive uses third-party sources without contacting the target.
- Active Subdomain Enumeration: Queries the target's DNS servers directly (e.g., DNS brute-forcing). Active queries the target DNS servers directly, which leaves logs.

### Analytical Workflow: Analyzing DNS TXT Records

DNS TXT records contain descriptive text, but they serve critical security functions for email validation.

Analyst Action Plan: What information do TXT DNS records typically reveal about a domain? They contain SPF and DMARC records which reveal which services are authorized to send email. Inspecting these records shows which third-party marketing, support, or security services the target routes their email through.

## Geolocation & Maps

### Core Concept: Convergent Evidence

Never rely on a single geolocation data point. GPS coordinates can be modified, and satellite images can be outdated.

Field Fact: Why is combining multiple geolocation methods more reliable than using a single method? No single method is foolproof but when multiple independent methods converge on the same location confidence increases dramatically.

### Technical Overview: Solar Telemetry with SunCalc

SunCalc calculates the solar position, altitude, and shadows for any geographic coordinate at any date and time.

How to verify photos:

- Note the direction and length of shadows in the image.
- Match it with the sun position calculated by SunCalc.
- Takeaway: It calculates the sun position for any location and time, allowing you to verify that shadow directions match the claimed origin. If the photo claims to be in London at noon but the shadows point north, the photo is fake.

### Analytical Workflow: Mapped WiFi Access Points (Wiggle.net)

Wigle.net is a global repository of wireless networks mapped by coordinates.

Analyst Trap - The BSSID Lock: What makes Wigle.net valuable for geolocating mobile devices? It maps WiFi access point locations, and connecting to a specific WiFi network places a device in approximate physical proximity. If you capture a device log showing connection to a specific BSSID, you can map that BSSID on Wigle.net to find the exact building the device visited.

