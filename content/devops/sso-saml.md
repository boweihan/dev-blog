---
title: "SSO, SAML and SLO"
date: 2016-12-25T21:53:48-04:00
draft: false
type: "post"
---

Let’s talk about Single Sign-On (SSO), Security Assertion Markup Language (SAML) and Single Logout (SLO).

![](https://cdn-images-1.medium.com/max/2000/1*_hDf5cmKUnIIRxssl9N44g.png)

<br/>
***What is Single Sign-On?***

Have you ever thought it would be nice to have one credential to all of your online accounts? Welcome to the world of Single Sign-On!

SSO is a process that allows users to authenticate into multiple services after logging into a primary service. You might have already noticed this with Gmail, which lets you sign on to a myriad of sites with your google credentials alone.

![](https://cdn-images-1.medium.com/max/2000/1*JVfyJgkVq70S678r-NQQTg.png)

The base process for SSO is as follows:

* Log into main service (i.e. Gmail).

* When you visit a new service (i.e. Medium), it redirects you to the main service to check if you are logged in.

* Main service returns a one time password (OTP) token.

* OTP token is verified by new service, and user is granted access.

The three most common protocols for implementing SSO are OAuth, OpenID and SAML. The example with Gmail uses OAuth, which is an **authorization protocol**. SAML is different because it provides a means for users to access multiple domains with the **same credentials** (i.e. authentication, which is not covered by OAuth). Moving forward, we’ll be discussing how SSO is implemented with SAML.

<br/>
***SAML what?***

SAML (Security Assertion Markup Language) was developed in the early 2000s “to define an XML framework for exchanging authentication and authorization information.” Think of it as the standard for exchanging authentication and authorization data between two parties. SAML is critical to Single Sign-On because it allows a user’s identity to be passed from one place to another with digitally signed XML (eXtensible Markup Language) documents.

![Example of a SAML AuthNRequest](https://cdn-images-1.medium.com/max/4332/1*7VqWpxOm7mxa_dAa5rbuEA.png)<center>*Example of a SAML AuthNRequest*</center>

The wealth of information that needs to be included in a SAML request — such as Version, ID, ProviderName, IssueInstant, Destination, ProtocolBinding, AssertionConsumerServiceURL, and Issuer — are critical to providing parties with adequate user context.

<br/>
***SAML and SSO***

This diagram (clockwise) sums up the flow for SAML SSO.

![SAML SSO User Flow](https://cdn-images-1.medium.com/max/3380/1*FfZ238hvS5cuC-iLJX-a7g.png)<center>*SAML SSO User Flow*</center>
<br/>
* User (Principle) accesses external service provider (SP) (i.e. your website).

* Service provider identifies user’s origin (i.e. through IP or subdomain) and sends authentication request (AuthnRequest) to Identity Provider (i.e. Okta, ADFS).

* User logs into IdP if necessary and initiates a session.

* IdP builds XML authentication response containing the users email or username, signs it, and POST’s it back to service provider.

* The service provider validates the response based on certificate fingerprint and identity provider.

* User is granted access!

This flow has significant advantages over logging in with a username/password because there is no need to type or remember credentials and no possibility of having a weak password. Amazing.

<br/>
***The Logout Problem***

While it is all good and fine to recklessly log into various web applications using SSO, this is not without drawbacks. Here are two:

* Users rarely remember to log out of every session established during SSO. This opens the door to vulnerabilities such as session hijacking, where session tokens can be stolen through sniffing, XSS, and man-in-the-middle attacks.

* Signing out of a service provider doesn’t mean you are signing out from the IdP, it only clears the local session. This means that if your IdP redirects unauthenticated users to a service provider automatically (a common scenario), signing out won’t have any effect because users will be redirected to the IdP and then redirected back to the service provider with a brand new session.

This is where Single Logout (SLO) with SAML becomes useful.

<br/>
***Single Logout***

There are various configurations for SLO — we’ll discuss the “front channel” (HTTP redirect) model initiated by the service provider.

![](https://cdn-images-1.medium.com/max/2292/1*xMbaV6E4060zic28pewbsQ.png)

* User hits logout.

* Service provider creates a digitally signed SAML LogoutRequest and returns it to the user’s browser.

* Browser sends request to IdP Single Logout URL which is appended with the LogoutRequest (base64 & base64url encoded) in the query string.

* IdP identifies all other SSO sessions for user and iteratively generates SLOs for each service provider.

* After each service provider has successfully terminated its own session with the user, the IdP terminates its own session with the user and sends a LogoutResponse to the original service provider.

* User is logged out of IdP and all service providers.

Using SLO to log out of the IdP and service provider simultaneously will mitigate the risk of orphaned SSO sessions and allow users to log out with (more) confidence.

Comment on [Medium](https://medium.com/@BoweiHan/elijd-single-sign-on-saml-and-single-logout-624efd5a224)