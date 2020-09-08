---
layout: post
title: Salesforce APEX トリガーコールアウトチェーン
---
[English](http://iandrosov.github.io/2017-09-10-Trigger-Callout-Chain/)

私は最近、Salesforceの制限で課題に直面しました。通常は不可能なApexトリガーからのチェーンコールアウトを行う必要がありました。APEXトリガーからのコールアウトはすべて制限されており、トランザクションごとに10回までの呼び出しに制限されている `@future` メソッド呼び出しでしかできません。

この場合、私は既に外部APIへの`@future`コールアウトを行っているトリガーを持っており、このコールから返されたデータを使用して別のAPIコールアウトを行う必要がありました。また、同じオブジェクトトリガー、例えばAccountやContactを使っていても、`@future`メソッドから開始される連続的な更新は、この非同期@futureコンテキストの制限によって制限されてしまうので、それはできません。

私ができたのは、この問題を回避するために[Queueable Apex](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_queueing_jobs.htm)を使うことです。これは、トリガーが@futureメソッドでAPIコールアウトを行っている間、このAPIコールは非同期（サーバー側では別スレッド）で実行され、その@futureメソッド内の全てのDML操作は単一トランザクションの制限に該当するということです。Queueable Apex（キューアブルアペックス）を使うと、@futureコンテキストとは別の作業をするために新しい非同期プロセスを起動して、その制限を慎重に回避することができます。

実際には、Salesforce はこの作業を、リソースが利用可能になるとスケジュール通りに実行されるバッチジョブのようなキューに置いています。この作業を完了させるためのトリガー @future メソッドから非同期のバッチジョブを起動するようなものです。

これは、Queueable APEX クラスの実装です。

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

このジョブは、監視可能なジョブIDを返すSystemメソッドの呼び出しによって開始されます。

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

これは、新しいキューアブルクラスのメソッドをカバーするためのユニットテストです。

```
@isTest
private class SampleServiceQueueTest {
	static testMethod void SampleQueueTest(){
        List<Id> idList = new List<Id>();
        SamlpleServiceQueue mq = new SampleServiceQueue(idList);
    }
}
```

このテストメソッドは、デプロイのために100%のテストカバレッジを得る必要があります。
これでAPEX Triggerのコールアウトのためのチェイニングは完了です。 