# frida-idiotify

a helper library for calling il2cpp methods in unity games using frida.
it builds a runtime index of unity il2cpp metadata, handles assemblies, namespaces, and enums automatically, and includes simple wrapper classes for common unity objects like `GameObject` and `Transform`.

built on top of `frida-il2cpp-bridge`.

---

## requirements

* frida 16+
* `frida-il2cpp-bridge`
* unity il2cpp game (android or ios)
* frida gadget already injected (or attaching with frida)

---

## setup and run

1. push the scripts to the device:

```bash
adb push agent.js /data/local/tmp/agent.js
adb push frida-idiotify.js /data/local/tmp/frida-idiotify.js
adb push wrappers.js /data/local/tmp/wrappers.js
```

2. launch the game and then try injecting using frida/frida gadget.
   if injection worked, you should see in logcat or the unity console:

```
[idiotify] injected successfully, have fun doing stuff easily!
```

---

## basic usage

all calls must be performed inside `Il2Cpp.perform`.

```js
Il2Cpp.perform(() => {
    Idiotify.init();

    Idiotify.call("Debug.Log", "calling debug.log through frida-idiotify");
});
```

---

## wrapper example: spawn and move a cube

with the included GameObject and Transform wrappers:

```js
Il2Cpp.perform(() => {
    Idiotify.init();

    const cube = new GameObject("PrimitiveType.Cube");
    cube.transform.position = Il2Cpp.Vector3(0, 1, 0);
    cube.active = true;

    Idiotify.call("Debug.Log", "cube created via wrapper");
});
```

---

## calling methods directly

### static methods

```js
Idiotify.call("Debug.Log", "hello");
```

### instance methods

```js
Idiotify.call("Button.set_interactable", buttonInstance, true);
```

### assembly override

```js
Idiotify.call("Assembly-CSharp::PlayerController.Jump", playerInstance);
```

---

## notes

* class and method lookups are cached for performance.
* enum values can be passed as strings (`PrimitiveType.Cube`).
* if multiple classes share a name, unityengine and assembly-csharp are preferred automatically.
* wrapper classes (`GameObject`, `Transform`) provide clean syntax for common operations.

---

## license
frida-idiotify is released under the **mit license**. you are free to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of this software, provided that you include the original copyright and license notice in all copies or substantial portions of the software. 

this software is provided "as is", without warranty of any kind, express or implied, including but not limited to the warranties of merchantability, fitness for a particular purpose, and noninfringement. in no event shall the authors be liable for any claim, damages, or other liability arising from the use of the software.
