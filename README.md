# OWASP Juice Shop — Secure Login Form

**Course:** CSCE 477 — Risk Analysis  
**Author:** Caleb Mandapat  

## Overview

This project implements a login form inspired by OWASP Juice Shop's login page for a security design and exploitation assignment. The form includes client-side validation and simulated server-side authentication, but contains an intentional vulnerability that can be exploited via XSS to demonstrate real-world attack vectors.

## Project Structure

```
├── index.html    # Login form with validation + exploitable vulnerability
└── README.md
```

## How to Run

```bash
# Open directly in a browser
open index.html

# Or serve locally
python3 -m http.server 8080
```

## Security Features Implemented

| Feature | Implementation |
|---------|---------------|
| **Empty field prevention** | `validateEmail()` and `validatePassword()` reject blank inputs before submission |
| **Email format check** | Verifies input contains `@` and matches a basic RFC-style regex (`/^[^\s@]+@[^\s@]+\.[^\s@]+$/`) |
| **Password length enforcement** | Requires a minimum of 8 characters |
| **Simulated parameterized queries** | Auth uses a dictionary lookup instead of string-concatenated SQL |
| **Inline error messages** | Errors display beneath each field using safe `textContent` assignment |

## Known Vulnerability (Intentional)

The login result message is rendered using `innerHTML`, which reflects the user's email input directly into the DOM without sanitization. While the client-side regex validation blocks most XSS payloads through normal form interaction, an attacker can bypass client-side validation using the browser console:

### Exploit Steps

1. Open `index.html` in a browser
2. Open the browser console (F12 → Console)
3. Paste and run:
   ```javascript
   validateEmail = function() { return null; };
   document.getElementById('email').value = '<img src=x onerror=alert("XSS")>';
   document.getElementById('password').value = 'anything1';
   handleLogin();
   ```
4. The XSS payload executes because the attacker overrides `validateEmail()` to bypass client-side validation entirely, then the malicious email value gets injected into the DOM via `innerHTML`.

Alternatively, to bypass validation entirely:
```javascript
document.getElementById('login-result').innerHTML = '<img src=x onerror=alert("XSS")>';
document.getElementById('login-result').style.display = 'block';
```

### Fix

Replace all `innerHTML` usage with `textContent`, or sanitize input before DOM insertion:

```javascript
// BEFORE (vulnerable)
resultDiv.innerHTML = 'Invalid credentials for <strong>' + email + '</strong>';

// AFTER (secure)
resultDiv.textContent = 'Invalid credentials for ' + email + '. Please try again.';
```

## Technologies

- HTML5 / CSS3
- Vanilla JavaScript (no frameworks)
