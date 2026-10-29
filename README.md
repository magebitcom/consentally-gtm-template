# ConsentAlly for Google Tag Manager

A Google Tag Manager tag template that sets Google Consent Mode's defaults and loads the
ConsentAlly banner for your store, for sites that can only add scripts through Tag Manager.

If you can edit your site's `<head>`, use ConsentAlly's install tag there instead: it runs before
anything else on the page, so it can hold every third-party script, including those outside Tag
Manager. This template loads the same banner later, from inside your container.

Full guide: https://consentally.com/docs/google-tag-manager

## What the tag does

1. Sets the Consent Mode defaults with `setDefaultConsentState`: `ad_storage`, `ad_user_data`,
   `ad_personalization`, `analytics_storage`, `functionality_storage` and
   `personalization_storage` denied, `security_storage` granted, `wait_for_update` 500 ms. It also
   sets `ads_data_redaction` to `true`, and `url_passthrough` to `true` when you tick the box.
2. Loads ConsentAlly's script for your store from `https://edge.consentally.com/k/<store key>`,
   the only address it may load from. That script loads the banner with your published settings.
3. When the visitor decides, the banner sends a Consent Mode update on the `dataLayer`, which
   your container applies before its next event.

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

## Limits

- The container uses the standard `dataLayer` name. With a renamed data layer the banner's
  updates do not reach the container.
- Tags in the container that fire before the script has loaded are not held by it: give them
  consent checks in their **Consent Settings**. Google's own tags read Consent Mode either way.
- A page whose Content Security Policy uses nonces gets no nonce on the injected script.

## Licence

Apache License 2.0, see [LICENSE](LICENSE).
