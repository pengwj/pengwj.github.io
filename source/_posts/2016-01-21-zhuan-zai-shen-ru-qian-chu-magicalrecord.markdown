---
layout: post
title: "[转载]深入浅出MagicalRecord"
date: 2016-01-21 13:53:55 +0800
comments: true
categories: 
---

写的蛮不错的一篇关于MagicalRecord的文章，怕以后需要时忘记了，所以转载在自己的博客里面，原文链接如下    
[http://childhood.logdown.com/posts/208957/easy-magicalrecord-01
](http://childhood.logdown.com/posts/208957/easy-magicalrecord-01
)     

[http://childhood.logdown.com/posts/209933/easy-magicalrecord-02
](http://childhood.logdown.com/posts/209933/easy-magicalrecord-02
)

[http://childhood.logdown.com/posts/211016/easy-magicalrecord-03
](http://childhood.logdown.com/posts/211016/easy-magicalrecord-03
)

[http://childhood.logdown.com/posts/211076/easy-magicalrecord-04
](http://childhood.logdown.com/posts/211076/easy-magicalrecord-04
)

============================
##一、深入浅出MagicalRecord-02

###1、CoreData与MagicalRecord

在 ios 开发中，我们会使用CoreData来进行数据持久化。但是在使用CoreData进行存取等操作时，代码量相对较多。而 MagicalRecord 正是为方便操作 CoreData 而生。

MagicalRecord 的三个目标：

1.简化 CoreData 相关代码   
2.清晰、简单、单行获取数据    
3.当需要优化请求的时候，仍然允许修改 NSFetchRequest
#####[MagicalRecord-Github地址](https://github.com/magicalpanda/MagicalRecord)

============================

###2、 安装配置

#####方法1：

