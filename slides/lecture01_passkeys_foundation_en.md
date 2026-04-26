# Lecture 1: Introduction to Passkeys

## Purpose of This Material

This lecture note is an introductory guide designed to bridge the gap between beginner-level understanding and implementation-level thinking about passkeys. Rather than treating passkeys as just a convenient login feature, this material explains the problems with traditional password-based authentication, what FIDO2 and WebAuthn are trying to solve, and where implementers need to pay attention.

This repository is designed as two complementary parts:

- `slides/`: explanatory materials for conceptual understanding
- `app/`: a hands-on application for trying registration, authentication, and failure cases

---

## 1. Why Do We Need Passkeys?

### The Limits of Password Authentication

For many years, web services have mainly relied on a model where users type an ID and password, and the server decides whether to allow login based on a match. But this model has structural weaknesses.

- Users tend to choose weak passwords
- Password reuse is common
- If users type secrets into phishing sites, those secrets can be stolen directly
- Even if the server stores password hashes, a breach can still have serious impact
- Adding MFA can help, but may increase complexity and still be weak against some attacker-in-the-middle scenarios

In short, a password is a secret that people must remember and re-enter. That property itself makes it vulnerable to theft and reuse.

### What Passkeys Aim to Change

Instead of asking users to remember and type a secret string, passkeys use credentials based on public-key cryptography and stored on the user's device. The key goals are:

- do not send the secret to the server
- do not make humans type the secret
- only allow use under the correct origin
- combine security with familiar device authentication such as biometrics or device unlock

---

## 2. What Is a Passkey?

### A Simple Definition

A passkey generally refers to a public-key credential based on FIDO2 and WebAuthn, packaged by operating systems and browsers in a way that is easy for users to use.

From the user's point of view, the experience looks like this:

1. Choose "register a passkey" on a site
2. Confirm identity using Face ID, Touch ID, Windows Hello, a device PIN, or screen unlock
3. Later sign in from that device without typing a password

### The Core Idea in One Sentence

The essence of a passkey is this: instead of sharing a secret string with the server, the device signs with a private key kept on the device, and the server verifies that signature using a public key.

In the password world, the user and the server both depend on the same secret. If that secret is stolen, someone else can act as the user.

In the passkey world, responsibilities are split:

- Device:
  - holds a private key that never leaves the device
- Server:
  - stores only the matching public key

This means the server can verify the user without storing the user's secret itself. That is the biggest conceptual shift.

### Understanding the Public-Key Model Simply

Public-key cryptography uses two keys.

- Private key
  - held only on the user's device
  - never sent out
- Public key
  - safe to send to the server
  - used to verify whether a signature is genuine

An easy mental model is:

- the private key is like a seal only the real owner can use
- the public key is the reference information used to check whether the seal is genuine

Whenever a user authenticates, the server sends a fresh `challenge`, which is a one-time piece of data. The device signs that challenge with the private key. The server then uses the public key to confirm that the signature is valid and that it was made for the exact challenge the server just issued.

Because of this design, the same secret does not need to be sent every time. That greatly reduces the risk of stealing and replaying something like a password.

### The Technical Reality

At the center of passkeys is public-key cryptography.

- On the device side:
  - the private key is stored securely
- On the server side:
  - the public key is stored
- During authentication:
  - the server issues a `challenge`
  - the device signs it with the private key
  - the server verifies it with the public key

So the server stores a public key, not a shared secret that becomes catastrophic if leaked.

### A More Intuitive Comparison with Passwords

- Passwords:
  - prove that the user knows a secret
  - require that secret to be entered
  - can be reused by anyone who steals the secret
- Passkeys:
  - rely on the user's device holding the private key
  - do not send the private key itself
  - send only a signature result

The crucial point is this: the system can prove that the correct secret exists on the device without sending that secret. That is the strength of the public-key model.

### The Three Minimum Things to Remember

When first learning about passkeys, these three points help the rest make sense:

1. A passkey is not just a password with a different UI
2. Authentication relies on a private key on the device and a public key on the server
3. Each authentication signs a fresh `challenge`, which improves phishing and replay resistance

Once these points are clear, the WebAuthn registration and authentication flows become much easier to understand.

### The User Experience of Passkeys

For users, passkeys are not only "a system that uses advanced cryptography." They are also a login experience that removes the need to remember and type passwords again and again.

In many environments, users confirm themselves with methods such as:

