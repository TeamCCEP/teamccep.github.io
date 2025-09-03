---
title: Webex API From WxCC
date: 2025-07-18
layout: post
---

**Webex CC Integrations** are great for handling OAuth token management to your external API services, however, the methods which are required by Webex API, are not supported by WxCC Integrations.  That leads me to this article, where I am going to show an alternate method, completely contained within Webex, to **access the Webex APIs from within WxCC Flow Designer**, oh and as a bonus, **from WxConnect Flows as Custom Nodes** as well!

# Setup a Webex Service App

## Create the Service App

1. Go to [http://developer.webex.com/](http://developer.webex.com/) and sign in with your Webex account
2. Click on your Avatar and then select My Webex Apps
3. Create a new Service App
   1. Name it something cool like "Taserface", or something more tame like "WxCC Flow Access."  It's up to you!
   2. Select or upload your preferred icon
   3. Description is...stay with me here...where you describe what this integration does
   4. Contact email is whatever email you want to use
   5. The scopes are important and I cannot tell you what to check, but basically you need to know a few things:
      1. You cannot select all of them, there's a limit to it
      2. Each API endpoint doc page lists which scopes are required, so use that as your guide, to which and how many scopes to check off
         1. For me, I am going to only select `spark-admin:people_read`, so that I can lookup users in the directory
   6. Click to Add Service App
4. Copy your Client ID and Secret to your notes

## Authorize the Service App

1. Go to Control Hub [https://admin.webex.com/](https://admin.webex.com/)
2. Select Apps > Service Apps > Scroll the list and find "Taser Face" or whatever you called yours
   * **TIP:** The search is by ID and not name, so if you search for "Taser" and see nothing, that's why
3. After selecting the Service App, flip on the Authorize togggle

## Grab an Access Token and Refresh Token

1. At this point, you'll need to either refresh the Service App page you were on in the previous section, or just click off of it, and then back on to it, to see the new section titled "Org Authorizations"
   1. If you don't see "Org Authorizations", click refresh again
2. Select your Org Name from the drop down list
3. Paste your Client Secret into the field below the org name, and generate tokens
4. Copy your Refresh and Access tokens to your notes

# Configure WxCC Flow Designer

Now let's move to the WxCC specific configurations, where we'll setup a Global Variable, Subflow, and Main flow.

## Store Access Token in WxCC Global Variable

When it comes to dynamic storage areas within WxCC, we don't have much, but we do have read/write capabilities with our Global Variables, so that's what we're going to use, to grab the current access token, and also update it when it expires.

1. Control Hub > Contact Center > Flow > Global Variables > Create
   1. Name is something like "WEBEX_ACCESS_TOKEN"
   2. Variable type is String
   3. Default value is your copied Access Token
   4. Save
2. Click back into your newly created Global Variable and copy its Name and ID from the top of the page and put it in your notes

## Enable WxCC API Integration

Since we will be updating the Global Variable dynamically, it will require us to access the WxCC API from within a Flow, and this is not on by default.  If you already have a read/write integration already created, you can ignore this step, and we'll just re-use what you already have.

1. Control Hub > Contact Center > Integrations > Webex Contact Center > Add Connector
   1. Name like "WxCC API Access"
   2. Read-Write is required for us
   3. I authorize
   4. Click Add

## Create a Subflow to Refresh the Token

While you do have the valid access token now available to your flows via the Global Variable, it will eventually expire, and then your API calls will start to fail.

What we need is, a subflow, so it's reusable by all Flows, that will refresh our access token for us, and then update the Global Variable dynamically.  The subflow will update the Global Variable but it will also return the access token as an output variable, so the already executing flow can obtain the new value immediately.

I have already created this subflow for you, we just need to import it, and tweak it a little.

1. Control Hub > Contact Center > Flows > Subflows > Manage > Create > Import
   1. Import the file from this [folder](https://github.com/TeamCCEP/teamccep.github.io/tree/master/assets/files/WebexAPIFromWxCC)
2. Now with the Flow Designer open, perform the following:
   1. Edit Flow variable `params` with the values from your notes
   2. Update the HTTP Request activity with name `UpdateGlobalVariable` to use your connector from the previous section
   3. Save, Validate & Publish

## Create a Main Flow

The main flow will use the Global Variable to gain access to the current access token, so that you can call Webex APIs, but the main flow will also trigger the sublow above, as needed, just in case we receive a 401 Unauthorized response from the API.

I have already created this main flow for you, we just need to import it, and tweak it a little.

1. Control Hub > Contact Center > Flows > Manage > Create > Import
   1. Import the file from this [folder](https://github.com/TeamCCEP/teamccep.github.io/tree/master/assets/files/WebexAPIFromWxCC)
2. Now with the Flow Designer open, perform the following:
   1. Update the flow variable `target_user` with an email address of a user in your Org
   2. Double check your Global Variable to make sure your's is in the flow, and not mine: the one that came along with the import.
      1. Most likely, you will need to swap out the Global Variable in the flow with your own, as the export/import of Global Variables is a bit weird, in that the system will let you import a flow which references a Global Variable which doesn't exist.
      2. This means that you will need to double check three activities:
         1. Your HTTP Request activity named `GetPersonDetails` to use your Global Variable name as the Bearer token.
         2. Your Subflow activity named `RefreshTheToken` to map your Global Variable
            1. There should be one single output mapping from `access_token` to your Global Variable holding your access token
         3. Your Condition activity named `HaveAccessToken` to check your Global Variable
         4. Then delete the Global Variable that came along with the import if you did all of that work to update it to your own
   3. Save, Validate & Publish

## Test

You should now be ready to attach this main flow to an Entry Point and phone number and then test it.

# Configure WxConnect Flow Custom Nodes

We are going to borrow the work we did in the section **Setup a Webex Service App**.  It's ok that WxCC and WxConnect use the same integration; they will each get their own tokens, and that's ok, but feel free to create two separate integrations if that's more your style.

Buckle up though, as this process is not easy your first time through.  Though, once you do create this, and it works, you should be able to enahance it beyond the example I lead you through.

## Create a Custom Node

1. Control Hub > Contact Center > Webex Connect > Assets > Integrations > Add Integration > Custom Node
   1. Name is something like "Webex API"
   2. Rest API
   3. Description as you see fit
   4. Node Category as you see fit
   5. Creation Type from blank
   6. SVG for Node icon ([there's one here](https://github.com/TeamCCEP/teamccep.github.io/tree/master/assets/files/WebexAPIFromWxCC) you can use)

Now, this next page you see is both the integration details (e.g., client ID and secret), but also the API call too (aka Request Method).  The way WxConenct works is, each API call you would like to make, needs it's own integration detail added, and each API call will obtain its own token.  You are allowed 25 API calls per custom node.

## Add a Request Method

I will replicate the same person lookup as we did for WxCC flow.

1. Request Name is "GetPersonByEmail" or similar
2. Request and Conenction Timeout is 2000ms or thereabouts
3. Type is GET
4. Resource URL is `https://webexapis.com/v1/people`
5. Authorization
   1. Type is OAuth 2.0
   2. Grant Type is Authorization Code
   3. Consumer ID is your Client ID
   4. Consumer Secret is your Client Secret
   5. Authorization URL is `https://webexapis.com/v1/authorize`
   6. Scope is `spark-admin:people_read spark:kms`
      1. You may notice the `spark:kms` is new, but that's not specific to WxConnect, it's actually a hidden scope, that was just hidden from us previously, but we need to know about now.
   7. Access Token URL is `https://webexapis.com/v1/access_token`
   8. Refresh Token URL is `https://webexapis.com/v1/access_token`
   9. Click the Access Token button
   10. A window should pop-up for you to complete the auth
   11. Confirm you now have an access token, and refresh token
   12. Scroll down to URL Parameters and add two parameters:
       1.  Parameter is `email` for one and `callingData` for the other
       2.  Parameter Type is "Dynamic" for `email` and "Static" for `callingData`
       3.  Field Name is "email" for `email`
       4.  Parameter Value is `true` for `callingData`
   13. Scroll down to Response section and add two Node Events (like node outcomes)
       1.  Node Name is "Success" for one, and "Error" for the other
       2.  Switch Body to HTTP Status for both
       3.  Condition is "equals" for Success and "not equals" for Error
       4.  Value is `200` for both
       5.  Node Edge is "Success" and "Error" respectively
   14. Still in the Response section, let's add one response object
       1.  Parameter name is "Person" is similar
       2.  Body is Body
       3.  Response Path is `$.items[0]`
           1.  This means that if there is a returned person from the API call, the customer node will have an output variable called `person` that you can work with in the flow.
   15. Click Save
   16. [Optional] Click Test to perform a quick test
       1.  Select Method Name from Drop Down
       2.  Enter the email of one of your users
       3.  Click Test and observe the output
   17. Click to go back by "Manage Custom Node" header at the top of the page
   18. Looking at the list of integrations, notice that your new integration is not toggled on right now, so you need to toggle it on, so that it's visible in your WxConnect Flows.
6.  

_Note: If you need to make more than just this one People lookup API call, you will need to repeat steps 1 - 15 again for each API call you configure._

## Build a WxConnect Flow

We'll build a simple Webhook based flow for testing the integration.

1. WxConnect > Services > [choose any service or create a new one] > Flows > Create Flow
2. Flow Name is "Testing Webex API" or similar
3. Method is New Flow
4. Select "Start from Scratch"
5. Click Create
6. Select Webhook as the Trigger
7. Choose "Crete new event" radio button to create a new webhook URL
   1. Copy this URL for your notes, we'll need it in a minute
8. Name is "Testing Webex API" or similar
9. Click Save
10. Drag in your custom node (mine is called Webex API and has the Webex icon)
11. Connect the Configure Webhook start node to it
12. Double-click the Webex API node to open its properties
    1.  Select the Method Name
    2.  Enter the email address of one of your users
    3.  Click Save to close the properties
13. Drag in an Evaluate node
14. Connect the custom node to it
15. Double-click the Evaluate node to open its properties
    1.  Use the following code in the node:

    ~~~ javascript
    const person_json = JSON.parse("$(n3.person)");
    const person_name = person_json.displayName;
    const person_status = person_json.status;

    const debug_data = JSON.stringify({
        "person_name": person_name,
        "person_status": person_status,
        "person_json": person_json
    });

    0;
    ~~~

    2. Script Output is `0`
    3. Branch Name is "Success"
    4. Click Save to close the properties
16. Click Make Live and wait until its Live
17. Open Postman, or a similar app, and issue a POST request to your Webhook URL, with an empty JSON body of `{}`
18. Go back to your Flow and click on the debugger, then click decrypt logs
19. Find your transaction (there will likely only be one) and select it
20. Look at your Evalute node's outcome and you should see the `debug_data` variable populated with data from the API call (you'll see some other variables there too)
