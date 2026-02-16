# Race Condition Vulnerabilities – Practical Guide

This repository documents common **Race Condition vulnerabilities** encountered during Web Application Security Testing (VAPT) and Bug Bounty assessments.

Race conditions occur when an application fails to properly handle multiple concurrent requests, allowing unintended behavior such as bypassing limits, duplicating transactions, or escalating privileges.

---

## ⚠️ Disclaimer

This guide is intended strictly for **educational and authorized security testing purposes**.  
Test only applications you own or have explicit permission to assess.  
The author is not responsible for misuse.

---

# 🧠 What is a Race Condition?

A race condition occurs when:

- Multiple requests are processed simultaneously
- The application does not properly synchronize state updates
- Business logic checks are performed before state changes are committed

This allows attackers to exploit timing issues in the backend.

---

# 1️⃣ Limit Overrun Race Condition

### Scenario:
A coupon, balance, or discount is supposed to be usable only once.

### Steps:

1. Capture the request while applying a coupon.
2. Send the request to **Repeater**.
3. Duplicate the same request multiple times.
4. Group them together.
5. Send the group in parallel using **Single Packet Attack**.

### Result:
The coupon or discount may be applied multiple times before the backend updates the usage state.

---

# 2️⃣ Bypassing Rate Limit Using Race Condition

### Tool Required:
- Burp Suite
- Turbo Intruder Extension

---

## Steps:

1. Install **Turbo Intruder** from BApp Store.
2. Identify the parameter you want to brute force (e.g., password, OTP).
3. Send the request to **Turbo Intruder**.
4. Select **Single Packet Attack** option.

---

## Modify the Default Script

Replace:

```python
for i in range(20):
    engine.queue(target.req, gate='race1')
```

With:

```python
for word in wordlists.clipboard:
    engine.queue(target.req, word, gate='race1')
```

### Important:
- Ensure your wordlist is copied to the clipboard.
- Turbo Intruder will send all requests in a single TCP packet.

---

### Result:
If rate limiting is poorly implemented, multiple attempts may bypass restrictions and reveal the correct password.

---

# 3️⃣ Multi-Endpoint Race Condition

### Scenario:
You want to purchase a ₹1000 product but only have ₹100 credit.

The application should prevent checkout due to insufficient balance.

---

## Steps:

1. Capture request for:
   - Adding small product to cart
   - Adding big product to cart
   - Checkout request

2. Send all requests to **Repeater**.
3. Group:
   - Big product request
   - Checkout request

4. First, send the small product add-to-cart request.
5. Refresh the page and confirm product added.
6. Now send grouped requests in parallel (Single Packet Attack).

---

### Result:
Due to delayed balance validation, checkout of the big product may succeed despite insufficient credit.

---

# 4️⃣ Single Endpoint Race Condition

### Scenario:
Update Email Functionality

1. Enter new email.
2. System sends confirmation link to that email.

---

## Steps:

1. Capture the update email request.
2. Send it to Repeater.
3. Duplicate the same request.
4. Group both requests.
5. Send them in parallel.

---

### Result:

- Two confirmation emails may be generated.
- Confirming one may unintentionally confirm the other.
- Depending on implementation, attacker may control which email gets verified.

---

# 🔥 Why Race Conditions Occur

- Lack of transaction locking
- Improper database isolation level
- Missing synchronization
- Business logic validation performed before commit
- Separate validation and update operations

---

# 💥 Security Impact

Race conditions can lead to:

- Coupon abuse
- Double spending
- Account takeover
- Bypassing OTP validation
- Balance manipulation
- Business logic abuse

Severity ranges from **Medium to Critical** depending on impact.

---

# 🛡 Mitigation & Prevention

To prevent race conditions:

- Use proper transaction locking
- Apply atomic database operations
- Implement server-side synchronization
- Use row-level locking
- Validate and update in a single transaction
- Apply strict rate limiting with server-side tracking
- Avoid separate validation and update calls

---

# 🧩 CWE & OWASP Mapping

- CWE-362 – Race Condition
- CWE-367 – Time-of-check Time-of-use (TOCTOU)
- OWASP A04 – Insecure Design
- OWASP A01 – Broken Access Control (impact dependent)

---

# 📚 References

- PortSwigger Web Security Academy – Race Conditions
- OWASP Business Logic Vulnerabilities
- CWE-362 Documentation

---

# 📜 License

This repository is intended for educational and authorized security testing purposes only.

---

## ✅ End of Guide
