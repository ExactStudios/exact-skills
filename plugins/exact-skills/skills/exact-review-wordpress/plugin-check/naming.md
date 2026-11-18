# Plugin name, owner and description

Source: Plugin Check 2.2.0, `prompts/ai-check-prereview.md` and `prompts/ai-check-prereview-output.md`, rewritten. The prereview prompt ends with an "Output Format: disallowed" block whose identifiers ("core-supports", "script-insertion") appear nowhere else in it; it is left out. The similar-name check (`ai-check-similar-name.md`) is left out because it needs a web search.

Categories: WP.org 17 (name, slug, owner), WP.org 9 (superlative claims in descriptions), WP.org 12 (keyword stuffing in the name or description).

## Inputs

- Display name: the `Plugin Name` header and the `=== Name ===` line of `readme.txt`. They should resemble each other; check both.
- Slug: the plugin folder name.
- Owner: the `Author` and `Author URI` headers, the readme `Contributors`, and the domain of any `Plugin URI`. A brand that matches the owner is the author's own.
- Description: the `Description` header, the readme short description and the `== Description ==` section.

## Name rules

The name passes only when every rule below passes.

1. **Too generic.** Not allowed: one or a few words that don't say what the plugin does, or a description of the function so broad that it seems to cover every case and doesn't name its context. An invented, distinctive term is the exception, and it goes at the start of the name.
   For a UPS shipping tracker integrated with WooCommerce: "Shipping" is too generic. "Ecommerce Shipping" and "Ecommerce Tracker" describe it but miss the UPS context. "ShipGlex Shipping" is probably fine (invented term). "Shipping Tracker for UPS" is probably fine (says the context; WooCommerce is implied).
