# Unity Reference Cheat Sheet

#### _Code snippets, Techniques, Optimizations, Links_

**Unity:** An Entity-Component Game Engine

**Entity-Component Pattern:** Allows a single entity to span multiple domains without coupling the domains to each other.

**`GameObject`:** The single entity that spans multiple domains _(Transform, Physics, Rendering, ...)_.

**`Component`:** Keeps the domains isolated, where the code for each is placed in its own `Component` class.

In Unity,  `GameObject` is simply a container of `Components`.

-----------------------------------------------------------
![Entity-Component Pattern](Entity-Component.png)
-----------------------------------------------------------

# Index
1. [Architecture, Standards, Best Practices](#architecture-standards-best-practices)
1. [Techniques](#techniques)
    * [Singleton Pattern](#singleton-pattern)
    * [Coroutines](#coroutines)
    * [Events](#events)
1. [Optimizations](#optimizations)
    * TODO
1. [Quick Links](#quick-links)
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
# Architecture, Standards, Best Practices

##### Software Design 101

* Use C#
  * Strongly-typed, lexical analysis = debugging easy
  * Lots of documentation, libraries, community supported
* Stay Organized
  * Naming conventions
    * Descriptive names, standard capitilization
    * Unity handles spaces in names
  * Logical folder structure
    * Easy to locate assets
* Zero-tolerance for:
  * Warnings & Errors
  * Runtime Memory Allocaton
* Use the Profiler often
  * Keep code constantly optimized

-----------------------------------------------------------
# Techniques
-----------------------------------------------------------
## Implementing Gameplay

### Inter-Object Communication
* Public Static Classes\Methods
* Public Instance Methods
* Events/Messages

### Managers\Controllers

Most common patterns:
* Singletons
* Object Pools

### Singleton Pattern
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

##### Further Reading
* [Singleton: Game Programming Patterns Book](http://gameprogrammingpatterns.com/singleton.html)
* [Singleton example on Unify wiki](http://wiki.unity3d.com/index.php/Singleton)
* [Toolbox example](http://wiki.unity3d.com/index.php/Toolbox)
* [GameManager example](http://rusticode.com/2013/12/11/creating-game-manager-using-state-machine-and-singleton-pattern-in-unity3d/)
* [Good read about Singleton issues](http://programmers.stackexchange.com/questions/40373/so-singletons-are-bad-then-what/)

-----------------------------------------------------------
# Coroutines
-----------------------------------------------------------

**Execution Time Sharing** _(not multi-threading, concurrency, or parallelism)_

[[Video Tutorial]](https://unity3d.com/learn/tutorials/modules/intermediate/scripting/coroutines)
[[Unite Talk]](http://unity3d.com/learn/resources/extending-coroutines-power-and-glory)

* [`Coroutine`](http://docs.unity3d.com/ScriptReference/Coroutine.html) inherits from [`YieldInstruction`](http://docs.unity3d.com/ScriptReference/YieldInstruction.html)
* A function that can suspend its execution (yield) until the given `YieldInstruction` finishes.
    * Maintains local parameter references when called.
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

* [UnityEvents vs. C# Delegates](https://github.com/antfarmar/Unity/blob/master/Internal%20Unity%20Tips.md#unityevents)
* [Reddit Post](https://www.reddit.com/r/Unity3D/comments/2yh54n/unityengineevents_eventsystem_c_delegate/)
* [Reddit Post](https://www.reddit.com/r/Unity3D/comments/35oekm/delegate_events_vs_unityevent_which_one_is/)

-----------------------------------------------------------

**Events** are closely related/similar to the **Observer** software design pattern.

[Observer: gameprogrammingpatterns chapter](http://gameprogrammingpatterns.com/observer.html)

### Observer Pattern
* Simple, workable solution for **Decoupling** classes:
  * It helps us loosen the coupling between two pieces of code.
  * It lets a subject indirectly communicate with some observer without being statically bound to it.
  * It lets one piece of code announce that something interesting happened without actually caring who receives the notification.

**Problems:**
* Object heavy
* Have to implement an entire interface just to receive a notification.
* Can’t have a single class that uses different notification methods for different subjects.

Modern approach is for an “Observer” to be only a reference to a method or function, like C# delegates:

### C# Delegate Events
* [Unity Video Tutorial](http://unity3d.com/learn/tutorials/modules/intermediate/scripting/events)

#### Delegates

[Delegates (C# Programming Guide)](https://msdn.microsoft.com/en-us/library/ms173171.aspx)

* A delegate is a type that represents references to methods with a particular parameter list and return type. When you instantiate a delegate, you can associate its instance with any method with a compatible signature and return type. You can invoke (or call) the method through the delegate instance.
* Delegates are used to pass methods as arguments to other methods. 
* The following example shows a delegate declaration:
  * `public delegate int PerformCalculation(int x, int y);`

#### Delegate Events: C# Publisher-Subscriber Model

[Events (C# Programming Guide)](https://msdn.microsoft.com/en-us/library/awbftdfh.aspx)

* Events enable a class or object to notify other classes or objects when something of interest occurs.
* The class that sends (or raises) the event is called the publisher and the classes that receive (or handle) the event are called subscribers.
* The publisher determines when an event is raised; the subscribers determine what action is taken in response to the event.
* Event handlers are nothing more than methods that are invoked through delegates. You create a custom method, and a class can call your method when a certain event occurs.

* C# has "events" baked into the language.
  * The `event` keyword is used to declare an event in a publisher class.
    * `public event SampleEventHandler SampleEvent;`
* the "observer" you register is a "delegate" (delegates are a reference to a method).
    * `public delegate void SampleEventHandler(object sender, SampleEventArgs e);`
  
**Problems:**
* Setting a delegate ALLOCATES memory.
  * Using a single delegate and constantly setting it will cause a GC call to reclaim memory.
  * Better to pre-set an array of delegates in the `Awake` function to overcome this.


### Unity Events
* [Manual Entry](http://docs.unity3d.com/Manual/EventSystem.html)
* [Scripting API](http://docs.unity3d.com/ScriptReference/Events.UnityEvent.html)
* [UnityEvent:](https://github.com/antfarmar/Unity/blob/master/Internal%20Unity%20Tips.md#unityevents)
  * used in Unity UI system.
  * Serialized in editor.

* Examples:
  * example

-----------------------------------------------------------
# Optimizations
-----------------------------------------------------------

### Section One

* TODO



-----------------------------------------------------------
# Quick Links
-----------------------------------------------------------

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
   * [Implementations in Unity](https://github.com/antfarmar/Unity-Scripting-Tutorials#unity-scripting-tutorials)
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
* [Rigidbody](http://docs.unity3d.com/ScriptReference/Rigidbody.html)|[2D](http://docs.unity3d.com/ScriptReference/Rigidbody2D.html)
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

