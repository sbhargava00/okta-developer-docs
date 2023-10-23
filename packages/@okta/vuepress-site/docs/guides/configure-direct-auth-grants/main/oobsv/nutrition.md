<ApiLifecycle access="ea" />
<ApiLifecycle access="ie" />

This guide explains how to implement a direct authentication out-of-band (OOB) flow for your app in Okta. The guide focuses on using SMS or Voice with the Phone authenticator.

---

**Learning outcomes**

* Understand the OAuth 2.0 direct authentication OOB flow.
* Set up your app with the OOB grant type.
* Implement the OOB flow in Okta.

**What you need**

* [Okta Developer Edition organization](https://developer.okta.com/signup)
* An app that you want to implement OAuth 2.0 direct authentication OOB with Okta
* A test user in your org enrolled in the Phone authenticator with both **SMS** and **Voice call** enabled
* The Direct Authentication feature enabled for your org. Contact [Okta Support](mailto:support@okta.com) to enable this EA feature.

<ApiAmProdWarning />