2. **Unrelated to the plugin.** Compare the name with the description. An invented term is again the exception.
3. **Keyword stuffing.** The name repeats a keyword or near-synonyms, or reads as a keyword list.
4. **Trademarks and project names the author doesn't own.** Treat any distinctive term you don't recognize as a possible trademark too.
   - Fine when the owner matches the brand (an integration plugin for the author's own service), or when the term comes after a phrase that shows no affiliation: "for", "with", or a "-" separator.
   - Wrong at the start of the name, or anywhere without such a phrase.
   - Portmanteaus count: a name ending in "-Press" plays on "WordPress".
   - Examples for an author at nicedev.com: "WooCommerce Pricing Rates" is wrong. "Pricing Rates" is fine. "Nicedev Pricing Rates" is probably fine. "Bank of Germany Pricing Rates" is wrong (implies an official institution). "Pricing Rates for WooCommerce" is fine. "Locaki" is probably wrong (an original-looking name that doesn't match the author). "Pricing Rates for WhatsApp" is wrong (banned). "Pricing Rates for WP" is wrong (discouraged). "PricingPress" is wrong (portmanteau). "Paypal Payment Gateway for WooCommerce" is wrong, and so is "Nicedev Paypal for WooCommerce"; "Nicedev Payment Gateway with Paypal for WooCommerce" is right.
5. **Banned and discouraged terms** (lists below) are not allowed anywhere in the name or slug, not even after "for" or "with". "for WordPress" at the end is still wrong: the plugin is in the WordPress directory, so saying so is redundant. "PRT Text editor for WP" and "PRT Text editor for WordPress" should both become "PRT Text editor".

## Description rules

- WP.org 9: no claims of being better than other plugins, the best, the first or the only solution: "better than…", "best plugin for…", "#1 plugin for…", "the most … plugin for…", "first option for users", "the only plugin for…". The same words are fine when not comparing the plugin with others.
- WP.org 12: no keyword stuffing, meaning repeated keywords or keyword lists.

## Banned terms

- Meta, whose lawyers don't permit their marks in plugin names, slugs or banners: Facebook (FB, fbook), WhatsApp (WA, Whats, vvhatsapp, vva, wh4tsapps, Whats App), Instagram (Insta, Gram, INS), Threads, Oculus.
- WordPress Foundation: WordPress (wordpess, wpress).
- Other brands: Trustpilot, Binance Pay.

## Discouraged terms

- "Plugin" when it only says the plugin is a plugin ("SEO Plugin"), and always as the first word.
- Superlatives: best, #1, first, perfect, the most.
- "Free" when redundant ("(free)", "free" on its own); every directory plugin is free.
- "WP" meaning WordPress, anywhere: as a standalone word, in "for WP", "W P", at the start or the end. Allowed only as part of a bigger word that is a different trademark. Plugin Check lists this under both banned and discouraged; it is kept here once.
- Gutenberg (gberg, guten, berg). The editor is called the block editor.

## Other trademarks and project names

Allowed only by rule 4: owned by the author, or after "for", "with" or "-". A reference list, not complete.

- Automattic: WooCommerce (Woo, WC, W C, WooCom, vvcommerce, vvoo, ووکامرس), Akismet, Jetpack.
- Google: Google Analytics (GA4, ganalytics), Google Maps, Google, AdSense, AdWords, Android, Chrome, Gmail, GDrive, YouTube (YT).
- Apple: Apple, Apple Pay, macOS, iOS, iPhone, iPad, Macintosh.
- Microsoft: Microsoft, Bing, Windows, LinkedIn (LinkIN), GitHub, Internet Explorer, Edge, Skype.
- Other brands: Adobe, Amazon (AMZ, azon, AWS, CloudFront, Route53), Booking.com, Bootstrap, OpenAI (Sora, ChatGPT, Chat GPT), DeepSeek (Deep Seek), Cloudflare (Turnstile), cPanel, WHMCS, Disqus, Dropbox, Envato, FedEx, Firefox, Font Awesome, GTmetrix (GTM), HubSpot, Mailchimp, Klaviyo, MailerLite, SendPulse, MailWizz, Matomo, OnlyFans (Only Fans), Opera, Paddle, PayPal, Pinterest, Stripe, Slack, TikTok (tik tok), Twitch, Twitter, Bluesky, Discord, Telegram, UPS, USPS, Watson, Yandex (Яндекс), Yahoo, DPD, Rive, Aparat, Bitcoin Lightning, Bitcoin, Rutube, bKash, Nagad, M-Pesa, Stax, SSLCommerz, Odoo, Aliyun Bailian (Aliyun), Alibaba Cloud (Alibaba), Satim, Sendcloud, Payinn, Web Monetization, Sentry.
- WordPress plugins: Contact Form 7 (CF7), Advanced Custom Fields (ACF), bbPress, BuddyPress, Divi, Dokan, Easy Digital Downloads (EDD), Elementor (Eleme), GiveWP, Gravity Forms (GF), Ninja Forms, Yoast SEO (Yoast), Polylang, WPML, WPMLS, All in One (AIO), WPForms, PixelYourSite (PYS, Pixel Your Site), Beaver Builder, Tutor LMS, BuddyBoss, Content Views, SearchWP, TranslatePress, Rank Math, Pods, WPS, Sentinel, Easy Table of Contents (EasyTOC), Ultimate Addons.
- Other projects: jQuery (j query), TinyMCE (tiny MCE).

## Recording a finding

- Flag only clear cases; Plugin Check's own rule is that a person checks every flag afterwards. Use High confidence for a banned term or a trademark at the start of the name, Medium for generic, unrelated or possible-trademark cases.
- Severity: High for a banned term or a trademark used as if official; Medium for a discouraged term, a generic or unrelated name, keyword stuffing, or a superlative claim.
- Say what the issue is and which rule it breaks. Name the trademarks you found in the display name.
- Fix: suggest a display name and, if the slug has the same issue, a slug.
  - Put trademarks the author doesn't own at the end, after "for" or "with"; several go as "for X and/with Y and/with Z". Or leave them out.
  - Never use a banned or discouraged term.
  - If the result is too generic, start it with the company, brand or author's username, or an invented term.
  - Don't suggest a name close to a well-known plugin, even with extra words.
  - Slug: derived from the name (it can be shorter), lowercase, no special characters, spaces as "-", at most 50 characters.
