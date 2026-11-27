# App Review for messaging strangers (Instagram and Messenger)

Researched 2026-11-27 against primary Meta sources only: developers.facebook.com docs (their `.md` mirrors under `/documentation/...` and the `/docs/...` pages) and the Meta Business Help Center. Builds on `research/messaging-without-webhooks`, `research/policies-and-naming` and `research/app-setup-and-oauth`; it does not repeat them.

Context: every user brings their own Business-type app, runs it with Standard Access, and touches only their own Page and Instagram account. Live-confirmed before this research: in development mode `GET /{page}/conversations?platform=instagram` silently omits threads with people who hold no app role; private replies to a no-role commenter do send; `HUMAN_AGENT` to a role holder is accepted before review. Messenger with a stranger is untested.

## Short answer

Reaching DMs from people without a role needs Advanced Access for `instagram_manage_messages` / `pages_messaging`, and Advanced Access needs App Review plus Business Verification of a registered legal entity. Meta has a lighter "own business" track for Instagram messaging (verification details optional, credentials left blank), but three things in the docs work against a webhook-free CLI: the Instagram review checklist says "Ensure your app can process webhooks correctly"; the Instagram Platform overview limits apps with "no user interface" to `instagram_basic` and `instagram_manage_comments`; and every messaging screencast must show a Facebook Login flow. Messenger is the exception that the docs disagree on: the Messenger overview and a January 2024 changelog say own-Page messaging needs no review, while the Conversations API, the access-levels row and error 200/2018028 say non-role users need review (and a Live app).

## 1. The "own business" App Review track

### What the Instagram own-business guide asks for

