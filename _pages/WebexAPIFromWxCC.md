---
title: Webex API From WxCC
date: 2025-07-18
layout: post
---

# !!! Under Construction !!!

**Webex CC Integrations** are great for handling OAuth token management to your external API services, however, the methods which are required by Webex API, are not supported by WxCC Integrations.  That leads me to this article, where I am going to show an alternate method, completely contained within Webex, to **access the Webex APIs from within WxCC Flow Designer and WxConnect Flows as Custom Nodes**.

# Setup a Webex Service App

## Create the Service App

1. Go to [http://developer.webex.com/](http://developer.webex.com/) and sign in with your Webex account
2. Click on your Avatar and then select My Webex Apps
3. Create a new Service App
   1. Name it something cool like "Lazer Face", or something more tame like "WxCC Flow Access."  It's up to you!
   2. Select or upload your preferred icon
   3. Description is...stay with me here...where you describe what this integration does
   4. Contact email is whatever email you want to use
   5. The scopes are important and I cannot tell you what to check, but basically you need to know a few things:
      1. You cannot select all of them, there's a limit to it
      2. Each API endpoint doc page lists which scopes are required, so use that as your guide, to which and how many scopes to check off
         1. For me, I am going to only select `spark-admin:people_read`, so that I can lookup users in the directory
   6. Click to Add Service App
4. Copy your Client Secret to some place safe, as you'll need it in the future, and you can never get it again

## Authorize the Service App

1. Go to Control Hub [https://admin.webex.com/](https://admin.webex.com/)
2. Select Apps > Service Apps > Scroll the list and find "Lazer Face" or whatever you called yours
   * **TIP:** The search is by ID and not name, so if you search for "Lazer" and see nothing, that's why
3. After selecting the Service App, flip on the Authorize togggle

## Grab an Access Token and Refresh Token

1. At this point, you'll need to either refresh the Service App page you were on in the previous section, or just click off of it, and then back on to it, to see the new section titled "Org Authorizations"
2. Select your Org Name from the drop down list
3. Paste your Client Secret into the field below the org name, and generate tokens
4. Copy your Refresh and Access tokens to the same place you pasted your Client Secret
5. Grad your Client ID from this page while your at it, so you'll have four things in your notes:
   1. Client ID
   2. Client Secret
   3. Refresh Token
   4. Access Token

# Configure WxCC Flow Designer

Now let's move to the WxCC specific configurations, where we'll setup a Global Variable, Subflow, and Main flow.

## Store Access Token in WxCC Global Variable

When it comes to dynamic storage areas within WxCC, we don't have much, but we do have read/write capabilities with our Global Variables, so that's what we're going to use, to grab an access token, and also update it when it expires.

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
   1. Import the file from the download link
2. Now with the Flow Designer open, perform the following:
   1. Edit Flow variables with the values from your notes:
      1. `global_variable_id`
      2. `global_variable_name`
      3. `client_id`
      4. `client_secret`
      5. `refresh_token`
   2. Update the HTTP Request activity with name `UpdateGlobalVariable` to use your connector from the previous section
   3. Save, Validate & Publish

## Create a Main Flow

The main flow will use the Global Variable to gain access to the current access token, so that you can call Webex APIs, but the main flow will also trigger the sublow above, as needed, just in case we receive a 401 Unauthorized response from the API.

I have already created this main flow for you, we just need to import it, and tweak it a little.

1. Control Hub > Contact Center > Flows > Manage > Create > Import
   1. Import the file from the download link
2. Now with the Flow Designer open, perform the following:
   1. Update the flow variable `target_user` with an email address of a user in your Org
   2. Double check your Global Variable to make sure your Webex API Token one is in the flow, if not, add it
   3. Double check your Subflow activity to make sure it's your Subflow from above
      1. There should be one single output mapping from `access_token` to your Global Variable holding your access token
   4. Save, Validate & Publish

## Test

You should now be ready to attach this main flow to an Entry Point and phone number and then test it.

# Configure WxConnect Flow Custom Nodes