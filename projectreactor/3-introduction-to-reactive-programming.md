# 3-响应式编程简介

Reactor 是 Reactive Programming 范式的一个实现，可以总结如下：

> 响应式编程是一种异步编程范式，涉及数据流和变化的传播。这意味着可以通过所采用的编程语言轻松表达静态（例如数组）或动态（例如事件发射器）数据流。
> —— https://en.wikipedia.org/wiki/Reactive_programming

作为响应式编程方向的第一步，微软在 .NET 生态系统中创建了响应式扩展 (Rx) 库。然后 RxJava 在 JVM 上实现了响应式编程。随着时间的推移，通过 Reactive Streams 的努力，出现了 Java 的响应式标准，该标准为 JVM 上的响应式库定义了一组接口和交互规则。此接口已集成到 Java 9 的 Flow 类下。

响应式编程标准的实现通常以面向对象的语言呈现，作为观察者设计模式的扩展。您还可以将主要的流模式与熟悉的 Iterator 设计模式进行比较，因为所有这些库中的 Iterable-Iterator 对都具有二元性。两者主要区别之一是：迭代器是基于拉的，响应式流是基于推的。

**迭代**是一种命令式编程模式，即访问值的方法完全是 Iterable 的责任。在响应式中，由开发人员选择何时访问序列中的 next() 项。在响应式中，是发布者-订阅者模式。但是是发布者在新可用值到来时通知订阅者，而这种推送方面的特性是响应式的关键。此外，应用于推送值的操作是以声明式而不是命令式表达的：程序员表达计算的逻辑而不是描述其确切的控制流程。

除了推送值之外，错误处理和完成方面也以明确定义的方式进行了规范。发布者可以向其订阅者推送新值（通过调用 onNext），也可以发出错误信号（通过调用 onError）或完成（通过调用 onComplete）。错误信号和完成信号都会终止数据流。这可以总结如下：

```
onNext x 0..N [onError | onComplete]
```

上述方法非常灵活。该模式支持没有值、一个值或 n 个值（包括无限的数据流，例如时钟的连续滴答）。

但是我们首先要回答一个问题：为什么我们需要这样一个异步响应式库？

## 3.1.阻塞造成浪费

现代应用程序可以支持大量并发用户，尽管现代硬件的性能不断提高，但现代软件的性能仍然是一个关键问题。

总的来说，有两种方法可以提高程序的性能：

- 并发以使用更多线程和更多硬件资源。
- 使用当前硬件资源需求更高效率。

通常，Java 开发人员使用阻塞代码编写程序。这种做法很好，直到出现性能瓶颈。然后适时引入额外的线程，运行类似的阻塞代码。但是这种资源利用率的扩展会很快引入争用和并发问题。

更糟糕的是，阻塞会浪费资源。如果您仔细观察，一旦程序涉及一些延迟（特别是 I/O，例如数据库请求或网络调用），资源就会被浪费。因为线程（可能很多线程）处于空闲状态，等待数据返回。

所以并发不是银弹。其需要访问硬件的全部功能，不仅逻辑复杂，且容易浪费资源。

## 3.2.异步解决问题

前面提到的第二种方法，寻求更高的效率，可以解决资源浪费问题。通过编写异步、非阻塞代码，您可以让代码切换到另一个使用相同底层资源的活动任务，并在异步处理完成后返回到当前进程。

但是如何在 JVM 上生成异步代码呢？ Java 提供了两种异步编程模型：

- **回调**：异步方法没有返回值，但需要一个额外的回调参数（一个 lambda 或匿名类），当结果可用时会调用该参数。一个众所周知的例子是 Swing 的 EventListener 层次结构。
- **Futures**：异步方法立即返回 Future<T>。异步进程计算 T 值，Future 对象包装了对它的访问。该值不会立即可用，以轮询该对象直到该值可用。例如，运行 Callable<T> 任务的 ExecutorService 使用 Future 对象。

这些技术够好吗？并非适用于每个场景，两种方法都有局限性。

回调很难组合在一起，很快导致代码难以阅读和维护（称为“回调地狱”）。

