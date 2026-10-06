# 🔐 NearKey - Hackathon Edition

### Proximity-based authentication without one-time codes.

[![Rowdy Hacks XII](https://img.shields.io/badge/Rowdy%20Hacks%20XII-2nd%20Place-gold)](https://devpost.com/software/nearkey)
[![Focus](https://img.shields.io/badge/Focus-Authentication%20%26%20Identity-blue)](#security-model)
[![Android](https://img.shields.io/badge/Android-Supported-green)](#mobile-authenticator)
[![iOS](https://img.shields.io/badge/iOS-Prototype-lightgrey)](#mobile-authenticator)
[![Status](https://img.shields.io/badge/Status-Active%20Development-orange)](#project-status)

**NearKey** is a proximity-assisted authentication platform that uses a trusted nearby phone to cryptographically authenticate sign-in requests.

Instead of asking users to retrieve and manually enter a one-time passcode, NearKey generates a short-lived authentication challenge, sends it to an enrolled mobile device, and verifies a device-generated cryptographic signature before allowing access.

> **Your trusted phone is nearby. NearKey verifies it. You sign in.**

NearKey began as a hackathon project focused on improving authentication for financial services. The current version is a working research prototype designed to explore faster, lower-friction authentication while reducing reliance on transferable OTP codes that can be exposed through phishing or social engineering.

🌐 **Live Demo:** https://nearkey.vercel.app/ (refresh if it does not load) 

🏆 **Devpost:** https://devpost.com/software/nearkey

---

## 🏆 Recognition

NearKey earned **2nd Place in the Beginner Track at Rowdy Hacks XII**, which featured **273 participants and 99 submitted projects overall**.

The project was also built around the **SWIVEL financial-services challenge**, where the team explored NearKey as a potential authentication layer for banking and other high-trust applications.

During judging, NearKey received strong feedback from industry professionals. A Microsoft judge specifically encouraged the team to continue developing the project and investigate whether portions of the authentication architecture may be suitable for patent protection.

NearKey is **not currently patented or patent pending**. Intellectual-property protection is being explored as development continues.

---

## 💡 Why NearKey?

Traditional two-factor authentication often looks like this:

```text
Enter password
      ↓
Wait for code
      ↓
Open phone
      ↓
Find message or authenticator app
      ↓
Copy or memorize code
      ↓
Return to website
      ↓
Enter code
```

NearKey explores a different flow:

```text
Enter password
      ↓
Trusted phone is nearby
      ↓
Server creates one-time challenge
      ↓
Phone signs the challenge
      ↓
Server verifies the signature
      ↓
Authenticated
```

There is no six-digit authentication code for the user to copy, type, or disclose to another person.

The longer-term goal is **risk-adaptive authentication**: routine, low-risk sign-ins should require minimal interaction, while sensitive actions can require explicit confirmation such as Face ID, Touch ID, or Android biometrics.

---

## ✨ Current Features

- Nearby-device authentication using **Bluetooth Low Energy (BLE)**
- Short-lived, single-use login challenges
- **P-256 ECDSA** challenge signing
- Session-, account-, phone-, and nonce-bound authentication context
- Android private-key storage through **AndroidKeyStore**
- QR-based phone enrollment
- Browser-to-phone Bluetooth verification
- Background Android authentication service
- Redis-backed hosted state for Vercel deployments
- WebSocket-based phone and browser communication
- Saved application dashboard for demonstrating future relying-party integrations
- Native iOS prototype using the same authentication concept

---

## 🧠 How It Works

NearKey currently consists of three primary components:

```text
┌──────────────────────┐
│   Website / Browser  │
└──────────┬───────────┘
           │
           │ HTTPS / WSS
           ▼
┌──────────────────────┐
│    NearKey Server    │
│                      │
│ Sessions             │
│ Challenges           │
│ Device Enrollment    │
│ Signature Validation │
└──────────┬───────────┘
           │
           │ Authentication challenge
           ▼
┌──────────────────────┐
│ NearKey Authenticator│
│                      │
│ Android / iPhone     │
│ Device-bound key     │
│ BLE communication    │
└──────────────────────┘
```

### 1. First-factor sign-in

The user begins a login in the browser.

A successful first factor creates a **pending authentication session** rather than immediately granting access.

### 2. Challenge generation

The server generates a cryptographically random, short-lived, single-use authentication challenge.

The challenge is bound to context including:

- account
- enrolled phone
- browser session
- service
- random nonce
- expiration time

Challenges currently expire after a maximum of **60 seconds**.

### 3. Device signature

The enrolled phone receives the challenge and signs the immutable challenge payload with its private key.

NearKey currently uses **ECDSA with the P-256 curve and SHA-256**.

On Android, the private key is generated and stored through AndroidKeyStore and is not exposed to the browser or server.

### 4. Bluetooth exchange

The browser communicates with the nearby authenticator over BLE and receives the signed authentication proof.

### 5. Server verification

The server verifies the signature and validates the challenge context before promoting the pending session to an authenticated session.

A consumed challenge cannot simply be replayed to authenticate again.

---

## 🔒 Security Model

NearKey is designed around **cryptographic proof of device-key possession**, not Bluetooth presence alone.

The current prototype includes:

### Short-lived challenges

Authentication challenges expire quickly and are tied to the login session that created them.

### Single-use authentication

Successfully consumed challenges cannot be reused.

### Device-bound cryptographic identity

During enrollment, the phone creates a public/private key pair. The server stores the public key while the private key remains on the mobile device.

### Session binding

Authentication proofs are tied to the browser session that initiated the request.

### Account and phone binding

Signed challenge context includes account and device information so a valid response cannot simply be moved between unrelated users or phones.

### Origin validation

Hosted NearKey deployments validate the expected public origin rather than relying on arbitrary forwarded headers.

### Persistent hosted enrollment

Vercel deployments use Redis-backed state so device enrollment can survive server-instance changes and redeployments.

---

## ⚠️ Current Security Boundaries

NearKey is an **authentication research prototype**, not a production banking authentication system.

### BLE does not mathematically prove distance

Bluetooth provides a useful nearby-device signal, but connectivity alone does not prove that the authenticator is a specific number of feet away. Relay attacks remain an important research area.

### Automatic signing does not always prove user intent

The Android prototype can automatically sign a valid authentication request without requiring the user to tap **Approve** every time.

That creates the low-friction experience NearKey is exploring, but it also means possession of the device and explicit human authorization are different security properties.

Our intended production direction is therefore **risk-adaptive authentication**:

```text
Known device + normal login
          ↓
Nearby trusted phone
          ↓
Automatic authentication

High-risk action
          ↓
Nearby trusted phone
          ↓
Explicit biometric confirmation
          ↓
Authentication / authorization
```

Potential step-up events include:

- password resets
- enrollment of a new authenticator
- account recovery
- changes to security settings
- logins from unfamiliar devices
- suspicious authentication activity
- high-value financial transactions
- transfers to new recipients

### Hardware backing varies

AndroidKeyStore keeps the key non-exportable through the normal application interface, but hardware-backed storage depends on the device.

### Public deployment is a shared demo

The hosted version uses demo-specific behavior to make judging and testing easier. Those settings are not intended to represent a production account-recovery model.

---

## 🏦 Financial Services Direction

NearKey was originally designed with financial authentication in mind.

One of the project's main questions is:

> **Can strong authentication become easier for normal users while reducing reliance on OTP codes that can be phished or socially engineered?**

A future banking integration could use NearKey for both sign-in authentication and sensitive transaction authorization.

For example:

```text
LOGIN
Institution: Example Bank
Device: Known MacBook
Risk: Low

→ Nearby trusted device verifies the request
→ Authentication completes with minimal interaction
```

A higher-risk transaction could instead require explicit approval:

```text
TRANSFER AUTHORIZATION

Amount: $4,250.00
Recipient: New Recipient
Institution: Example Bank

→ Nearby device detected
→ Transaction details cryptographically bound
→ Face ID / fingerprint required
→ Signed authorization returned
```

A future NearKey protocol could cryptographically bind the **relying party, action, amount, recipient, transaction ID, session ID, nonce, and expiration** into the signed authorization request.

---

## 📱 Mobile Authenticator

### Android

Android currently provides the most complete NearKey experience.

The authenticator supports:

- QR-based enrollment
- device key generation
- AndroidKeyStore private-key storage
- P-256 challenge signing
- BLE peripheral communication
- background authentication through an Android foreground service
- hosted HTTPS connections
- automatic challenge processing

### Android build requirements

Install:

- **JDK 17**
- Android Studio **or** Android SDK command-line tools
- Android SDK Platform supporting API 26+
- Android SDK Build Tools
- Android Debug Bridge (`adb`) for USB installation/testing
- a physical Android 8.0+ phone with BLE peripheral/advertising support

The repository includes the Gradle wrapper, so a separate Gradle installation is not required.

Build the debug APK:

```sh
cd android
./gradlew --no-daemon :app:assembleDebug
```

The APK will be generated at:

```text
android/app/build/outputs/apk/debug/app-debug.apk
```

Install it from the repository root with:

```sh
adb install -r android/app/build/outputs/apk/debug/app-debug.apk
```

For local USB testing:

```sh
adb reverse tcp:5173 tcp:5173
```

This allows the Android app to reach the local NearKey server at:

```text
http://localhost:5173
```

### Android release build

The hosted NearKey website can distribute the signed Android build from:

```text
/downloads/nearkey.apk
```

To rebuild the release:

```sh
./android/tools/release.sh
```

Release signing credentials should remain outside the repository and should never be committed to Git.

---

### iPhone

NearKey also includes a native iOS prototype using the same general challenge-response architecture.

The main limitation is currently **background Bluetooth behavior**. During the current prototype flow, NearKey must remain active during iPhone authentication.

Longer-term work includes:

- improved background BLE behavior
- TestFlight distribution
- production Apple signing
- biometric step-up authentication
- a more persistent authentication experience comparable to Android

Free Apple Personal Team development builds are temporary and are intended only for development testing.

---

## 🌐 Web Application

The browser application currently contains three primary routes:

```text
#/login
```

Starts first-factor authentication.

```text
#/loading
```

Handles device enrollment and nearby-phone authentication.

```text
#/dashboard
```

Displays the authenticated NearKey account and saved app list.

Client-side routing alone cannot bypass authentication. The server remains the source of truth for the session state.

---

## ☁️ Deploying to Vercel

The Hackathon Edition supports deployment through **Vercel**.

The repository includes:

```text
vercel.json
api/index.mjs
```

Hosted deployments use Redis-backed state for sessions, enrollment, saved applications, and phone communication across server instances.

### Required production environment variables

```env
PUBLIC_ORIGIN=https://your-project.vercel.app

DEMO_USERNAME=your-demo-user
DEMO_PASSWORD=your-demo-password

UPSTASH_REDIS_REST_URL=...
UPSTASH_REDIS_REST_TOKEN=...

NEARKEY_STATE_KEY=nearkey:production:v1
```

NearKey also accepts compatible Vercel storage variables:

```env
KV_REST_API_URL
KV_REST_API_TOKEN
```

For the public shared demonstration:

```env
DEMO_PHONE_SETUP=true
```

For a stricter environment:

```env
DEMO_PHONE_SETUP=false
```

The relaxed setting exists only to make the public demo easier to test and should not be treated as a production device-recovery policy.

After deployment, open the final production URL and enroll the mobile authenticator using the new QR code generated by that deployment.

---

## 🧪 Demo Walkthrough

1. Open NearKey in a Chromium-based browser on macOS.
2. Sign in using the demo account.
3. If no authenticator is enrolled, NearKey generates a temporary setup QR code.
4. Open the NearKey Authenticator on Android and scan the QR code.
5. Review the server address and enroll the phone.
6. Grant Nearby Devices/Bluetooth permissions when prompted.
7. Grant the browser Bluetooth permission the first time NearKey requests it.
8. NearKey generates a short-lived authentication challenge.
9. The phone signs the challenge with its device-bound private key.
10. The browser relays the signed proof to the server.
11. The server validates the signature, challenge, device, and session context.
12. The dashboard opens only after NearKey confirms the second factor.

Subsequent logins can reuse the enrolled authenticator without repeating device enrollment.

---

## 🧪 Hackathon / Demo-Only Features

### Simulated Google sign-in

The **Continue with Google** button currently simulates completion of an external first-factor identity provider.

It does **not** use Google OAuth.

After it is selected, the user still enters the same NearKey phone-authentication flow.

### Saved applications

The dashboard allows a user to save application names and optional URLs.

These records demonstrate the concept of a NearKey identity dashboard. They are currently **metadata only** and are not live integrations with those services.

---

## 🚀 Roadmap

NearKey is continuing beyond the original hackathon build.

### Relying-party SDK / API

Allow third-party applications to integrate NearKey without modifying the core NearKey codebase.

### Standalone bank demo

Build a separate financial-services application that consumes NearKey as a real authentication provider.

### Risk-adaptive authentication

Automatically handle familiar low-risk activity while requiring biometric confirmation for high-risk actions.

### Transaction binding

Bind sensitive operation details directly into signed authorization requests.

Possible fields include:

```text
Relying Party
Action
Amount
Recipient
Transaction ID
Session ID
Nonce
Expiration
```

### Per-service device keys

Generate separate cryptographic credentials for different relying parties instead of using one shared device identity everywhere.

### Improved iOS background operation

Bring the iPhone experience closer to the current Android background-authentication flow.

### Device and key attestation

Allow the server to make stronger decisions about the environment in which authentication keys were generated and stored.

### Credential protection and rotation

Further protect local enrollment credentials and support secure credential rotation/revocation.

### Security audit logging

Record device enrollment, authentication, credential revocation, challenge creation, and risk events.

### Relay-resistance research

Explore stronger mechanisms for demonstrating actual physical proximity rather than treating BLE connectivity itself as proof of distance.

---

## 🛡️ Threat Model Areas

NearKey is being developed with the following attack classes in mind:

- OTP phishing
- social engineering
- replay attacks
- MFA fatigue
- session hijacking
- authentication relay
- device theft
- enrollment hijacking
- credential cloning
- malicious relying parties
- compromised browsers
- compromised servers
- account-recovery abuse

The project intentionally documents known limitations instead of assuming that Bluetooth proximity alone provides complete security.

---

## 🧰 Technology

### Backend

- Node.js
- HTTP APIs
- WebSockets
- Vercel Functions
- Upstash Redis

### Web

- JavaScript
- HTML / CSS
- Web Bluetooth
- WebSocket communication

### Android

- Kotlin
- Android BLE APIs
- AndroidKeyStore
- foreground services

### iOS

- Swift
- Core Bluetooth
- native device cryptography

### Cryptography

- ECDSA
- P-256
- SHA-256
- cryptographically random nonces
- short-lived single-use challenges

---

## 📊 Project Status

NearKey should currently be considered:

> **A working authentication research prototype and award-winning hackathon project under active development.**

The Hackathon Edition demonstrates the complete concept of:

```text
Device enrollment
        ↓
Cryptographic identity
        ↓
Server challenge
        ↓
Nearby device discovery
        ↓
Challenge signature
        ↓
Server verification
        ↓
Authenticated session
```

It is **not yet intended for production financial accounts or other security-critical environments** without further review, hardening, testing, and integration work.

The next stage is moving NearKey from a demonstration authentication provider toward an integration-oriented identity and authentication platform.

---

## 👥 Team

NearKey was created during **Rowdy Hacks XII** and earned **2nd Place in the Beginner Track**.

The team continues to develop the project beyond the hackathon with a focus on:

**Identity • Authentication • Applied Cryptography • Mobile Security • Financial Security • Anti-Phishing • Proximity Authentication**

---

## ⚖️ Disclaimer

NearKey is an experimental security project.

Do not use the current Hackathon Edition to protect production financial accounts, sensitive organizational infrastructure, or other high-risk systems without appropriate security review and hardening.

---

## ⭐ NearKey

### **Authentication shouldn't require memorizing another code.**

NearKey explores what authentication looks like when the trusted device already in your pocket can prove its identity securely, locally, and with minimal friction.