- fingerprint authentication
- face recognition
- a device PIN
- screen unlock

The important point is that the biometric data or PIN itself is not sent to the website. Those local checks are used by the device to decide whether it should allow the private key to be used.

So although the experience feels like "logging in with a face or fingerprint," the technical reality is "the device uses the private key to produce a signature." The site is not receiving biometric data or the PIN and checking it remotely.

### Not Sending Password-Related Secrets Outside

With password authentication, users type a password every time, and the server authenticates based on information derived from that secret. That creates room for abuse if the destination is a phishing site or if the server later suffers a breach.

With passkeys, there is no shared secret string that must be sent to the site for verification.

- the private key stays on the device
- the server stores only the public key
- the login flow returns a signature result

Because of this, passkeys can authenticate without sending the password itself or a password-derived shared secret outside. Users do not have to hand over a secret string, and the server does not have to store it.

### Comparing Passwords and Passkeys

| Aspect | Passwords | Passkeys |
|---|---|---|
| User action | Remember and type a string | Confirm with biometrics or a device PIN |
| What is sent to the site | Depends on the password and its verification flow | A signature over the challenge |
| What the server mainly stores | Password hash | Public key and credential ID |
| Secret kept on the device | Often not strongly protected as a cryptographic credential | Private key is kept on the device |
| Phishing resistance | Easy to steal if typed into a fake page | Harder to abuse because it is bound to origin/RP context |
| Impact of server breach | Can lead to hash cracking or password reuse abuse | A public key alone does not directly allow login |
| Usability | Requires memory, typing, and resets | Close to normal device unlock behavior |

The key point in this table is that passkeys change both the user experience and the authentication model. They are not just a cosmetic replacement for passwords.

---

## 3. Related Terms and Technical Positioning

### FIDO2

FIDO2 is a framework for passwordless and phishing-resistant authentication. In practice, it is easiest to understand it as the combination of two major pieces:

- WebAuthn
  - the standard API used by browsers and web applications
- CTAP
  - the protocol used for communication between clients and authenticators or security keys

### The Technical Relationship Between WebAuthn and FIDO2

These terms look similar, but they are not the same thing.

- FIDO2:
  - the overall framework for passwordless authentication
- WebAuthn:
  - the standard API used by web browsers and web applications within that framework

From a web developer's point of view, WebAuthn is the entry point exposed in the browser, while FIDO2 is the broader ecosystem and model behind it.

Put more concretely:

- WebAuthn:
  - the specification for using authenticators from JavaScript in web pages
- CTAP:
  - the specification for communication between browsers or operating systems and authenticators or security keys
- FIDO2:
  - the overall authentication framework built from those components

### A Simple Relationship Map

1. A web application calls `navigator.credentials.create()` or `navigator.credentials.get()` in the browser
2. Those APIs are provided by WebAuthn
3. The browser or operating system talks to the authenticator when needed
4. That authenticator communication uses CTAP
5. The overall model is commonly referred to as FIDO2-based authentication

So it is helpful to think of WebAuthn as "the web-facing entry point" and FIDO2 as "the larger system around that entry point."

### WebAuthn

WebAuthn is a W3C standard API for using authenticators from JavaScript. In frontend implementations, the two most important calls are:

- `navigator.credentials.create()`
  - registers a new credential
- `navigator.credentials.get()`
  - authenticates with an existing credential

### Authenticator

An authenticator is the component that uses the private key to create signatures. Examples include:

- a built-in smartphone authenticator
- a laptop biometric platform
- a USB security key

---

## 4. Registration Flow

Let us follow passkey registration at a conceptual level.

1. A signed-in user clicks "register passkey"
2. The server generates a random `challenge`
3. The server returns RP information, user information, the `challenge`, and related options to the browser
4. The browser calls `navigator.credentials.create()`
5. The device verifies the user locally and generates a key pair
6. The resulting attestation or registration response is sent back to the server
7. The server verifies the `challenge` and related fields, then stores the public key and credential ID

### What the Server Stores During Registration

Implementation details vary, but the server typically stores items like:

- user ID
- credential ID
- public key
- sign count
- transports
- creation time or a label

The private key is not stored on the server.

---

## 5. Authentication Flow

Now let us look at the authentication flow.

1. The user selects "sign in with passkey"
2. The server returns a `challenge` and allowed credential candidates
3. The browser calls `navigator.credentials.get()`
4. The authenticator signs based on the `challenge` and origin context
5. The browser sends the assertion to the server
6. The server verifies the signature, `challenge`, origin, and RP ID
7. If verification succeeds, the login succeeds

