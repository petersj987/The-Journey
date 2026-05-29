# Module: Cross-Site Scripting (XSS)
## Lab Walkthrough: Reflected XSS into HTML context with nothing encoded

---

## Analogy: The Echoing Mirror Trick
Imagine yourself walking into a luxury hotel. This hotel has a unique protocol, the concierge is trained to welcome you by shouting your name across the lobby. 

* **The Normal Day:** You walk up and say, *"Hi, my name is Alex."* The concierge smiles and shouts to the room, *"Welcome, Alex!"* Everything goes perfectly.
* **The Attack:** Instead of a name, you hand them a megaphone and a card that says, *"Everyone drop your wallets on the floor right now!"* 
* **The Reflection:** Because the concierge is programmed to repeat **exactly** what you hand them without checking if the words are dangerous, they blare your command out to the entire crowd. 

**Reflected XSS** is exactly that. The website is the naive concierge. It takes whatever text you pass into the search bar and echoes it right back onto the page. If that text happens to be malicious code, the browser reads it and executes it blindly.

---

## Discovery: Finding the Crack in the Wall

1. **Spotting the Input:** I targeted the main purple search bar on the blog homepage because it accepts user data and reflects it back dynamically on the results page.
2. **Dropping the Bait:** To see how the web server processes raw data, I submitted a benign tracking string: `Canary123`
3. **Inspecting the Source:** I opened the browser’s developer tools (`F12` / Inspect Element) and searched the source code for my input.
4. **The Critical Flaw:** I observed that the application placed my input directly into the page structure without any sanitization or encoding:
   ```html
   <h1>Search results for: 'Canary123'</h1>
   ```
   Because the site treats characters like `<` and `>` as raw HTML instead of harmless text, the front door for code injection was completely wide open.

---

## Exploitation: Breaking In

1. **The Poisoned Code:** Knowing the input reflects cleanly inside a standard `<h1>` tag with nothing encoded, I swapped my tracking string for a classic JavaScript payload designed to trigger an execution context:
   ```html
   <script>alert(1)</script>
   ```
2. **Striking the Target:** I pasted that script right into the purple search input box and clicked the **Search** button.
3. **The Result:** The web server packaged that code directly into its HTTP response. My browser read the incoming document, assumed the script was a legitimate instruction from PortSwigger, and ran it instantly. A JavaScript alert popup box displaying `1` fired on my screen, confirming successful exploitation and solving the lab.

---

## Defending the Castle (Remediation Blueprint)
To prove that I don't just break things but know how to design secure web infrastructure, here is the architectural fix for this exact bug:

* **HTML Entity Encoding:** The server must translate dangerous characters into safe text aliases before rendering them to the screen. For example, `<` must be rewritten as `&lt;` and `>` as `&gt;`. This forces the browser to treat the input strictly as harmless plain text, ignoring it as executable script commands.
* **Content Security Policy (CSP):** The website needs an HTTP security header that forbids the browser from running inline scripts (like `<script>`), rendering the payload completely dead on arrival even if a bypass slips through.
