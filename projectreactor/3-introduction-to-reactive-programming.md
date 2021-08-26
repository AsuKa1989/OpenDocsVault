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

响应式库，例如 Reactor，旨在解决 JVM 上“经典”异步方法的这些缺点，同时还关注一些其他方面：

- 可组合性和可读性
- 数据将被视为各种操作符（operators）操作的流（flow）

- 在被订阅（subscribe）之前，什么都不会发生
- 背压（Backpressure）或者消费者向生产者发出emission过高的信号的能力

- 与并发无关的高难度但高价值的抽象概念

### 3.3.1.可组合性和可读性

“可组合性”是指编排多个异步任务的能力。我们使用先前任务的结果作为输入提供给后续任务。或者，我们可以以 fork-join 方式运行多个任务。此外，我们可以在更高级别的系统中将异步任务重用为离散组件。

编排任务的能力与代码的可读性和可维护性紧密相关。随着异步进程层数和复杂性的增加，编写和读取代码变得越来越困难。正如我们所见，回调模型很简单，但它的主要缺点之一是，对于复杂的流程，您需要从一个回调中执行一个回调，而回调本身又嵌套在另一个回调中，依此类推。这种混乱被称为“回调地狱”。您可以猜到（或从经验中知道），这样的代码很难回溯和推理。

Reactor 提供了丰富的组合选项，其中代码反映了抽象过程的组织，并且所有内容通常都保持在同一级别（最小化嵌套）。

### 3.3.2.装备线类比

您可以将响应式应用程序处理的数据视为在装配线移动的原材料。Reactor 既是传送带又是工作站。原材料从一个来源（原始Publisher）产出，最终成为准备好推送给消费者（或Subscriber）的成品。

原材料可以经过各种转换和其他中间步骤，或者成为将中间件聚合在一起的更大装配线的一部分。如果某一点出现故障或堵塞（可能装箱产品花费的时间过长），受影响的工作站可以向上游发出信号以限制原材料的流动。

### 3.3.3.操作符

在 Reactor 中，操作符是我们装配类比中的工作站。每个操作符向发布者（Publisher）添加行为，并将上一步的发布者包（Publisher）装到一个新实例中。整个链因此被链接起来，这样数据来自第一个发布者（Publisher）并沿着链向下移动，由每个链接转换。最终，订阅者（ Subscriber）通过订阅完成该过程。请记住，在订阅者（Subscriber）订阅发布者（ Publisher）之前什么都不会发生，我们很快就会看到。

了解操作符创建新实例可以帮助您避免常见错误，该错误会导致您认为您在链中使用的操作符没有被应用。请参阅常见问题解答中的此项。

虽然 Reactive Streams 规范没有指定操作符，但 Reactor 等响应式库的最佳附加值之一是它们提供的丰富的操作符。这些操作符涵盖了很多方面，从简单的转换和过滤到复杂的编排和错误处理。

### 3.3.4.订阅之前，无事发生

在 Reactor 中，当您编写发布者（Publisher）链时，默认情况下不会开始向其中注入数据。相反，您创建的是异步流程的抽象描述（这有助于可重用性和组合）。

通过订阅（subscribing,）行为，您将发布者（Publisher）与订阅者（Subscriber）联系起来，这会触发整个链中的数据流。这是通过来自订阅者的单个请求（ request）信号在内部实现的，该信号向上游传播，一直返回到源发布者（Publisher）。

### 3.3.5.背压

向上游传播信号也用于实现背压（backpressure），我们在装配线类比中将其描述为当工作站处理比上游工作站慢时向上线发送的反馈信号。

Reactive Streams 规范定义的真实机制非常接近类比：订阅者可以在无界模式下工作，让源以最快的速度 push 所有数据，或者它可以使用请求机制向源发出信号准备处理最多 n 个元素。

中间操作符还可以更改传输中的请求。想象一个缓冲区（buffer）运算符，它将元素以 10 个为一组进行分组。如果订阅者请求一个缓冲区，源生成十个元素是可以接受的。一些操作符还实现了预取（prefetching）策略，这避免了 request(1) 反复请求。如果在请求之前生成元素不会太耗费资源，则这是有利的。

被压将单纯的 push 模型转变为 push-pull 混合模型。如果上游元素随时可用，下游可以从上游 pull n 个元素。但是如果元素没有准备好，这些元素则会在生产好后被上游 push。

### 3.3.6.热流 VS 冷流

Rx 系列响应式库区分了两大类响应式流：热流和冷流。这种区别主要与响应式流如何对订阅者做出反应有关：

- 冷流为每个订阅者重新开始，包括在数据源处。例如，如果源包装了一个 HTTP 调用，则为每个订阅发出一个新的 HTTP 请求。
- 热流不会为每个订阅者从头开始。相反，后来的订阅者会收到订阅后发出的信号。但是请注意，一些热流可以全部或部分缓存或重放排放历史。从一般的角度来看，当没有订阅者正在监听时，热流甚至可以产生数据（“订阅前什么都不会发生”规则的一个例外）。

有关 Reactor 中热流与冷流的更多信息，请参阅 [reactor-specific](https://projectreactor.io/docs/core/release/reference/#reactor.hotCold) 部分。