### How This Differs from Password Authentication

- Passwords:
  - the user enters a secret
  - the server verifies a shared-secret-based flow
- Passkeys:
  - the user does not enter the secret key
  - the device signs with the private key
  - the server verifies the signature with the public key

---

## 6. Why Passkeys Have Strong Phishing Resistance

One of the biggest reasons passkeys attract attention is their phishing resistance.

### Why They Are Strong

- authentication is bound to origin context
- a fake site does not satisfy the correct RP ID and origin conditions
- it is much harder to steal something and reuse it the way attackers do with passwords

For example, a credential registered for `example.com` cannot simply be used on a lookalike domain such as `examp1e.com`. That is a major difference from passwords.

### Put Even More Simply

With passwords, users type a secret into the page in front of them. So if the page is fake and the user trusts it, the secret can be stolen directly.

With passkeys, users are not handing a secret string to the site. Instead, the device uses a private key intended for that site and returns a signature. That makes it much harder for a fake site to obtain usable credentials for the real site.

In short, passwords are a model where "a human gives the secret away," while passkeys are a model where "the device returns a site-bound signature." That is the source of phishing resistance.

---

## 7. Why Passkeys Are Considered Safer

Passkeys are not considered safer just because they are newer. They are safer because the things attackers used to steal easily become structurally harder to steal or reuse.

Here we focus on three important properties:

- phishing resistance
- server-breach resistance
- replay-attack resistance

### 7-1. Phishing Resistance

Phishing resistance means that even if a user is tricked by a fake site, the authentication material is much harder to steal and reuse directly.

With passkeys, authenticators operate in a context tied to origin and RP ID. A credential created for `example.com` cannot simply be used correctly on a different malicious domain.

This is because the server is not merely checking whether "a string matched." It is checking conditions such as:

- whether the signature corresponds to the current `challenge`
- whether the response is bound to the expected origin and RP ID
- whether the signature corresponds to a registered credential

So even if an attacker builds a login page that looks almost identical to the real one, it is much harder to extract and immediately reuse authentication material the way password phishing does.

### 7-2. Server-Breach Resistance

Server-breach resistance means that even if the server-side database is leaked, the leaked data does not immediately let the attacker log in as the user.

In password systems, the server stores password hashes. Even though hashes are better than plaintext passwords, they still become targets for brute force, cracking, and reuse abuse after a breach.

In passkey systems, the server mainly stores:

- public keys
- credential IDs
- usage metadata

The important point is that a public key alone cannot create a valid signature. Authentication requires the private key, and that private key remains on the user's device.

So even if the server is compromised, the attacker gets verification material, not the secret material required to impersonate the user. That is a major advantage over password systems.

Of course, a server breach is still serious. User data leakage, credential enumeration, and session leakage can still matter. But at least the common failure mode of "stolen authentication data leads directly to login" becomes much less likely.

### 7-3. Replay-Attack Resistance

A replay attack means capturing valid authentication data and sending it again later to impersonate the user.

Passkeys are designed to resist this because the authentication data is not static. The server sends a fresh `challenge` every time.

In simplified form:

1. The server sends a one-time `challenge`
2. The device signs that `challenge`
3. The server verifies that the response matches the exact `challenge` it just issued

Because of this, copying and replaying an old response does not work if the `challenge` is different.

In password systems, once an attacker knows the correct secret, that secret can often be reused repeatedly. With passkeys, each response must match the fresh `challenge`, so replaying a previous result is much harder.

### Summary of the Main Security Properties

- Phishing resistance:
  - makes it much harder to steal reusable credentials on fake sites
- Server-breach resistance:
  - the server does not hold the private key, so a breach does not directly expose the login secret
- Replay resistance:
  - each authentication signs a fresh `challenge`, so old responses are difficult to reuse

These three properties are why passkeys are not only convenient, but also structurally better positioned to improve authentication security.

### Important Remaining Caveats

This does not mean every attack disappears.

- session hijacking
- theft of an already-enrolled device
- weak recovery flows
- weak identity verification during account recovery
- server-side implementation mistakes such as missing `challenge` validation

Passkeys are not a magic key. They are a stronger foundation for authentication.

---

## 8. Challenges of Device Dependence and Ecosystem Dependence

