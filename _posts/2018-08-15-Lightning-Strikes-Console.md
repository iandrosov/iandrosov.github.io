---
layout: post
title: Lightning Strikes the Service Console
---

I have been working in Lightning and building Components for a couple of years now. Lightning is a great platform to work with, finally, we can work in JS and build manageable units of work as components that can assemble complex apps. This was hard and clunky before with just JavaScript/CSS framework, require lots of work and skills that normally not available on projects. 

Any great framework has flaws and Lightning no different. This problem appeared as was trying to open a component as subtab in a console app. Things work differently in the console as you may know from Service Cloud.

We build a nice set of UI components that display a complex table of data that need a full-screen real-estate to display full table. I tested this as a tab in Lightning Record Page on the opportunity and it worked great as Lightning Component on Sales app. And on my large screen, it all shows nicely, job well done, maybe? We try to open this in a console where space is limited for main table display. Make it worse our screens at work are way smaller so data need to scroll resulted in really bad UX.

We attempted to solve this by opening component as a subtab on a console as part of same Opportunity record and here is another problem - using app builder created a page very easy task, assuming it will be Lightning App Page we can open it as full screen but NOT. 

* App Builder page builds flexipage metada
* Different from <aura:app>
* Can open new page from a record but ID not passed
* Result - compnent cannot initialize data

#### Some try-error fun
I did several attempts to launch component form various places, button component, quick action, Aura app, nothing seem to work right. While it is possible to pass parameters to aura:app it was not working in navigation inside a console. Finally reached out to support and after few weeks finally, git some great insights that helped to open a component as a subtab in Lightning Console.

This is good resource working with console [Salesforce Console Developer Guide](https://developer.salesforce.com/docs/atlas.en-us.api_console.meta/api_console/sforce_api_console_lightning_openSubtab.htm) has some examples for workspace tabs, subtabs and icons.

Next use of new lightning components is essential like the one below.

```
lightning:isUrlAddressable
```
Here is full list of [Lightning Interfaces](https://developer.salesforce.com/docs/component-library/bundle/lightning:isUrlAddressable/documentation) we can work with in Lightning Components.

Using this interface and some navigation it is possible to build a simple component that will open another component as a subtab.

```
<aura:component implements="force:appHostable,flexipage:availableForAllPageTypes,flexipage:availableForRecordHome,force:hasRecordId" access="global" >
    <aura:attribute name="recordId" type="String" />
    <lightning:workspaceAPI aura:id="workspace" />
    <lightning:navigation aura:id="navService"/>
    <lightning:button label="My Plan" onclick="{! c.openTabWithSubtab }" />	
</aura:component>
```

This component will show a button that can be placed on a record page and will launch a component as a subtab using JavaScript controller code. We assume our custom component name `MyPlanComponent`, keep close attention that reference to our components is done by name with global namespace prefix `c__MyPlanComponent`

```
({
    openTabWithSubtab : function(component, event, helper) {
        var recTempId = component.get("v.recordId"); // Record ID for Oppty
        var workspaceAPI = component.find("workspace");
        
        var navService = component.find("navService");
        
        var pf = {
            type: 'standard__component',
            attributes: {
                componentName: 'c__MyPlanComponent'
            },
            state: {
                "recId": recTempId
            }
        };     
        
        
        /** OPEN My PLan Lightning component in subtab **/
		workspaceAPI.openTab({
            url: '/lightning/r/Opportunity/'+recTempId+'/view',
            focus: true
        }).then(function(response) {
            workspaceAPI.openSubtab({
                parentTabId: response,
                pageReference: pf,
                focus: true
            });
        })
        .catch(function(error) {
            console.log(error);
        });        
        
    }
})
```
Notice we also used an explicit attribute to pass record ID `recordId` and defining a page parameter for our component enables to pass ID all the way to component APEX controller to initialize.

```
.......
state: {
  "recId": recTempId
}
```

This is one solution on how to launch custom components in console subtab and initialize it with record context while it is a full page.
the same idea can also apply if we wanted to launch the same component from Quick Action instead of adding a new button.

This sample code below helps to make a component for Quick Action. This component does not use any APEX code, only Lightning.

```
<aura:component implements="force:appHostable,flexipage:availableForAllPageTypes,flexipage:availableForRecordHome,force:hasRecordId,force:lightningQuickAction" access="global" >
    <aura:attribute name="recordId" type="String" />
    <lightning:workspaceAPI aura:id="workspace" />
    <lightning:navigation aura:id="navService"/>
    <aura:handler name="init" value="{!this}" action="{!c.openTabWithSubtab}" />	
</aura:component>
```

Controller JS code

```
({
    openTabWithSubtab : function(component, event, helper) {
        var recTempId = component.get("v.recordId"); // Record ID for Oppty
        var workspaceAPI = component.find("workspace");
        
        var navService = component.find("navService");
        
        var pf = {
            type: 'standard__component',
            attributes: {
                componentName: 'c__MyPlan'
            },
            state: {
                "recId": recTempId
            }
        };     
        
        
        /** OPEN My PLan Lightning component in subtab **/
		workspaceAPI.openTab({
            url: '/lightning/r/Opportunity/'+recTempId+'/view',
            focus: true
        }).then(function(response) {
            workspaceAPI.openSubtab({
                parentTabId: response,
                pageReference: pf,
                focus: true
            });
        })
        .catch(function(error) {
            console.log(error);
        }); 	
	}
})
```

This is that APEX-Less console components that works in Lightning console, hopefully these samples will help you to build some owesome UX. Go Lightning....



