# Unity Cheat Sheet
***Code snippets, Techniques, Optimizations, Links***
-----------------------------------------------------------
![Entity-Component Pattern](Entity-Component.png)
-----------------------------------------------------------
# Index
1. [Code Snippets](#code-snippets)
2. [Techniques](#techniques)
  * [Coroutines](#coroutines)
  * [Events](#events)
3. [Optimizations](#optimizations)
  * to do
4. [Quick Links](#quick-links)
  * [Unity3D Website](#unity-website)
  * [Documentation](#documentation)
  * [Community](#community)
  * [Scripting](#scripting)
  * [GameObject|Component|Mono|Behaviour|Transform](#gameobjectcomponentmonobehaviourtransform)
  * [Vectors|Math|Physics|Input](#vectorsmathphysicsinput)
  * [Optimizations](#optimizations)
  * [Miscellaneous](#miscellaneous)
 
-----------------------------------------------------------
# Code Snippets

-----------------------------------------------------------
# Techniques
## Coroutines
**Execution Time Sharing** _(not multi-threading, concurrency, or parallelism)_

[[Video Tutorial]](https://unity3d.com/learn/tutorials/modules/intermediate/scripting/coroutines)
[[Unite Talk]](http://unity3d.com/learn/resources/extending-coroutines-power-and-glory)

* [`Coroutine`](http://docs.unity3d.com/ScriptReference/Coroutine.html) inherits from [`YieldInstruction`](http://docs.unity3d.com/ScriptReference/YieldInstruction.html)
* A function that can suspend its execution (yield) until the given `YieldInstruction` finishes.
  * No return values or error handling (but can be overcome, if necessary)
* Can be used as a way to spread an effect over a period time. It is also a **useful optimization**:
  * Replaces state machines elegantly
  * Prevents execution blocking

> When a task does not need to be needlessly repeated quite so frequently, you can put it in a coroutine to get an update regularly but not in every single frame (where it would block execution). Similarly, calling an expensive function every frame in `Update()` will introduce significant slowdown. To overcome this, use a coroutine to call it, say, only every tenth of a second instead of every frame update.

For example:
```csharp 
void Start() {
    StartCoroutine(SomeCoroutine);
}

void Update() {
    //ExpensiveFunction();  // muh framerates :( 
}

IEnumerator SomeCoroutine() {
    while(true) {
        ExpensiveFunction();
        yield return new WaitForSeconds(.1f);
  }
}
```
* **A common pattern effectively handled by coroutines:**
  * Operations that take more than 1 frame...
  * Where we don't want to block execution...
  * And want to know when finished running.
  * **Examples:**
    * Cutscenes, Animation
    * AI Sequences/State Machines
    * Expensive Operations

* Coroutines also admit a nice, slick, readable game loop:
```csharp
void Start() {
    StartCoroutine (GameLoop());    // Let's play!
}

// This is called from Start() and will run each phase of the game one after another.
private IEnumerator GameLoop() {
    yield return StartCoroutine (LevelStart());    // Start the level: Initialize, do some fun GUI stuff, ..., yield WaitForSeconds if setup too fast.
    yield return StartCoroutine (LevelPlay());  // Let the user(s) play the level until a win or game over condition is met, then return back here.
    yield return StartCoroutine (LevelEnd());   // Find out if some user(s) "won" the level or not. Also, do some cleanup.
    
    if (WinCondition) {     // Check if game level progression conditions were met.
        Application.LoadLevel(++level);   // or Application.LoadLevel(Application.loadedLevel) if using same scene
    } else {    // Let the user retry the level by restarting this (non-yielding) coroutine again.
        StartCoroutine (GameLoop());
    }
}
```
### Usage
Use [StartCoroutine](http://docs.unity3d.com/ScriptReference/MonoBehaviour.StartCoroutine.html) methods to start a coroutine.
```csharp
public Coroutine StartCoroutine(IEnumerator method); // Typical usage. Pass the name of the method in code.
public Coroutine StartCoroutine(string methodName, object value = null); // Higher runtime overhead to start the coroutine this way; can pass only one parameter.
```
Use [StopCoroutine](http://docs.unity3d.com/ScriptReference/MonoBehaviour.StopCoroutine.html) methods to stop a coroutine.
```csharp
public void StopCoroutine(IEnumerator method); // Stops the coroutine stored in method running on this behaviour.
public void StopCoroutine(string methodName); // Stops the first coroutine named methodName.
public void StopAllCoroutines(); // Stops all coroutines running on this behaviour.
```
>*Note: If you call multiple coroutines with the same name, even a single StopCoroutine with that name will destroy them all!*

### Coroutine Return Types
Normal coroutine updates are run after the `Update()` function returns. Different uses of Coroutines:
```csharp
yield                       // The coroutine will continue after all Update functions have been called on the next frame.
yield WaitForSeconds        // Continue after a specified time delay, after all Update functions have been called for the frame
yield WaitForFixedUpdate    // Continue after all FixedUpdate has been called on all scripts
yield WaitForEndOfFrame     // Continue after all FixedUpdate has been called on all scripts
yield WWW                   // Continue after a WWW download has completed.
yield StartCoroutine        // Chains the coroutine, and will wait for the MyFunc coroutine to complete first.
```
In actual C# code:
```csharp
yield return null;                      
yield return new WaitForSeconds(t);     
yield new WWW(url);                     
yield return new WaitForFixedUpdate();  
yield StartCoroutine(routine)           
```

**Example usages:**

* **By method name as code:**
```csharp	
// Need a reference to a specific coroutine instance to stop it this way.
IEnumerator instance = null;
 
// Start coroutine
instance = SomeCoroutine(a, b, c);
StartCoroutine(instance); // or instance = StartCoroutine(SomeCoroutine (a, b, c)); (Coroutine continue failure?)
 
// Stop coroutine
StopCoroutine(instance);
``` 
* **By method name as string:**

```csharp
IEnumerator Start() {
    StartCoroutine("DoSomething", 2.0F);
    yield return new WaitForSeconds(1);
    StopCoroutine("DoSomething");
}

IEnumerator DoSomething(float someParameter) {
 	while (true) {
		print("DoSomething Loop");
		yield return null;
	}
}
```
-----------------------------------------------------------
## Events
###Unity Events
### C# Delegate Events
-----------------------------------------------------------
# Optimizations
### Section One
* to do
-----------------------------------------------------------
# Quick Links
## Unity Website
### Documentation
* [Unity Manual](http://docs.unity3d.com/Manual/index.html)
* [Scripting API](http://docs.unity3d.com/ScriptReference/index.html)
* [Unity3D Custom Google Search](http://unity3d.com/search)
* [Execution Order of Event Functions & Script Lifecycle Flowchart](http://docs.unity3d.com/Manual/ExecutionOrder.html)

### Community
* [Unity Forums](http://forum.unity3d.com/)
* [Unity Answers](http://answers.unity3d.com/)

### Scripting
* [Scripting API](http://docs.unity3d.com/ScriptReference/index.html)
* [Scripting Tutorials: Concise Videos & Code Snippets](http://unity3d.com/learn/tutorials/topics/scripting)
* [Good Coding Practices](https://unity3d.com/learn/tutorials/modules/intermediate/scripting/coding-practices)
* [Script Serialization](http://docs.unity3d.com/Manual/script-Serialization.html)

### GameObject|Component|Mono|Behaviour|Transform
* [GameObject](http://docs.unity3d.com/Manual/GameObjects.html)|[Class](http://docs.unity3d.com/ScriptReference/GameObject.html)
* [Component](http://docs.unity3d.com/Manual/UsingComponents.html)|[Class](http://docs.unity3d.com/ScriptReference/Component.html)
* [Transform](http://docs.unity3d.com/Manual/Transforms.html)|[Class](http://docs.unity3d.com/ScriptReference/Transform.html)
* [Mono](http://docs.unity3d.com/ScriptReference/MonoBehaviour.html)|[Behaviour](http://docs.unity3d.com/ScriptReference/Behaviour.html)

### Vectors|Math|Physics|Input
* [Vector Cookbook](http://docs.unity3d.com/Manual/UnderstandingVectorArithmetic.html)
* [Vector3](http://docs.unity3d.com/ScriptReference/Vector3.html)|[Vector2](http://docs.unity3d.com/ScriptReference/Vector2.html)
* [Mathf](http://docs.unity3d.com/ScriptReference/Mathf.html)
* [Physics](http://docs.unity3d.com/ScriptReference/Physics.html)|[Raycast](http://docs.unity3d.com/ScriptReference/Physics.Raycast.html)|[Hit](http://docs.unity3d.com/ScriptReference/RaycastHit.html)
* [Physics2D](http://docs.unity3d.com/ScriptReference/Physics2D.html)|[Raycast](http://docs.unity3d.com/ScriptReference/Physics2D.Raycast.html)|[Hit2D](http://docs.unity3d.com/ScriptReference/RaycastHit2D.html)
* [Rigidbody](http://docs.unity3d.com/ScriptReference/Rigidbody.html)|[2D](http://docs.unity3d.com/ScriptReference/Rigidbody.html)
* [Input](http://docs.unity3d.com/Manual/ConventionalGameInput.html)|[Class](http://docs.unity3d.com/ScriptReference/Input.html)

### Mouse|Collision|Trigger

### Debug|Gizmos

### Optimizations
* [Optimizing Graphics Performance](http://docs.unity3d.com/Manual/OptimizingGraphicsPerformance.html)
* [Internal Unity Tips](http://unity3d.com/learn/resources/internal-unity-tips-and-tricks)

### Miscellaneous
* [Unify Community Wiki](http://wiki.unity3d.com/index.php/Main_Page)