1.从github上下载MagicalRecord源码 [MagicalRecord-Github地址](https://github.com/magicalpanda/MagicalRecord)。

2.将MagicalRecord文件夹拖放并添加到Xcode项目中。

3.添加 CoreData.framework。

4.在项目的预编译头文件中（PCH）中导入CoreData+MagicalRecord.h。或者在将要使用的类中单独导入。

5.开始编码。

#####方法2：

1.如果已经安装了CocoaPods（不清楚CocoaPods使用的朋友们可以看唐巧的这篇[用CocoaPods做iOS程序的依赖管理](http://blog.devtang.com/blog/2014/05/25/use-cocoapod-to-manage-ios-lib-dependency/)）。

在终端中输入pod search MagicalRecord，结果如下：



复制pod 'MagicalRecord', '~> 2.2'到项目中的Podfile并保存文件。命令行cd到工程根目录下，并执行pod install。

2.以下步骤同方法1的3、4、5步骤。

注意：
如果你在使用 MagicalRecord 方法的时候不想带前缀MR_（比如用 findAll 代替 MR_findAll），只需在PCH文件中，在CoreData+MagicalRecord.h之前增加#defin MR_SHORTHAND即可。

============================

###3.环境需求

iOS SDK 6.x 及以上
OS X SDK 10.8 及以上
使用Xcode5来运行github仓库中包含的测试Tests，但MagicalRecord库建立在Xcode4上。

##二、深入浅出MagicalRecord-02

这一节我们一起粗略的了解下 CoreData 中的一些核心概念以及 MagicalRecord 的入门准备。只有对 CoreData 理解深入了，才能更轻松的使用 MagicalRecord。

###1、CoreData 的核心概念

先上两幅关键的概念图

####(1)NSManagedObjectModel 托管对象模型（MOM）

这个MOM由实体描述对象，即NSEntityDescription实例的集合组成，实体描述对象介绍见下面第7条。

作用：添加实体的属性，建立属性之间的关系

####(2)NSManagedObjectContext 托管对象上下文（MOC）

在概念图2中，托管对象上下文（MOC）通过持久化存储协调器（PSC）从持久化存储（NSPersistentStore）中获取对象时，这些对象会形成一个临时副本在MOC中形成一个对象集合，该对象集合包含了对象以及对象彼此之间的一些关系。我们可以对这些副本进行修改，然后进行保存，然后MOC会通过 PSC 对 NSPersistentStore 进行操作，持久化存储就会发生变化。

CoreData中的NSManagedObjectModel 托管对象的数据模型（MOM），通过 MOC 进行注册。MOC有插入、删除以及更新数据模型等操作，并提供了撤销和重做的支持。

作用：插入数据，更新数据，删除数据

####(3)NSPersistentStoreCoordinator 持久化存储协调器（PSC）


在应用程序和外部数据存储的对象之间提供访问通道的框架对象集合统称为持久化堆栈（persistence stack）。在堆栈顶部的是托管对象上下文（MOC），在堆栈底部的是持久化对象存储（persistent object stores）。在托管对象上下文和持久化对象存储之间便是持久化存储协调器（PSC）。应用程序通过类NSPersistentStoreCoordinator的实例访问持久化对象存储。

持久化存储协调器为一或多个托管对象上下文提供一个访问接口，使其下层的多个持久化存储可以表现为单一一个聚合存储。一个托管对象上下文可以基于持久化存储协调器下的所有数据存储来创建一个对象图。持久化存储协调器只能与一个托管对象模型（MOM）相关联。

####(4)NSManagedObject 托管对象（MO）

托管对象必须继承自NSManagedObject或者NSManagedObject的子类。NSManagedObject能够表述任何实体。它使用一个私有的内部存储，以维护其属性，并实现托管对象所需的所有基本行为。托管对象有一个指向实体描述的引用。NSEntityDescription 实体描述表述了实体的元数据，包括实体的名称，实体的属性和实体之间的关系。

####(5)Controller 控制器

概念图1中绿色的 Array Controller，Object Controller，Tree Controller 这些控制器，一般都是通过 control+drag 将 Managed Object Context 绑定到它们，这样我们就可以在 nib 中可视化地操作数据

####(6)NSFetchRequest 获取数据请求

使用托管对象上下文来检索数据时，会创建一个获取请求（fetch request）。类似Sql查询语句的功能。

####(7)NSEntityDescription 实体描述
实体描述对象提供了一个实体的元数据，包括实体名（Name），类名（ClassName），属性（Properties）以及实体属性与其他实体的一些关系（Relationships）等。

####(8).xcdatamodeld

里面是.xcdatamodeld文件，用数据模型编辑器编辑，编译后为.momd或.mom文件。

我们可以选中我们的应用程序（路径类似为/Users/Childhood/Library/Application Support/iPhone Simulator/7.1/Applications/005D926F-5763-4305-97FE-AE55FE7281A4），右键显示包内容，我们看到是这样的。

我们着重理解下他们之间的协同工作关系。

这里有一个简单的demo [CoreDataDemo](https://github.com/dabing1022/CoreDataDemo)。



结合图示和代码，我们来看下他们的运作机制（下面参考[罗朝辉（飘飘白云）[Cocoa]深入浅出 Cocoa 之 Core Data（1）- 框架详解](http://blog.csdn.net/kesalin/article/details/6739319))

1.应用程序先创建或读取模型文件（后缀为xcdatamodeld）生成 NSManagedObjectModel 对象。   
2.生成 NSManagedObjectContext 和 NSPersistentStoreCoordinator 对象，MOC会设置自身的持久化存储协调器(PSC)，通过PSC来对数据文件进行读写。   
3.NSPersistentStoreCoordinator 负责从数据文件(xml，sqlite，二进制文件等)中读取数据生成 Managed Object，或保存 Managed Object 写入数据文件。    
4.NSManagedObjectContext 参与对数据进行各种操作的整个过程，它可以持有多个 Managed Object。我们通过它来监测 Managed Object。监测数据对象有两个作用：支持 undo/redo 以及数据绑定。    
5.Array Controller，Object Controller，Tree Controller 这些控制器一般与 NSManagedObjectContext 关联，因此我们可以通过它们在 nib 中可视化地操作数据对象


###2.MagicalRecord的入门准备
如上篇提到的，在工程的PCH预编译头文件中导入CoreData+MagicalRecord.h文件。因为该头文件包括了所有需要的MagicalRecord头文件。

在我们的app delegate中，或者在awakeFromNib中都可以，我们可以使用下列的方法来设置CoreData堆栈。

setup系列方法   

+ (void) setupCoreDataStack;
+ (void) setupAutoMigratingCoreDataStack;
+ (void) setupCoreDataStackWithInMemoryStore;
+ (void) setupCoreDataStackWithStoreNamed:(NSString *)storeName;
+ (void) setupCoreDataStackWithAutoMigratingSqliteStoreNamed:(NSString *)storeName;

通过调用上面的方法，我们就可以实例化一块CoreData堆栈，并且为该实例提供 getter 和 setter 方法。

需要注意的一点是，当我们的编译器在 DEBUG 模式下（DEBUG的flag为1），如果改变了定义的数据模型而没有创建新的数据模型，那么 MagicalRecord 则会删除老的存储并且会自动创建一份新的，不用在每次改变的时候进行卸载/重新安装。

在我们的app退出时，我们可以使用下面这个方法来做清理工作。

cleanUp   
[MagicalRecord cleanUp];

###3. 参考学习
[MagicalRecord的官方资料](https://github.com/magicalpanda/MagicalRecord/blob/develop/Docs/GettingStarted.md)     
[CoreDataClassOverview](http://cocoadevcentral.com/articles/000086.php)   
[iphone数据存储之 – Core Data的使用](http://www.cnblogs.com/xiaodao/archive/2012/10/08/2715477.html)    
[[Cocoa]深入浅出 Cocoa 之 Core Data（1）- 框架详解](http://blog.csdn.net/kesalin/article/details/6739319)



##三、深入浅出MagicalRecord-03
这节我们来一起学习下MagicalRecord对数据的增删改查，内容主要来自于 MagicalRecord的github资料。

###1. 增-创建实体
创建实体   

`Person *myPerson = [Person MR_createEntity];`   

指定创建的上下文中创建实体   

`Person *myPerson = [Person MR_createInContext:otherContext];`

###2. 删-删除实体
删除一个实体   

`[myPerson MR_deleteEntity];`   

删除特定上下文中的实体


`[myPerson MR_deleteInContext:otherContext];`     

删除所有实体

`[Person MR_truncateAll];`

删除特定上下文中的所有实体

`[Person MR_truncateAllInContext:otherContext];`

###3. 改-修改实体   

`Person *person = ...;`

`person.lastname = "xxx";`

###4. 查-查询实体
查询的结果通常会返回一个NSArray结果。

#####a) 基本查询
从持久化存储（PersistantStore）中查询出所有的Person实体   

`NSArray *people = [Person MR_findAll];`

查询出所有的Person实体并按照 lastName 升序（ascending）排列

`NSArray *peopleSorted = [Person MR_findAllSortedBy:@"lastName" ascending:YES];`

查询出所有的Person实体并按照 lastName 和 firstName 升序（ascending）排列

`NSArray *peopleSorted = [Person MR_findAllSortedBy:@"lastName,firstName" ascending:YES];`

查询出所有的Person实体并按照 lastName 降序，firstName 升序（ascending）排列

`NSArray *peopleSorted = [Person MR_findAllSortedBy:@"lastName:NO,firstName" ascending:YES];`

//或者

`NSArray *peopleSorted = [Person MR_findAllSortedBy:@"lastName,firstName:YES" ascending:NO];`

查询出所有的Person实体 firstName 为 Forrest 的实体

`Person *person = [Person MR_findFirstByAttribute:@"firstName" withValue:@"Forrest"];`

#####b) 高级查询
使用NSPredicate来实现高级查询。

`NSPredicate *peopleFilter = [NSPredicate predicateWithFormat:@"department IN %@", @[dept1, dept2]];`

`NSArray *people = [Person MR_findAllWithPredicate:peopleFilter];`

#####c)返回 NSFetchRequest


`NSPredicate *peopleFilter = [NSPredicate predicateWithFormat:@"department IN %@", departments];`

`NSFetchRequest *people = [Person MR_requestAllWithPredicate:peopleFilter];`

#####d)自定义 NSFetchRequest

`NSPredicate *peopleFilter = [NSPredicate predicateWithFormat:@"department IN %@", departments];`

`NSFetchRequest *peopleRequest = [Person MR_requestAllWithPredicate:peopleFilter];`

`[peopleRequest setReturnsDistinctResults:NO];`

`[peopleRequest setReturnPropertiesNamed:@[@"firstName", @"lastName"]];`

`NSArray *people = [Person MR_executeFetchRequest:peopleRequest];`

#####e)查询实体的个数
返回的是 NSNumber 类型

`NSNumber *count = [Person MR_numberOfEntities];`

基于NSPredicate查询条件过滤后的实体个数

`NSNumber *count = [Person MR_numberOfEntitiesWithPredicate:...];`

返回的是 NSUInteger 类型

+ (NSUInteger) MR_countOfEntities;
+ (NSUInteger) MR_countOfEntitiesWithContext:(NSManagedObjectContext *)context;
+ (NSUInteger) MR_countOfEntitiesWithPredicate:(NSPredicate *)searchFilter;
+ (NSUInteger) MR_countOfEntitiesWithPredicate:(NSPredicate *)searchFilter inContext:(NSManagedObjectContext *)
#####f)合计操作

`NSInteger totalFat = [[CTFoodDiaryEntry MR_aggregateOperation:@"sum:" onAttribute:@"fatCalories" withPredicate:predicate] integerValue];`


`NSInteger fattest  = [[CTFoodDiaryEntry MR_aggregateOperation:@"max:" onAttribute:@"fatCalories" withPredicate:predicate] integerValue];`


`NSArray *caloriesByMonth = [CTFoodDiaryEntry MR_aggregateOperation:@"sum:" onAttribute:@"fatCalories" withPredicate:predicate groupBy:@"month"];`


#####g)从指定上下文中查询

`NSArray *peopleFromAnotherContext = [Person MR_findAllInContext:someOtherContext];`


`Person *personFromContext = [Person MR_findFirstByAttribute:@"lastName" withValue:@"Gump" inContext:someOtherContext];`


`NSUInteger count = [Person MR_numberOfEntitiesWithContext:someOtherContext];`
     
##四、深入浅出MagicalRecord-04
这节我们来一起学习下 MagicalRecord 对数据的存储，内容主要来自于 MagicalRecord的github资料。

###存储的时机
一般情况下，我们应该在数据发生变化时就进行存储操作。有些应用选择在退出的时候存储，然而在大多数情况下这是不必要的。事实上，如果你只是当应用退出的时候进行存储，你有可能会丢失数据！如果你的应用崩溃了呢？用户会丢失他们改变的数据，这是很糟糕的体验，应该极力去避免出现这种情况。

如果你发现存储比较耗时，有下面几点你可以考虑下：

#####1.利用后台线程存储

MagicalRecord 提供了一个简洁的API来操作后台线程对实体改变的存储。例如：

`[MagicalRecord saveWithBlock:^(NSManagedObjectContext *localContext) {
    //在这里做存储工作，该闭包代码块的工作将会在后台线程运行
} completion:^(BOOL success, NSError *error) {
    [application endBackgroundTask:bgTask];
    bgTask = UIBackgroundTaskInvalid;
}];`

#####2.将存储任务拆分成细小的存储

类似导入大量数据这样的任务应该被拆分成多个小模块。一次性存储多少量的数据并没有统一的标准，所以你需要使用 Apple’s Instruments 的来测试下你的应用的性能。

###处理长时存储
#####ios平台
当退出 ios 应用时，有机会来整理和存储数据到磁盘上。如果你知道存储操作要持续一会，那么最好的方法就是请求应用延期退出。如下：


`UIApplication *application = [UIApplication sharedApplication];`

`__block UIBackgroundTaskIdentifier bgTask = [application beginBackgroundTaskWithExpirationHandler:^{
    [application endBackgroundTask:bgTask];
    bgTask = UIBackgroundTaskInvalid;
}];`

`[MagicalRecord saveWithBlock:^(NSManagedObjectContext *localContext) {
    //这里做存储操作
} completion:^(BOOL success, NSError *error) {
    [application endBackgroundTask:bgTask];
    bgTask = UIBackgroundTaskInvalid;
}];`

请确保认真阅读了 [read the documentation for beginBackgroundTaskWithExpirationHandler](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIApplication_Class/index.html)，因为不适当或者不必要的延长应用程序的生命周期可能会在审核的时候遭到拒绝。

#####OSX平台
在 OS X Mavericks (10.9) 以及后面的版本中，App Nap 可以使得应用在后台的时候可以有效的被终止退出。如果你知道存储操作要持续一会，那么最好的方法就是暂时禁用自动终止和突然终止功能（前提是你的应用支持这些功能）：


`NSProcessInfo *processInfo = [NSProcessInfo processInfo];`


`[processInfo disableSuddenTermination];`


`[processInfo disableAutomaticTermination:@"Application is currently saving to persistent store"];`


`[MagicalRecord saveWithBlock:^(NSManagedObjectContext *localContext) {
    //这里做存储操作
} completion:^(BOOL success, NSError *error) {
    [processInfo enableSuddenTermination];
    [processInfo enableAutomaticTermination:@"Application has finished saving to the persistent store"];
}];`

和 ios 实现一样，在实现之前确保阅读 [read the documentation on NSProcessInfo](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSProcessInfo_Class/index.html)来避免被拒绝。

#####变化
在 MagicalRecord 2.2 中，存储API更加一致并遵循CoreData的命名模式。在这个版本中，已经加入了自动化测试来确保将来更新时，存储工作（新的API和废弃的API）能够继续工作。

MR_save被暂时恢复到当前线程的同步运行和存储到持久存储(persistent store)的原始状态。然而，MR_save方法被标记为“将被废弃”(deprecated)，将会在下个大版本 MagicalRecord 3.0 中移除掉。你应该使用MR_saveToPersistentStoreAndWait同功能函数来替代它。

######a)新的方法
新增加了下面几个方法：

`NSManagedObjectContext+MagicalSaves`


`- (void) MR_saveOnlySelfWithCompletion:(MRSaveCompletionHandler)completion;`


`- (void) MR_saveToPersistentStoreWithCompletion:(MRSaveCompletionHandler)completion;`


`- (void) MR_saveOnlySelfAndWait;`


`- (void) MR_saveToPersistentStoreAndWait;`


`- (void) MR_saveWithOptions:(MRSaveContextOptions)mask completion:(MRSaveCompletionHandler)completion;
MagicalRecord+Actions`


`+ (void) saveWithBlock:(void(^)(NSManagedObjectContext *localContext))block;`


`+ (void) saveWithBlock:(void(^)(NSManagedObjectContext *localContext))block completion:(MRSaveCompletionHandler)completion;`


`+ (void) saveWithBlockAndWait:(void(^)(NSManagedObjectContext *localContext))block;`


`+ (void) saveUsingCurrentThreadContextWithBlock:(void (^)(NSManagedObjectContext *localContext))block completion:(MRSaveCompletionHandler)completion;`



`+ (void) saveUsingCurrentThreadContextWithBlockAndWait:(void (^)(NSManagedObjectContext *localContext))block;`

######b)标记为废弃的函数
下面这些函数被标记为“废弃的”，将会在 MagicalRecord 3.0 版本移除掉，推荐使用替代的函数。

`NSManagedObjectContext+MagicalSaves`


`- (void) MR_save;`
`- (void) MR_saveWithErrorCallback:(void(^)(NSError *error))errorCallback;`
`- (void) MR_saveInBackgroundCompletion:(void (^)(void))completion;`
`- (void) MR_saveInBackgroundErrorHandler:(void (^)(NSError *error))errorCallback;`
`- (void) MR_saveInBackgroundErrorHandler:(void (^)(NSError *error))errorCallback completion:(void (^)(void))completion;`
`- (void) MR_saveNestedContexts;`
`- (void) MR_saveNestedContextsErrorHandler:(void (^)(NSError *error))errorCallback;`
`- (void) MR_saveNestedContextsErrorHandler:(void (^)(NSError *error))errorCallback completion:(void (^)(void))completion;
MagicalRecord+Actions`



