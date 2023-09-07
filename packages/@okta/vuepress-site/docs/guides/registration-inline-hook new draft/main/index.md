---
title: Registration inline hook
excerpt: Code an external service for a registration inline hook
layout: Guides
---

<ApiLifecycle access="ie" />

This guide provides examples of an Okta registration inline hook for profile enrollment (self-service registration) and progressive profile enrollments. It uses the web site [Glitch.com](https://glitch.com) to act as an external service to receive and respond to registration inline hook calls. The sample project provides external code for three registration inline hook use cases:

* A simple profile enrollment (self-service registration)
* Progressive profile enrollment updates
* Profile enrollment and progressive profile enrollments together

> **Note:** This document is only for Okta Identity Engine. If you're using Okta Classic Engine, see [Registration inline hook for Classic Engine](/docs/guides/archive-registration-inline-hook/nodejs/main/). See [Identify your Okta solution](https://help.okta.com/okta_help.htm?type=oie&id=ext-oie-version) to determine your Okta version.

---

**Learning outcomes**

* Understand the Okta inline hook calls and responses for profile enrollment (SSR) and progressive profile enrollment.
* Implement working examples of a registration inline hook with a Glitch.com project.
* Preview and test a registration inline hook for profile enrollment (SSR) and progressive profile enrollment.

**What you need**

* [Okta Developer Edition organization](https://developer.okta.com/signup/)
* [Glitch.com](https://glitch.com) project or account
* [A Profile Enrollment policy](https://help.okta.com/okta_help.htm?type=oie&id=ext-create-profile-enrollment)

**Sample code**

[Okta Registration Inline Hook Example](https://glitch.com/edit/#!/okta-inlinehook-registrationhook-oie)

---

## About registration inline hook implementation

In the following examples, the external service code parses requests from Okta and responds with commands that indicate the following:

* Whether the end user's email domain is valid and allowed to register (for profile enrollment)
* Whether the end user's employee number is valid and allowed to be added to their profile (for progressive profile enrollment)

In these examples, you set up your registration inline hook to handle either the simple profile enrollment (SSR) scenario, progressive profile enrollment scenario, or both, and configure a profile enrollment policy to define these scenarios and implement the registration hook. The registration hook customizes how to add new users to your okta org or update profiles of existing users.

The following is a high-level workflow for the profile enrollment (self-service registration) scenario:

1. An end user attempts to self-register with your Okta org.
1. A registration inline hook triggers during this process and sends a call to the external service with the end user's data.
1. The external service evaluates the Okta call to ensure that the end user is from domain `okta.com`.
1. The external service responds to Okta with a command to allow or deny the registration based on the email domain.

The following is a high-level workflow for a progressive profile enrollment scenario:

1. An existing registered end user attempts to sign in to their profile.
1. A profile enrollment policy presents a custom sign-in form that asks for additional data from the end user.
1. A registration inline hook triggers during this process and sends a call to the external service with the end user's data.
1. The external service responds to Okta with a command to allow or deny the addition of the new data to the end user's profile.

## Profile enrollment (self-service registration) requests

The following is an example of a JSON request received from Okta. The request properties contain data submitted by the end user who is trying to self register.

See the [request properties](/docs/reference/registration-hook/#objects-in-the-request-from-okta) of a registration inline hook for full details.

>**Note:** The `requestType` is `self.service.registration`.

```json
{
    "eventId": "04Dmt8BcT_aEgM",
    "eventTime": "2022-04-25T17:35:27.000Z",
    "eventType": "com.okta.user.pre-registration",
    "eventTypeVersion": "1.0",
    "contentType": "application/json",
    "cloudEventVersion": "0.1",
    "source": "regt4qeBKU29vSoPz0g3",
    "requestType": "self.service.registration",
    "data": {
        "context": {
            "request": {
                "method": "POST",
                "ipAddress": "127.0.0.1",
                "id": "123dummyId456",
                "url": {
                    "value": "/idp/idx/enroll/new"
                }
            }
        },
        "userProfile": {
            "firstName": "Rosario",
            "lastName": "Jones",
            "login": "rosario.jones@example.com",
            "email": "rosario.jones@example.com"
        },
        "action": "ALLOW"
    }
}
```

## Progressive profile enrollment requests

The following JSON example provides the end user's profile data to the external service for evaluation.

See the [request properties](/docs/reference/registration-hook/#objects-in-the-request-from-okta) of a registration inline hook for full details.

>**Note:** The `requestType` is `progressive.profile`.

```json
{
    "eventId": "vzYp_zMwQu2htIWRbNJdfw",
    "eventTime": "2022-04-25T04:04:41.000Z",
    "eventType": "com.okta.user.pre-registration",
    "eventTypeVersion": "1.0",
    "contentType": "application/json",
    "cloudEventVersion": "0.1",
    "source": "regt4qeBKU29vS",
    "requestType": "progressive.profile",
    "data": {
        "context": {
            "request": {
                "method": "POST",
                "ipAddress": "127.0.0.1",
                "id": "123dummyId456",
                "url": {
                    "value": "/idp/idx/enroll/update"
                }
            },
            "user": {
                "passwordChanged": "2022-01-01T00:00:00.000Z",
                "_links": {
                    "groups": {
                        "href": "/api/v1/users/00u48gwcu01WxvNol0g7/groups"
                    },
                    "factors": {
                        "href": "/api/v1/users/00u48gwcu01WxvNol0g7/factors"
                    }
                },
                "profile": {
                    "firstName": "Rosario",
                    "lastName": "Jones",
                    "timeZone": "America/Los_Angeles",
                    "login": "rosario.jones@example.com",
                    "locale": "en_US"
                },
                "id": "00u48gwcu01WxvNo"
            }
        },
        "action": "ALLOW",
        "userProfileUpdate": {
            "employeeNumber": "1234"
        }
    }
}
```

## Set up for profile enrollment (SSR) scenario

The simple scenario of profile enrollment (self-service registration) involves new users self-registering from the **Sign up** link with the default three sign-up fields (Email, First name, and Last name). With this use case, the registration inline hook triggers and evaluates the domain in the Email field. If the domain is from `okta.com`, the user can register. If not, the user is denied registration. To implement this scenario:

* Set up your Glitch project and review your external service code
* Activate and enable your registration inline hook
* Create an enrollment policy to use the registration inline hook

### Set up your external service code response for profile enrollment

You need to remix your own version of the Okta sample Glitch project and confirm that it is live.

1. Go to the [Okta Registration Inline Hook Example](https://glitch.com/edit/#!/okta-inlinehook-registrationhook-oie).
1. Click **Remix your own**.
1. Click **Share**.
1. In the **Live site** field, click the copy icon. This is your external service URL. Make a note of it as you need it later.
1. Review the project code with endpoint `/registrationHookSSR`.
1. Click **Logs**. If you see a "Your app is listening on port {XXXX}" message, the app is ready to receive Okta requests.

The external service responds to Okta indicating whether to accept the end user's self-registration. The response returns a `commands` object in the body of the HTTPS response. This object contains specific syntax that indicates whether the user is allowed or denied to self-register or to update their profile with Okta.

See the [response properties](/docs/reference/registration-hook/#response-objects-that-you-send) of a registration inline hook for full details.

```javascript
app.post('/registrationHookSSR', async (request, response) => {
  console.log();
  var returnValue = {};

  if (request.body.requestType === 'self.service.registration') {
    var emailRegistration = (request.body.data.userProfile['login']).split('@');
    if (emailRegistration[1].includes('okta.com')) {
      console.log(request.body.data.userProfile['firstName'] + " " + request.body.data.userProfile['lastName'] + " " + request.body.data.userProfile['email'] + " has registered!");
      returnValue = {
        'commands':[
          {
            type: 'com.okta.action.update',
            value: {
              'registration': 'ALLOW',
            }
          }
        ]
      };
    } else {
      console.log(request.body.data.userProfile['firstName'] + " " + request.body.data.userProfile['lastName'] + " " + request.body.data.userProfile['email'] + " denied registration!");
        returnValue = {
        'commands':[
          {
            type: 'com.okta.action.update',
            value: {
              'registration': 'DENY',
            },
          }
        ],
        'error': {
          'errorSummary':'Incorrect email address. Please contact your admin.',
          'errorCauses':[{
            'errorSummary':'Only okta.com emails can register.',
            'reason':'INVALID_EMAIL_DOMAIN',
            'locationType':'body',
            'location':'data.userProfile.email',
            'domain':'end-user'
          }]
        }
      };
    }
  }

 response.status(200).send(JSON.stringify(returnValue));
})
```

### Add your registration hook for profile enrollment

Configure your registration inline hook for your Okta org to use the glitch project for profile enrollment (SSR).

1. In the Admin Console, go to **Workflow** > **Inline Hooks**.
1. Click **Add Inline Hook** and select **Registration** from the dropdown menu.
1. Add a name for the hook (in this example, use "Profile Enrollment SSR").
1. Add your external service URL (see [Set up your Glitch project](#set-up-your-glitch-project)), and append it with the endpoint. For example, use your Glitch project name with the endpoint (`registrationHookSSR`):

   `https://your-glitch-projectname.glitch.me/registrationHookSSR`

1. Include the authentication field and secret. In this example:

   * **Authentication Field** = `authorization`
   * **Authorization Secret** = `Basic YWRtaW46c3VwZXJzZWNyZXQ=`

   > **Note**: If you want to use OAuth 2.0 to secure your inline hooks, see [Add Authentication method](/docs/guides/common-hook-set-up-steps/nodejs/main/#add-authentication-method).

1. Click **Save**.

> **Note:** You can also set up an inline hook using the API. See [Inline Hooks Management API](/docs/reference/api/inline-hooks/#create-inline-hook).

### Create an enrollment policy for profile enrollment (SSR)

To enable the registration inline hook, you must associate it with a Profile Enrollment policy. In this example, you create an enrollment policy specifically for your hook. See [enable and configure a Profile Enrollment policy](https://help.okta.com/okta_help.htm?type=oie&id=ext-create-profile-enrollment).

> **Note:** Profile Enrollment and registration inline hooks are only supported by the [Okta Sign-In Widget](/docs/guides/embedded-siw/) version 4.5 or later.

To associate the registration inline hook with a profile enrollment policy:

1. In the Admin Console, go to **Security > Profile Enrollment**.
1. Click **Add Profile Enrollment Policy**.
1. Give your policy a name (in this example, use "Inline Hook"), and then click **Save**.
1. From the list of Enrollment Policies, find **Inline Hook** and click the pencil icon.
1. Click **Manage Apps**, and then click **Add Apps to this Policy**.
1. Locate the **Okta Admin Console**, click **Apply**, and then click **Close**.
1. Click **Back to Profile Enrollment Policy**.
1. In **Profile enrollment**, click **Edit**.
1. For **Self-service registration**, select **Allowed**.
1. From the **Inline hook** dropdown menu, select the hook that you set up and activated earlier. See [Add your registration hook for profile enrollment](#add-your-registration-hook-for-profile-enrollment).

   > **Note:** You can associate only one inline hook at a time with your profile enrollment policy.

1. In **Run this hook**, select **When a new user is created**.
1. Click **Save**.

Your registration inline hook is configured for profile enrollment (self-service registration). Go to the [Preview the registration inline hook](#preview-the-registration-inline-hook) or [Test your registration inline hook](#test-your-registration-inline-hook) to preview and run the sample.

## Set up for progressive profile enrollment scenario

### External service code response for progressive profile

### Activate and enable your registration hook for progressive profile

## Set up for profile enrollment (SSR) and progressive profile enrollment scenario

### External service code response for profile and progressive enrollment

### Activate and enable your registration hook for profile and progressive enrollment


## Set up, activate, and enable

You must set up, activate, and enable the registration inline hook within your Okta Admin Console.

### Set up your Glitch project

You need to remix your own version of the Okta sample Glitch project and confirm that it is live.

1. Go to the [Okta Registration Inline Hook Example](https://glitch.com/~okta-inlinehook-registrationhook-v2).
1. Click **Remix your own**.
1. Click **Share**.
1. In the **Live site** field, click the copy icon. This is your external service URL. Make a note of it as you need it later.
1. Update the project with your own code. For example, you can use the [Send response](#send-response) sample in the `server.js` file.

### Set up and activate the registration inline hook

1. In the Admin Console, go to **Workflow** > **Inline Hooks**.
1. Click **Add Inline Hook** and select **Registration** from the dropdown menu.
1. Add a name for the hook (in this example, use "Guide Registration Hook Code").
1. Add your external service URL (see [Set up your Glitch project](#set-up-your-glitch-project)), and append it with the endpoint. For example, use your Glitch project name with the endpoint (`registrationHook`):

   `https://your-glitch-projectname.glitch.me/registrationHook`

1. Include the authentication field and secret. In this example:

   * **Authentication Field** = `authorization`
   * **Authorization Secret** = `Basic YWRtaW46c3VwZXJzZWNyZXQ=`

   > **Note**: If you want to use OAuth 2.0 to secure your inline hooks, see [Add Authentication method](/docs/guides/common-hook-set-up-steps/nodejs/main/#add-authentication-method).

1. Click **Save**.
1. In your Glitch project, click **Logs**. If your set up is successful, a "Your app is listening on port {XXXX}" message appears.

The registration inline hook is now set up with an active status.

> **Note:** You can also set up an inline hook using the API. See [Inline Hooks Management API](/docs/reference/api/inline-hooks/#create-inline-hook).

### Set up the employee number attribute

In the Progressive Enrollment example, end users are asked to submit a valid employee number. The `employeeNumber` attribute is read-only by default. You need to change `employeeNumber` to read-write.

1. In the Admin Console, go to **Directory** > **Profile Editor**.
1. Select **User (default)**.
1. Under **Attributes**, find **Employee Number** and click the information icon.
1. In the **Employee Number** dialog, under **User permission**, select **Read-Write**.
1. Click **Save Attribute**.

End users can now update the employee number in their profile.

### Enable the registration inline hook

To enable the registration inline hook, you must associate it with a Profile Enrollment policy. In this example, you create an enrollment policy specifically for your hook. See [enable and configure a Profile Enrollment policy](https://help.okta.com/okta_help.htm?type=oie&id=ext-create-profile-enrollment).

> **Note:** Profile Enrollment and registration inline hooks are only supported by the [Okta Sign-In Widget](/docs/guides/embedded-siw/) version 4.5 or later.

To associate the registration inline hook with a Profile Enrollment policy:

1. In the Admin Console, go to **Security > Profile Enrollment**.
1. Click **Add Profile Enrollment Policy**.
1. Give your policy a name (in this example, use "Inline Hook"), and then click **Save**.
1. From the list of Enrollment Policies, find **Inline Hook** and click the pencil icon.
1. Click **Manage Apps**, and then click **Add Apps to this Policy**.
1. Locate the **Okta Admin Console**, click **Apply**, and then click **Close**.
1. Click **Back to Profile Enrollment Policy**.
1. In **Profile enrollment**, click **Edit**.
1. For **Self-service registration**, select **Allowed**.
1. From the **Inline hook** dropdown menu, select the hook that you set up and activated earlier. See [Set up and activate the registration inline hook](#set-up-and-activate-the-registration-inline-hook).

   > **Note:** You can associate only one inline hook at a time with your Profile Enrollment policy.

1. In **Run this hook**, select **Both**. For our examples, this trigger allows both an SSR request and a Progressive Enrollment data update request.
1. Click **Save**.
1. Under **Profile Enrollment Form**, click **Add form input**.
1. From the dropdown menu, select **Employee number**.
1. In the **Add form input** dialog, under **Customize form input**, set **Input requirement** as **Optional**.
1. Click **Save**.

Your registration inline hook is configured for Profile Enrollment. You are now ready to preview and test the example.

## Preview the registration inline hook

Your Okta org is set up to call the sample external service using a registration inline hook, and the external service is ready to receive and respond to an Okta call.

In your Okta org, you can preview the request and response JSON in the Admin Console.

1. In the Admin Console, go to **Workflow** > **Inline Hooks**.
1. Select the registration inline hook name (in this example, select **Guide Registration Hook Code**).
1. Click **Preview**.
1. In the **Configure Inline Hook request** block, select an end user from your org in the **data.userProfile** field. That is, select a value from your `data.user.profile` object.
1. To test an SSR request, under **requestType**, select **Self-Service Registration**.
1. From the **Preview example Inline Hook request** block, select **Generate Request**.
   The end user's request information in JSON format, that is sent to the external service, appears.
1. Click **Edit** to update your request before previewing the response. For this example, change the email domain to `@example.com`.
1. From the **View service's response** block, click **View Response**.
   The response from your external service in JSON format appears, which indicates that self-registration was either allowed or denied.
1. To test a profile update, select an end user from your org in the **data.userProfile** field, and then under **requestType** select **Progressive Profile**.
1. From the **Preview example Inline Hook request** block, select **Generate Request**.
1. Click **Edit** to update `userProfileUpdate`:
   ```javascript
   "userProfileUpdate": {
      "employeeNumber": "1234"
      }
   ```
   > **Note:** Make sure that you delete the comma after the value you enter for an employee number. It isn't valid JSON otherwise.
1. From the **View service's response** block, click **View Response**.
   The response from your external service in JSON format appears, which indicates that the profile update was either allowed or denied.

## Test your registration inline hook

You can also test the code directly with self-registering or profile-updating end users.

### Test the SSR inline hook

To run a test of your SSR inline hook, go to the Okta sign-in page for your Okta org, click the **Sign Up** link, and attempt to self-register.
> **Note:** The **Employee number** field appears as optional. To test SSR, you can leave **Employee number** blank.

* If you use an allowable email domain, such as `rosario.jones@example.com`, the end user registration goes through.
* If you use an incorrect email domain, the end user registration is denied. Review the error message that displays the error summary from the external service code and is passed back to Okta. See [error](/docs/reference/registration-hook/#error).

### Test the progressive enrollment inline hook

To test the progressive enrollment inline hook, you need to make **Employee number** required and manually add a user with a password.

1. Sign in to the Okta Admin Console as an admin.
1. Go to **Security** > **Profile Enrollment**.
1. Under **Profile enrollment form**, find **Employee number**, and then click **Edit**.
1. Set the **Input requirement** to **Required**.
1. Click **Save**.
1. Go to **Directory** > **People**, and then click **Add person**.
1. Enter the credentials for your test user and select **I will set password**.
1. Enter a password, and then click **Save**.
1. Sign out from the Admin Console and sign in with your new `@example.com` credentials.

* If you use valid sign-in credentials, the **Employee number** field appears on the next page.
* If you enter an employee number in a valid format (4 digits), the update goes through.
* If you enter an employee number in an invalid format, the update is denied. Review the error message that displays the error summary from the external service code and is passed back to Okta. See [error](/docs/reference/registration-hook/#error).

> **Note:** Review [Troubleshooting hook implementations](/docs/guides/common-hook-set-up-steps/nodejs/main/#troubleshoot-hook-implementations) for help with any difficulties during setup or configuration.

## Next steps

Review the following guides to implement other inline or event hook examples:

* [Event hook](/docs/guides/event-hook-implementation/)
* [Password import inline hook](/docs/guides/password-import-inline-hook/)
* [Token inline hook](/docs/guides/token-inline-hook/)
* [Telephony inline hook](/docs/reference/telephony-hook/)

## See also

For a complete description of this inline hook type, see [Registration inline hook](/docs/reference/registration-hook/).