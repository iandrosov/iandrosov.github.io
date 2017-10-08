---
layout: post
title: Offline Salesforce Data For Native iOS Apps – Salesforce Mobile SDK 2.0 SmartStore
---

For those of us involved in mobile development with Salesforce.com, we know all too well that offline data typically a critical feature that customers request. [Salesforce Mobile SDK](https://developer.salesforce.com/devcenter/mobile) provides some useful features for SmartStore to help provide Salesforce data to mobile users offline. Local storage in mobile solutions is a big thing. It makes things great for users, usability and adoption. At the same time it can be daunting for developers due to increased complexity. Basically you carry a sort of mini RDBMS on your mobile device which can get  quite complicated. Unfortunately there is not a lot of detailed information available on how to do this in native iOS Application. In this post I’d like to discuss some of the details around the use of SmartStore SOUP objects for local storage in Native iOS Application written in Objective-C. First of all SmartStore is only Salesforce specific local storage designed for use with Salesforce Object data. SmartStore is supported on iOS and Android devices in native and Hybrid apps. Smart Store is designed to store salesforce data offline in a simple and secure manner. Local data is automatically encrypted by the SDK so nothing special needs to be done by the developer to get that! SmartStore stores data in its original JSON format that is returned by Salesforce REST requests, so there is no need to convert data into various formats. This has the added benefit of running your application from the same set of code/data parsers online and offline, which is very convenient. Data is stored in what is called a SOUP object with SmartStore. Hopefully this has been a helpful intro, now on to some code and details.

## Basic operations to use SmartStore:

+ Create/Register a SOUP by its unique name (it is like a DB table name)
+ Insert data into store with SOUPs
+ Query stored data

Basic CRUD operations are supported and there were many options added with SDK 2.x for querying of data that we discuss later on. This example explains how to use SmartStore and also shows how to convert existing apps using earlier versions of SmartStore. First you will need to import some headers:

```
#import "SFSmartStore.h"
#import “SFQuerySpec.h
```

Before using any SmartStore operations you need to register a SOUP. To register a SOUP all you need is the smart store object, soup name and index specs.

```
SFSmartStore *my_store  [SFSmartStore sharedStoreWithName:kDefaultSmartStoreName];
```

After creating a shared SmartStore object – a sort of a DB handle – we need to create a SOUP – a sort of DB table – to store our data.

```
#define MY_SOUP @"MY_TABLE_SOUP"
BOOL exists = [my_store soupExists:MY_SOUP];

// check if store exists and create it if not
if(!exists) {
   NSMutableDictionary *columnIndex1 = [[NSMutableDictionary alloc] init];
   [columnIndex1 setObject:@"sfid" forKey:@"path"];
   [columnIndex1 setObject:@"string" forKey:@"type"];
   NSMutableDictionary *columnIndex2 = [[NSMutableDictionary alloc] init];
   [columnIndex2 setObject:@"Name" forKey:@"path"];
   [columnIndex2 setObject:@"string" forKey:@"type"];
  [my_store registerSoup:MY_SOUP
withIndexSpecs:[NSArray arrayWithObjects:columnIndex1,columnIndex2, nil]];

// This code will apply for older SDK version such as 1.5 where there is NO ARC // support need explicit release of index objects.
       [columnIndex1 release];
       [columnIndex2 release];
   }
```   

That is all. Now we have a “table” SOUP with some indexes where we can store and query our data. For registration of a SOUP this code is mostly the same on SDK 1.5 or 2.x.

## To insert/update data

Lets say we have a simple record structure to insert, ID, Name, Status, Client. We will have to set up NSMutableDictionary object to hold our data and set data fields.

```
 NSMutableDictionary *dict = [[NSMutableDictionary alloc] init];
    [dict setValue:[NSString stringWithString:@"si1312341234"] forKey:@"sfid"];
    [dict setValue:[NSString stringWithString:@"my record"] forKey:@"Name"];
    [dict setValue:[NSString stringWithString:@"ABC Co."] forKey:@"Client"];
    [dict setValue:[NSString stringWithString:@"PENDING"] forKey:@"Status"];
    NSArray *records = [NSArray arrayWithObjects: dict, nil];
    // --------- LOCAL STORE -------
    // Save records data to smartstore
    NSError *error = nil;
    [my_store upsertEntries:records toSoup:MY_SOUP
                                   withExternalIdPath:@"sfid" error:&error];
    NSLog(@"Upsert: %@", error);
    if(error==nil){
       return TRUE;
    }else{
       return FALSE;
    }
```

## To delete a soup is simple

```
[my_store removeSoup:MY_SOUP];
```

This is the only line of code to delete your table containing your data.

## Query data

Here is one of the methods that is similar to earlier versions of SDK, so it is easy to demonstrate the difference. This is an example of code from SDK 2.0:

```
NSMutableDictionary *queryDictionary = [[ NSMutableDictionary alloc] init];
  [queryDictionary setObject:@"range" forKey:@"queryType"];
  [queryDictionary setObject:@"Name" forKey:@"indexPath"];
  [queryDictionary setObject:@"ascending" forKey:@"order"];
  [queryDictionary setObject:[NSNumber numberWithInt:5] forKey:@"pageSize"];
  SFQuerySpec *querySpec = [[SFQuerySpec alloc]   initWithDictionary:queryDictionary                              
withSoupName:MY_SOUP];
  NSArray *results = [my_store queryWithQuerySpec:querySpec pageIndex:0];
```

In the SDK 1.5 or prior you would have seen the following code instead, which can be converted into 2.x.

```
SFSoupQuerySpec *querySpec = [[SFSoupQuerySpec alloc] initWithDictionary:queryDictionary];
NSArray *results = [my_store querySoup:MY_SOUP withQuerySpec:querySpec pageIndex:0];
       [queryDictionary release];
       [querySpec release];
```

At this point we can see how to use SmartStore to store and access your data off-line. Now all you need to do is make a REST request to Salesforce to bring the data to your mobile app. Lets review a simple example on how to do this here to close the loop. For demonstration purposes, I will use simple query with standard REST API provided by Salesforce.com. In the real world your applications will likely need to call custom services for details on calling custom APEX REST service. For more info reference this article: [REST with APEX Service in native iOS Apps](http://iandrosov.github.io/Custom-REST-Service-Native-iOS-App). We will make a simple SOQL query request to get list of Contact records and store them locally in order to use this data offline. The simple way to think about this “complex” offline problem is to make your mobile app work with local data for the most part.  This simplifies things with error handling, connectivity and data sync. In some cases user requests to get this data refreshed and synced with their Salesforce Organization. Once we have set up the Native Application with Salesforce Mobile SDK it will provide a boilerplate code for making secure OAuth connections to Salesforce and we can start coding our request. In your controller method we will make a query request with an SOQL query string to get our contacts data.

```
- (void)viewDidLoad
{
  [super viewDidLoad];
  self.title = @"SmartStore Sample App";
  //Here we use a query that should work on either Force.com or Database.com
  SFRestRequest *request = [[SFRestAPI sharedInstance] requestForQuery:
       @"SELECT Id, FirstName, LastName, Email, Phone FROM Contact LIMIT 10"];
  [[SFRestAPI sharedInstance] send:request delegate:self];
}
```

Notice a standard string SOQL query SELECT Id, FirstName, LastName, Email, Phone FROM Contact LIMIT 10. This will return some of the fields from the standard Contact Object and I have limited the query to 10 rows for simplicity. Now that the request been fired, our Rest Delegate will respond with some results in a short while. Next we need to handle response. We will implement SFRestDelegate methods didLoadResponse to get JSON data. We will also need to implement several error handler methods, in case our request errored out, so that we will get notified.

```
- (void)request:(SFRestRequest *)request didLoadResponse:(id)jsonResponse {
      NSArray *records = [jsonResponse objectForKey:@"records"];
      self.dataRows = records;
     if (records != nil && [records count] > 0){
         NSError *error = nil;
         SFSmartStore *my_store =
         [SFSmartStore sharedStoreWithName:kDefaultSmartStoreName];
         BOOL exists = [my_store soupExists:MY_SOUP];
         // check if store exists and create it if not
         if(!exists) {
           NSMutableDictionary *columnIndex1 =
                            [[NSMutableDictionary alloc] init];
           [columnIndex1 setObject:@"recordId" forKey:@"path"];
           [columnIndex1 setObject:@"string" forKey:@"type"];
           [my_store registerSoup:MY_SOUP withIndexSpecs:
                 [NSArray arrayWithObjects:columnIndex1, nil]];
         }
         exists = [my_store soupExists:MY_SOUP];
         // Store data here with upsert operation
         [my_store upsertEntries:records
                  toSoup:MY_SOUP
                  withExternalIdPath:@"recordId"
                  error:&error];
      }
}
- (void)request:(SFRestRequest*)request didFailLoadWithError:(NSError*)error {
      NSLog(@"request:didFailLoadWithError: %@", error);
      //add your failed error handling here
}
- (void)requestDidCancelLoad:(SFRestRequest *)request {
      NSLog(@"requestDidCancelLoad: %@", request);
      //add your failed error handling here
}
- (void)requestDidTimeout:(SFRestRequest *)request {
      NSLog(@"requestDidTimeout: %@", request);
      //add your failed error handling here
}
```

At this point we got the data from Salesforce and stored this locally for later access offline in our SmartStore under name MY_SOUP. Note that we did not have to use any field mappings to persist the data. SmartStore will handle records as they are and we can use the same parsing logic to handle remote response, as well as local data, stored in MY_SOUP. This data has the identical format and the same field names. Now we can query and use this data any time within our application using the Query Data method described earlier. One important thing to remember is that SDK stores data locally with some security measures.  All data is encrypted on the device and can only be accessed within context on the application and via SDK methods. Local data also has been bound to user session context, which means that while the user has a valid OAuth token the application will continue to sync and store data with Salesforce. However, once the user decides to logout or the session has been revoked by the Administrator, the SDK part of the application will automatically delete all local data from all SmartStore locations and reset the OAuth session. Users will have to once again establish a valid connection to Salesforce and download the data for local storage. Hopefully this provided some needed details on how to use SmartStore with [Salesforce Mobile SDK](https://developer.salesforce.com/devcenter/mobile) to take your data with you wherever you go. In conclusion I would like to mention that Mobile SDK is available on [github](https://github.com/forcedotcom/SalesforceMobileSDK-iOS) as a free open download.
