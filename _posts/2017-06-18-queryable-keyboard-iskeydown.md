---
layout: post
title: Creating a Queryable keyboard utility
excerpt: "Use DOM events to create a IsKeyDown map of currently and previously pressed keys"
categories: ["Game Development"]
tags: ["keyboard","typescript"]
comments: true
image:
  feature: https://i.imgur.com/SZb1kCU.png
---

Getting input from the browser requires us to listen for dom events. As our game runs in a loop we might have the need to use a typical "Is the A button currently down" method, rather than getting an event telling us it was pressed down. 

### Listen for key events

The simplest way would be to bind to any keyboard events, and once they are pushed put them in a map for later consumption. This would require us to have a method to set up binding of events when you want you keyboard to start listening.

**ts/game.ts**
{% highlight typescript %}
    export class Keyboard {
        private static keyMap: KeyMap = [];
        public static init(): void {
            window.addEventListener("keydown", (event) => {
                if (!Keyboard.keyMap[event.keyCode] === true) {
                    Keyboard.keyMap[event.keyCode] = true;
                    console.log("Key Down:" + String.fromCharCode(event.keyCode), event.keyCode);
                }
            }, false);
        }
    }
{% endhighlight %}

>You might notice that the type of the keymap is `KeyMap`. This is just an alias for  `{ [id: number]: boolean }`. Add `type KeyMap = { [id: number]: boolean };` to create the alias in your file.

Now we have an overview of any keys that has been pressed, but the is one issue. It's a one-time keyboard, once a key is pressed it stays pressed forever. Fixing this requires us to add a keyup event listener as well.
Our 

**ts/game.ts**
{% highlight typescript %}
    window.addEventListener("keyup", (event) => {
        Keyboard.keyMap[event.keyCode] = false;
        console.log("Key up:" + String.fromCharCode(event.keyCode), event.keyCode);
    }, false);
{% endhighlight %}

We now have a simple utility holding a map of which keys are currently pressed.

Creating simple is down and up allows us to test against this map

**ts/game.ts**
{% highlight typescript %}
    public static IsKeyDown(keycode: number): boolean {
        return !!Keyboard.keyMap[keycode];
    }
{% endhighlight %}

You might notice that we pass a number as the keycode, rather than a character. To be able to use the name of the key rather than the code we can add fields to our keyboard.

