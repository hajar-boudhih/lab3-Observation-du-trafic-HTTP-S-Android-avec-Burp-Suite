# lab3-Observation-du-trafic-HTTP-S-Android-avec-Burp-Suite



# Mobile Security Analysis Lab – HTTP Interception

## Introduction

This repository documents a hands-on mobile security laboratory focused on HTTP traffic interception. The objective of this lab was to understand how unencrypted network communications can be observed, analyzed, and potentially abused when proper security controls are not in place.

The experiment was conducted in a fully controlled environment using **Burp Suite Community Edition** as the interception proxy and an **Android 10 emulator** as the target device. The purpose was strictly educational: to visualize data exposure risks and to better understand how to defend against them.

This report presents the configuration, observations, identified risks, and recommended countermeasures.

---

## Technical Environment

The lab was carried out using the following setup:

* Analysis tool: Burp Suite Community Edition
* Analysis host IP address: 192.168.11.101
* Proxy port: 8080
* Target platform: Android 10 Emulator
* Test date: February 25, 2026

All components were connected within a local network to ensure a safe and isolated testing environment.

---

## Laboratory Configuration

### 1. Proxy Configuration with Burp Suite

The first step consisted of configuring Burp Suite to act as an interception proxy. A proxy listener was created with the following parameters:

* Binding address: 192.168.11.101
* Listener port: 8080
* Proxy listener enabled and actively accepting connections

Once activated, Burp Suite was ready to capture and log HTTP traffic originating from the emulator.



<img width="1078" height="455" alt="Image" src="https://github.com/user-attachments/assets/bffb31ca-9bcf-45a4-bbb4-bf94e1e37c38" />


### 2. Android Emulator Proxy Setup

The Android emulator was configured to route its traffic through the proxy manually. The WiFi network settings were modified as follows:

* Manual proxy configuration enabled
* Proxy hostname set to 192.168.11.101
* Proxy port set to 8080
* Local addresses added to the bypass list

This configuration ensured that all HTTP requests from the emulator passed through Burp Suite before reaching their intended destination.



<img width="655" height="426" alt="Image" src="https://github.com/user-attachments/assets/91b128a6-506c-4dcb-bdf4-029482266e31" />

<img width="594" height="263" alt="Image" src="https://github.com/user-attachments/assets/ea51ba96-0f9b-4361-aa51-17a423c9fbf7" />


<img width="632" height="523" alt="Image" src="https://github.com/user-attachments/assets/4efe8a9c-9b3a-4241-8623-64a9e3e37013" />



<img width="227" height="473" alt="Image" src="https://github.com/user-attachments/assets/3ff70296-e95c-4593-982c-8802ebaf192c" />


### 3. Certificate Installation

To properly intercept HTTPS traffic, Burp’s Certificate Authority (CA) certificate was exported in DER format and installed on the emulator.

During installation, Android displayed a warning about installing a custom CA certificate. This was expected behavior in a lab environment and was acknowledged for testing purposes. This step allowed encrypted HTTPS traffic to be decrypted and inspected by Burp Suite.

---


<img width="566" height="649" alt="Image" src="https://github.com/user-attachments/assets/b6f208ff-76d9-4d27-9096-01cc2be8856e" />

<img width="295" height="550" alt="Image" src="https://github.com/user-attachments/assets/c12f240a-f3d1-4b43-9c80-b7d7c887cc22" />

<img width="819" height="266" alt="Image" src="https://github.com/user-attachments/assets/0ad87e04-15d9-488d-8687-61b51cb3b4fb" />

<img width="921" height="471" alt="Image" src="https://github.com/user-attachments/assets/e9f0f1a7-b4c4-4361-b1b9-638605b94bee" />

<img width="234" height="262" alt="Image" src="https://github.com/user-attachments/assets/613951dc-2d87-462f-97a5-41bfeeb4f43a" />

## Traffic Analysis and Observations

Once the environment was configured, test traffic was generated and monitored.

### Test Traffic – neverssl.com

The website neverssl.com was intentionally chosen because it operates over HTTP without encryption. When accessed from the emulator, Burp Suite successfully captured the full HTTP request.



<img width="204" height="460" alt="Image" src="https://github.com/user-attachments/assets/3ddc5c90-2e51-4408-9204-17995db2a809" />

<img width="1140" height="251" alt="Image" src="https://github.com/user-
   attachments/assets/73b1a561-5378-4a82-be00-9bfad7d3b178" />
   

<img width="788" height="532" alt="Image" src="https://github.com/user-attachments/assets/ca43fd49-8580-4674-ad86-5c3795c3b3ba" />