思考一个例子：在用户界面上显示用户最喜欢的前五个选项，如果没有她最喜欢的则给出推荐。这会经过三个服务（一个提供最喜欢选项的 ID，第二个获取最喜欢选项的详细信息，第三个提供带有详细信息的推荐），如下所示：

```java
userService.getFavorites(userId, new Callback<List<String>>() {  // 我们有基于回调的服务：一个回调接口，其中一个方法在异步过程成功时调用，一个在发生错误时调用。
  public void onSuccess(List<String> list) {  // 第一个服务使用收藏夹 ID 列表调用其回调。
    if (list.isEmpty()) { // 如果列表为空，我们必须转到推荐服务。 
      suggestionService.getSuggestions(new Callback<List<Favorite>>() {
        public void onSuccess(List<Favorite> list) {  // 推荐服务将 List<Favorite> 提供给第二个回调。
          UiUtils.submitOnUiThread(() -> {  // 由于我们处理的是 UI，我们需要确保我们的消费代码在 UI 线程中运行。
            list.stream()
                .limit(5)
                .forEach(uiList::show);  // 我们使用 Java 8 Stream 将处理的推荐数量限制为 5，并在 UI 的图形列表中显示它们。
            });
        }

        public void onError(Throwable error) {  // 在所有情况，我们都以相同的方式处理错误：我们在弹出窗口中显示它们。
          UiUtils.errorPopup(error);
        }
      });
    } else {
      list.stream()  // 回到最喜欢选项的ID服务。如果服务返回了一个完整的列表，我们需要去 favoriteService 获取详细的 Favorite 对象。由于我们只需要五个，因此我们首先流式传输 ID 列表以将其限制为五个。
          .limit(5)
          .forEach(favId -> favoriteService.getDetails(favId,  // 又一次回调。这次我们得到了一个完全成熟的收藏对象，我们将其推送到 UI 线程内的 UI。
            new Callback<Favorite>() {
              public void onSuccess(Favorite details) {
                UiUtils.submitOnUiThread(() -> uiList.show(details));
              }

              public void onError(Throwable error) {
                UiUtils.errorPopup(error);
              }
            }
          ));
    }
  }

  public void onError(Throwable error) {
    UiUtils.errorPopup(error);
  }
});
```

以上代码，有点难理解并且有重复的部分。看看它在 Reactor 中的等价物：

```
userService.getFavorites(userId)  // 我们从最喜欢的选项 ID 流开始。
           .flatMap(favoriteService::getDetails)  // 我们将这些异步转换为详细的收藏对象（flatMap）。我们现在有一个收藏流。
           .switchIfEmpty(suggestionService.getSuggestions())  // 如果Favorite的流程是空的，我们通过suggestService切换到fallback。
           .take(5)  // 我们最多只对结果流中的五个元素感兴趣。
           .publishOn(UiUtils.uiThreadScheduler())  // 最后，我们要在 UI 线程中处理每条数据。
           .subscribe(uiList::show, UiUtils::errorPopup);  // 我们通过描述如何处理数据的最终形式（在 UI 列表中显示）以及在出现错误时如何处理（显示弹出窗口）来触发流程。
```

如果您想确保在 800 毫秒内检索到最喜欢选项的 ID，或者如果需要更长的时间，从缓存中获取它们，该怎么办？在基于回调的代码中，这是一项复杂的任务。在 Reactor 中，在链中添加超时操作符非常简单，如下所示：

```java
userService.getFavorites(userId)
           .timeout(Duration.ofMillis(800)) // 如果上面的部分在 800 毫秒内什么都不产生，产生一个错误
           .onErrorResume(cacheService.cachedFavoritesFor(userId))  // 如果出现错误，请回退到 cacheService。
           .flatMap(favoriteService::getDetails)  // 链的其余部分与前面的示例类似。
           .switchIfEmpty(suggestionService.getSuggestions())
           .take(5)
           .publishOn(UiUtils.uiThreadScheduler())
           .subscribe(uiList::show, UiUtils::errorPopup);
```

