---
layout: post
title: Salesforce APEX Trigger Callout Chaining
---
[日本語](http://iandrosov.github.io/2017-09-10-Trigger-Callout-Chain-JP/)

I recently faced a challenge with Salesforce limits. I needed to make a chained callout from Apex Trigger that is typically not possible. Any callouts from APEX triggers are restricted, only possible with `@future` method calls that are limited to 10 calls per transaction.

In this cases, I had a trigger that was already making `@future` callout to external API and I needed to use the data returned from this call to make another API callout, this is chaining the API calls that are NOT allowed in APEX Trigger or `@future methods` that call other @future. And even using the same object trigger say Account or Contact it will not be possible, since any continuous updates started from `@future` method will be restricted by this limitation of asynchronous @future context.

What I was able to do is to use [Queueable Apex](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_queueing_jobs.htm) to work around this problem. What this means is while the trigger makes an API callout with @future method this API call runs asynchronous (different thread on server side), all DML operations inside that @future method fall under a limit restriction of a single transaction. Queueable apex allows us to start a new asynchronous process to do separate work from @future context and carefully avoid that limit.

In reality, Salesforce puts this work on a queue sort of a batch job that will execute on schedule as resources are available. It is like starting an asynchronous batch job from a trigger @future method to complete this task.

This is the Queueable APEX class implementation

```
public class SampleServiceQueue implements Queueable, Database.AllowsCallouts {
	public final List<Id> idList;
    public SampleServiceQueue(List<Id> idList){
        this.idList = idList;
    }    
    public void execute(QueueableContext context) {
       // This method will make a REST API call
		SampleService.accountPaymentService(idList);
    }
}
```

This job starts by a call to System method that returns a job ID that can be monitored.

```
List<Account> acctList = Trigger.new; // Get this from a trigger call
// Assemble List of IDs for queueable method
List<Id> acctIdList = new List<Id>();
for (Account a : acctList){
     acctIdList.add(a.Id);    
}

if (acctIdList != null && acctIdList.size()>0){   
	Id jobID = System.enqueueJob(new SampleServiceQueue(acctIdList));
}

```

This is a unit test to cover the new queueable class method.

```
@isTest
private class SampleServiceQueueTest {
	static testMethod void SampleQueueTest(){
        List<Id> idList = new List<Id>();
        SamlpleServiceQueue mq = new SampleServiceQueue(idList);
    }
}
```

This test method should get 100% test coverage for deployment. Chaining for APEX Trigger callout is complete.
 
