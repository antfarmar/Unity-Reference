Source video: http://unity3d.com/learn/resources/internal-unity-tips-and-tricks

# Don't use FindObject!
* **FindObject() method is very sloooow.**
    * Parses every object in a Scene.
    * The more objects in a Scene, the slower FindObject() will be.
    
* **Use Object Registration/Registry Manager (Singleton)**
    * When an object is enabled, add it to a manager. Remove on disable.
    * Be sure Manager is initialized before other objects.
        * Configure Script Execution Order (make singleton run first)
    * **Nice implementation:**
        * Write a base class that tracks all the instances of a type that are active.
        * Base extends MonoBehavior, that you then extend your classes off.
    
# Smart Data Structure Layout
* **Code reuse**
    * Abstract classes used in many places
    * Simplifies drawing (PropertyDrawers)
    * Code easier to read/understand/maintain
* (Serialization examples: new GUI)
    
# Memory Debugging: Profiler & GC
* Use profiler frequently during project, don't wait until the end.
* Better to use profiler remotely on debug builds.
    * Some editor overhead when used in editor.
* Always check allocations per frame.
    * Try to keep at 0
    * GC is expensive!

* **Example**
    * Allocations every frame should be cached instead.
    * Class will use more memory, but...
        * GC will be called less.
        * Memory vs. Performance

* **Runtime Memory State**
    * Use the memory snapshot:
        * Found in the Profiler's Memory Tab: detailed view option.
    * Always name your objects meaningful names for debugging.

# Graphics Debugging: Profiler & Rendering
* Apps will run slow if too much is being rendered.
    * Draw calls, Overdraw (fillrate)
* **CPU side (how/what needs to render) vs. GPU side (rendering)**
    * **CPU:** Camera.Render is where most time is spent.
        * Fixes: Batching, Render lists, reduce what to render
    * **GPU:** Use external 3rd party tools to debug.
        * Pix for Windows, Intel GPA, VS2012 GFX Debugger.

# UnityEvents
* **Allow for:**
    * **Persistent callbacks** (configured in editor)
        * Only Component listeners can be hooked up.
    * **Runtime callbacks** (won't be serialized) `public void AddListener(UnityAction call)`
        * Anything! can be hooked.
    * **Weak references** (won't hold onto objects after they should be GC'd)

* **Versus C# Delegate Events**
    * Need to write scripts to hook them up.
    * They don't save/serialize.
    * Has problems with referencing:
        * If an object is pointed to by a delegate,
            * and something somewhere else holds a reference to that delegate,
                * the object will never be GC'd.
        * If the object is deleted, you don't want that delegate to be called.

* **Weak Referencing on Persistent Events**
    * If a game object is destroyed, Unity Events won't call functions on it.
    * The reference won't be considered by the GC to be a real reference.
    * Hence, UnityEvents are safer than delegates, and objects will be GC'd when destroyed.

# How Do Unity Events Work?
* Generic based
* UnityAction is actually a base delegate type.
* Component listeners will only be executed if the component is active (no need for checking state).
* **To create a UnityEvent**
    * Simply extend UnityEvent with desired type arguments:
```csharp
[Serializable]
public class ButtonClickedEvent : UnityEvent<Button> { }
```
* **To call a UnityEvent**
    * Call Invoke() with the required arguments:
```csharp
void OnClickEvent() { onClick.Invoke(this); }
```
