---
layout: post
title: Using C# 6 and dotnet 4.6 in Unity3d
excerpt: "Lets have a look at the new features we can use with c# 6 in unity 4.6"
categories: ["Game Development"]
tags: ["Unity3d","dotnet", "game development"]
comments: true
image:
  feature: https://i.imgur.com/sYp4tIU.png
---

Unity3d 2017.1 includes experimental support for c# 6 and 4.6. This enables us to use a bunch of new features.

### Activating dotnet 4.6 support

To enable .net 4.6 support go to `Edit->Project Settings->Player->Other Settings->Configuration->Scripting Runtime Version.`

![Change scripting runtime version](https://i.imgur.com/sYp4tIU.png)

### Read-only auto properties

Properties are already available in Unity3d. But c# 6 gives us the ability to have read only properties. These are defined using a single getter and can only be set in a constructor. 

```csharp
public class ReadOnlyProperties : MonoBehaviour {

    public string Greeting { get; }

    public ReadOnlyProperties() {
        Greeting = "Hello";
    }
    
	// Use this for initialization
	private void Start () {
        //Greeting = "Hi there!"; //Uncomment this and you will get a compiling error
        Debug.Log("Your current Greeting is: " + Greeting);
	}
    ...

}
```
### Auto-Property Initializers

Properties can now be initialized "inline". No need to set them in the constructor.
If we expand on our Greeting example from above we can remove the constructor and change the `Greeting` property to

```csharp
public string Greeting { get; } = "Hello";
```

### Expression-bodied function members
The body of our methods might consist of only one statement that can be represented as an expression.
In our greeting example we can create a `Format Debug log message` that generates the string we log to the console.

```csharp
public class ReadOnlyProperties : MonoBehaviour {

    public string Greeting { get; } = "Hello";

	// Use this for initialization
	private void Start () {
        //Greeting = "Hi there!"; //Uncomment this and you will get a compiling error
        Debug.Log(GenerateDebugString(Greeting));
        Debug.Log(GenerateEBFDebugString(Greeting));
	}
	
    //Normal method
    private string GenerateDebugString(string greeting) {
        return "Your current Greeting is: " + greeting;
    }
    //Expression-bodied function member
    public string GenerateEBFDebugString(string greeting) => "Your current Greeting is: " + greeting;
}
```


### Null-conditional operators

The null conditional operation, or as its also know the YEEES-NO-MORE-HUGE-IF-CHECKS-WHEN-FINDING-COMPONENTS-ON-A-GAMEOBJECT operator is one of the coolest and most help feature in c# 6 perhaps.

Lets assume we have the following.
```csharp
string superData1 = GameObject.Find("ObjectWithNullValue").GetComponent<MySuperComponent>().SuperData;
```

The risk of getting a NullReferenceException here is quite high if either `ObjectWithNullValue` is not in the scene or it doesn't have the `MySuperComponent` component on itself.

This is where the Null-conditional operatior helps out a lot. We can rewrite our line as:

```csharp
string superData1 = GameObject.Find("ObjectWithNullValue")?.GetComponent<MySuperComponent>()?.SuperData;
```
The following will run smoothly without any null reference exceptions
```csharp
string superData1 = GameObject.Find("ObjectWithNullValue")?.GetComponent<MySuperComponent>()?.SuperData;
string superData2 = GameObject.Find("ObjectWithoutNull")?.GetComponent<MySuperComponent>()?.SuperData;
Debug.Log(superData1);
Debug.Log(superData2);
```

Output 

```
Null
Hi I'm Not null
```

### String Interpolation

Another neat feature available in c# 6 is string interpolation built into the syntax.
Previously you would use the string.format feature or perhaps concatinating together string.
No more! 

We can turn the following
```csharp
string fullGreeting1 = string.Format("{0} {1}", Greeting, Name);
string fullGreeting2 = Greeting + " " + Name;
```

Into 

```csharp
string fullGreeting = $"{FirstName} {LastName}";
```

### But wait! There's more!

There are a lot of other features as well.
For a further intro i would suggest having a look at the dotnet documentation.
[What's new in c# 6](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-6)