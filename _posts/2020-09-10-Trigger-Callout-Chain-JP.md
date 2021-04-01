---
layout: post
title: Salesforce APEX トリガーコールアウトチェーン
---
[English](http://iandrosov.github.io/Trigger-Callout-Chain/)

最近、Salesforceの制限で課題に直面しました。通常は不可能なApexトリガーからのチェーンコールアウトを行う必要がありました。APEXトリガーからのコールアウトはすべて制限されており、トランザクションごとに10回までの呼び出しに制限されている @future メソッド呼び出しでしかできません。

この場合、既に外部APIへの@futureコールアウトを行っているトリガーがあり、このコールから返されたデータを使用して別のAPIコールアウトを行う必要がありました。また、同じオブジェクトトリガー、例えばAccountやContactを使っても、@futureメソッドから開始される継続的なアップデートは、この非同期@futureコンテキストの制限によって制約されてしまいます。


私ができたのは、この問題を回避するために[Queueable Apex](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_queueing_jobs.htm)

この問題は[Queueable Apex](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_queueing_jobs.htm)を使うことで、回避できます。これは、トリガーが@futureメソッドでAPIコールアウトを実行している間、APIコールは非同期（サーバー側では別スレッド）で実行され、その@futureメソッド内の全てのDML操作は単一トランザクションの制限に制約されてしまいます。しかし、ここで[Queueable Apex](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_queueing_jobs.htm)（キューアブルアペックス）を使うと、@futureコンテキストとは別の作業をするために、新しい非同期プロセスを起動して、その制限を回避することが可能となります。

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

これは、新しいQueueable Class　(キューアブルクラス）のメソッドをカバーするためのユニットテストです。

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
