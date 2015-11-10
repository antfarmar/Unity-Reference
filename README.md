# Unity Cheat Sheet
*Code snippets, Techniques, Optimizations, Links*

# Index
1. [Code Snippets](#code-snippets)
  * [Coroutines](#coroutines)
2. [Quick Links](#quick-links)
  * [Official](#unity-website)
  * [Documentation](#documentation)
  * [Community](#community)
  * [Sripting](#scripting)
3. [Declarations](#declarations)


# Code Snippets
## Coroutines
[(video tutorial)](https://unity3d.com/learn/tutorials/modules/intermediate/scripting/coroutines)
* [`Coroutine`](http://docs.unity3d.com/ScriptReference/Coroutine.html) inherits from [`YieldInstruction`](http://docs.unity3d.com/ScriptReference/YieldInstruction.html)
* A function that can suspend its execution (yield) until the given `YieldInstruction` finishes.
* Can be used as a way to spread an effect over a period time, but it is also a **useful optimization**:
 * When a task doesn’t need to be repeated quite so frequently, you can put it in a coroutine to get an update regularly but not every single frame. *i.e. Calling an expensive function every frame in `Update()` might introduce significant overhead. To overcome this, use a coroutine to call it, say, every tenth of a second instead.*
```csharp 
void Update() {
    StartCoroutine(SomeCoroutine);
}

IEnumerator SomeCoroutine() {
    while(true) {
        ExpensiveFunction();
        yield return new WaitForSeconds(.1f);
    }
}
```
### Usage
Use [StartCoroutine](http://docs.unity3d.com/ScriptReference/MonoBehaviour.StartCoroutine.html) methods to start a coroutine.
```csharp
public Coroutine StartCoroutine(IEnumerator method); // Typical usage. Just pass the method call as parameter.
public Coroutine StartCoroutine(string methodName, object value = null); // Higher runtime overhead to start the coroutine this way; can pass only one parameter.
```
Use [StopCoroutine](http://docs.unity3d.com/ScriptReference/MonoBehaviour.StopCoroutine.html) methods to stop a coroutine.
```csharp
public void StopCoroutine(IEnumerator routine); // Stops the coroutine stored in routine running on this behaviour.
public void StopCoroutine(string methodName); // Stops the first coroutine named methodName.
public void StopAllCoroutines(); // Stops all coroutines running on this behaviour.
```
Example usage:
```csharp	
// Need a reference to a specific coroutine instance to stop it this way.
IEnumerator instance = null;
 
 // Start coroutine
 instance = SomeCoroutine(a, b, c);
 StartCoroutine(instance); // or instance = StartCoroutine(SomeCoroutine (a, b, c)); (Coroutine continue failure?)
 
 // Stop coroutine
 StopCoroutine(instance);
``` 

```csharp
void Start() {
  print("Starting " + Time.time);
  StartCoroutine(WaitAndPrint(2.0F));
  print("Before WaitAndPrint Finishes " + Time.time);
}

IEnumerator WaitAndPrint(float waitTime) {
  yield return new WaitForSeconds(waitTime);
    print("WaitAndPrint " + Time.time);
}
```

### Coroutine Return Types
```csharp	
yield return null; // A yield instruction that just returns.
yield return new WaitForSeconds(t); // A yield instruction that waits for t seconds.
yield new WWW(url); // A yield instruction that waits for the retrieval of contents of URLs.
yield return new WaitForFixedUpdate(); // Waits until next fixed frame rate update function. 
yield StartCoroutine(routine) // Can also wait for a coroutine to finish execution.
```

# Quick Links
## Unity Website
### Documentation
* [Unity Manual](http://docs.unity3d.com/Manual/index.html)
* [Scripting API](http://docs.unity3d.com/ScriptReference/index.html)

### Community
* [Unity Forums](http://forum.unity3d.com/)
* [Unity Answers](http://answers.unity3d.com/)

### Scripting
* [Scripting API](http://docs.unity3d.com/ScriptReference/index.html)
* [Scripting Tutorials: Concise Videos & Code Snippets](http://unity3d.com/learn/tutorials/topics/scripting)

![image title](image.png)
