# Module: Cross-Site Scripting (XSS)
## Lab Walkthrough: Stored XSS into HTML context with nothing encoded

---

## Analogy: The Poisoned Water Supply
Think of Reflected XSS like a splash of dirty water thrown directly at one specific person. But **Stored XSS** is a completely different beast—it is like slipping a slow-dissolving poison directly into the town's central water tower.

* **Setting the Trap:** The attacker doesn't target a specific victim. Instead, they walk up to the public comment section (the water tower) and drop a malicious script inside. 
* **The Storage:** The web application accepts the script and saves it permanently in its database. The poison is now mixed into the system.
* **The Outbreak:** The attacker leaves. Days later, hundreds of innocent users open that specific blog post to read it. By opening the page, they are "drinking from the tower." The website pulls the malicious code out of the database and serves it to every single visitor, executing the exploit automatically in their browsers.

With Stored XSS, you attack the platform itself to automatically compromise any user who visits it later.

---

## Discovery: Finding the Permanent Storage Point

1. **Targeting the Feature:** I navigated into a blog post and located the comment section functionality, which accepts a comment string, a name, an email, and a website.
2. **Dropping the Tracking Canary:** To test if the website saves input without cleaning it, I submitted a comment containing a unique test string: `StoredCanary999`
3. **Inspecting the Layout:** I refreshed the webpage and opened the Browser Developer Tools (`F12` / Inspect Element) to see exactly where my comment was sitting.
4. **The Critical Flaw:** I found my text permanently embedded directly inside the page source code within the comment display area:
   ```html
   <p>StoredCanary999</p>
   ```
   Because the backend database accepted my input exactly as written and rendered it raw to the page, the injection window was fully confirmed.

---

## Exploitation: Poisoning the Well

1. **Crafting the Persistent Payload:** Knowing the comment section reflects data cleanly inside the HTML structure with zero character encoding, I swapped my test string for a classic execution script:
   ```html
   <script>alert(1)</script>
   ```
2. **Executing the Request:** I filled out the required name and email fields, pasted the script into the comment text area, and clicked **Post comment**.
3. **The Result:** The web server processed the input and stored it permanently in its database. Upon navigating back to the blog post, the page fetched the comment history. My browser read the stored `<script>` tags as native commands instead of harmless text, instantly firing a JavaScript alert box popup displaying `1`. The lab was successfully solved.

---

## Defending the Castle (Remediation Blueprint)
Because Stored XSS is incredibly dangerous and impacts multiple users passively, I recommend implementing these core architectural defenses:

* **Context-Aware Output Encoding:** The server must safely encode all raw data pulled from the database before inserting it into the webpage HTML. Characters like `<` must display safely as `&lt;` and `>` as `&gt;` so the browser prints the code as text instead of running it.
* **Input Sanitization:** Run all incoming comment data through a trusted server-side HTML sanitization library (like DOMPurify) to automatically strip out dangerous HTML tags before they ever get saved to the database.
