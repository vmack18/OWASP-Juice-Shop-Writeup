## OWASP Juice Shop Write-up: Empty User Registration (Using Burp Suite)

### 📌 Challenge Overview

### 📊 Details

* **Challenge Title:** Empty User Registration
* **Category:** Improper Input Validation
* **Difficulty:** ⭐ (1/6 - Beginner)
* **Objective:** Register a user with empty credentials or bypass client-side form restrictions to submit an empty registration payload.

---

### ❓ Why This Matters

When designing web applications, developers often use frontend validation (such as HTML `required` attributes, disabled submit buttons, or input length checks) to prevent users from submitting incomplete forms.

However, relying solely on client-side controls is a security anti-pattern. Because client-side code runs locally on the user's browser, it can easily be bypassed, modified, or ignored entirely. If the backend API fails to perform proper **Input Validation**, attackers can submit empty, malformed, or unauthorized data payloads directly to server endpoints, potentially leading to database pollution, broken logic flows, or unauthorized account creation.

---

### 🛠️ Tools Used

* **Burp Suite:** An interception proxy used to capture, modify, and replay HTTP/HTTPS requests.
* **Standard Web Browser:** Google Chrome, Mozilla Firefox, or Microsoft Edge (configured to route traffic through Burp Suite).

---

### 🚀 Methodology and Solution

The registration form in OWASP Juice Shop enforces client-side rules that prevent you from clicking the **Register** button if the email, password, or security question fields are left blank. We can use Burp Suite to intercept a valid registration request and clear out the required parameters.

#### Step 1: Configure Burp Suite and Browser Proxy

1. Open **Burp Suite** and ensure **Intercept** is turned **on** under the **Proxy > Intercept** tab.
2. Ensure your browser is configured to route web traffic through Burp Suite's local proxy listener (`127.0.0.1:8080`).

#### Step 2: Capture a Registration Request

1. Navigate to your local OWASP Juice Shop instance (`http://localhost:3000/#/register`).
2. Type dummy values into the fields (e.g., Email: `test@test.com`, Password: `Password123`, select a security question, and provide an answer) so that the frontend validation permits you to click **Register**.
3. Click the **Register** button.
4. Your browser will pause as Burp Suite intercepts the outgoing HTTP `POST` request to `/api/Users/`.

#### Step 3: Modify and Empty the Payload in Burp Suite (The Solution)

1. Switch to your **Burp Suite** window and view the intercepted request.
2. Locate the JSON payload body at the bottom of the request window, which looks similar to this:

```json
{
  "email": "test@test.com",
  "password": "Password123",
  "passwordRepeat": "Password123",
  "securityQuestion": "1",
  "securityAnswer": "answer"
}

```

3. Edit the JSON body to clear out the required fields (leaving them empty strings or removing values as needed):

```json
{
  "email": "",
  "password": "",
  "passwordRepeat": "",
  "securityQuestion": null,
  "securityAnswer": ""
}

```

4. Click the **Forward** button in Burp Suite to send the modified request to the server.
5. Once the server accepts the empty registration payload, the challenge success notification will immediately pop up in your browser!

---

## Screenshot

### 🧠 Technical Explanation

The front-end user interface used form validation rules and disabled the register button until text was entered. However, the underlying backend API endpoint (`/api/Users/`) did not enforce corresponding server-side validation rules requiring non-empty fields for user creation.

By capturing the request in Burp Suite and stripping out the field values, the backend blindly processed the empty payload, successfully registering a blank user account and exposing the improper input validation flaw.

---

### 🛡️ Remediation and Best Practices

To prevent improper input validation and secure API endpoints against bypass techniques:

* 🛡️ **Enforce Strict Server-Side Validation:** Never rely on frontend form validation (`required` tags, disabled buttons) alone. Always validate data types, formats, and emptiness on the backend API server.
* 🚦 **Reject Incomplete Payloads:** Implement validation checks (such as express-validator or schema validators) that immediately reject requests with empty or missing mandatory fields with a `400 Bad Request` status code.
* 🔒 **Sanitize Database Inputs:** Ensure business logic constraints are securely enforced at the database and application controller layers simultaneously.
