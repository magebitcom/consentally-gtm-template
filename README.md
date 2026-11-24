# ConsentAlly for Google Tag Manager

A Google Tag Manager tag template that sets Google Consent Mode's defaults and loads the
ConsentAlly banner for your store, for sites that can only add scripts through Tag Manager.

If you can edit your site's `<head>`, use ConsentAlly's install tag there instead: it runs before
anything else on the page, so it can hold every third-party script, including those outside Tag
Manager. This template loads the same banner later, from inside your container.

Full guide: https://consentally.com/docs/google-tag-manager

## What the tag does

1. Sets ConsentAlly's Google developer ID (`developer_id.dY2MxZT`), then the Consent Mode
   defaults with `setDefaultConsentState`: one call per row of **Regional defaults** for the
   regions it lists (see below), then one without a region for all other regions, out of the box
   `ad_storage`, `ad_user_data`, `ad_personalization`, `analytics_storage`,
   `functionality_storage` and `personalization_storage` denied and `security_storage` granted.
   Each call has `wait_for_update` 500 ms. It also sets `ads_data_redaction` to `true`, and
   `url_passthrough` to `true` when you tick the box. When your **Global Privacy Control
   signal** variable is true, `ad_storage`, `ad_user_data` and `ad_personalization` are denied in
   every call, whatever a row grants (see below).
2. Sets the page global `__consentallyTpl`, which tells ConsentAlly's script that this tag set
   the defaults, then loads that script for your store from
   `https://edge.consentally.com/k/<store key>`, the only address it may load from. That script
   loads the banner with your published settings. When the banner cannot start (a malformed key,
   a script that fails to load, no published settings for the key, a banner that has not started
   10 seconds after that script started), every type but `security_storage` is updated to
   denied, so no grant from your defaults stays without a banner. When a Tag Manager policy on
   your site blocks this tag from writing `__consentallyTpl`, reading `ConsentAlly` or reading the
   `ca_state_<store key>` cookie, it sets no regional row and only the built-in defaults for all
   other regions, and says why in the browser console in Preview mode.
3. When the banner loads, it sends a Consent Mode update on the `dataLayer` with what your
   region rules in the ConsentAlly dashboard allow for this visitor (or the visitor's stored
   choice), which your container applies before its next event, and then the event
   `consentally_consent_update`, which tags waiting for consent can fire on (see below). Each
   choice the visitor makes sends both again. The script copies both, and the denial when the
   banner cannot start, to every other data layer a Google tag on the page reads, so a container
   with a renamed data layer gets them too.

Basic or advanced Consent Mode is a setting of your banner in the ConsentAlly dashboard, not of
this tag.

## Regional defaults

The tag's defaults apply until the ConsentAlly banner loads. Each row of the
**Regional defaults** table sets them for the regions it lists: ISO 3166-2 codes separated by
commas, a country such as `ES` or a state or province such as `US-CA`, and **Granted** or
**Denied** for each consent type. Google applies the most specific region, so a `US-CA` row wins
over a `US` row in California. Visitors from a region without a row get **Defaults for all other
regions**, out of the box every type denied but `security_storage`. The tag trims and upper-cases
the codes. The country must be on the ISO 3166-1 list (`GB`, not `UK`); a state or province is
checked by its format only, and a code Google does not recognise matches no visitor, so those
visitors get the defaults for all other regions. An invalid code is left out and a row with no
valid code is skipped, each with a note in the browser console in Preview mode.

Once the banner loads, it applies the region rules you set in the ConsentAlly dashboard, in every
region: its update replaces these defaults. If the banner cannot load, or has not started 10
seconds after its script started, every type is denied except `security_storage`.

