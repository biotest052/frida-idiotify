# frida-idiotify

a small helper library that makes calling il2cpp methods from frida easier.
it builds a runtime index of unity il2cpp metadata and lets you invoke methods without worrying about assemblies, namespaces, or enum values.

built on top of `frida-il2cpp-bridge`.

---

## requirements

* frida 16+
* `frida-il2cpp-bridge`
* unity il2cpp game (android or ios)
* frida gadget already injected (or attaching with frida)

---

## how to run (frida gadget)

1. push the scripts to the device:

```bash
adb push agent.js /data/local/tmp/agent.js
adb push frida-idiotify.js /data/local/tmp/frida-idiotify.js
```

2. configure frida gadget to load the agent:

```json
{
  "interaction": {
    "type": "script",
    "path": "/data/local/tmp/agent.js"
  }
}
```

3. launch the game.

if injection worked, you should see in logcat or the unity console:

```
[idiotify] injected successfully
```

---

## basic usage

all calls must be made inside `Il2Cpp.perform`.

```js
Il2Cpp.perform(() => {
    Idiotify.init();
    Idiotify.call("Debug.Log", "frida-idiotify loaded");
});
```

---

## example: spawn a cube

creates a unity cube and moves it upward:

```js
Il2Cpp.perform(() => {
    Idiotify.init();

    const cube = Idiotify.call(
        "GameObject.CreatePrimitive",
        "PrimitiveType.Cube"
    );

    const transform = Idiotify.call(
        "GameObject.get_transform",
        cube
    );

    Idiotify.call(
        "Transform.set_position",
        transform,
        Il2Cpp.Vector3(0, 1, 0)
    );
});
```

---

## calling methods

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

* class and method lookups are cached after first use.
* enum values can be passed as strings (`PrimitiveType.Cube`).
* if multiple classes share a name, unityengine and assembly-csharp are preferred automatically.

---

## license

frida-idiotify is released under the **mit license**.
you are free to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of this software, provided that you include the original copyright and license notice in all copies or substantial portions of the software.

this software is provided "as is", without warranty of any kind, express or implied, including but not limited to the warranties of merchantability, fitness for a particular purpose, and noninfringement. in no event shall the authors be liable for any claim, damages, or other liability arising from the use of the software.
