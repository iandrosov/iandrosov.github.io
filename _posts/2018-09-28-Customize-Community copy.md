---
layout: post
title: Tips to Customize Communities
---

Quick update 2018-09-28 since Summer 18 & Winter 19 number of improvements had been made to Community Branding and some of the items noted here now can be edited via UI in Builder settings. The CSS may still be useful but for compatibility, it is best to use available system features.

If you recently worked with Salesforce Communities you may have noticed some simple things not easy to customize. For example, trying to remove global search or notification from your pages is not possible without a hack of some code. Here I try to collect various tips on how to get those details done and blaze new trails in your Community.

#### Favicon
Add favicon to your pages seem simple and should be available in Community Builder branding settings declarative but it is not. We can do it, however, with a small bit of code in the community header.
There is an [idea](https://success.salesforce.com/ideaView?id=0873A000000TtX7QAK) for this, vote on it if you like to see this feature added.

In Community Builder navigate to Settings->Advanced and press Edit Head Markup. Add a line of HTML code and Save then Publish your changes.

```
<link rel="shortcut icon" href="<COMMUNITY BASE>/resource/<Static Resource name>/img/favicon.ico" />
```
Best practice to use public URLs for such image assets, CMS storage, or Static resources. Need to use public access Community URLs as shown in the example above. The public will allow your images to show on both secured and public pages such as login or password change.

#### Global Search Bar

A search bar is part of Community templates such as Napili or Customer Account etc. Removing it for some reason is not an option, why? Here is some code to remove it for some specific use cases.
Here we need to override some CSS in the Branding part of Community Builder. This gets a little interesting because it is different per template

For Customer Account template is code below

```
.cSearchContainer {
   visibility: hidden
}
```

For Napili template

```
.forceCommunitySearchBox {
    visibility: hidden
}
```

#### Notification Bell

Community notifications are a handy feature if you are actually using this but in some use cases, it is not needed. Simple is BEST! Here is more branding CSS code to hide the notification bell.

This code works in Customer Account and Napili templates as of WIN18

```
.cNotifications {
   visibility: hidden
}
```

#### User Profile Menu

This is another item that has some good options from Builder settings but seems incomplete. We can hide/show these items in the menu with declarative component direct from Builder by selecting checkboxes

+ Include Contact Support
+ Include My Messages
+ Show User Settings in Menu

Here is CSS for the Napili template to hide the user settings option from the menu even though we can use the builder for that now.

```
.userSettings{
  display: none;  
}
```

In some cases we want to slim down even further and limit user actions, removing `My Account` from the menu.
Here is an example code CSS for this in Napili template

```
.myAccount{
  display: none;  
}
```

By now you get the idea that with some effort in CSS we can override some simple things that are not available as declarative features. One thing to keep in mind as I write this post I am using the Winter 2018 release of Salesforce and any such CSS changes will require maintenance in the future when salesforce changes Template markup or add new Community templates to the list or if you use one of the custom templates. Here is why it is best practice to use declarative features to customize and make CSS hacking last resort as needed.

This will be my last post for 2017 finishing this year of tech blogging. As I work more in Community and find more tips I will update this post with more details.
