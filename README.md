# Background

I used to love Blueprint, and it still holds a special place in my heart.
It was the first true programming language that I learned, and it was the gateway drug for me to get into C# with Unity,
which is still my most favorite programming language.

However, as I have grown as a developer and have branched out deeper into game and even web development, I always come
back to Blueprint with mixed emotions.
While Blueprint is highly accessible, it also holds the engine itself back in several ways.

The lack of proper generics in Blueprint holds Unreal Engine itself back in several ways. For example, it is not
possible to have multidimensional `TArray`s, `TMap`s, or `TSet`s.
It isn't possible to create a `TArray<TArray<int32>>` in C++ like you can in basically any other language, including in
C++ itself where you can make a `std::vector<std::vector<int32>>`.
If you want to be able to work with generic types with proper engine support,
you need to either use a custom K2_Node (which is extremely tedious) or a Blueprint Macro (which has several limitations
of its own).

Another big language limitation is the lack of first-class functions. Again, to get an equivalent experience, you must
work with either Blueprint Macros or make a custom K2_Node.
Blueprint Macros are pretty reasonable to create, but needing to make a K2_Node takes substantially more effort.

Both of these limitations became extremely clear whenever working on an implementation for ReactiveX for Blueprint.
Getting it to work in C++ land was straightforward enough, I was able to just add ReactivePlusPlus, and it all worked
fine from there.
But trying to get it to also work with Blueprint was a much greater feat; I ended up making a separate library
specifically for Blueprint that went all-in on Blueprint Macros.

But the biggest problem that I face with using Blueprint nowadays is because I primarily use laptops, and I am
oftentimes away from a mouse.

I have an Asus ProArt P16 with a high-resolution touchscreen. You might think that using a touchscreen would be great
for Blueprints because of how graphical it is.
In reality, it's the exact opposite. Using a touchscreen with Blueprint is a nightmare and it is practically unusable.
Whenever you drag on an empty space, it doesn't drag the graph around; it creates a highlight box.
You can't pull any pins off of nodes. You can't even drag any nodes around. Clicking on anything on the screen rarely
registers as a proper click.
I imagine that using Blueprint is far better whenever using a drawing tablet because it typically maps to mouse
movements, and tapping the pen usually maps to a left-mouse click.
Tapping a touchscreen does NOT map to a left-mouse click; it maps to a touch, which Unreal Engine hardly supports.

Furthermore, the Asus ProArt P16 has a giant trackpad. Whenever I try panning with a trackpad, it just zooms in and out
because it registers it as scrolling vertically and horizontally.
The only way to pan with a trackpad is by holding down the right click with one of my fingers and then using another
finger to drag...

So while Blueprint presents itself as a way for designers to be able to do a bit of programming, it doesn't support
input methods that designers would be more likely to use.

Another big issue with using Blueprint is the system resource usage while just programming (not even testing anything).
Unreal Engine's UI, including Blueprint, is built using Unreal Engine itself wth a combination of Slate and UMG.
This is usually a good thing for a variety of reasons; dogfooding means that Epic Games is able to work through issues
that they themselves face during the development of the engine.
That also means that you have a lot of source-available examples to look through on how to use Slate and UMG to
implement some very complex UI.
However, Unreal Engine is a game engine so Slate and UMG are catered for people on high-end desktops, the same kinds of
environments that are used to play those games.
Slate and UMG are horribly inefficient because they re-render as fast as my computer and framerate limit will let it.
With just a Blueprint open and nothing else, my iGPU usage spikes up and stays very high, even if there is no motion,
until I tab out.
As a result, my battery drains extremely fast whenever using Blueprint in comparison to Visual Studio, VSCode, any
JetBrains IDEs, etc.

Note that I didn't even mention Blueprint's performance itself. While, relatively speaking, it is very slow, it is
typically fast enough for most things you'd work on.
Generally, the problem with Blueprint is that the UX is abysmal whenever using a laptop. Using C++ is far more pleasant
whenever working with a laptop.
However, the compilation times are very long with C++ and there are many times when the hot-reload just doesn't work.
For scripting languages, are there any options other than Blueprint?

Yes, there are.

# Goal

I am going to try out different programming languages in Unreal Engine and evaluate the pros and cons to using them.
The goal is to find the best alternative language to use for Unreal Engine while waiting out for Unreal Verse to be
added as an official option in Unreal Engine 6.
The options here are ordered by language and ranked by viability for run-time programming.

## C++

> One of the key moments in Unreal Engine 4's development was,
> we had a series of debates about UnrealScript – the scripting language I'd built that we'd carried through three
> generations.
> And what we needed to do to make it competitive in the future. And we kept going through bigger and bigger feature
> lists of what we needed to do to upgrade it,
> and who could possibly do the work, and it was getting really, really unwieldy.
> And there was this massive meeting to try and sort it out, and try to cut things and decide what to keep,
> and plan and...there was this point where I looked at that and said 'you know, everything you're proposing to add to
> UnrealScript is already in C++.
> Why don't we just kill UnrealScript and move to pure C++? You know, maximum performance and maximum debuggability. It
> gives us all these advantages.'
> - Tim Sweeney

- Fast runtime performance
- Very slow compile times
- Allows for integration with any C++ library given you can make a wrapper around it that plays nicely with Unreal
  Engine
