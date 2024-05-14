+++
author = "Abhishek AN"
title = "Cloud Firestore🔥 with Kotlin Coroutines and Flow API"
date = "2021-02-16"
description = "Implementing a real time listener using Firestore with coroutines"
tags = [
    "firebase",
    "firestore",
    "kotlin",
    "coroutines",
    "flow"
]
categories = [
    "firebase",
    "android"
]
series = ["Firebase"]
+++

We’ve heard a lot about Kotlin flow since its launch in March last year.

So what is a flow?

> A flow is a type that can emit multiple values sequentially, as opposed to suspend functions that return only a single value.

If you’ve used Firestore before you would know that you can retrieve data once or have realtime updates for that data using its realtime listener.

### Real time listener for a document with Firestore

Before we begin, make sure you have the coroutines dependency

```
implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.4.2'
```

Now let’s begin by creating a State class for handling the states of operations that happen.

{{< highlight html >}}
sealed class State<out T> {
    class Loading<out T> : State<T>()
    data class Success<out T>(val data: T) : State<T>()
    data class Failed<out T>(val message: String) : State<T>()
}
{{< /highlight >}}

Let’s make the repository that gets and listens to updates in realtime for our document.

{{< highlight html >}}
@ExperimentalCoroutinesApi
class PostRepository {
    fun getPostData(): Flow<State<PostModel>> = callbackFlow {
        offer(State.Loading())
        val postDocument = Firebase.firestore
                .collection(AppConfig.POSTS_COLLECTION)
                .document("post1")

        val subscription = postDocument.addSnapshotListener { snapshot, exception ->
            exception?.let {
                offer(State.Failed(it.message.toString()))
                cancel(it.message.toString())
            }
            if (snapshot!!.exists()) {
                offer(State.Success(snapshot.toObject(PostModel::class.java)!!))
            }
        }
        awaitClose { subscription.remove() }
    }
}
{{< /highlight >}}

As you can see above I’ve used callbackFlow as you cannot use emit() with the snapshot listener as it is already an async call. Therefore the callbackFlow provides a synchronized way of doing it with offer().

We are also closing the flow stream if any exception occurs while attempting to read data. In the end we are removing the listener that was attached in **awaitClose**. This lambda is called when the parent coroutine is cancelled.

Now the viewmodel for the interaction between activity and repository.

{{< highlight html >}}
@ExperimentalCoroutinesApi
class PostViewModel(private val repo: PostRepository) : ViewModel() {

    fun getPost() = repo.getPostData()

}
{{< /highlight >}}

Here the viewmodel receives the Flow with its state.

Now the code inside the **callbackFlow** {…} in the repository is not called until that flow is collected.

Now in the activity / UI, the collect method is a suspended method thus it has to be called from a suspend method.

{{< highlight html >}}
CoroutineScope(Dispatchers.Main).launch {
            loadPost()
}

private suspend fun loadPost() {
  viewModel.getPost().collect {
            when (it) {
                is State.Loading -> {
                    Timber.i("LOADING")
                }
                is State.Success -> {
                    Timber.i("${it.data}")
                    Timber.i("SUCCESS")
                }
                is State.Failed -> {
                    Timber.i(it.message)
                }
            }
        }
}
{{< /highlight >}}

Now you can collect the post data from the flow as updates happen in real time and handle it if the State is successful.

And that’s all, I hope you find this article interesting and useful. If you like this post do share it with others! Thank you for reading!

You can also read this amazing [article](https://medium.com/firebase-developers/firebase-ing-with-kotlin-coroutines-flow-dab1bc364816) by [Shreyas Patil](https://medium.com/@patilshreyas)