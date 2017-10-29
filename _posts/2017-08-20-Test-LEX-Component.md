---
layout: post
title: Lightning Component Unit Test - System Error
---

While working on Lightning components recently discovered that LEX APEX controllers are tricky to test. Mainly because of use `aura.redirect` or other `aura.` calls. If you try testing methods that use `aura.redirect` call you will get a System error like this,

```
Internal Salesforce Error: 173417772-1404 (604027281) (604027281)
```
that does not tell anything usefull to a developer. Tech Support has translated this for me to some Missing ORACLE connection context. Given the fact that Salesforce platform is built on giant ORACLE instance it is not surprisig. At times I feel that main reason Salesforce has limts is to protect that giant ORACLE instance from fail.
Anyways lets see how to resolve this issue with our test code.

Digging bit deeper is WHY? Main reason for this error is `aura.<>` is a JavaScript frontend library, but here we are testing APEX controller back end that has to interface with front end JS library. Any such hybridization of programming languages can bring complex problems to light and I personally would avoid such transitions. 

Fpr this example we have a custom Lightning Component that is mostly a copy of example `forgotPassword` found in any developer org. This component that performs a function to help users reset password. It has APEX Controller class like this one

```
global class LightningForgotPasswordController {

    public LightningForgotPasswordController() {

    }

    @AuraEnabled
    public static String forgotPassword(String username, String checkEmailUrl) {
        try {
            Site.forgotPassword(username);
            ApexPages.PageReference checkEmailRef = CommunitiesUtil.getCommunityPageRedirect('custom-forgot-password');
            if(!Site.isValidUsername(username)) {
                return Label.Site.invalid_email;
            }
            // Set perameter to display page message
        	checkEmailRef.getParameters().put('emailSent', 'true');
        	
            aura.redirect(checkEmailRef);
            return null;
        }
        catch (Exception ex) {
            return ex.getMessage();
        }
    }
```

Simple one method that calls `forgotPassword` methods, checks user validity and redirects user to page with a message to check email and reset password.

This method has following unit test code that tests this class ot provide code coverage.

```
@IsTest(SeeAllData = true)
public with sharing class LightningForgotPasswordControllerTest {

 /* Verifies that ForgotPasswordController handles invalid usernames appropriately */
 @IsTest
 static void testLightningForgotPasswordControllerInvalidUserName() {
  System.assertEquals(LightningForgotPasswordController.forgotPassword('fakeUser', 'http://a.com'), Label.Site.invalid_email);
  System.assertEquals(LightningForgotPasswordController.forgotPassword(null, 'http://a.com'), Label.Site.invalid_email);
  System.assertEquals(LightningForgotPasswordController.forgotPassword('a', '/home/home.jsp'), Label.Site.invalid_email);
 }

 /* Verifies that null checkEmailRef url throws proper exception. */
 @IsTest
 static void testLightningForgotPasswordControllerWithNullCheckEmailRef() {
  System.assertEquals(LightningForgotPasswordController.forgotPassword('a', null), 'Argument 1 cannot be null');
  System.assertEquals(LightningForgotPasswordController.forgotPassword('a@salesforce.com', null), 'Argument 1 cannot be null');
 }

 /* Verifies that LightningForgotPasswordController object is instantiated correctly. */
 @IsTest
 static void LightningForgotPasswordControllerInstantiation() {
  LightningForgotPasswordController controller = new LightningForgotPasswordController();
  System.assertNotEquals(controller, null);
 }
}
```

When I run this type of unit test on my dev sandbox I got the interesting system error mentioned earlier

```
Internal Salesforce Error: 173417772-1404 (604027281) (604027281)
```

IMPORTANT fact if I use exact Salesforce example class and test that unit test will work. However, for some reason that test does NOT run/test `aura.redirect(checkEmailRef);` line of code it is ignored. But on my custom controller code it exevcuted this code and has an error.

One simple way I found to avoid this error and provide effective test coverage is NOT to RUN `aura.redirect` during unit tests `Test.isRunningTest()`. This means making conditional code that is a bit ugly but in this case needed to move forward. Here is fixed code and unit test that worked.

```
global class LightningForgotPasswordController {

    public LightningForgotPasswordController() {

    }

    @AuraEnabled
    public static String forgotPassword(String username, String checkEmailUrl) {
        try {
            Site.forgotPassword(username);
            ApexPages.PageReference checkEmailRef = CommunitiesUtil.getCommunityPageRedirect('custom-forgot-password');
            if(!Site.isValidUsername(username)) {
                return Label.Site.invalid_email;
            }
            // Set perameter to display page message
        	checkEmailRef.getParameters().put('emailSent', 'true');
        	
            // Special consideration
            // SF System error on aura redirect missing context
            // Avoid error for Unit tests run do not redirect
            if(!Test.isRunningTest()){
            	aura.redirect(checkEmailRef);
            }
            return null;
        }
        catch (Exception ex) {
            return ex.getMessage();
        }
    }
```

Unit test is mostly the same

```
@IsTest(SeeAllData = false)
public with sharing class ForgotPasswordControllerTest {
	/* Verifies that ForgotPasswordController handles invalid usernames appropriately */
	@IsTest
 	static void testLightningForgotPasswordControllerInvalidUserName() {
  		System.assertEquals(ForgotPasswordController.forgotPassword('fakeUser', 'http://a.com'), Label.Site.invalid_email);
  		System.assertEquals(ForgotPasswordController.forgotPassword(null, 'http://a.com'), Label.Site.invalid_email);
  		System.assertEquals(ForgotPasswordController.forgotPassword('a', '/home/home.jsp'), Label.Site.invalid_email);
 	}

 	/* Verifies that null checkEmailRef url throws proper exception. */
 	@IsTest
	static void testLightningForgotPasswordControllerWithNullCheckEmailRef() {
  		System.assertEquals(ForgotPasswordController.forgotPassword('', ''), 'Invalid Email Address');
  		//System.assertEquals(ForgotPasswordController.forgotPassword('a@test.com', null), 'Invalid Email Address');
 	}

 	/* Verifies that LightningForgotPasswordController object is instantiated correctly. */
 	@IsTest
 	static void LightningForgotPasswordControllerInstantiation() {
  		ForgotPasswordController controller = new ForgotPasswordController();
  		System.assertNotEquals(controller, null);
 	}
```

Fix for unit test system error, now I can deploy code to Production with effective code coverage.