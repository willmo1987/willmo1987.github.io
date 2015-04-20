---
layout: post
title: "CoreData多线程下NSManagedObjectContext的使用"
tags: [CoreData]
categories: [iOS]
---

> **本博客为个人原创，转载需在明显位置注明出处**

&emsp;&emsp;本人去年10月份才接触IOS项目的，项目中用到了CoreData，由于需要调用子线程进行API请求，所以需要在子线程中使用NSManagedObjectContext进行数据的增删改查。因此当时百度了很多相关东西，但是没有得到我要的答案，后来硬着头皮Google了很多英文博客和论坛（本人英文水平其实还可以，英文文档看起来也没那么吃力，但总归不是母语，到现在都有一种抵触感），研究的也是一知半解，由于项目进度不能耽搁，也没有深究，最后选用了persistentStoreCoordinator<-mainContext<-privateContext这样的结构，让我完成了需求。

&emsp;&emsp;由于刚过完年，手头上的工作不多，想着重构一下之前写的代码。由于之前CoreData的代码虽然让我完成了项目需求，但是始终没有让我感到有安全感，所以重构代码的第一件事就是好好研究CoreData。在Google的时候，我发现了这样两篇老外的博客，[博客1](http://floriankugler.com/blog/2013/4/2/the-concurrent-core-data-stack) ，[博客2](http://floriankugler.com/blog/2013/4/29/concurrent-core-data-stack-performance-shootout)。前者是介绍NSManagedObjectContext在多线程下的三种设计，后者是博主对这三种设计进行的性能测试。下面我将一一介绍：

1. persistentStoreCoordinator<-mainContext<-privateContext

    &emsp;&emsp;这种设计就是我之前在项目中使用的，也是阻塞UI线程最严重的一种设计。它总共有两个Context，一个是UI线程中使用的mainContext，一个是子线程中使用的privateContext，他们的关系是privateContext.parentContext = mainContext，而mainContext是与Disk连接的Context，所以这种设计下，每当子线程privateContext进行save操作以后，它会将数据库所有变动Push up到其父Context，也就是mainContext中去，注意：这时子线程的save操作并没有任何关于Disk IO的操作。而后mainContext在UI线程又要执行一次save操作才能真正将数据变动写进数据库中，这里的save操作就与Disk IO有关了，而且又是在主线程，所以说这种设计是最阻碍UI线程的。

2. persistentStoreCoordinator<-backgroundContext<-mainContext<-privateContext

    &emsp;&emsp;这种设计是第一种的改进设计，也是上述的老外博主推荐的一种设计方式。它总共有三个Context，一是连接persistentStoreCoordinator也是最底层的backgroundContext，二是UI线程的mainContext，三是子线程的privateContext，后两个Context在1中已经介绍过了，这里就不再具体介绍，他们的关系是privateContext.parentContext = mainContext, mainContext.parentContext = backgroundContext。下面说说它的具体工作流程。

    &emsp;&emsp;在应用中，如果我们有API操作，首先我们会起一个子线程进行API请求，在得到Response后要进行数据库操作，这是我们要创建一个privateContext进行数据的增删改查，然后call privateContext的save方法进行存储，这里的save操作只是将所有数据变动Push up到它的父Context中也就是mainContext中，然后mainContext继续call save方法，将数据变动Push up到它的父Context中也就是backgroundContext，最后调用backgroundContext的save方法真正将数据变动存储到Disk数据库中，在这个过程中，前两个save操作相对耗时较少，真正耗时的操作是最后backgroundContext的save操作，因为只有它有Disk IO的操作。

3. persistentStoreCoordinator<-mainContext&emsp;persistentStoreCoordinator<-privateContext

    &emsp;&emsp;第三种设计是最直观的一种设计，无论是mainContext还是privateContext都是连接persistentStoreCoordinator的。这种设计的工作流程是：首先在ViewController中要添加一个名为NSManagedObjectContextDidSaveNotification的通知，然后子线程中创建privateContext，进行数据增删改查操作，直接save到本地数据库，这时在ViewController中会回调之前注册的NSManagedObjectContextDidSaveNotification的回调方法，在该方法中调用mainContext的mergeChangesFromContextDidSaveNotification:notification方法，将所有的数据变动merge到mainContext中，这样就保持了两个Context中的数据同步。由于大部分的操作都是privateContext在子线程中操作的，所以这种设计是UI线程耗时最少的一种设计，但是它的代价是需要多写mergeChanges的方法。（注：前两种parent,child的Context，一旦childContext调用save方法，其parentContext不用任何merge操作，CoreData自动将数据merge到parentContext当中）

####总结：

&emsp;&emsp;第一种设计是失败的设计，完全可以不考虑，第二种设计比较复杂繁琐，但是它是最方便而且UI线程的阻塞时间也是相对较少的一种。第三种设计是最少阻塞UI的一种，但是这种设计操作比较繁琐，应用场合是数据量比较大的应用，一般会应用在企业应用中，所以如果你不是企业级的应用或者不是数据量很大的应用，我还是推荐第二种设计。我根据第二种设计写了一个Demo，下面附上一些关键代码供大家参考：

#####注：在参考代码之前大家可能要对NSManagedObjectContext的concurrencyType有所了解，还有就是performBlock和performBlockAndWait:方法的区别。本人也是初学者，头一次写博客，敬请各位大侠斧正。

这是appDelegate中的backgroundContext  

{% highlight objective-c linenos %}
-(NSManagedObjectContext *)rootObjectContext {  
  if (nil != _rootObjectContext) {  
    return _rootObjectContext;  
  }   
  NSPersistentStoreCoordinator *coordinator = [self persistentStoreCoordinator];  
  if (coordinator != nil) {  
    _rootObjectContext = [[NSManagedObjectContext alloc] initWithConcurrencyType:NSPrivateQueueConcurrencyType];  
    [_rootObjectContext setPersistentStoreCoordinator:coordinator];  
  }  
  return _rootObjectContext;  
}
{% endhighlight %}
	
这是mainContext  

{% highlight objective-c linenos %}
- (NSManagedObjectContext *)managedObjectContext  {  
  if (nil != _managedObjectContext) {  
    return _managedObjectContext;  
  }  
  _managedObjectContext = [[NSManagedObjectContext alloc] initWithConcurrencyType:NSMainQueueConcurrencyType];  
  _managedObjectContext.parentContext = [self rootObjectContext];  
  return _managedObjectContext;  
    
  //  _managedObjectContext = [[NSManagedObjectContext alloc] init];  
  //  _managedObjectContext.persistentStoreCoordinator = [self persistentStoreCoordinator];  
  //  return _managedObjectContext;  
}
{% endhighlight %}
	
AppDelegate中saveContext方法，每次privateContext调用save方法成功之后都要call这个方法  

{% highlight objective-c linenos %}
- (void)saveContextWithWait:(BOOL)needWait  {  
  NSManagedObjectContext *managedObjectContext = [self managedObjectContext];  
  NSManagedObjectContext *rootObjectContext = [self rootObjectContext];  
      
  if (nil == managedObjectContext) {  
    return;  
  }  
  if ([managedObjectContext hasChanges]) {  
    NSLog(@"Main context need to save");  
    [managedObjectContext performBlockAndWait:^{  
      NSError *error = nil;  
      if (![managedObjectContext save:&error]) {  
        NSLog(@"Save main context failed and error is %@", error);  
      }  
    }];  
  }  
      
  if (nil == rootObjectContext) {  
    return;  
  }  
      
  RootContextSave rootContextSave = ^ {  
    NSError *error = nil;  
    if (![_rootObjectContext save:&error]) {  
      NSLog(@"Save root context failed and error is %@", error);  
    }  
  };  
      
  if ([rootObjectContext hasChanges]) {  
    NSLog(@"Root context need to save");  
    if (needWait) {  
      [rootObjectContext performBlockAndWait:rootContextSave];  
    }  
    else {  
      [rootObjectContext performBlock:rootContextSave];  
    }  
  }  
}
{% endhighlight %}
	
这是伪API方法，仅供Demo使用  

{% highlight objective-c linenos %}
+(void)getEmployeesWithMainContext:(NSManagedObjectContext *)mainContext completionBlock:(CompletionBlock)block {  
  NSManagedObjectContext *workContext = [NSManagedObjectContext generatePrivateContextWithParent:mainContext];  
  [workContext performBlock:^{  
    Employee *employee = [NSEntityDescription insertNewObjectForEntityForName:@"Employee" inManagedObjectContext:workContext];  
    [employee setRandomData];  
    NSError *error = nil;  
    if([workContext save:&error]) {  
      block(YES, nil, nil);  
    }  
    else {  
      NSLog(@"Save employee failed and error is %@", error);  
      block(NO, nil, @"Get emploree failed");  
    }  
  }];  
}
{% endhighlight %}
      
这是NSManagedObjectContext的Category  

{% highlight objective-c linenos %}
@implementation NSManagedObjectContext (GenerateContext)  
	
+(NSManagedObjectContext *)generatePrivateContextWithParent:(NSManagedObjectContext *)parentContext {  
  NSManagedObjectContext *privateContext = [[NSManagedObjectContext alloc] initWithConcurrencyType:NSPrivateQueueConcurrencyType];  
  privateContext.parentContext = parentContext;  
  return privateContext;  
}  
  
+(NSManagedObjectContext *)generateStraightPrivateContextWithParent:(NSManagedObjectContext *)mainContext {  
  NSManagedObjectContext *privateContext = [[NSManagedObjectContext alloc] init];  
  privateContext.persistentStoreCoordinator = mainContext.persistentStoreCoordinator;  
  return privateContext;  
}
  
@end
{% endhighlight %}
	
这是ViewController里API操作和UI刷新的相关代码，从refreshData方法  

{% highlight objective-c linenos %}
-(void)refreshData {  
  [EmployeeTool getEmployeesWithMainContext:[self mainContext] completionBlock:^(BOOL operationSuccess, id responseObject, NSString *errorMessage) {  
    if ([NSThread isMainThread]) {  
      NSLog(@"Handle result is main thread");  
      [self handleResult:operationSuccess];  
    }  
    else {  
      NSLog(@"Handle result is other thread");  
      [self performSelectorOnMainThread:@selector(handleResult:) withObject:[NSNumber numberWithBool:operationSuccess] waitUntilDone:YES];  
    }  
  }];  
}  
  
-(void)handleResult:(BOOL)operationSuccess {  
  if (operationSuccess) {  
    NSLog(@"Operation success");  
    [self saveContext];  
  }  
  [self.refreshControl endRefreshing];  
}  
  
-(void)saveContext {  
  WMAppDelegate *appDelegate = (WMAppDelegate*)[UIApplication sharedApplication].delegate;  
  [appDelegate saveContextWithWait:NO];  
}
{% endhighlight %}