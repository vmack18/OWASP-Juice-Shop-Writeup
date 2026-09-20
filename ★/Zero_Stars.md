## OWASP Juice Shop Write-up: Zero Stars (Using Burp Suite)

### 📌 Challenge Overview

### 📊 Details

* **Challenge Title:** Zero Stars
* **Category:** Improper Input Validation
* **Difficulty:** ⭐ (1/6 - Beginner)
* **Objective:** Give a devastating zero-star feedback to the store.

---

### ❓ Why This Matters

When web applications rely entirely on client-side restrictions (such as graphical sliders or form limitations) instead of robust backend validation, users or attackers can easily bypass the user interface. By interacting directly with backend APIs using tools like Burp Suite, malicious payloads or out-of-range inputs can be submitted undetected, leading to **Improper Input Validation** vulnerabilities.

---

### 🛠️ Tools Used

* **Burp Suite:** An interception proxy used to capture, modify, and replay HTTP/HTTPS requests.
* **Standard Web Browser:** Google Chrome, Mozilla Firefox, or Microsoft Edge (configured to route traffic through Burp Suite).

---

### 🚀 Methodology and Solution

Because the customer feedback frontend restricts the star rating slider from 1 to 5, we can use Burp Suite to intercept the submission request and change the rating value to `0`.

#### Step 1: Configure Burp Suite and Browser Proxy

1. Open **Burp Suite** and start a temporary project.
2. Navigate to the **Proxy** tab and ensure **Intercept** is turned **on**.
3. Configure your web browser (or Burp's built-in browser) to route traffic through the local proxy proxy listener (typically `127.0.0.1:8080`).

#### Step 2: Capture the Feedback Request

1. Navigate to your local OWASP Juice Shop instance (`http://localhost:3000`).
2. Open the sidebar menu and select **Customer Feedback**.
3. Type your comment (e.g., `"Devastating zero-star feedback!"`).
4. Set the rating slider to the lowest allowed limit (**1 star**), complete the captcha, and click **Submit**.
5. Your browser will hang as Burp Suite intercepts the outgoing HTTP `POST` request to `/api/Feedbacks/`.

#### Step 3: Modify the Payload in Burp Suite (The Solution)

1. Switch back to your **Burp Suite** window and go to the **Proxy > Intercept** tab where the intercepted request is paused.
2. Locate the JSON body payload at the bottom of the request window, which looks similar to this:

```json
{
  "comment": "Devastating zero-star feedback!",
  "rating": 1,
  "captchaId": 1,
  "captcha": "..."
}

```

3. Change `"rating": 1` to **`"rating": 0`**:

```json
{
  "comment": "Devastating zero-star feedback!",
  "rating": 0,
  "captchaId": 1,
  "captcha": "..."
}

```

4. Click the **Forward** button in Burp Suite to release the modified request to the server.
5. Once the server accepts the out-of-range value, the challenge success notification will immediately appear on your browser screen!

---

## Screenshot

### 🧠 Technical Explanation

The front-end user interface restricted star inputs using slider constraints (1–5), but the underlying server-side API endpoint (`/api/Feedbacks/`) lacked proper boundary checks.

When the modified request carrying `rating: 0` arrived at the server via Burp Suite interception, the application logic directly committed the numerical value to the database without validating whether it fell within the expected business logic limits. This highlights why client-side validation provides zero security guarantees against direct API tampering.

---

### 🛡️ Remediation and Best Practices

To prevent improper input validation vulnerabilities, ensure that data constraints are enforced at every layer of the application stack:

* 🛡️ **Enforce Server-Side Validation:** Never rely on front-end UI limits (like HTML sliders, form disabled attributes, or max/min attributes) to secure backend endpoints. Always validate and sanitize all incoming parameters on the server.
* 🚦 **Implement Strict Boundary Checks:** Validate data types, string lengths, and numerical ranges explicitly in your controller or input validation middleware (e.g., ensuring ratings strictly fall between 1 and 5).
* 🔒 **Reject Malformed Payloads:** Automatically reject requests containing unexpected values or out-of-bounds parameters with an appropriate `400 Bad Request` HTTP status response.
