# ConsentAlly for Google Tag Manager

A Google Tag Manager tag template that sets Google Consent Mode's defaults and loads the
ConsentAlly banner for your store, for sites that can only add scripts through Tag Manager.

If you can edit your site's `<head>`, use ConsentAlly's install tag there instead: it runs before
anything else on the page, so it can hold every third-party script, including those outside Tag
Manager. This template loads the same banner later, from inside your container.

Full guide: https://consentally.com/docs/google-tag-manager

## What the tag does

1. Sets ConsentAlly's Google developer ID (`developer_id.dY2MxZT`), then the Consent Mode
   defaults with `setDefaultConsentState`: `ad_storage`, `ad_user_data`,
   `ad_personalization`, `analytics_storage`, `functionality_storage` and
   `personalization_storage` denied, `security_storage` granted, `wait_for_update` 500 ms. It also
   sets `ads_data_redaction` to `true`, and `url_passthrough` to `true` when you tick the box.
2. Loads ConsentAlly's script for your store from `https://edge.consentally.com/k/<store key>`,
   the only address it may load from. That script loads the banner with your published settings.
3. When the visitor decides, the banner sends a Consent Mode update on the `dataLayer`, which
   your container applies before its next event, and then the event
   `consentally_consent_update`, which tags waiting for consent can fire on (see below). A later
   page view with the visitor's stored choice sends both again.

Basic or advanced Consent Mode is a setting of your banner in the ConsentAlly dashboard, not of
this tag.

## Setup

1. In your container, open **Templates**, then **Search Gallery** under Tag Templates, and add
   **ConsentAlly**.
2. Add a tag of the type **ConsentAlly** and enter your **Store key**: the 16 characters from
   the script install page of your ConsentAlly dashboard (the value of `data-ca-key`).
3. Set its trigger to **Consent Initialization - All Pages** and publish the container.

Use the template or the install tag, not both. If both are on a page, the second copy of the
script does nothing, and both set the same defaults.

## Firing tags when the visitor decides

A tag whose consent check fails when its trigger fires does not run on that trigger. In basic
mode, a Google tag that fires only on **Initialization - All Pages** sends its page view before the
visitor decides; ConsentAlly holds that request and it is not sent again, so the page where the
visitor accepts records no page view. Set the tag up like this:

1. In **Triggers**, choose **New**, then **Custom Event**. Enter `consentally_consent_update` as
   the event name, keep **All Custom Events** and save the trigger.
2. Open your Google tag. In **Advanced Settings**, under **Consent Settings**, choose **Require
   additional consent for tag to fire** and add `analytics_storage`.
3. Under **Tag firing options**, choose **Unlimited**. Not **Once per page**: Tag Manager counts the
   attempt its consent check blocked on Initialization as the page's one firing, so the tag would
   never run on the new trigger.
4. In **Triggering**, keep **Initialization - All Pages** and add the new trigger. Save the tag and
   publish the container.

The tag then sends one page view on each page view where analytics is allowed. A visitor who allows
analytics and then, on the same page, allows more sends a second page view for that page.

Any other tag that waits for consent works the same way: require its consent type and add the
trigger.

## Limits

- The container uses the standard `dataLayer` name. With a renamed data layer the banner's
  updates do not reach the container.
- Tags in the container that fire before the script has loaded are not held by it: give them
  consent checks in their **Consent Settings**. Google's own tags read Consent Mode either way.
- A page whose Content Security Policy uses nonces gets no nonce on the injected script.

## Releasing

Until the gallery lists the template, consentally.com offers this file for import
(https://consentally.com/files/ca-tag-template.tpl): in a container, **Templates**, **New** under
Tag Templates, the menu at the top right, **Import**.

The template's source and tests live in ConsentAlly's own repository (`integrations/gtm-template`),
with the site's copy for that download in `apps/marketing/files/ca-tag-template.tpl`, which a test
holds to this `template.tpl`;
the public repository holds `template.tpl`, `metadata.yaml`, `LICENSE` and this file at its root.
To release a version, commit the new `template.tpl` there, then add that commit's full SHA at the
top of `versions` in `metadata.yaml` with a short `changeNotes`, and commit that. The gallery
shows the update within two to three days.

## Licence

Apache License 2.0, see [LICENSE](LICENSE).
