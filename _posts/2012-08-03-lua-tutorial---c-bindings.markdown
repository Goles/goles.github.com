---
title: Lua Tutorial - C Bindings
layout: post
tags:
---

### Intro
In this blog post I want to make a rather short but straight to the point Lua Tutorial. The objective is to learn how to manipulate the Lua stack with the Lua C API in order to create Lua bindings.

### Motivation
I wrote this tutorial to prove that it's rather easy to manipulate the Lua stack and create Lua C bindings yourself without any sort of third-party frameworks. 

It's rather surprising how a lot of people seem to be looking for the **holy grial** of the Lua binding Frameworks.

They come in all flavors and styles, and all of them have their strengths and weaknesses, but it's not a bad idea to think a bit before just picking one of this frameworks "blindly" or because of fear.

### Myths
1.Using the Lua C API is hard

This one is a classic, I've heard about it a thousand times. Using the Lua C API is not hard at all, specially if you're doing rather simple things like creating single function bindings which is what most of people want to do in the first place. While manipulating the Lua stack can get a bit more complex when doing very specific stuff, normally it's not.

2.Using the Lua C API is a waste of time

Using the API is often a great excersise for several reasons, one of the most important ones is that it will reveal the real complexity of the Lua integration that you're trying to achieve for your application thus allowing you to take a more informed decision if things get just too hard or time consuming and you need to pickup a Lua Binding Framework. If you really tried to make the integration yourself, you will get a good grasp of what features will be more important for your project.

On the other side, maybe you'll end-up creating a rather simple set of bindings which work wonders for your project and that you can extend on-demand instead of adding a huge framework just for binding a couple of function calls.

### Include Lua in your C project

1. Get Lua
<pre><code class="bash">wget http://www.lua.org/ftp/lua-5.2.1.tar.gz
tar -zxvf lua-5.2.1.tar.gz
</code></pre>

2. To link against Lua you can compile using the following args

<pre><code class="bash">gcc main.c -o output -Ilua-5.2.1/include/ -Llua-5.2.1/lib/ -llua</code></pre>

### Let's run a Lua script from your C project

sample\_script.lua
    <pre><code class="lua">print("Hello Lua World!, this is a Lua C Tutorial :D")</code></pre>

main.c

    #include "lua.h"
    #include "lualib.h"
    #include "lauxlib.h"

    int main() {

        // first we create an instance of a Lua state and Load the lua libraries
        lua_State *L = lua_open();
        luaL_openlibs(L);

        // Tell Lua to load and run the file sample_script.lua
        lua_dofile(L, "sample_script.lua");

        // Close the Lua state
        lua_close(L);

        return 0;
    }


sample_script.lua

    function sum (param1, param2)
        result = param1 + param2
        return result
    end

As you can notice, it's very simple to just run a Lua script from your C code. This is already quite powerful, but the next logical step would be calling Lua functions and passing arguments to them from C to Lua.

### Calling Lua Functions from C

sample_script.lua

<pre><code class="lua">

    function sum (param1, param2)
        result = param1 + param2
        return result
    end

</code></pre>

main.c

    #include <stdio.h>
    #include "lua.h"
    #include "lualib.h"
    #include "lauxlib.h"

    int main() {

        // first we create an instance of a Lua state and Load the lua libraries
        lua_State *L = lua_open();
        luaL_openlibs(L);

        // Tell Lua to load and run the file sample_script.lua
        lua_dofile(L, "sample_script.lua");

        // Push the function declared in sample_script.lua to the top of the "Lua Stack"
        lua_getglobal(L, "sum");

        // Little check to see if the Top of the Lua stack is actually a function.
        assert(lua_isfunction(L, -1));

        // Let's push two numbers to the sum function
        int a = 19;
        int b = 1;
        lua_pushnumber(L, a); // push a to the top of the stack
        lua_pushnumber(L, b); // push b to the top of the stack

        // Perform the function call (2 arguments, 1 result) */
        if (lua_pcall(L, 2, 1, 0) != 0) {
            error(L, "error running function `f': %s", lua_tostring(L, -1));
        }

        // Let's retrieve the function result
        if (!lua_isnumber(L, -1)) { // Check if the top of the Lua Stack has a numeric value
            error(L, "function `sum' must return a number");
        }

        double c = lua_tonumber(L, -1); // retrieve the result
        lua_pop(L, 1); // Pop the retrieved value from the top of the Lua stack

        printf("The number returned by sum is %f\n", c);

        // Close the Lua state
        lua_close(L);

        return 0;
    }

### Calling C from Lua

test.lua

<pre><code class="lua">

    sin = mysin(5)
    print(sin)

</code></pre>

main.c

    #include <stdio.h>
    #include "lua.h"
    #include "lualib.h"
    #include "lauxlib.h"

    // Function that we will expose to lua
    static int l_sin (lua_State *L) {
        double d = luaL_checknumber(L, 1); // check if the argument is a number
        lua_pushnumber(L, sin(d)); // Push the result
        return 1;  // number of results
    }

    int main() {
        // first we create an instance of a Lua state and Load the lua libraries
        lua_State *L = lua_open();
        luaL_openlibs(L);

        // Expose the mysin function to the lua environment
        lua_pushcfunction(l, l_sin);
        lua_setglobal(l, "mysin");

        // Tell Lua to load and run the file sample_script.lua
        lua_dofile(L, "test.lua");

        return 0;
    }

### Outro

We did some really basic Lua bindings, I really hope that you learned a bit more and that you can now start experimenting on your own projects. I may write a part 2 of this series in the nearby future. If you would like me to cover any particular subject you can always write to me on [Twitter](http://www.twitter.com/ngoles).

### Further Lecture

1. [Lua tutorial directory](http://lua-users.org/wiki/TutorialDirectory)
2. [Lua Directory](http://lua-users.org/wiki/LuaDirectory)
3. [Online Interpreter](http://repl.it/)


