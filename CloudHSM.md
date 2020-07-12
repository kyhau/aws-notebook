# CloudHSM & On-Premises HSM

## CloudHSM
- CloudHSM is a dedicated HSM (hardware security module) which runs within your VPC, accessible only to you in a
  single tenant architecture.
- AWS manages and maintains hardware, but has **no access to the cryptographic component**.
- Interaction with CloudHSM is via **industry standard APIs**, no normal AWS APIs.
  - **PKCS#11**
  - **Java Cryptography Extensions (JCE)**
  - **Microsoft CrytoNG (CNG)**
- Keys can be transferred between CloudHSM and other hardware solutions (on premises).
- Keys are shared between cluster members. 
- Applications can be OUTSIDE the VPC - Direct Connect, Peered, or VPN.

Use Case
- AWS CloudHSM would be an appropriate key management solution for requirements such as:
  1. No access to the key enclave for ANYONE but security admins. 
  2. Compliance with FIPS140-2 Level 3 or above.
  3. Support one or more of APIs (e.g. PKCS#11, JCE)
  4. Ideally, no self-owned hardware.


## On-Premises HSM
- On-Premises HSM:  if you really need to control your own physical hardware.
