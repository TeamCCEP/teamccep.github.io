---
title: Enable Customer Journey Data Services (CJDS)
date: 2025-01-09
layout: post
---

> `Note:` Please refer to [Webex CC Labs](https://webexcc.github.io/pages/JDS_XM/#customer-journey-data-services-cjds) for better understanding of CJDS feature in Webex Contact Center.

### Quick Links

> Control Hub: **[https://admin.webex.com](https://admin.webex.com){:target="\_blank"}**\
> Portal: **[https://portal.wxcc-us1.cisco.com/](https://portal.wxcc-us1.cisco.com/){:target="\_blank"}**\
> Agent Desktop: **[https://desktop.wxcc-us1.cisco.com](https://desktop.wxcc-us1.cisco.com){:target="\_blank"}**\
> Developer Portal: **[https://developer.webex-cx.com/documentation/getting-started](https://developer.webex-cx.com/documentation/getting-started){:target="\_blank"}**\
> Feature Overview: **[https://app.vidcast.io/share/889c2cbf-51b2-4cc9-94f8-9143078dca83](https://app.vidcast.io/share/889c2cbf-51b2-4cc9-94f8-9143078dca83){:target="\_blank"}**\
> Example Use Case: **[https://app.vidcast.io/share/b90e50f4-d085-416c-9aae-29426fa18f53](https://app.vidcast.io/share/b90e50f4-d085-416c-9aae-29426fa18f53){:target="\_blank"}**\
> Readme: **[https://github.com/CiscoDevNet/cjaas-widgets/blob/main/CustomerJourney/README_VERSION_10.0.0.md](https://github.com/CiscoDevNet/cjaas-widgets/blob/main/CustomerJourney/README_VERSION_10.0.0.md){:target="\_blank"}**\
> JDS API Reference: **[https://developer.webex-cx.com/documentation/journey](https://developer.webex-cx.com/documentation/journey){:target="\_blank"}**\
> Adding Customer Identities: **[https://app.vidcast.io/share/4c4a5909-7770-4662-b25e-2f31b34f9c63](https://app.vidcast.io/share/4c4a5909-7770-4662-b25e-2f31b34f9c63){:target="\_blank"}**\
> JDS Contact Resolver Subflow: **[https://app.vidcast.io/share/cab5b686-9745-4394-98d4-b8b58c45f1a3](https://app.vidcast.io/share/cab5b686-9745-4394-98d4-b8b58c45f1a3){:target="\_blank"}**\
> Oauth Token Creation for Postman:**[https://app.vidcast.io/share/6b54f08d-0cd2-481f-979b-c8643c1d6a13](https://app.vidcast.io/share/6b54f08d-0cd2-481f-979b-c8643c1d6a13){:target="\_blank"}**\

## JDS Feature Overview

<div style="padding-bottom:60.25%; position:relative; display:block; width: 100%">
	<iframe src="https://app.vidcast.io/share/embed/889c2cbf-51b2-4cc9-94f8-9143078dca83" width="100%" height="100%" title="Introduction to Experience Management" frameborder="0" loading="lazy" allowfullscreen style="position:absolute; top:0; left: 0"></iframe>
</div>

CJDS is an API-first service that enables organizations to:

- `Listen`: Integrate with any data source or third-party applications to listen to disparate data sources (_e.g. customer
  called Support_).
- `Identify`: Create a dynamic customer profile capturing propensity drivers, such as a customer's preferred mode of communication or preferred language (_e.g. how many times has the customer contacted us in the last week in each channel?_).
- `Analyze`: Apply different aggregation techniques to all customer data collected (_e.g. What is the CSAT score of the customer interaction? Is he using telephony,chat or other channel to contact?_).
- `Act`: Use the data and insights within CJDS to dynamically change the flow within Webex Contact Center Flow Control and personalize the customer experience at a granular level. These insights are visible to customer-facing teams in real time through Agent Desktop. (_e.g. bypass normal queue when customer calls for the third time in 24 hours and offer premium support_).

A comprehensive summary of the feature is available in the [Developer Portal](https://developer.webex-cx.com/documentation/guides/journey---getting-started), where you can find all the vital information & step-by-step guide to enable JDS for the first time in your own tenant.

## Provision JDS

1. `Fill out` this [form](https://app.smartsheet.com/b/form/7776df72239e47d0bbb73a392e32927f) to have CJDS provisioned for your tenant enabled, as it **not** enabled by default. If applicable, please work together with your Customer Success Manager (CSM) to ensure a smooth process and enablement. Post the initial request, the Cisco team will provision the CJDS instance within 72 hours.

2. When JDS is provisioned for the tenant, `Customer Journey Data` tab appears in the Control Hub. **It will only be visible to full customer admins. Partner admins will not see this tab.**

3. You can set how long to retain data that's captured for CJDS. We recommend that you keep a minimum of 180 days to make sure that there are sufficient end-user journey data. By default, data is retained for 365 days.

   a. Sign in to Control Hub and go to `Customer Journey Data` > `Settings`.

   b. Enter the **number of days** that you want to retain data in CJDS.

4. Click `Save`.

![jdsprov](/assets/images/JDS/Provision_JDS.gif)

> `Note:` JDS is already provisioned in the provided lab tenant, so you do not need to perform them. You may follow the above steps when provisioning for your own tenant.

## Setup Desktop Customer Journey Widget

The customer Journey widget provides a single pane of glass view to the customer’s journey across all channels and applications, giving you the necessary contextual data for a more personalized customer experience and reducing average handling time.

### Create a journey project and Activate Webex Contact Center connector

Journey `projects` help organizations manage multiple data sources. Each journey project has a unique identifier which is required to leverage CJDS APIs payloads. Note that `project IDs` in Control Hub are referred to as `workspace IDs` in APIs.
Jouney project may be activated with the `Webex Contact Center connector`. This allows the journey project to capture all customer events and send it to CJDS from Contact Center. Once the connector is activated in one project, it cannot be activated in another. **At any point in time, it can be enabled for only one project.**

1. Sign in to **Control Hub** and go to `Customer Journey Data > Journey projects`.

2. You can use the default Sandbox Project or click `Create a Journey Project`.

3. Enter a **name** and a **description** for the journey project.

4. Select the journey `project` that you created in previous step.

5. Toggle the `Activate` connector in the Webex Contact Center section to ON.

![jdsactivateconnector](/assets/images/JDS/jds_activate_connector.gif)

### Add User Identities to a Journey Project

1. Select the journey `project` for which connector was activated.

2. Select `Identities`. Click on `Add identities`.

3. Download the sample **template**.

4. In the downloaded CSV file, add the Customer identities [First Name,Last Name,Email Addresses,Phone Numbers,Customers Ids].

   - Each customer identity must have **at least** an email address, phone number, or customer ID or else the CSV file will return an error.
   - If you want to add **multiple** email addresses, phone numbers or customer IDs, you need to **use the pipe “|” delimiter** between them. For example, try to add your phone number both with and without a plus sign.
   - For the **Id column**, make sure to leave each row `empty`. When you upload the CSV file, this field will auto-generate.

   ![jdscreatedcsv](/assets/images/JDS/jds_created_csv.png)

5. Upload the **CSV file** that you created for customer identities, and then click `Next`.

6. If the CSV file is valid, a window appears to show you if the import was successful. Once you're done, select `Close`. You should see a list of all the uploaded customer identities.

![jdsuploadidentities](/assets/images/JDS/jds_upload_ids.gif)

### Enable Customer Journey Widget on an Agent Desktop

1. We recommend adding JDS to your existing desktop layout using a code snippet. You can get the relevant code [here](https://github.com/CiscoDevNet/cjaas-widgets/blob/main/CustomerJourney/README_VERSION_10.0.0.md#how-to-add-the-cjds-widget-into-an-existing-desktop-layout).

Here is a screenshot of the block in place (notice it is after `IVR_TRASNCRIPT` and before `WXM_JOURNEY_TAB`).

![JDSWidgetCode](/assets/images/JDS/JDS_Widget_Code.png)

Alternatively, Download this [Desktop Layout JSON file](/assets/desktoplayouts/JDS10_AIAssistant.json) which includes the JDS widget and AI Assistant.

2. Sign in to Control Hub and go to `Contact Center > Desktop Layouts`.

3. Create a `New Layout`.

4. Assign an `Agent Team`.

5. Upload the Desktop Layout JSON file that you downloaded or created in **step 1**.

6. Click `Save`.

![jdsuploadwidget](/assets/images/JDS/jds_upload_widget.gif)

### View Customer Journey Widget on an Agent desktop

1. Login as an Agent into `Agent Desktop`. Ensure that the **desktop layout** of the Agent's Team has the JDS widget configured as per the previous steps.

2. On **Accepting** an incoming request, the `CJDS Widget` will appear in the desktop. If you are using AI Agent, it may load the IVR Transcript by default. Navigate to the 'Customer Journey' tab to see it.

The widget displays insights such as the number of times the customer has called or was contacted across all channels during a given duration. It also displays the journey history with default information such as `wrap-up code`, `queue name`, `agent ID` etc., and also allows to customize the event history with options such as third-party (non Webex CC) events and custom icons.

The `progressive profile` allows for alignment of different phone numbers and emails under one profile, ensuring accurate and comprehensive interaction data.

![JDSWidgetDesktop](/assets/images/JDS/jds_desktop_widget.gif)

Once JDS is set up and configured, we recommend using the <a href="https://github.com/TeamCCEP/JDSContactResolver" target="_blank">contact resolver subflow</a> to retrieve customer details into variables in existing flows. This allows you to do things like greet callers by name, channel switch to Email, SMS etc, and a variety of other activities based around knowing your customer.

There is a great <a href="https://github.com/WebexSamples/webex-contact-center-api-samples/tree/main/customer-journey-samples/cjds-postman-example" target="_blank">Postman Collection</a> available that gives many API's that you can use to show examples of overdue payment notifications in WxCC Agent desktop. You will need to set up postman with the relevant OAuth token using <a href="https://app.vidcast.io/share/6b54f08d-0cd2-481f-979b-c8643c1d6a13" target="_blank">this guide</a>

![Overdue Payment Notification](/assets/images/JDS/JDSPayments.png)