Passkeys are powerful, but in real deployments they depend heavily on devices, operating systems, browsers, and account-sync ecosystems. This is a different kind of challenge from traditional password systems.

### 8-1. Device-Dependent Challenges

Because passkeys rely on a device or authenticator that holds the private key, the user experience is influenced by the user's actual hardware environment.

- login becomes harder if the needed passkey is not available on the device the user is using
- device loss, replacement, or failure creates migration and recovery challenges
- older devices or browsers may provide an incomplete experience
- ease of adoption depends on whether biometrics or device PINs are already set up

In other words, passkeys reduce the burden of remembering secrets, but they make device availability and device management much more visible.

### 8-2. Ecosystem-Dependent Challenges

The passkey experience does not exist only inside a single website. It depends on cooperation across operating systems, browsers, password managers, cloud sync systems, and security keys.

- there are experience differences across Apple, Google, Microsoft, and other platforms
- browser and OS implementations may differ in UI and supported features
- synchronization and cross-device transfer behavior can vary by ecosystem
- managed enterprise devices or restricted environments may be harder to support

Because of this, designers need to ask not only "is this possible in the specification?" but also "will this work smoothly in the user's actual environment?"

### 8-3. Things to Plan for During Adoption

When introducing passkeys, surrounding flows matter as much as the core cryptography.

- how account recovery works after a lost device
- how users register multiple devices
- how fallback works in environments that do not support passkeys well
- which devices are trusted in organizational deployments

If the design only improves security but does not help users recover when something goes wrong, the operational experience can still become painful.

---

## 9. Implementation Points

To build a passkey-enabled application, implementers need to understand both the client and server sides.

### Client-Side Points

- correctly receive `PublicKeyCredentialCreationOptions`
- handle Base64URL and ArrayBuffer conversion
- handle errors from `navigator.credentials.create()` and `get()`
- provide a path when the user cancels

### Server-Side Points

- generate `challenge` values securely and keep them only for a short time
- always verify the returned `challenge`
- verify origin and RP ID
- perform signature verification correctly
- manage sign counts and credential metadata carefully

### Common Difficult Areas

- binary encoding conversion
- browser differences
- HTTPS requirements in development
- managing multiple devices and multiple credentials

---

## 10. Threat Model for Learning

This material is meant to go beyond "it works" and help learners see what becomes dangerous when the implementation is broken. The hands-on lab is planned to explore questions such as:

- what happens if the `challenge` is fixed
- what becomes dangerous if origin validation is removed
- what happens if post-login session handling is weak
- why weak account recovery still matters even with passkeys
- where users get confused if the UI is poorly designed

In other words, learning passkeys means more than learning to use public keys. It also means understanding the surrounding design decisions.

---

## 11. Plan for the Hands-On Application

The `app/` directory is intended to host a small educational web application with features such as:

- user registration
- password login
- passkey registration
- passkey authentication
- registered credential listing
- safe and intentionally unsafe comparison modes

### Learning Experience Goals

- first experience the normal registration and authentication flow
- then inspect the flow of `challenge` and credential handling through logs and UI
- finally observe what happens when parts of the implementation are intentionally weakened

This sequence makes the topic much easier to understand than reading specifications alone.

---

## 12. Summary

- Passkeys are a phishing-resistant authentication approach built on public-key cryptography
- users do not have to remember and type a secret string
- the server stores a public key and verifies signatures
- WebAuthn registration and authentication are based on `challenge` flows
- implementation correctness around origin, RP ID, and `challenge` validation is critical
- biometrics and device PINs are used locally to allow private-key use, not sent to the site as remote secrets
- passkeys improve security and usability, but they also introduce device and ecosystem dependencies
- to learn passkeys well, it is important to study not only the happy path but also failure cases and threat models

### Closing Summary

Passkeys represent a major shift from "authentication by entering a shared secret" to "authentication by signing with a private key stored on the device." That shift is what makes stronger resistance to phishing, server breaches, and replay attacks possible.

At the same time, usability depends heavily on the surrounding device and platform ecosystem, so deployment must consider recovery flows, multi-device use, and operational reality. In that sense, passkeys are not just a login feature. They are a design topic that sits at the intersection of security, UX, and operations.

---

## Possible Future Lectures

- Lecture 2: Follow the WebAuthn registration flow in code
- Lecture 3: Authentication flow and signature verification
- Lecture 4: UX and account recovery design for passkey deployment
- Lecture 5: Implementation mistakes from an attacker's point of view
