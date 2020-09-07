---
layout: post
title: Community Dynamic Branded Login Page
---

Salesforce Communities is becoming more mature with every release as a unified business portal. While Communities branding and customizations have been improving with custom templates, Community Builder, and Lightning components, the Login functionality has been fairly basic.

Most Portal UX customers need to present a totally custom self-registration and Login experience. A standard Login page or OAuth login flow allows for minor branding only with company logo, the rest remains a Salesforce UX. That may be ok for many cases, but enterprises have higher demands for custom flow and UX. The community has full support for building custom pages with Visual Force or Lightning Components and make a custom registration flows with some APEX development.

However, this provides for 1-to-1 Community branding. If we have a customer portal for a business it is a single portal with Single Community presenting a single brand, a simple case. What do we do in cases where business needs to support multiple brands but Portal UX is the same for all those users and brands? This is a challenge we explore here.

One option is to architect Community per brand, by configuring many Communities and each will carry a specific brand but code and config are mostly all the same for all. This approach has some advantages

+ Separate community allows detailed brand customization
+ Extending different UX parts
+ Changes for single-brand possible
+ No impact on other brands

There are also disadvantages especially as this extends to more than 3-6 separate communities.

+ Admin management overhead
+ Additional resources to handle updates
+ Same change across all Portals duplicates work
+ Multiple Deployments

This works for smaller cases with 2-3 brands that can be ok but if this case is for extending business that calls for kind of multitenancy and quick onboarding we need a better solution to handle all brands with a single community.

Keeping a single Community with unified content but with dynamic branding. For our Custom pages that are not a problem, we can brand that based on some configuration or parameters.
How about our Login pages? This turns out a problem in Community, especially while using OAuth 2.0 flow mainly because security does not allow to pass any extra parameters into our OAuth flow and blocking any possibility for dynamically branded Login/Registration pages. This is true for Login public pages.

In Summer 17 salesforce delivered a new feature to handle this case by adding a new brand parameter for OAuth Login pages. As part of URL, we can pass a brand parameter to drive this dynamic logic

`expid_<value>`

This feature allows for some flexibility, while we cannot pass huge objects it is reasonable to add some key value to drive our branding.

For OAuth and SAML endpoints

`services/oauth2/authorize/expid_op123?response_type=code&teken....`

In Apex Login controller we can get this value as follows

`Sttring brandId = Site.getExperienceId();`

We can dynamically load styles for UX pages

`<div class="{!brandId}" id="mybrand">`

Note, this works well in Visual Force pages, but here I discovered a problem that is identified as a bug for Lightning Components on Community Builder pages. Calling `Site.getExperienceId()` in the component controller from the Builder page always returns null no values are passed through. So, for now, this does not work in Community LEX. I have no doubt thought that since I reported this bug in summer 2017, DEV team is at work to fix this. Hopefully in Winter 18 or later it will be fully working.
I will check back and update this blog once new info is available.
 