The rows are for visitors who have not chosen yet. A visitor with a saved choice for your store
(the banner's `ca_state_<store key>` cookie, whatever it holds) gets no row: the tag sets only its
built-in defaults for all other regions, every type denied but `security_storage`, and the banner
applies the saved choice when it loads, so a visitor who refused does not start with a row's grants while that cookie is there (see Limits for a choice kept only in local storage).
The tag reads only that cookie. Its permission allows reading any cookie, since the name holds
your store key and Tag Manager's cookie permission takes exact names only; a Tag Manager policy
that blocks reading it gets the same built-in defaults.

The banner sets Google's types by your dashboard's categories, so it can repeat only defaults that
follow those categories:

| Consent type                                       | Once the banner loads                   |
| -------------------------------------------------- | --------------------------------------- |
| `security_storage`                                 | Always granted                          |
| `analytics_storage`                                | Analytics                               |
| `functionality_storage`, `personalization_storage` | Functional: both granted or both denied |
| `ad_storage`, `ad_user_data`, `ad_personalization` | Marketing: all granted or all denied    |

So set each row per category to match your ConsentAlly rules for its regions: `security_storage`
granted, the two functional types alike and the three ad types alike. Then nothing changes when
the banner takes over. A row that sets the types of one category differently, or denies
`security_storage`, holds only until the banner loads.

For example, a store whose banner asks first in Europe and only offers an opt-out in the United
States adds a row with the region `US` and every type **Granted**, and keeps the defaults for all
other regions denied. The tag then sets:

```js
// The row: every type granted in the United States.
setDefaultConsentState({
  ad_storage: "granted",
  ad_user_data: "granted",
  ad_personalization: "granted",
  analytics_storage: "granted",
  functionality_storage: "granted",
  personalization_storage: "granted",
  security_storage: "granted",
  wait_for_update: 500,
  region: ["US"],
});
// All other regions: every type denied but security_storage.
setDefaultConsentState({
  ad_storage: "denied",
  ad_user_data: "denied",
  ad_personalization: "denied",
  analytics_storage: "denied",
  functionality_storage: "denied",
  personalization_storage: "denied",
  security_storage: "granted",
  wait_for_update: 500,
});
```

Grant a type only in regions where your ConsentAlly rules do not ask for consent before tracking,
so the defaults and the banner agree.

### Global Privacy Control

A Tag Manager template cannot read the browser's Global Privacy Control signal
(`navigator.globalPrivacyControl`): its sandbox refuses access to `navigator`. To apply it in the
defaults, create a **Custom JavaScript** variable:

```js
function () {
  return navigator.globalPrivacyControl === true;
}
```

and choose it as the tag's **Global Privacy Control signal**. When it is true, the tag sets
`ad_storage`, `ad_user_data` and `ad_personalization` denied in every row and in the defaults for
all other regions, until the banner loads and applies your Global Privacy Control setting from
the ConsentAlly dashboard. Without the variable, the banner applies the signal when it loads.

ConsentAlly's script, loaded by this tag, sets no Google default of its own, so it never replaces
the tag's. Loaded from your page's `<head>` instead (the install tag), it sets every type denied
but `security_storage` for all regions. Only the page global this tag sets before loading it
(`__consentallyTpl`) makes the script leave Google's defaults to the tag: the same `/k/<store key>`
address loaded any other way, or the install tag with its `data-ca-key`, gets the script's own
defaults.

## Setup

1. In your container, open **Templates**, then **Search Gallery** under Tag Templates, and add
   **ConsentAlly**.
2. Add a tag of the type **ConsentAlly** and enter your **Store key**: the 16 characters from
   the script install page of your ConsentAlly dashboard (the value of `data-ca-key`).
3. Set its trigger to **Consent Initialization - All Pages** and publish the container.

Use the template or the install tag, not both. If both are on a page, the second copy of the
script does nothing but keep a deadline of its own. When the tag runs before the banner has
started, its regional defaults apply in their regions, the defaults for all other regions are
those of whichever sets them last, and the banner's update on load then applies your dashboard
rules over them in every region, as with the template alone; if the banner then cannot start,
every type but `security_storage` is denied, at the latest 10 seconds after the tag loaded its
copy of the script, even when the install tag's script had already given up before the tag ran.
When the banner has already started, the tag sets only its built-in default, every type denied but
`security_storage`, and no regional row, since the banner has already decided; a failure to load
its own copy of the script then changes nothing. The install tag's
script does not copy updates to a renamed data layer: in a container whose data layer is renamed,
with no Google tag reading `window.dataLayer`, a row's grants then stay for the page view whatever
the banner or the visitor decides, so keep the rows denied there or remove the install tag.

## Firing tags when the visitor decides

A tag whose consent check fails when its trigger fires does not run on that trigger, and a tag
that fires on **Initialization - All Pages** under a regional row that grants its consent type runs
before the banner has applied your rules. With this tag, the banner sends its update and the event
`consentally_consent_update` on every page view once it loads, so tags that need consent fire on
that event instead. Set the tag up like this:

1. In **Triggers**, choose **New**, then **Custom Event**. Enter `consentally_consent_update` as
   the event name, keep **All Custom Events** and save the trigger.
2. Open your Google tag. In **Advanced Settings**, under **Consent Settings**, choose **Require
   additional consent for tag to fire** and add `analytics_storage`.
3. Under **Tag firing options**, choose **Unlimited**. Not **Once per page**: Tag Manager counts the
   attempt its consent check blocked when the banner loaded as the page's one firing, so the tag
   would never run on the visitor's choice.
4. In **Triggering**, replace **Initialization - All Pages** with the new trigger, so a row that
   grants `analytics_storage` does not fire the tag a second time when the banner loads. Save the
   tag and publish the container.

The tag then sends one page view on each page view where your rules or the visitor allow analytics.
A visitor who allows analytics and then, on the same page, allows more sends a second page view for
that page. If the banner cannot load, every type is denied, so a tag that requires consent does
not fire.

Any other tag that waits for consent works the same way: require its consent type and add the
trigger.

## Limits

- The copies to a renamed data layer start from the `dataLayer` the script found when it loaded:
  a page script that later replaces `window.dataLayer` with a new array stops them.
- Tags in the container that fire before the script has loaded are not held by it: give them
  consent checks in their **Consent Settings**. Google's own tags read Consent Mode either way.
- A page whose Content Security Policy uses nonces gets no nonce on the injected script.
- The tag finds a saved choice by its cookie only. The banner also keeps the choice in the
  browser's local storage, which a template cannot read under a name that holds the store key;
  when only that copy is left, the page starts with the regional rows until the banner loads,
  applies the choice and writes the cookie again, and every page does while the browser refuses
  that cookie. Where a saved refusal has to hold before the banner loads, keep the types it covers
  denied in the rows.
- A Custom JavaScript variable for the Global Privacy Control field cannot run under a Content
  Security Policy without `unsafe-eval`: Tag Manager returns `undefined`, which the tag reads as
  no signal. Check its value in Preview mode.
- The 10 seconds start when ConsentAlly's script starts. A request for that script that never
  completes, neither loading nor failing, leaves a row's grants until the page is left: a Tag
  Manager template has no timer API to time it, the same for any consent template that grants a
  default.

## Releasing

Until the gallery lists the template, consentally.com offers this file for import
(https://consentally.com/files/ca-tag-template.tpl): in a container, **Templates**, **New** under
Tag Templates, the menu at the top right, **Import**.

The template's source and tests live in ConsentAlly's own repository (`integrations/gtm-template`),
with the site's copy for that download in `apps/marketing/files/ca-tag-template.tpl`, which a test
holds to this `template.tpl`. A version that needs a newer ConsentAlly script than the edge serves
reaches the site and the gallery only after that script is released;
the public repository holds `template.tpl`, `metadata.yaml`, `LICENSE` and this file at its root.
To release a version, commit the new `template.tpl` there, then add that commit's full SHA at the
top of `versions` in `metadata.yaml` with a short `changeNotes`, and commit that. The gallery
shows the update within two to three days.

If ConsentAlly's edge is rolled back to a script from before this version, the banner still
starts: that script reads the store key from the same `/k/<store key>` address, ignores
`__consentallyTpl` and sets its own default, every type denied but `security_storage`, which
replaces only the defaults for all other regions. A regional row's grants stay until the banner's
update, which a current banner always sends on load, because it reads `__consentallyTpl` itself; a
banner from before this version may leave that update out where the rules ask first, so while the
template is published, never roll both the script and the banner back past this version. That older script
neither denies the types when the banner cannot start nor copies updates to a renamed data layer,
so until the script is released again a row's grants can stay for the page view when the banner
fails to start, and in a container whose data layer is renamed, with no Google tag reading
`window.dataLayer`, whatever the visitor chooses.

## Licence

Apache License 2.0, see [LICENSE](LICENSE).