Future 对象比回调要好一些，但它们在组合方面仍然表现不佳。尽管 CompletableFuture 在 Java 8 中进行了改进。将多个 Future 对象编排在一起是可行的，但并不简单。此外，Future 还有其他问题：

- 通过调用 get() 方法很容易导致 Future 对象出现另一种阻塞情况。
- 它们不支持惰性计算。

- 它们缺乏对多个值和高级错误处理的支持。

思考另一个例子：我们得到一个 ID 列表，我们想从中获取一个名称和一个统计信息，并将这些成对地组合起来，所有这些操作都是异步的。以下示例使用 CompletableFuture 类型的列表执行此操作： 

```java
CompletableFuture<List<String>> ids = ifhIds();  // 我们从一个 Future 开始，它为我们提供了要处理的 id 值列表。

CompletableFuture<List<String>> result = ids.thenComposeAsync(l -> {  // 一旦我们得到列表，我们想开始一些更深层次的异步处理。
	Stream<CompletableFuture<String>> zip =
			l.stream().map(i -> {  // 遍历列表中的每个元素
				CompletableFuture<String> nameTask = ifhName(i);  // 异步获取关联的名称。
				CompletableFuture<Integer> statTask = ifhStat(i); // 异步获取关联的任务。

				return nameTask.thenCombineAsync(statTask, (name, stat) -> "Name " + name + " has stats " + stat);  // 合并两个结果。
			});
	List<CompletableFuture<String>> combinationList = zip.collect(Collectors.toList());  // 我们现在有一个代表所有组合任务的期货列表。要执行这些任务，我们需要将列表转换为数组。
	CompletableFuture<String>[] combinationArray = combinationList.toArray(new CompletableFuture[combinationList.size()]);

	CompletableFuture<Void> allDone = CompletableFuture.allOf(combinationArray);  // 将数组传递给 CompletableFuture.allOf，它会在所有任务完成后输出一个完成的 Future。
	return allDone.thenApply(v -> combinationList.stream()
			.map(CompletableFuture::join)  // 棘手的一点是 allOf 返回 CompletableFuture<Void>，所以我们重申期货列表，通过使用 join() 收集它们的结果（这里不会阻塞，因为 allOf 确保期货全部完成）。
			.collect(Collectors.toList()));
});

List<String> results = result.join();  // 一旦整个异步管道被触发，我们等待它被处理并返回我们可以断言的结果列表。
assertThat(results).contains(
		"Name NameJoe has stats 103",
		"Name NameBart has stats 104",
		"Name NameHenry has stats 105",
		"Name NameNicole has stats 106",
		"Name NameABSLAJNFOAJNFOANFANSF has stats 121");
```

由于 Reactor 有更多的开箱即用的组合操作符，因此可以简化此过程，如下所示：

```java
Flux<String> ids = ifhrIds();  // 首先，我们异步操作中获取 id 流（Flux<String>）。

Flux<String> combinations =
		ids.flatMap(id -> {  // 对于流中的每个元素，我们异步处理它（在函数体 flatMap 调用中）两次。
			Mono<String> nameTask = ifhrName(id);  // 获取关联的名称。
			Mono<Integer> statTask = ifhrStat(id); // 获取相关的统计信息。

			return nameTask.zipWith(statTask,  // 异步组合这两个值。
					(name, stat) -> "Name " + name + " has stats " + stat);
		});

Mono<List<String>> result = combinations.collectList();  // 当这些值可用时，将它们聚合到一个列表中。

List<String> results = result.block(); // 在实际使用中，我们将通过进一步组合或订阅 Flux 来继续异步使用 Flux。最有可能的是，我们会返回结果 Mono。这里由于我们在测试，我们改为阻塞，等待处理完成，然后直接返回聚合的值列表。
assertThat(results).containsExactly(  // 断言结果。
		"Name NameJoe has stats 103",
		"Name NameBart has stats 104",
		"Name NameHenry has stats 105",
		"Name NameNicole has stats 106",
		"Name NameABSLAJNFOAJNFOANFANSF has stats 121"
);
```

使用回调和 Future 对象的风险是相似的。响应式编程解决是通过"发布者-订阅者"对解决问题。

## 3.3.从命令式编程到响应式编程