<img width="523" height="583" alt="Image" src="https://github.com/user-attachments/assets/ee07b2de-92a7-4f9e-a907-66636e279067" />


The intercepted request clearly displayed:

* The request method (GET)
* The full URL path
* Host header
* User-Agent string
* Referer header
* Language and encoding preferences

This demonstrated how easily plaintext HTTP traffic can be read when intercepted.

### System Application Traffic – Google Update Service

During analysis, background traffic from Google services was also captured. Requests to Google update services revealed structured JSON payloads containing application identifiers and update checks.

Even though some services use secure channels, this observation highlighted how metadata such as:

* Application IDs
* Version numbers
* Updater information
* Device operating system

can potentially be exposed if not properly encrypted.

---

## Identified Security Risks

The interception exercise revealed several important risks.

### 1. Plaintext Data Exposure

When HTTP is used instead of HTTPS, all transmitted data is visible in clear text. This includes:

* URL paths and query parameters
* Request headers
* Referer information
* Potential user identifiers

Anyone positioned between the client and server (for example, on public WiFi) could intercept and read this information.

### 2. Application Information Leakage

Requests may contain valuable metadata about the device and installed applications. For example:

* Application identifiers
* Version numbers
* Platform details from User-Agent strings

This information can assist attackers in fingerprinting devices and targeting known vulnerabilities.

### 3. Session and Authentication Risks

If cookies are transmitted without proper security flags:

* They can be intercepted on unencrypted connections.
* Session hijacking becomes possible.
* Sensitive tokens in URLs may be logged or cached unintentionally.

This creates a serious risk for user accounts and authenticated sessions.

---

## Defensive Recommendations

Based on the findings, several defensive measures are strongly recommended.

### For Mobile Application Developers

Applications should enforce secure communications by default.

* Disable cleartext traffic in the Android manifest using cleartextTrafficPermitted="false".
* Implement certificate pinning for critical backend services.
* Use a Network Security Configuration file to define per-domain security rules.

Cookies must be configured securely:

* Use the Secure flag to restrict transmission to HTTPS only.
* Apply the HttpOnly flag to prevent JavaScript access.
* Configure the SameSite attribute appropriately.

Sensitive information should never be included in URLs. Instead:

* Use HTTPS with POST requests.
* Place sensitive data in the request body.
* Consider implementing request signing for critical operations.

### For Server Administrators

Servers should enforce strong HTTP security headers, including:

* Strict-Transport-Security (HSTS)
* Content-Security-Policy
* X-Content-Type-Options
* X-Frame-Options

These headers significantly reduce exposure to downgrade attacks, content injection, and clickjacking.

### For End Users

Users also play a role in protecting themselves:

* Always verify the HTTPS padlock in browsers.
* Avoid installing untrusted certificates.
* Do not perform sensitive transactions on public WiFi networks without protection.
* Consider using a trusted VPN service on untrusted networks.

---

## Reproducibility of the Lab

This lab can be reproduced with the following prerequisites:

* Burp Suite Community Edition or later
* Android SDK with an emulator running Android 10 (API level 29)
* A host machine connected to the same local network

The general steps are:

1. Launch Burp Suite and configure a proxy listener on port 8080.
2. Export the Burp CA certificate in DER format.
3. Start the Android emulator and install the certificate.
4. Configure the emulator’s WiFi proxy manually.
5. Access [http://neverssl.com](http://neverssl.com) to verify interception.
6. Observe captured traffic in Burp’s HTTP history tab.

Following these steps will reproduce the same traffic interception behavior described in this report.

---



<img width="512" height="624" alt="Image" src="https://github.com/user-attachments/assets/3cdaf269-f506-44d6-aa84-e01d5ee25dc3" />

## Ethical and Legal Notice

This security analysis was conducted strictly within a controlled laboratory environment for educational purposes. All intercepted traffic was generated intentionally for testing.

Unauthorized interception of network communications may violate privacy laws and cybersecurity regulations. Security testing must always be performed with explicit authorization from system owners.

---

## Conclusion

This lab clearly demonstrates how vulnerable unencrypted HTTP communication can be. Through a simple proxy setup, it was possible to observe complete request details, metadata, and application behavior.

More importantly, the exercise reinforces a critical lesson in mobile security: encryption is not optional. Proper HTTPS enforcement, secure cookie configuration, certificate validation, and defensive server headers are essential to protect user data.

Understanding how attacks work is the first step toward building secure systems. This laboratory exercise serves as both a technical demonstration and a reminder of the importance of secure-by-design development practices.



