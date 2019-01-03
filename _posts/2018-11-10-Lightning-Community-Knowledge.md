---
layout: post
title: Edit Knowledge in Salesforce Communities
---

Salesforce Communities has extended many features in recent releases, including new branding sets, themes, and audiences. We had a need to open Knowledge to Comunity users that is a standard out of box feature. Most Community types can view and search Knowledge articles for self-service. 

How about editing or contribute an article in Lightning Community? As it turned out as always all in the details, editing Knowledge in Lighting Community is not fully supported and requires development. While it is possible to use standard Knowledge editor for Community built with VisualForce and Tabs, there is no Edit/Author option for Community Builder pages and Lightning.

This article post will explore that topic, how to build Lightning Component to edit Knowledge articles in Lightning Community. I was expecting this would be solved already but after searching in the documentation, samples or GitHub there was no solution created yet, and Knowledge documentation is a bit sparse on the subject, especially for Lightning Knowledge.

### Lightning Knowledge

Salesforce Knowledge in Lightning has changed the data model behind the scenes. In classic, there were sets of Knowledge custom objects created for each Article type.
If we create a new article type that = building a new Custom Object in Salesforce with new fields and permissions.

Lightning changed all that to use single Object `Knowledge__kav` and Record Types as new article types. I am guessing `_kav` stands for KNowledge Article Version. It becoming more clear now why that model change causes issues and makes Knowledge transition to a Lightning bit harder than it should be.

### Knowledge in Community

Community Builder provides many standard components to view Knowledge, Topics and articles search. This post will focus on building a component to edit an article.

To edit text we will use [Base Lightning Component](https://developer.salesforce.com/docs/component-library/overview/components) RichText component to view or edit the actual text 

* Edit `lightning:inputRichText`
* View `ightning:formattedRichText`

Lightning Component [Code example](https://github.com/iandrosov/sfdx-lex-knowledge-edit)

```
<aura:component implements="forceCommunity:availableForAllPageTypes,force:lightningQuickAction" access="global" >
	<aura:handler name="init" value="{! this }" action="{! c.init }" />
    <aura:attribute name="richtext" type="String"/>

    <div aura:id="container">
        <p>
            <lightning:inputRichText value="{!v.richtext}"/>
        </p> 
        <br/>
        <h2>Text Output for sample only to show live text edits</h2>
        <p>
            <lightning:formattedRichText aura:id="kvaRichText" value="{!v.richtext}" />
        </p>
    </div>
</aura:component>
```

Component client JavaScript controller

```
    init: function(cmp) {
  
		var content = "<SOME ARTICLE TEXT be Loaded here>" ; 
        cmp.set("v.richtext", content);
      
    }

```

### Access Knowledge via REST API
To edit a Knowledge article we need to first create and then find the article and load its content using the REST API. For this example, we create 2 test articles and a single category called `Policy`, in real production there can be a complex web of categories and related articles.

For REST API Documentation can be found at [Salesforce Knowledge Developer Guide](https://developer.salesforce.com/docs/atlas.en-us.knowledge_dev.meta/knowledge_dev/knowledge_REST_retrieve_article_version.htm). To query article data we can use [WorkBench](https://workbench.developerforce.com) and run SOQL query to find article IDs

```
SELECT ArticleNumber,Id,KnowledgeArticleId,Title,VersionNumber FROM Knowledge__kav

Result
 	ArticleNumber	Id	KnowledgeArticleId	Title	VersionNumber
1	000001001	ka0J00000005YlFIAU	kA0J00000005859KAA	Random Article	1
2	000001000	ka0J00000005YlAIAU	kA0J00000005854KAA	Test QA Process	1
```

Another way to query Lightning Knowledge to get content is using `KnowledgeArticleVersion` object as shown below.

```
SELECT Id,KnowledgeArticleId,Summary,Title FROM KnowledgeArticleVersion

 	Id	KnowledgeArticleId	Summary	Title
1	ka0J00000005YlFIAU	kA0J00000005859KAA	Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book. It has survived not only five centuries, but also the leap into electronic typesetting, remaining essentially unchanged. It was popularised in the 1960s with the release of Letraset sheets containing Lorem Ipsum passages, and more recently with desktop publishing software like Aldus PageMaker including versions of Lorem Ipsum.	Random Article
2	ka0J00000005YlAIAU	kA0J00000005854KAA		Test QA Process


```

Now we can try Knowledge Management REST API to get article content so we can load text into our component. This API will return an Article record by ID

```
/services/data/v44.0/knowledgeManagement/articles/kA0J00000005859KAA

additionalInformation
canArchive: true
canDelete: false
canEdit: true
canPublish: false
canUnpublish: true
hasArchivedVersions: false
hasTranslations: false
urls
OnlineMasterVersion: /services/data/v44.0/knowledgeManagement/articleVersions/masterVersions/ka0J00000005YlFIAU
archivedArticleMasterVersionId:
articleId: kA0J00000005859KAA
articleType: Knowledge__kav
draftArticleMasterVersionId:
masterLanguage: en_US
onlineArticleMasterVersionId: ka0J00000005YlFIAU
```

As you see this returned an article metadata and importantly a URL to next call to get actual article data. We will call that next.

```
/services/data/v44.0/knowledgeManagement/articleVersions/masterVersions/ka0J00000005YlFIAU

additionalInformation
canArchive: true
canDelete: false
canEdit: true
canPublish: false
canTranslate: false
data: /services/data/v44.0/sobjects/Knowledge__kav/ka0J00000005YlFIAU
lastVersion: true
masterVersion: true
summary: Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book. It has survived not only five centuries, but also the leap into electronic typesetting, remaining essentially unchanged. It was popularised in the 1960s with the release of Letraset sheets containing Lorem Ipsum passages, and more recently with desktop publishing software like Aldus PageMaker including versions of Lorem Ipsum.
title: Random Article
type: Knowledge__kav
url: /services/data/v44.0/knowledgeManagement/articleVersions/masterVersions/ka0J00000005YlFIAU
archiveScheduleDate: null
articleId: kA0J00000005859KAA
articleNumber: 000001001
articleVersionId: ka0J00000005YlFIAU
assigneeId: null
dueDate: null
instruction: null
language: en_US
locked: false
publishScheduleDate: null
publishStatus: Online
versionNumber: 1

```
Now we got our content to load and edit as part of JSON response. In this case, it is a Summary field with text that can be edited. Knowledge object can carry custom fields for text content as well.
In this case, we are working with Lightning Components that will be hosted in Community pages. Salesforce Lightning can’t make calls to third-party APIs from client-side code. To get our content we will use APEX controller to load/edit data we need for Knowledge article.



