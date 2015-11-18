# Unity Cheat Sheet
***Code snippets, Techniques, Optimizations, Links***
-----------------------------------------------------------
![Entity-Component Pattern](Entity-Component.png)
-----------------------------------------------------------
# Index
1. [Code Snippets](#code-snippets)
2. [Techniques](#techniques)
    * [Singleton Pattern](#singleton-pattern)
    * [Coroutines](#coroutines)
    * [Events](#events)
3. [Optimizations](#optimizations)
    * TODO
4. [Quick Links](#quick-links)
    * [Unity3D Website](#unity-website)
    * [Documentation](#documentation)
    * [Community](#community)
    * [Scripting](#scripting)
    * [GameObject|Component|Mono|Behaviour|Transform](#gameobjectcomponentmonobehaviourtransform)
    * [Math|Random|Vectors|Physics|Input](#mathrandomvectorsphysicsinput)
    * [Debug|Gizmos](#debuggizmos)
    * [Optimizing](#optimizing)
    * [Miscellaneous](#miscellaneous)

-----------------------------------------------------------
# Code Snippets

-----------------------------------------------------------
# Techniques

## Singleton Pattern
***Game Managers, Static Classes, Global Variables, & You***

> Singletons are basically an object enforced to have a single instance only and always. You can access it anywhere, any time, without needing to instantiate it. That's why it's so closely related to `static`. For comparison, `static` is basically the same thing, except it's _not an instance_. We don't need to instantiate it, and we can't, because it's automagically allocated. And that can and does bring problems.

**The Singleton design pattern is a very specific type of single instance, specifically one that is:**

* Accessible via a global, static instance field
* Created either on program initialization or upon first access (lazy instantiation)
* No public constructor (cannot instantiate directly)
* Never explicitly freed (implicitly freed on program termination)

**This pattern introduces several potential long-term problems:**

* Inability to use abstract or interface classes
* Inability to subclass
* High coupling across the application (difficult to modify)
* Difficult to test (can't fake/mock in unit tests)

### Game Managers as Singletons
_One object to control them all_

**Game Manager Basic Requirements:**
* Be easily extendible
* Have only one instance allowed
* Persist across all the scenes (shouldn’t be destroyed when a new scene is loaded)
* Provide global variable access to other classes

**So why be Singleton? A non-instantiable static class could do the job, no?!**
* You can’t extend MonoBehaviour with a static class, hence it can't be a Script Component.
* You can’t implement an interface with a static class.
* You can’t pass around a static class as a parameter.

#### GameManager Singleton Examples

**Minimal Quick Hack:**
```csharp
public class GameManager : MonoBehaviour {
    public static GameManager instance = null;  // Static instance of GameManager which allows it to be accessed by any other script.

    void Awake() {                          // Awake is always called before any Start functions.
        if (instance == null)               // Check if instance already exists.
            instance = this;                // If not, set instance to this.
        else if (instance != this)          // If instance already exists and it's not this:
            Destroy(this.gameObject);       // Then destroy this. This enforces our singleton pattern, meaning there can only ever be one instance of a GameManager.
        DontDestroyOnLoad(this.gameObject); // Sets this to not be destroyed when loading new scenes.
    }
}
```

* [Singleton on Unify wiki](http://wiki.unity3d.com/index.php/Singleton)
* [Toolbox example](http://wiki.unity3d.com/index.php/Toolbox)
* [GameManager example](http://rusticode.com/2013/12/11/creating-game-manager-using-state-machine-and-singleton-pattern-in-unity3d/)

##### Further Reading

* [Good read] (http://programmers.stackexchange.com/questions/40373/so-singletons-are-bad-then-what/)

-----------------------------------------------------------

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

> When a task does not need to be needlessly repeated quite so frequently, you can put it in a coroutine to get an update regularly but not in every single frame. Similarly, calling an expensive function every frame in `Update()` will introduce significant slowdown, since it would block execution. To overcome this use a coroutine to call it, say, only every tenth of a second instead of _every_ frame update.

For example:
```csharp
void Start() {
    StartCoroutine(SomeCoroutine);
}

void Update() {
    // ExpensiveFunction();     // muh framerates :(
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

* **Coroutines also admit a slick, readable game loop:**
```csharp
void Start() {
    StartCoroutine (GameLoop());    // Let's play!
}


// This is called from Start() and will run each phase of the game one after another.
IEnumerator GameLoop() {
    yield return StartCoroutine (LevelStart()); // Start the level: Initialize, do some fun GUI stuff, ..., yield WaitForSeconds if setup too fast.
    yield return StartCoroutine (LevelPlay());  // Let the user(s) play the level until a win or game over condition is met, then return back here.
    yield return StartCoroutine (LevelEnd());   // Find out if some user(s) "won" the level or not. Also, do some cleanup.

    if (WinCondition) {                 // Check if game level progression conditions were met.
        Application.LoadLevel(++level); // or Application.LoadLevel(Application.loadedLevel) if using same scene
    } else {
        StartCoroutine (GameLoop());    // Let the user retry the level by restarting this (non-yielding) coroutine again.
    }
}

// The Coroutines
IEnumerator LevelStart() { Debug.Log("Start"); yield return new WaitForSeconds(1f); }
IEnumerator LevelPlay () { while(alive) yield return null; }
IEnumerator LevelEnd  () { Debug.Log("End.."); yield return new WaitForSeconds(1f); }
```

### Usage

Use [StartCoroutine](http://docs.unity3d.com/ScriptReference/MonoBehaviour.StartCoroutine.html) methods to start a coroutine.
```csharp
public Coroutine StartCoroutine(IEnumerator method);                        // Typical usage. Pass the name of the method in code.
public Coroutine StartCoroutine(string methodName, object value = null);    // Higher runtime overhead to start the coroutine this way; can pass only one parameter.
```

Use [StopCoroutine](http://docs.unity3d.com/ScriptReference/MonoBehaviour.StopCoroutine.html) methods to stop a coroutine.
```csharp
public void StopCoroutine(IEnumerator method);  // Stops the coroutine stored in method running on this behaviour.
public void StopCoroutine(string methodName);   // Stops the first coroutine named methodName.
public void StopAllCoroutines();                // Stops all coroutines running on this behaviour.
```

>*Note: If you call multiple coroutines with the same name, even a single StopCoroutine with that name will destroy them all!*

### Coroutine Return Types
Normal coroutine updates are run after the `Update()` function returns.
Different uses of Coroutines by **return type:**

```csharp
yield                       // The coroutine will continue after all Update functions have been called on the next frame.
yield WaitForSeconds        // Continue after a specified time delay, after all Update functions have been called for the frame
yield WaitForFixedUpdate    // Continue after all FixedUpdate has been called on all scripts
yield WaitForEndOfFrame     // Continue after all FixedUpdate has been called on all scripts
yield WWW                   // Continue after a WWW download has completed.
yield StartCoroutine        // Chains the coroutine, and will wait for the MyFunc coroutine to complete first.
```

**In actual C# code:**
```csharp
yield return null;
yield return new WaitForSeconds(t);
yield new WWW(url);
yield return new WaitForFixedUpdate();
yield StartCoroutine(routine)
```

### Example usages of `StartCoroutine()` & `StopCoroutine()`:

* **Passing method name as code:**
```csharp
IEnumerator instance = null;        // Need a reference to a specific coroutine instance to stop it.
instance = SomeCoroutine(a, b, c);
StartCoroutine(instance);           // Start coroutine
                                    // or instance = StartCoroutine(SomeCoroutine (a, b, c)); (Coroutine continue failure?)
StopCoroutine(instance);            // Stop this specific coroutine instance.
```

* **Passing method name as string:**
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
# Events
[UnityEvents vs. C# Delegates](https://github.com/antfarmar/Unity/blob/master/Internal%20Unity%20Tips.md#unityevents)

### Unity Events
### C# Delegate Events
-----------------------------------------------------------
# Optimizations
### Section One

* TODO



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

### Math|Random|Vectors|Physics|Input
* [Mathf](http://docs.unity3d.com/ScriptReference/Mathf.html)
* [Random](http://docs.unity3d.com/Manual/RandomNumbers.html)|[Class](http://docs.unity3d.com/ScriptReference/Random.html)
* [Vector Cookbook](http://docs.unity3d.com/Manual/UnderstandingVectorArithmetic.html)
* [Vector3](http://docs.unity3d.com/ScriptReference/Vector3.html)|[Vector2](http://docs.unity3d.com/ScriptReference/Vector2.html)
* [Physics](http://docs.unity3d.com/ScriptReference/Physics.html)|[Raycast](http://docs.unity3d.com/ScriptReference/Physics.Raycast.html)|[Hit](http://docs.unity3d.com/ScriptReference/RaycastHit.html)
* [Physics2D](http://docs.unity3d.com/ScriptReference/Physics2D.html)|[Raycast](http://docs.unity3d.com/ScriptReference/Physics2D.Raycast.html)|[Hit2D](http://docs.unity3d.com/ScriptReference/RaycastHit2D.html)
* [Rigidbody](http://docs.unity3d.com/ScriptReference/Rigidbody.html)|[2D](http://docs.unity3d.com/ScriptReference/Rigidbody.html)
* [Collision](http://docs.unity3d.com/ScriptReference/Collision.html)
* [Input](http://docs.unity3d.com/Manual/ConventionalGameInput.html)|[Class](http://docs.unity3d.com/ScriptReference/Input.html)

### Debug|Gizmos
* [Debug](http://docs.unity3d.com/ScriptReference/Debug.html)
* [Gizmos](http://docs.unity3d.com/ScriptReference/Debug.html)

### Optimizing
* [Optimizing Graphics Performance](http://docs.unity3d.com/Manual/OptimizingGraphicsPerformance.html)
* [2D Sprite Rendering Optimizations](http://unity3d.com/learn/resources/boosting-graphics-performance-2d-games)
* [Internal Unity Tips](http://unity3d.com/learn/resources/internal-unity-tips-and-tricks)
* [Architecting Games in Unity](https://www.youtube.com/watch?v=64uOVmQ5R1k)
* [Profiling (Mobile)](http://docs.unity3d.com/Manual/MobileProfiling.html)

### Miscellaneous
* [Unify Community Wiki](http://wiki.unity3d.com/index.php/Main_Page)

