---
layout: post
title: Tips to Customize Communities
---

If you recently worked with Salesforce Communities you may have noticed some simple things not easy to customize. Example be trying ot remove global search or notification from your pages is not possible without hack of some code. Here I try to collect various tips on how to get those details done and blaze new trails in your Community.

#### Favicon
Add favicon to your pages seem simple and should be available in Community Builder branding settings declarative but it is not. We can do it however, with small bit of code in the community header.
There is an [idea](https://success.salesforce.com/ideaView?id=0873A000000TtX7QAK) for this, vote on it if you like to see this feature added.

In Community Builder navigate to Settings->Advanced and press Edit Head Markup. Add line of HTML code and Save then Publish your changes.

```
<link rel="shortcut icon" href="<COMMUNITY BASE>/resource/<Static Resource name>/img/favicon.ico" />
```
Best practice to use public URLs for such image assets, CMS storage or Static resources. Need to use public access Community URLs as shown in example above. Public will allow your images ot show on both secured and public pages such as login or password change.

#### Global Search Bar

Search bar is part of Community templates such as Napili or Customer Account etc. Removing it for some rason is not an option, why? Here is some code to remove it for some specific use cases.
Here we need ot override some CSS in Branding part of Comunity Builder. This gets a little interesting because its is different per template

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

Community notifications is a heandy feature if you are actually using this but in some use cases it is not needed. Simple is BEST! Here is more branding CSS code to hide the notification bell.

This code works in Customer Account and Napili templates as of WIN18

```
.cNotifications {
   visibility: hidden
}
```

#### User Profile Menu

This is another item that has some good options from Builder settings but seem incomplete. We can hide/show these items in the menu with declarative component direct from Builder by selecting checkboxes

+ Include Contact Support
+ Include My Messages
+ Show User Settings in Menu

Here is CSS for Napili template to hide user settings option from menu even though we can use builder for that now.

```
.userSettings{
  display: none;  
}
```

In some cases we want to slim down even further adn limit user actions, removing `My Account` from the menu.
Here is example code CSS for this in Napili template

```
.myAccount{
  display: none;  
}
```

By now you get the idea that with some effort in CSS we can override some simple things that are not available as declarative features. One thing to keep in mind as I write this post I am using Winter 2018 release of Salesforce and any such CSS changes will require maintenance in the future when salesforce changes Template markup or add new Community templates to the list or if you use one of custom templates. Here is why it is best practice to use declarative features to customize and make CSS hacking last resort as needed.

This will be my last post for 2017 finishing this year of tech bloging. As I work more in Community and find more tips I will update this post with more details.
