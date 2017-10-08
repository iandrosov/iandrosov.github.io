---
layout: post
title: Custom REST APEX Service in native iOS Apps
---

The other day I was working on a mobile app and needed to call a custom APEX service. After looking through SDK documentation and searching the web, I realized that a lot is written about REST APIs, mobile development and Salesforce Mobile SDK, but none give good examples of how to call a custom service. Moreover most resources talk about hybrid mobile apps, so little is discussed on native Objective C side. So, I decided to write this post to restore the balance in the mobile universe.

Most mobile developers working with Salesforce have used [Mobile SDK](https://developer.salesforce.com/devcenter/mobile) to start the process, connect to a Salesforce Organization and work with data. The SDK provides a quick starting template that is a good starting point. The template is built around UIRootViewController displaying a list of Users and return results of a SOQL query from the organization you are connected to. That basic code uses standard REST CRUD operation services to execute SOQL query and most examples just stop there. The code starts with the assumption that the app has connected to Salesforce with OAuth already (SDK does handle that nicely) and goes something like this:

```
/Here we use a query that should work on either Force.com or Database.com
SFRestRequest *request = [[SFRestAPI sharedInstance] requestForQuery:@”SELECT Id, Name FROM User LIMIT 10″];    
[[SFRestAPI sharedInstance] send:request delegate:self];
The app defines several async delegate responder methods to process the results, success or error:

didLoadResponse
didFailLoadWithError
requestDidCancelLoad
requestDidTimeout
In the didLoadResponsemethod the code processes the result to set the data that will be populated in a Table View, if the query returned any data.

#pragma mark – SFRestAPIDelegate
– (void)request:(SFRestRequest *)request didLoadResponse:(id)jsonResponse {
   NSArray *records = [jsonResponse objectForKey:@”records”];
   NSLog(@”request:didLoadResponse: #records: %d”, records.count);
   self.dataRows = records;
   [self.tableView reloadData];  
}
```

That is all works great for standard CRUD operations on sObjects, but now what happens if we have a custom REST service hosted in Force.com. In most projects, calling a custom service is a base requirement. Normally this is a service written in APEX language with annotation to expose this class and its methods as a REST service. To create an APEX REST service we need to create an APEX class in the Force.com org, as the following example describes (More about rest APEX can be found in [REST Developer Guide](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/intro_what_is_rest_api.htm)). In this example I have chosen to query Contacts and return a list of custom records in common wrapper format:

```
// This annotation defines the service endpoint
@RestResource(urlMapping=’/mycontact/*’)
global with sharing class MyAPEXService {
   @HttpGet
   global static List doGet() {
   
		//APEX code goes here to query and create records
		List my_data = new List();
		List cnt = [select Id, FirstName, LastName from Contact limit 10];
		MyWrapper item = new MyWrapper();
		item.message = ‘found contact records’;
		item.myid = ‘X123’;
		item.contacts = cnt;
		my_data.add(item);
	 
	
		return my_data;
    }
}
```

Now we need to write a simple wrapper class. This is useful in many cases to mix information of custom fields and sObjects defined in Salesforce.

```
global class MyWrapper {
	global String message {get; set;}
	global String myid {get; set;}
	global List contacts {get; set;}
}
```

After we save this code, Salesforce will expose MyAPEXService class as REST endpoint at /mycontact. The actual endpoint URL will look something like this:

`https://instance.salesforce.com/services/apexrest/mycontact/`

Now for the fun part of building an Objective-C code to actually invoke this service from your mobile application. To send actual request we will use the same REST API send method as the standard code does. However, before we invoke APEX we need to manually set up the request object SFRestRequest. Mobile SDK provides some helper methods to set up standard CRUD requests automatically, so in standard calls that detail is hidden from the programmer. Here is an example of how we can set up request manually:

### Manually set up request object

```
// Manually set up request object
SFRestRequest *request = [[SFRestRequest alloc] init];
request.endpoint = @”/services/apexrest/mycontact”;
request.method = SFRestMethodGET;
request.path = @”/services/apexrest/mycontact”;

```

Now send requests to call the custom REST APEX service    

`[[SFRestAPI sharedInstance] send:request delegate:self];`

There are a couple of things worth noticing here:
The endpoint for custom services is/services/apexrest/
The method – is standard REST methods such as GET, POST, DELETE etc. but in SDK you’re required to have their corresponding definition, in this case SFRestMethodGET.
Here it is complete list of these definitions (saves you a trip to github to search through code):

```
SFRestMethodGET: methodName =@”GET”;
SFRestMethodPOST: methodName =@”POST”;
SFRestMethodPUT: methodName =@”PUT”;
SFRestMethodDELETE: methodName =@”DELETE”;
SFRestMethodHEAD: methodName =@”HEAD”;
SFRestMethodPATCH: methodName =@”PATCH”;
```

There are also parameters you can pass with this request. For purposes of this post I did not setup parameters in APEX to pass. To do so you would use NSDictionary queryParams, a simple name/value pair defining the parameters you want to pass to your service. This will be an asynchronous call that will return a response to one of your delegate methods we outlined earlier. A normal response will return to the didLoadResponse SFRestAPIDelegate delegate method. In this method we can process the returned results and set something on the UI for user to interact with. Something like this code below:

```
– (void)request:(SFRestRequest *)request didLoadResponse:(id)jsonResponse {
   
   // Use APEX service return JSON with complex structure
   NSArray *records = jsonResponse;
   NSLog(@”request:didLoadResponse: #records: %d”, records.count);
       
   // Parse & Initialize UI or other processing here from result records
   for (int i = 0; i < [records count]; i++) {
       // Get My Wrapper object data here
       NSDictionary *item = [records objectAtIndex:i];
       NSString *msg = [item objectForKey:@”message”]; // get wrapper field
 

       NSDictionary *contacts_list = [item objectForKey:@”contacts”];
       // Map object to local model
 for (int j = 0; j < [contacts_list count]; j++){
       NSDictionary *my_contact = [contacts_list objectAtIndex:j];
       NSString sfid = [my_contact objectForKey:@”Id”];
       NSString fname = [my_contact objectForKey:@”FirstName”];
       NSObject lname = [my_contact objectForKey:@”LastName”];
}
   
} // END method
```

This is all you need to call your custom APEX REST service and build mobile apps leveraging Salesforce.com Mobile SDK. In conclusion I want to say that while there is not a lot of documentation on Mobile SDK and Native mobile apps for iOS and Android, it is a great tool to connect your mobile apps to the “cloud” and Salesforce, as well as build mobile apps for the Enterprise. The code examples in this writing are based on SDK version 1.5. The new version of Salesforce Mobile SDK is being released shortly, so stay tuned for more on that.
 