**ts/game.ts**
{% highlight typescript %}
    export class Keyboard {


        public static W = 87;
        public static A = 65;
        public static S = 83;
        public static D = 68;
        .....
{% endhighlight %}

>These codes assume a QWERTY layout

Having these fields allows us to use the keyboard utility like 

```
if(Keyboard.IsKeyDown(Keyboard.A)) {
    //Do something
}
```

### Pressing and releasing key within a frame

```
+> GameLoop
|        +
|        v
|    Start Loop
|         +
|         v
|      Game logic code
|         +
|         v
|      Key Down
|         +
|         v
|      Key Up
|         +
|         v
|      Game Logic checking for key down
|         +
|         v
+--+ Game Loop End
```

There is a corner-case in that the key could be pressed and released within one update of our game, this would yield a false value even though we could consider the key being pressed.

To fix this, instead of clearing it in the up event, we will add the key from our `keyup` event to "KeysToRemove" map and process this at the end of our event frame

Add a field to our `Keyboard` class, and alter the `keyup` event slightly

{% highlight typescript %}
    private static keysReleased: Array<number> = [];

    ...

    window.addEventListener("keyup", (event) => {
            Keyboard.keysReleased.push(event.keyCode);
            console.log("Key Up:" + String.fromCharCode(event.keyCode), event.keyCode);
    }, false);

{% endhighlight %}

Now we need to actively remove the released keys from our current keymap.
To keep the keyboard agnostic of any game framework, we will explicitly have a `onFrame` method.

{% highlight typescript %}
    public static onFrame(): void {
        for (let i = 0, l = Keyboard.keysReleased.length; i < l; i++) {
            console.log('Key removed:' + String.fromCharCode(Keyboard.keysReleased[i]));
            Keyboard.keyMap[Keyboard.keysReleased[i]] = false;
        }
        Keyboard.keysReleased = [];
    }
{% endhighlight %}

### Implementing IsPressed and IsReleased keys

Currently our keyboard class can tell us if a key is down or not,
but another useful feature would be if it could tell us if the key was pressed or released this in this update. This would enable us to trigger an action once, rather than continuously while the key is down and the game is running.

To fix this we will keep a snapshot of the keymap for the previous update, and diff these.

Add a `previousKeyMap` field to our keyboard class.
{% highlight typescript %}
    private static previousKeyMap: KeyMap = [];
{% endhighlight %}

In our `onFrame` method we will name take a snapshot of our previous keymap.

{% highlight typescript %}
    public static onFrame(): void {
        ...
        this.previousKeyMap = JSON.parse(JSON.stringify(this.keyMap));
        ....
    }
{% endhighlight %}

Now we can have methods like

{% highlight typescript %}
    public static IsKeyPressed(keycode: number): boolean {
        return !!Keyboard.keyMap[keycode] && !Keyboard.previousKeyMap[keycode];
    }
{% endhighlight %}  

### Final file

If you have any problems or want to just get to the code, here is the entire file.

{% highlight typescript %}
type KeyMap = { [id: number]: boolean };
/**
 * Keeps and exposes the state of the keyboard from keydown events
 * 
 * @export
 * @class Keyboard
 */
export class Keyboard {


    public static W = 87;
    public static A = 65;
    public static S = 83;
    public static D = 68;
    public static X = 88;
    public static Q = 81;
    public static E = 69;
    public static R = 82;
    public static F = 70;
    public static Space = 32;
    public static Left = 37;
    public static Up = 38;
    public static Right = 39;
    public static Down = 40;
    public static Shift = 16;
    public static Ctrl = 17;

    /**
     * Our map of keys pressed down for the current frame
     * 
     * @private
     * @static
     * @type {KeyMap}
     * @memberOf Keyboard
     */
    private static keyMap: KeyMap = [];
    /**
     * The key map for the previous frame
     * 
     * @private
     * @static
     * @type {KeyMap}
     * @memberOf Keyboard
     */
    private static previousKeyMap: KeyMap = [];
    /**
     * The keys that was currently released this update
     * 
     * @private
     * @static
     * @type {Array<number>}
     * @memberof Keyboard
     */
    private static keysReleased: Array<number> = [];
    /**
     * Initializes the keyboard events needed to track state
     * Needs to be called before use
     * 
     * @static
     * 
     * @memberOf Keyboard
     */
    public static init(): void {

        window.addEventListener("keydown", (event) => {
            if (!Keyboard.keyMap[event.keyCode] === true) {
                Keyboard.keyMap[event.keyCode] = true;
                console.log("Key Down:" + String.fromCharCode(event.keyCode), event.keyCode);
            }
        }, false);
        window.addEventListener("keyup", (event) => {
            Keyboard.keysReleased.push(event.keyCode);
            console.log("Key Up:" + String.fromCharCode(event.keyCode), event.keyCode);
        }, false);
    }
    /**
     * Notify the keyboard of a new update and clear any pressed keys
     * 
     * @static
     * 
     * @memberof Keyboard
     */
    public static onFrame(): void {
        this.previousKeyMap = JSON.parse(JSON.stringify(this.keyMap));;
        for (let i = 0, l = Keyboard.keysReleased.length; i < l; i++) {
            console.log('Key removed:' + String.fromCharCode(Keyboard.keysReleased[i]));
            Keyboard.keyMap[Keyboard.keysReleased[i]] = false;
            console.log('Current keymap:', Keyboard.keyMap);
        }
        Keyboard.keysReleased = [];
    }
    /**
     * Query if the given keycode is currently pressed
     * 
     * @static
     * @param {number} keycode
     * @returns {boolean} 
     * 
     * @memberOf Keyboard
     */
    public static IsKeyDown(keycode: number): boolean {
        return !!Keyboard.keyMap[keycode];
    }
    /**
     * Query if the given keycode was pressed down this frame
     * 
     * @static
     * @param {number} keycode 
     * @returns {boolean} 
     * 
     * @memberOf Keyboard
     */
    public static IsKeyPressed(keycode: number): boolean {
        return !!Keyboard.keyMap[keycode] && !Keyboard.previousKeyMap[keycode];
    }
    /**
     * Query if the given keycode was released this frame
     * 
     * @static
     * @param {number} keycode 
     * @returns {boolean} 
     * 
     * @memberOf Keyboard
     */
    public static IsKeyReleased(keycode: number): boolean {
        return !Keyboard.keyMap[keycode] && !!Keyboard.previousKeyMap[keycode];
    }
}
{% endhighlight %}  
