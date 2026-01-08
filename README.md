# frida-idiotify

a helper library for calling il2cpp methods in unity games using frida.
it provides a lightweight abstraction over `frida-il2cpp-bridge`, allowing you to call unity and game methods using simple string paths instead of manually resolving classes and methods.

this version is designed to work **without full domain-wide assembly enumeration**, making it safer for mfuscator / stripped il2cpp builds.

---

## requirements

* frida 16+
* frida gadget or frida server running on device
* `frida-il2cpp-bridge`
* unity il2cpp game (android / quest / ios)

---

## build (important)

you **must compile** the agent before loading it.

```bash
frida-compile agent.ts -o agent.compiled.js
```

do **not** load raw `.ts` files.

---

## frida gadget config (listen mode)

your frida gadget **must** be configured like this:

```json
{
  "interaction": {
    "type": "listen",
    "address": "127.0.0.1",
    "port": 27042,
    "on_load": "resume"
  }
}
```

this allows frida to attach **after the game is already running**, avoiding freezes and early il2cpp crashes.

---

## running

1. launch the game normally
2. attach frida and load the compiled agent:

```bash
frida -U -n [app name] -l agent.compiled.js
```

example:

```bash
frida -U -n ProjectBlaze -l agent.compiled.js
```

if injection worked, you should see this in logcat:

```
[idiotify] injected successfully, have fun doing stuff easily!
```

---

## basic usage

all il2cpp interaction must happen inside `Il2Cpp.perform`.

```js
Il2Cpp.perform(() => {
    Idiotify.init();

    Idiotify.call(
        "UnityEngine.CoreModule::UnityEngine.Debug.Log",
        "hello from frida"
    );
});
```

---

## method paths

method paths follow this format:

```
assembly::namespace.class.method
```

examples:

```js
UnityEngine.CoreModule::UnityEngine.Debug.Log
Assembly-CSharp::PlayerController.Jump
```

assembly is optional, but **recommended** for stability.

---

## api reference

### `Idiotify.init()`

initializes idiotify.

* does **not** enumerate all assemblies
* safely touches known assemblies only
* required before calling anything

```js
Idiotify.init();
```

---

### `Idiotify.call(path, ...args)`

resolves and invokes a method.

* supports static and instance methods
* automatically converts js strings to `Il2Cpp.String`
* caches resolved methods for performance

#### static method

```js
Idiotify.call("UnityEngine.CoreModule::UnityEngine.Debug.Log", "hello world");
```

#### instance method

```js
Idiotify.call(
    "Assembly-CSharp::PlayerController.TakeDamage",
    playerInstance,
    5
);
```

#### with explicit assembly

```js
Idiotify.call(
    "Assembly-CSharp::PlayerController.Jump",
    playerInstance
);
```

---

### `Idiotify.findMethod(path, argCount?)`

resolves a method without invoking it.

```js
const log = Idiotify.findMethod(
    "UnityEngine.Debug.Log",
    1
);

log.invoke(Il2Cpp.string("manual invoke"));
```

---

## wrappers

idiotify includes optional wrapper helpers for common unity types.

### gameobject

```js
const cube = new GameObject("PrimitiveType.Cube");
cube.name = "frida cube";
cube.active = true;
```

### transform

```js
cube.transform.position = Vec3(0, 1, 0);
cube.transform.scale = Vec3(2, 2, 2);
```

### renderer + material

```js
cube.renderer.enabled = true;
cube.renderer.material.color = Color(1, 0, 0, 1);
```

---

## value types (vector3 / color)

unity value types are created manually to avoid constructor crashes:

```js
const pos = Vec3(0, 1, 0);
const col = Color(1, 0, 0, 1);
```

these return `Il2Cpp.ValueType` instances compatible with unity apis.

---

## stability notes

* **do not enumerate `domain.assemblies`** on protected games
* always attach using **listen mode**
* always run logic inside `Il2Cpp.perform`
* mfuscator / il2cpp stripping is supported
* a crash usually mean a method was resolved too early

---

## license

frida-idiotify is released under the **mit license**.

permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "software"), to deal in the software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the software.

the software is provided "as is", without warranty of any kind.