- Template params do not work with Unreal Engine's reflection system
- Unreal Engine is very particular about its reflection macros and will throw oftentimes unintelligible errors whenever
  a reflection macro isn't applied correctly
- Hot-reload only really works consistently whenever updating function logic; attempting to update signatures of any
  kind often will require a restart to work as expected

## Blueprint

Whenever Epic Games transitioned from Unreal Engine 3 to Unreal Engine 4, they opted to drop their custom scripting
language, Unreal Script,
in favor of going all-in on their visual scripting language, Kismet (which was renamed to Blueprint) coupled with C++.
With that, if you ever see a "K" prefix in front of anything in C++, it is likely a prefix for "Kismet." For example,
K2_Nodes; "K2" means Kismet 2.x

- Beginner-friendly; great gateway drug into learning other languages
- Can be simpler to follow the flow for
- Very tight integration with the editor. Gives you type-safety for editor-time things not exposed to C++ like objects
  contained in a level
- Simple support for Generics/Templates through Macros or custom K2_Nodes, but nothing further than that
- No native `any` or `void*` type, but it's pretty straightforward to make a thin wrapper around `FVariant`,
  `FInstancedStruct`, and `TVariant` in C++ to do the same.
  See something like [AnyValue](https://www.fab.com/listings/242b2e28-db6b-4e69-9744-067b4e171a0f) for that as a plugin,
  or I'll likely put up a throwaway GitHub of my own version that I made for it.
  But, I chose against this approach for making my plugin because constantly needing to cast to-and-from was getting to
  make for a terrible developer experience
- Functions are NOT first-class citizens
- Blueprints are completely stored in binary, which makes diffing for merging or reviewing code extremely difficult
- Slow runtime performance
- Fast compile times
- Compilations work consistently; no need to restart for any signature changes

## AngelScript

### [AngelScript](https://angelscript.hazelight.se/)

- Requires using a fork of Unreal Engine, which makes using plugins potentially more difficult
- Doesn't support interfaces
- Has access to all of Unreal Engine's reflection suite (minus reflection)

## [SkookumScript](https://skookumscript.com/unreal/)

Was originally a very well-maintained alternative scripting language which ended up getting bought by Epic Games and
became Verse

## [Verse](https://en.wikipedia.org/wiki/Verse_(programming_language))

- Only available in Unreal Editor for Fortnite
- Won't be available in Unreal Engine until Unreal Engine 6

## Python

### [UnrealEnginePython](https://github.com/20tab/UnrealEnginePython)

This project hasn't been updated since 2020-04-17.
Unfortunately, the project was abandoned because Epic Games made their own implementation of Python for Unreal Engine.
The nifty part about this project was that it was able to be used at runtime

### Python (Official)

Unfortunately, this is editor only. Runtime builds completely strip all the Python out of it.
Maybe it'd be possible to create a fork of the Unreal Engine source code which doesn't strip this out?
Regardless, this is particularly useful for making editor tools and doing editor automation,
but it is limited to what is exposed to it through Unreal Engine's reflection system.

### [Nitro Python Runtime (Python)](https://www.fab.com/listings/fb18f1b0-3977-4cd7-b24e-ae7f8355e559)

Only for calling Python scripts. Can push and pull variable names + values from/to the executed script but cannot
directly access UE5 internals

### [Yothon (Python)](https://www.yobiminds.com/yothon.html) and https://www.fab.com/listings/8d83689b-941a-49d9-ae22-f982d2d7e498

> Please note that, at this stage, Python scripts cannot directly access UE5 internals, e.g. UObjects.

As a result, this doesn't seem too different from just executing a Python command and dynamically passing in or pulling
those values as a JSON

## .NET

### [UnrealSharp (.NET 9+)](https://www.unrealsharp.com/)

### [UnrealCLR (.NET 6)](https://github.com/nxrighthere/UnrealCLR)

This project hasn't been updated since 2023-06-07

### [MonoUE](https://mono-ue.github.io/#:~:text=Mono%20for%20Unreal%20Engine%20is%20a%20plugin%20for,generated%20automatically%20for%20all%20Blueprint-accessible%20types%20and%20members.)

This project hasn't been updated since 2019-06-11

## JavaScript/TypeScript

### [Unreal.js (TypeScript/JavaScript)](https://github.com/ncsoft/Unreal.js/)

### [Puerts (TypeScript/JavaScript)](https://github.com/Tencent/puerts)

## Lua

### [UnLua (Lua)](https://github.com/Tencent/UnLua)

### [slua-unreal](https://github.com/Tencent/sluaunreal)

This is also by Tencent. Not quite sure what the difference is between this and UnLua but PUBG Mobile uses this

### [LuaMachine](https://github.com/rdeioris/LuaMachine)

## Rust

### [unreal-rust](https://github.com/MaikKlein/unreal-rust?tab=readme-ov-file)

This project hasn't been updated since 2022-10-02

## Haxe

### [Unreal.hx](https://github.com/proletariatgames/unreal.hx)

This project hasn't been updated since 2023-01-10

### [Haxe-UnrealEngine](https://github.com/SomeRanDev/Haxe-UnrealEngine5)

This project hasn't been updated since 2023-04-18