Source: [Apps For Your Own Business](https://developers.facebook.com/documentation/business-messaging/instagram-messaging/app-review/apps-for-your-own-business.md). Scope: "If your app is an automated experience or a custom inbox solution that can only process messages sent to your own Instagram Professional account, follow these guidelines".

- Screen recording. "Show the following: How to initiate a conversation with your app; How your app processes a message; How your app processes quick replies, ice breakers, or generic templates, if you have implemented them; How your app processes rich media; How your app processes messages the sender unsent." "Make sure that the app you are recording is the same app that you are submitting for review, otherwise your submission will be rejected."
- Verification details: "Providing verification details is optional." Platform: "Set the platform to **Website** ... and use your business's website URL." Credentials: "Leave this section blank."
- `instagram_manage_messages` description: "Tell us if your app is an automated experience, a custom inbox solution, or both"; "how to initiate a conversation with your app"; command list if automated. Plus "Enter your Instagram profile handle" and a Primary experience choice: **Automated** or **Live Agent** ("a custom inbox solution that agents can use to manually process (read, reply, delete, and so on) direct messages"). Live Agent is the fit for a human-driven CLI.
- `business_management`: call it out as "a dependency for the `pages_messaging` and `pages_show_list` permissions" and for `instagram_manage_messages`, and "In the app review screencast, clearly show the Page admin navigating the Facebook Login flow, picking the pages or Instagram handles and providing the necessary permissions to the app." Also "In your app, make sure to inform the Page admin that they need to explicitly provide permissions".
- Human Agent (optional): "Only request this feature if you have a custom inbox solution to process human agent escalations and your agents need up to 7 days to respond". Description: "Explain why it can take someone up to 7 days to reply to a direct message instead of 24 hours." Recording: reuse the `instagram_manage_messages` one if submitted together.

The parent page [App Review for Messenger API support for Instagram Apps](https://developers.facebook.com/documentation/business-messaging/instagram-messaging/app-review.md) adds a pre-submission checklist, including "Ensure your app can process webhooks correctly" and "Ensure your app can digest webhooks and use their contents to send and receive messages via the Send API". It applies to both tracks. A pull-only app cannot tick it truthfully; whether reviewers enforce it for a Live Agent inbox is not stated.

### What the permissions reference asks for, per permission

From the [Permissions Reference](https://developers.facebook.com/docs/permissions/reference) ("What to include in App Review submission" column):

- `instagram_manage_messages`: use case "Explain the messaging functionality your app offers to business customers who are onboarded onto the platform, and describe how they perform these functions." Screencast: "Demonstrate the complete Facebook login process on your app platform, showing how your app user grants your app this permission"; "Demonstrate your app sending a message to an Instagram user, and demonstrate the Instagram inbox client (either web or mobile app) receiving and displaying the sent message"; "Generate a cURL request that you can integrate into your app platform to send a message; You may use the API Integration Helper in Meta App Dashboard > Instagram to do this"; "Make sure it is a recording of the message being sent from the app to the user, instead of sharing screenshots".
- `pages_messaging`: screencast "Demonstrate the complete Facebook login process"; then "Demonstrate one of the following": inbound/outbound calls, or "Demonstrate your app sending a message to a Messenger user, and demonstrate the Messenger inbox client ... receiving and displaying the sent message", plus the same cURL / API Integration Helper item. The use-case text on this row reads "Provide specific examples of why your app requires creating and managing ads on behalf of other businesses for their pages", which looks copied from an ads permission.
- `instagram_manage_comments`: allowed usage "Read, update and delete comments of Instagram Business accounts." Its use-case and screencast cells describe `instagram_content_publish` ("Demonstrate creating a new photo post..."), another copy error.
- `pages_manage_metadata`: allowed usage "Subscribe to receive webhooks of your Page" and "Update settings of your Page"; screencast "Demonstrate how your app user subscribes to webhook events or updates the settings of their Facebook Page on your app platform". A pull-only CLI never subscribes; it would have to show a Page settings update.
- Dependencies must be submitted too: "If a permission or feature is dependent on another permission, you must include this permission in your submission." ([App Review for Instagram API](https://developers.facebook.com/documentation/instagram-platform/app-review.md))
- Private replies to strangers need "The `instagram_manage_comments` and `pages_messaging` permissions", "The Human Agent feature" and "Advanced Access" ([IG private replies](https://developers.facebook.com/documentation/business-messaging/instagram-messaging/features/private-replies.md)).

### Generic submission rules

From [App Review submission walkthrough](https://developers.facebook.com/documentation/resp-plat-initiatives/individual-processes/app-review/submission-guide.md):

- "Make at least 1 successful API call using each permission for which you are requesting advanced access. Calls must be made within 30 days of submitting"; the button "will remain grayed out until a successful API has been logged"; logging takes up to 2 days.
- Recordings: "Use English as the app UI language"; "provide captions and tool-tips" if not in English or not self-explanatory; "record in high-resolution, 1080 or better"; show "an app user granting your app each permission"; "Omit audio; our reviewers will not listen to it."; "When possible, use your mouse to interact with your app instead of your keyboard. If we can't see what you're doing, we can't test it!"
- "Each permission and feature must have its own description. **Do not copy and paste.**"
- App Purpose: "Set this to **Yourself or your own business** if your app is only available to people who have a role on your app, or a role in a Business that has claimed your app. Otherwise set it to **Clients**."
- "We will test your app using our own test accounts. **Do not include your _personal_ Meta Technologies app account's credentials.**"
- [Screen recordings guide](https://developers.facebook.com/docs/app-review/submission-guide/screen-recordings): "Before you start a recording, log out of any test accounts you are using." "Capture the entire login flow, from logged-out to logged-in." "Capture the complete authorization flow." "We recommend that you use annotations for all of the permissions and features that you are requesting."

### Does a CLI with no web UI pose a known problem?

Yes, in two documented places, plus a general one:

- [Instagram Platform overview](https://developers.facebook.com/documentation/instagram-platform/overview.md), "Private apps": "If reviewers are unable to test your app because it is behind a private intranet, has no user interface, or has not implemented Facebook Login for Business, you can request approval only for the following permissions: `instagram_basic`, `instagram_manage_comments`". Read literally, `instagram_manage_messages` is not available to an app reviewers cannot test. Whether a terminal counts as a "user interface" is not defined.
- [Common mistakes](https://developers.facebook.com/docs/app-review/submission-guide/common-mistakes): "If we can't access your app for any reason, your entire submission will be rejected"; "If we are unable to locate Facebook Login, or are unable to log into your app with Facebook Login" the whole submission is rejected.
- [App Review intro](https://developers.facebook.com/docs/app-review/introduction): "If we are unable to access your app to test it, your entire submission will be rejected."

What works for a CLI: the own-business track tells you to leave credentials blank, so reviewers test by messaging your Instagram handle, not by running your app. The screencast can show the CLI's browser OAuth flow (Facebook Login for Business), a stranger's DM arriving via the CLI, the CLI replying, and the reply in the Instagram app. The permissions reference explicitly accepts a cURL request as part of the messaging screencast. No doc mentions command-line, terminal or server-to-server apps.

### Turnaround and rejection reasons

- "It typically takes us less than one week to process your submission, and often takes only 2–3 days, but may take longer during peak periods." ([intro](https://developers.facebook.com/docs/app-review/introduction))
- Submission limits: 3 in progress at once, 15 per rolling 30 days per developer, 30 per business portfolio ([App Review process](https://developers.facebook.com/documentation/resp-plat-initiatives/individual-processes/app-review.md)).
- [Common mistakes](https://developers.facebook.com/docs/app-review/submission-guide/common-mistakes): "Your app is inaccessible"; "A screen recording of a permission or feature is missing"; "You are still developing your app"; "You are requesting a permission or feature you might need later"; "Facebook Login is not working or can't be located"; "You used a fake account". Also: recording a different app than the one submitted ([own-business guide](https://developers.facebook.com/documentation/business-messaging/instagram-messaging/app-review/apps-for-your-own-business.md)); requesting permissions "that your app does not use or does not align with the allowed usage" ([IG App Review](https://developers.facebook.com/documentation/instagram-platform/app-review.md)); "Selecting unneeded permissions is a common reason for rejection" ([Permissions Reference](https://developers.facebook.com/docs/permissions/reference)).

### Messenger equivalent

There is no Messenger own-business guide. The [Messenger Platform app review](https://developers.facebook.com/documentation/business-messaging/messenger-platform/app-review.md) page is written for public bots: "Publish the Facebook Page associated with your app"; "Ensure your webhook is functioning as expected and returning a `200 OK` response to webhook events within 20 seconds"; for gated functionality "provide a username and password in the submission notes" or "a trigger phrase". Its "Next steps" assume "release your app to the public".

## 2. Business Verification

- Required for Advanced Access: "Business Verification is required to get Advanced Access." ([Access Levels](https://developers.facebook.com/documentation/development/graph-api/overview/access-levels.md)); "Verification is required to obtain Advanced Access for any permission on Meta for Developers." ([About business verification](https://www.facebook.com/business/help/1095661473946872)); "Starting in 2023, your business must go through business verification and be verified in order to complete app review." ([Complete your App Review submission](https://developers.facebook.com/documentation/resp-plat-initiatives/individual-processes/app-review/content.md)). The Instagram overview: Advanced Access "requires App Review and Business Verification." It is therefore required for the own-business messaging track too: the track only grants Advanced Access, and no doc exempts it.
- Hedged wording to note: the access-levels and BV pages say apps "may have to be connected to a verified business" / "you might need to complete Business Verification". No doc lists permissions that are exempt.
- Who: "Ensure your business is a legal entity. Your business needs to be registered with local authorities and you'll need an official phone number or mailing address for your business." "Not all businesses need to or have the option to complete verification"; Security Center shows "Start verification" or "Ineligible for verification" ([About business verification](https://www.facebook.com/business/help/1095661473946872)). Only "Business portfolio owners with full control can verify" ([How to verify](https://www.facebook.com/business/help/2058515294227817)). The docs never mention sole proprietors or individuals. An unregistered individual cannot meet "registered with local authorities"; a registered sole proprietorship appears to qualify, but no page says so.
- Documents: "Certificate/Articles of Incorporation"; "Business Registration or License Document"; "Government Issued Business Tax Document ... Self-filed tax documents are not accepted"; "Business Bank Statement"; "Utility Bill" (address/phone only, not legal name). Must show the legal name and the address or phone, be unexpired and in a supported language ([Official documents](https://www.facebook.com/business/help/159334372093366)).
- Process: in Meta Business Suite > Security Center. Enter legal name, address, phone and website; "make sure your business website loads and is HTTPS compliant"; confirm by email, phone, SMS, WhatsApp or domain verification; "A decision on your verification submission may take up to 14 business days." Editing business details later means verifying again ([How to verify](https://www.facebook.com/business/help/2058515294227817)). A website that "fails to load, is not HTTPS compliant ... or contains a link that leads to an error" is a listed rejection reason ([Troubleshoot](https://www.facebook.com/business/help/2342133782492969)).
- The app must be connected to the verified portfolio: "Apps that request advanced access for permissions ... must be connected to a Business that has completed Business Verification." Connect it at App Dashboard > Settings > Basic > Verification; "anyone with an Administrator role on your app can connect it to a Business, but only someone with an Admin role in the Business will be able to complete the verification process." ([Business Verification](https://developers.facebook.com/documentation/development/release/business-verification.md)). Once connected, roles are managed in Business Manager ([App Roles](https://developers.facebook.com/documentation/development/build-and-test/app-roles.md)).
- Access Verification (Tech Provider) "is no longer required" ([Access Verification](https://developers.facebook.com/documentation/development/release/access-verification.md)).

## 3. After approval: Development vs Live, and ongoing obligations

### Does Advanced Access work without going Live?

The docs conflict:

- Business apps have no modes: "Business apps do not have app modes and rely exclusively on access levels." ([Publish](https://developers.facebook.com/documentation/development/release.md)); "apps that have been typed as Business apps do not have app modes and instead rely on access levels. To release new reviewable functionality for a Business app ... request Advanced Access" ([intro](https://developers.facebook.com/docs/app-review/introduction)); "Make sure your app is in Development mode or is a Business app type." (same).
- Use-case apps must be published: "To allow people, who do not have a role on your app, to use your app, you need to publish it"; "To publish your use case app go to the app dashboard **Publish > Go live**" ([Publish](https://developers.facebook.com/documentation/development/release.md)). Instagram: "You're required to connect your app to a business that has completed Business Verification before you can publish your app." ([Create an Instagram app](https://developers.facebook.com/documentation/instagram-platform/create-an-instagram-app.md))
- Messenger error table: `200` / `200-2018028` "Cannot message users who are not admins, developers or testers of the app until pages_messaging permission is reviewed and the app is live." ([Error codes](https://developers.facebook.com/documentation/business-messaging/messenger-platform/error-codes.md))
- DUC FAQ: "Apps in Dev Mode cannot access personal data of users who do not have a role on the app." ([DUC FAQ](https://developers.facebook.com/documentation/resp-plat-initiatives/individual-processes/data-use-checkup/faq.md))

The social-kit apps are Business-type apps created through use cases, and `research/app-setup-and-oauth` leaves them unpublished. So both statements could apply. Plan for having to publish; confirm after approval.

### What going Live requires

[Basic Settings](https://developers.facebook.com/documentation/development/create-an-app/app-dashboard/basic-settings.md) marks each of these "required to switch your app to Live mode": Display Name, Contact Email, Terms of Service URL, App Icon, Category, App Purpose. Privacy Policy URL and User Data Deletion URL are described without that tag, but Platform Terms §4.a requires the privacy URL in the dashboard (see `research/policies-and-naming`). "While verification is not required to Go Live, you will not be able to access data you do not own until verification is complete." DUC: developers "will need to complete DUC before the app can be switched to Live mode" ([DUC](https://developers.facebook.com/documentation/resp-plat-initiatives/individual-processes/data-use-checkup.md)). Webhooks also need a published app (already covered in `messaging-without-webhooks`).

### Ongoing obligations

- Data Use Checkup, annual: "required for developers whose apps have been published live with a use case, or have advanced access to permissions or features"; includes data handling questions and "Provide testing instructions for your application" ([DUC](https://developers.facebook.com/documentation/resp-plat-initiatives/individual-processes/data-use-checkup.md)). Notice 60 days ahead, reminders at 30/10/3 days ([DUC FAQ](https://developers.facebook.com/documentation/resp-plat-initiatives/individual-processes/data-use-checkup/faq.md)). Conflict: the same FAQ says "Only Live Mode apps require Data Use Checkup", while the access-levels page says "Apps that have Advanced Access for a permission or feature must complete Data Use Checkup".
- Data Protection Assessment: "a requirement for apps accessing advanced permissions", annual when enrolled, 60 days to answer ([Maintaining Data Access](https://developers.facebook.com/documentation/development/maintaining-data-access.md), [DPA](https://developers.facebook.com/documentation/resp-plat-initiatives/individual-processes/data-protection-assessment.md)).
- Ongoing reviews can happen "at any time", using the test instructions under Basic settings; "Remove any unused permissions from apps in the App Review tab"; reviewers check the privacy policy ([Ongoing Review best practices](https://developers.facebook.com/documentation/resp-plat-initiatives/individual-processes/compliance-checks/tutorial.md)).
- Removing access: trash icon under App Review > Permissions and Features; "Restoring Advanced Access to previously approved permissions or features does not require re-review"; "changing from Advanced to Standard will invalidate/deactivate any permission/feature for any app users who do not have a role" ([Access Levels](https://developers.facebook.com/documentation/development/graph-api/overview/access-levels.md)).
- Re-review triggers: "Any changes you make to your app's basic or advanced settings once you've submitted your app for App Review may require re-review before taking effect." ([intro](https://developers.facebook.com/docs/app-review/introduction)). Inactive apps (90 days with no logins, Graph calls or webhooks) lose tokens, and "any permissions that were removed from the app due to disuse while it was inactive must be re-approved through the App Review process" ([Maintaining Data Access](https://developers.facebook.com/documentation/development/maintaining-data-access.md)).

## 4. Policy URLs at review time

[Privacy Policy Expectations](https://developers.facebook.com/documentation/development/terms-and-policies/privacy-policy.md):

- Content: "what data is collected and how it is used, explain the purposes for which the data is being processed, and how users may request deletion of that data"; deletion route "may be an email or a contact form, which should be up to date and valid".
- "The linked page must be clearly marked as a privacy policy."
- "The privacy policy must be your own privacy policy, and not the policy of another company."
- Live, clickable, crawler-accessible, not geo-blocked; "Broken privacy policy links are considered violations"; check it returns 200 in the Sharing Debugger.
- Ongoing reviews: "Host your privacy policy on a publicly available link owned and managed by your organization." ([Ongoing Review best practices](https://developers.facebook.com/documentation/resp-plat-initiatives/individual-processes/compliance-checks/tutorial.md))

Applied to social-kit: no doc mentions github.com or says the policy must name the app. A page in the user's own GitHub repo or GitHub Pages site is arguably "owned and managed" by them. Linking to social-kit's own PRIVACY.md conflicts with "your own privacy policy, and not the policy of another company" and with "owned and managed by your organization". The safe pattern is for each user to publish a copy of the template under their own control, naming their business. The business website from Business Verification must be HTTPS and load anyway, so it can host the policy.

Terms of Service URL: required "to switch your app to Live mode" ([Basic Settings](https://developers.facebook.com/documentation/development/create-an-app/app-dashboard/basic-settings.md)). The App Review "Complete App Settings" step lists App Icon, Privacy Policy URL, App Purpose, App Category and Primary Contact, with no ToS ([submission walkthrough](https://developers.facebook.com/documentation/resp-plat-initiatives/individual-processes/app-review/submission-guide.md); same list in [IG App Review](https://developers.facebook.com/documentation/instagram-platform/app-review.md)). So ToS is needed to go Live, not to submit. The walkthrough also asks for a "1024x1024 compliant app icon".

## 5. Detecting the access level from the API

- `GET /{app-id}/permissions`: "Permissions that have been approved via Login Review", with a `status` filter whose values are `live` and `unapproved` ([Application permissions edge](https://developers.facebook.com/docs/graph-api/reference/application/permissions/)). It is the only documented review-status read. The page does not mention access levels, so whether Advanced Access shows as `live` needs a live check.
- Application node: no documented field for app mode, access level, business-verification status or review status ([Application reference](https://developers.facebook.com/docs/graph-api/reference/application/)). The `roles` edge reads app roles.
- `debug_token` returns `scopes`, `granular_scopes`, `is_valid`, `expires_at`, `data_access_expires_at`, and nothing about access level or mode ([debug_token](https://developers.facebook.com/docs/graph-api/reference/debug_token/)). A Standard Access token for a role user carries the same scopes.
- Error codes ([Messenger error codes](https://developers.facebook.com/documentation/business-messaging/messenger-platform/error-codes.md)):
  - `200`, and `200` / `2018028`: "Cannot message users who are not admins, developers or testers of the app until pages_messaging permission is reviewed and the app is live." The table lists 2018028 under code 200, not code 10. This is the only documented "no role" send error.
  - `10` / `2534022`: "This message is sent outside of allowed window." It is about the 24-hour window, not about access.
  - `551`: "User Block Error: This person isn't receiving messages from you right now." (`551` / `1545041`: "This person isn't available right now.") The [send guide](https://developers.facebook.com/documentation/business-messaging/messenger-platform/send-messages.md) adds: "The person may have blocked your Page, deactivated their account, or is otherwise unreachable." Not access-related.
  - `230`: not in Meta's Messenger error table or the Graph error-handling page. Docs are silent.
  - `1` / `99` "An unknown error occurred": "Missing permissions can cause this issue. Your app may need Advanced Access, Business Verification, or permissions".
  - Graph generic: `10` "API Permission Denied" and `200-299` "API Permission": "Permission is either not granted or has been removed." ([Error handling](https://developers.facebook.com/docs/graph-api/guides/error-handling))
- Reads produce no error at all: live testing showed no-role Instagram threads are silently omitted, and the docs describe no error for that case. Detection by error therefore only works on sends.

## 6. Instagram API with Instagram Login

- Same requirement: Advanced Access "requires App Review and Business Verification" ([Instagram Platform overview](https://developers.facebook.com/documentation/instagram-platform/overview.md)). [App Review for Instagram API](https://developers.facebook.com/documentation/instagram-platform/app-review.md) lists `instagram_business_manage_messages` and Human Agent as reviewable for Instagram Login. Its table puts "My app is only for a business I own or manage" at "Standard Access / Not required", for both login types.
- Different track: there is no own-business guide for Instagram Login. Submission starts from "Instagram > API setup with Instagram login > Complete app review". Required items include "Confirm that your app can be loaded and tested externally", "Verify that the login button or link is visible in your app and screencast", and verification details: "Provide detailed step-by-step instructions for Meta reviewers to log in and test your app" with credentials if needed. "Web or mobile Web is the only platform that currently supports Instagram API with Instagram Login." This is heavier for a CLI than the Facebook Login own-business track, which says leave credentials blank.
- Human Agent: "If you have requested the `instagram_business_manage_messages` permission, the Human Agent feature is added automatically." ([Create an Instagram app](https://developers.facebook.com/documentation/instagram-platform/create-an-instagram-app.md))
- Screencast for `instagram_business_manage_messages`: "Demonstrate the complete Instagram login process", a message sent and shown in the Instagram inbox, and "Demonstrate generating a cURL request ... You may use the API Integration Helper in Meta App Dashboard > Instagram" ([Permissions Reference](https://developers.facebook.com/docs/permissions/reference)).
- Different wording on Standard Access: the Instagram Login [Messaging API](https://developers.facebook.com/documentation/instagram-platform/instagram-api-with-instagram-login/messaging-api.md) and [Conversations API](https://developers.facebook.com/documentation/instagram-platform/instagram-api-with-instagram-login/conversations-api.md) pages frame access only by whose account is served: "Standard Access if your app serves Instagram professional accounts you own or manage". Unlike the Facebook Login pages, they never say Standard Access "can only interact with people who have a role on the app". Testers still "must have a role on your app". Whether the Instagram Login path also hides no-role threads under Standard Access is undocumented and worth a live check.
- Migration: an app moving from Facebook Login "might automatically be granted Advanced Access for the corresponding Instagram API permission" ([IG App Review](https://developers.facebook.com/documentation/instagram-platform/app-review.md)).
- The overview now lists a third setup, "Instagram API with Unified Login". Its review rules were not researched here.

## 7. Messenger (`pages_messaging`) on your own Page

The two quotes, both on the [Messenger overview](https://developers.facebook.com/documentation/business-messaging/messenger-platform/overview.md):

- Before you begin: "App Review (if your app needs Advanced Access). Not required if you only send and receive messages for your own Facebook Page." and "Business Verification (if your app is used by people without a Role on the app)".
- Key concepts, Access levels: "**Standard Access** (default) limits data to users with a Role on your app or Page. **Advanced Access** extends data access to all app users and requires App Review."

Supporting the "no review" reading: [Messenger changelog, January 10th, 2024](https://developers.facebook.com/documentation/business-messaging/messenger-platform/changelog.md), "Messenger app review update": "If you are developing the app to send and receive messages for your own page, you will not have to go through app review. Note that if you are planning to fetch user profile information, you still need to go through app review for the permissions required for accessing the User Profile API."

Against it:

- [Conversations API](https://developers.facebook.com/documentation/business-messaging/messenger-platform/conversations.md): "Advanced Access is required to access conversations between your business and people who do not have a role on your messaging app, your Instagram Professional account, your Facebook Page, or your business".
- Error `200-2018028`: "Cannot message users who are not admins, developers or testers of the app until pages_messaging permission is reviewed and the app is live."

Reconciliation: the "app users" in the access-levels row are the people who grant the app permissions (Page admins), not the people writing to the Page. Read that way, the overview says that an app used only by its own Page admin needs no review, which is true of all Standard Access. The 2024 changelog says more ("send and receive messages for your own page") and is the strongest evidence that Messenger strangers are reachable without review. The Conversations API page and error 2018028 say they are not. The docs do not settle it. A live check is cheap: have a no-role Facebook account message the Page, then list `/{page}/conversations?platform=messenger` and send one `RESPONSE` reply. If the reply returns 200/2018028, review is needed.

## Conflicts found

1. Messenger own Page without review (overview, 2024 changelog) vs. Advanced Access for non-role conversations (Conversations API) and error 200/2018028.
2. "Business apps do not have app modes" vs. "publish your use case app" / "app is live" in error 2018028 / "Apps in Dev Mode cannot access personal data of users who do not have a role" (DUC FAQ).
3. DUC needed for any Advanced Access (Access Levels, DUC page) vs. "Only Live Mode apps require Data Use Checkup" (DUC FAQ).
4. The Instagram review checklist requires webhook processing; the own-business guide and the permissions reference do not mention it.
5. "Private apps" with no UI are limited to `instagram_basic` + `instagram_manage_comments`, vs. the own-business track that needs no reviewer login.
6. Private replies: the docs require Advanced Access and say "Standard Access apps can only access data for people who have a role on the app", yet live testing sent a private reply to a no-role commenter under Standard Access.
7. Copy errors in the Permissions Reference: the `pages_messaging` use-case text is about ads, and the `instagram_manage_comments` cells describe content publishing.
8. Business Verification "is required" (Access Levels, Help Center) vs. "might need" / "may have to" (BV page, access-levels banner).

## Unknown until a live check or a real submission

- Messenger: does a stranger's thread appear, and does a `RESPONSE` reply send, under Standard Access? (live check)
- Does `/{app-id}/permissions?status=live` reflect Advanced Access for Business-type apps? (live check after approval; before approval, record what it returns)
- After approval, does Advanced Access reach strangers while the app stays unpublished, or must it go Live? (only after an approval)
- Will reviewers accept a CLI plus terminal screencast for `instagram_manage_messages`, given "no user interface" and the webhook checklist item? (only a submission tells)
- Can a sole proprietor or an individual pass Business Verification in a given country? (Security Center shows "Start verification" or "Ineligible")
- Does the Instagram Login path hide no-role threads under Standard Access the way the Facebook Login path does? (live check)
- What does error 230 mean in messaging? (undocumented)

## Sources

- https://developers.facebook.com/documentation/business-messaging/instagram-messaging/app-review/apps-for-your-own-business.md
- https://developers.facebook.com/documentation/business-messaging/instagram-messaging/app-review/apps-for-other-businesses.md
- https://developers.facebook.com/documentation/business-messaging/instagram-messaging/app-review.md
- https://developers.facebook.com/documentation/business-messaging/messenger-platform/app-review.md
- https://developers.facebook.com/documentation/business-messaging/messenger-platform/overview.md
- https://developers.facebook.com/documentation/business-messaging/messenger-platform/changelog.md
- https://developers.facebook.com/documentation/business-messaging/messenger-platform/conversations.md
- https://developers.facebook.com/documentation/business-messaging/messenger-platform/error-codes.md
- https://developers.facebook.com/documentation/business-messaging/messenger-platform/send-messages.md
- https://developers.facebook.com/documentation/business-messaging/instagram-messaging/overview.md
- https://developers.facebook.com/documentation/business-messaging/instagram-messaging/features/private-replies.md
- https://developers.facebook.com/docs/permissions/reference
- https://developers.facebook.com/docs/app-review/introduction
- https://developers.facebook.com/docs/app-review/submission-guide/screen-recordings
- https://developers.facebook.com/docs/app-review/submission-guide/common-mistakes
- https://developers.facebook.com/documentation/resp-plat-initiatives/individual-processes/app-review.md
- https://developers.facebook.com/documentation/resp-plat-initiatives/individual-processes/app-review/submission-guide.md
- https://developers.facebook.com/documentation/resp-plat-initiatives/individual-processes/app-review/content.md
- https://developers.facebook.com/documentation/resp-plat-initiatives/individual-processes/data-use-checkup.md
- https://developers.facebook.com/documentation/resp-plat-initiatives/individual-processes/data-use-checkup/faq.md
- https://developers.facebook.com/documentation/resp-plat-initiatives/individual-processes/data-protection-assessment.md
- https://developers.facebook.com/documentation/resp-plat-initiatives/individual-processes/data-handling-questions.md
- https://developers.facebook.com/documentation/resp-plat-initiatives/individual-processes/compliance-checks/tutorial.md
- https://developers.facebook.com/documentation/development/graph-api/overview/access-levels.md
- https://developers.facebook.com/documentation/development/build-and-test/app-modes.md
- https://developers.facebook.com/documentation/development/build-and-test/app-roles.md
- https://developers.facebook.com/documentation/development/release.md
- https://developers.facebook.com/documentation/development/release/business-verification.md
- https://developers.facebook.com/documentation/development/release/access-verification.md
- https://developers.facebook.com/documentation/development/create-an-app/app-dashboard/basic-settings.md
- https://developers.facebook.com/documentation/development/maintaining-data-access.md
- https://developers.facebook.com/documentation/development/terms-and-policies/privacy-policy.md
- https://developers.facebook.com/documentation/instagram-platform/overview.md
- https://developers.facebook.com/documentation/instagram-platform/app-review.md
- https://developers.facebook.com/documentation/instagram-platform/create-an-instagram-app.md
- https://developers.facebook.com/documentation/instagram-platform/instagram-api-with-instagram-login/messaging-api.md
- https://developers.facebook.com/documentation/instagram-platform/instagram-api-with-instagram-login/conversations-api.md
- https://developers.facebook.com/docs/graph-api/reference/application/
- https://developers.facebook.com/docs/graph-api/reference/application/permissions/
- https://developers.facebook.com/docs/graph-api/reference/debug_token/
- https://developers.facebook.com/docs/graph-api/guides/error-handling
- https://www.facebook.com/business/help/1095661473946872
- https://www.facebook.com/business/help/2058515294227817
- https://www.facebook.com/business/help/159334372093366
- https://www.facebook.com/business/help/2342133782492969
