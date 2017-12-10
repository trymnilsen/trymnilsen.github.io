---
layout: post
title: Capturing "this" context of a class function in typescript
excerpt: "Use class functions as callbacks with the correct context in typescript"
categories: ["Typescript"]
tags: ["Web","Typescript", "This context", "callbacks"]
comments: true
---

Typescript is fantastic, it increases my productivity massively.
But there is one thing that has always felt a bit off, using class functions as callbacks.

{% highlight typescript %}
class foo {
    public constructor() {
        functionExpectingACallback(this.onSomethingHappened);
    }
    private onSomethingHappened() {

    }
}
{% endhighlight %}

This code snippet will work and onSomethingHappened will be executed, but the `this` context might not be the class context.
There are of course some ways around it like.

{% highlight typescript %}
...
    public constructor() {
        functionExpectingACallback(this.onSomethingHappened.bind(this));
    }
...
{% endhighlight %}

Or 

{% highlight typescript %}
...
    public constructor() {
        functionExpectingACallback(()=> { this.onSomethingHappened(); });
    }
...
{% endhighlight %}

But both of those feels rather verbose or unnecessary. This is where our neat little solution comes in. What if we use a typescript arrow function which will automatically capture the this context for us, but instead of wrapping our class function, it *is* our class function.

{% highlight typescript %}
class foo {
    public constructor() {
        functionExpectingACallback(this.onSomethingHappened);
    }
    private onSomethingHappened = ()=> {

    }
}
{% endhighlight %}

Here we have defined our callback outside of the resulting prototype as just a member in our class. 
What I like about it:

    - It's is visually different from the other class functions making it easy to spot that this is a callback function.
    - It captures the this context of the class without having to wrap it or using `bind`