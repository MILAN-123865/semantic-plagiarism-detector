# REST API Authentication Guide

All protected endpoints in the plagiarism detector engine require authentication via JSON Web Tokens (JWT). Use the guide below to authenticate your scripts and API clients.

## 1. Obtain an Access Token

Authenticate by sending a `POST` request with your credentials to the `/api/v1/auth/login` endpoint. This returns a session token.

### cURL Example
```bash
curl -X POST "http://localhost:8000/api/v1/auth/login" \
     -H "Content-Type: application/json" \
     -d '{
       "username": "your_username",
       "password": "your_password"
     }'
```

### Python Example
```python
import requests

url = "http://localhost:8000/api/v1/auth/login"
payload = {
    "username": "your_username",
    "password": "your_password"
}
headers = {
    "Content-Type": "application/json"
}

response = requests.post(url, json=payload, headers=headers)
response_data = response.json()

# Extract the access token
access_token = response_data.get("token")
print(f"Token obtained successfully: {access_token[:10]}...")
```

---

## 2. Access Protected Scan Endpoints

To interact with protected scanning tools (such as the document scan endpoint `/api/v1/scan`), include the retrieved token in your request's `Authorization` header prefixed with `Bearer `.

### cURL Example
```bash
curl -X POST "http://localhost:8000/api/v1/scan" \
     -H "Authorization: Bearer YOUR_ACCESS_TOKEN_HERE" \
     -F "file=@/path/to/document.pdf"
```

### Python Example
```python
import requests

url = "http://localhost:8000/api/v1/scan"
headers = {
    "Authorization": f"Bearer {access_token}"
}
files = {
    "file": ("document.pdf", open("document.pdf", "rb"), "application/pdf")
}

response = requests.post(url, headers=headers, files=files)

if response.status_code == 200:
    print("Scan initiated successfully:")
    print(response.json())
elif response.status_code == 401:
    print("Authentication failed. Token might be invalid or expired.")
else:
    print(f"Failed to start scan: {response.status_code}")
```

---

## Testing with Swagger UI

When developing and auditing endpoints locally, you can use the interactive Swagger UI panel hosted at [http://localhost:8000/docs](http://localhost:8000/docs) to fire live requests against your workspace.

Secured endpoints require a valid JSON Web Token (JWT) Bearer token to authorize access. Follow these steps to authenticate your browser session:

### 🔐 How to Authorize Your Session

1. **Open the Documentation Core**: Navigate to [http://localhost:8000/docs](http://localhost:8000/docs) in your browser.
2. **Locate the Security Action Hook**: Click the lock icon button labeled **"Authorize"** positioned at the top right header section of the page.
3. **Inject the Authorization Token**: 
   * In the modal popup window, locate the text input field labeled **Value**.
   * Enter your token using the exact format: `Bearer <your_jwt_token_here>`
   * *Example*: `Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`
4. **Lock the Session Configuration**: Click the **Authorize** button within the modal window, then click **Close**.

Now, all subsequent interactive endpoint requests dispatched via the UI will automatically append the correct tracking header (`Authorization: Bearer <token>`) to your API request parameters.

### 🧪 Triggering an Interactive Request

* Expand any locked API route container (indicated by a closed lock icon).
* Click the **"Try it out"** button in the top right of the route container.
* Populate any required query parameters or JSON body payloads.
* Press the blue **"Execute"** button to fire the network request and review the server's response.
