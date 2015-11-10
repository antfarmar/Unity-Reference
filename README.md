# Unity Cheat Sheet
*Code snippets, Techniques Optimizations, Links*

# Code Snippets
## Co-Routines
```csharp
private IEnumerator myCoroutine() { 
  for (int i = 0; i < 10; i++) { 
    Debug.Log(i); 
    yield return 
  } 
  StopCoroutine("myCoroutine"); 
} 
StartCoroutine("myCoroutine"); 
```
## Coroutine Return Types 
```charp	
yield	
yield	WaitForSeconds	
yield	WWW	
yield	WaitForFixedUpdate	
yield	StartCoroutine
```

# Helpful Quick Links
## Official Unity Links
### Documentation
* [Unity Manual](http://docs.unity3d.com/Manual/index.html)
* [Scripting API](http://docs.unity3d.com/ScriptReference/index.html)
* 
### Community
* [Unity Forums](http://forum.unity3d.com/)
* [Unity Answers](http://answers.unity3d.com/)

### Scripting
* [Scripting API](http://docs.unity3d.com/ScriptReference/index.html)
* [Scripting Tutorials: Concise Videos & Code Snippets](http://unity3d.com/learn/tutorials/topics/scripting)

![image title](image.png)
