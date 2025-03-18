<!--
author:   Liad Magen
email:    liad.magen@rbinternational.com
version:  0.0.1
language: en
narrator: US English Female

comment:  A demo on Python and LiaScript.

icon:     https://pbs.twimg.com/profile_images/1388100423094087681/08Nvxq--_400x400.jpg

logo:     logo.jpg

comment:  Use the real Python in your LiaScript courses, by loading this
          template. For more information and to see, which Python-modules are
          accessible visit the [pyodide-website](https://alpha.iodide.io).

script:   ./pyodide/pyodide.js


@Pyodide.exec: @Pyodide.exec_(@uid,```@0```)

@Pyodide.exec_
<script run-once modify="# --python--\n">
async function installPackagesManually_exec(msg) {
    let module = msg.match(/ModuleNotFoundError: No module named '([^']+)/i)

    window.console.warn("Pyodide", msg)

    if (!module) {
        send.lia(msg, false)

    } else {
        if (module.length > 1) {
            module = module[1]

            if (window.pyodide_modules.includes(module)) {
                console.warn(msg)
                send.lia(msg, false)
            } else {
                send.lia("downloading module => " + module)
                window.pyodide_modules.push(module)
                await window.pyodide.loadPackage(module)
                await run_exec(code, true)
            }
        }
    }
}


async function run_exec(code, force = false) {
    if (!window.pyodide_running || force) {
        window.pyodide_running = true

        const plot = document.getElementById('target_@0')
        plot.innerHTML = ""
        document.pyodideMplTarget = plot

        if (!window.pyodide) {
            try {
                window.pyodide = await loadPyodide({ fullStdLib: false })
                window.pyodide_modules = []
                window.pyodide_running = true
            } catch (e) {
                send.lia(e.message, false)
                send.lia("LIA: stop")
            }
        }

        try {
            window.pyodide.setStdout((text) => console.log(text))
            window.pyodide.setStderr((text) => console.error(text))

            window.pyodide.setStdin({
                stdin: () => {
                    return prompt("stdin")
                }
            })

            window.pyodide.loadPackagesFromImports(code).then(async () => {
                const rslt = await window.pyodide.runPython(code)
    
                if (rslt !== undefined) {
                    send.lia(rslt)
                } else {
                    send.lia("")
                }
            }, installPackagesManually_exec);

        } catch (e) {
            installPackagesManually_exec(e.message)
        }
        send.lia("LIA: stop")
        window.pyodide_running = false
    } else {
        setTimeout(() => { run_exec(code) }, 1000)
    }
}

setTimeout(() => { run_exec(`# --python--
@1 # --python--
`) }, 500)

"calculating, please wait ..."

</script>

<div id="target_@0"></div>
@end





@Pyodide.eval: @Pyodide.eval_(@uid)

@Pyodide.eval_
<script>
async function installPackagesManually_eval(msg) {
    let module = msg.match(/ModuleNotFoundError: No module named '([^']+)/i);

    window.console.warn('Pyodide', msg);

    if (!module) {
        const err = msg.match(/File "<exec>", line (\d+).*\n((.*\n){1,3})/i);

        if (err !== null && err.length >= 3) {
            send.lia(msg,
                [[{
                    row: parseInt(err[1]) - 1,
                    column: 1,
                    text: err[2],
                    type: 'error',
                }]],
                false);
        } else {
            console.error(msg);
        }
    } else if (module.length > 1) {
        module = module[1];

        if (window.pyodide_modules.includes(module)) {
            console.error(msg);
        } else {
            console.debug('downloading module =>', module);
            window.pyodide_modules.push(module);
            await window.pyodide.loadPackage(module);
            await run_eval(code);
        }
    }
}

async function run_eval(code) {

    const plot = document.getElementById('target_@0')
    plot.innerHTML = ""
    document.pyodideMplTarget = plot

    if (!window.pyodide) {
        try {
            window.pyodide = await loadPyodide({ fullStdLib: false })
            window.pyodide_modules = []
            window.pyodide_running = true
        } catch (e) {
            console.error(e.message)
            send.lia("LIA: stop")
        }
    }

    try {
        window.pyodide.setStdout({
            write: (buffer) => {
                const decoder = new TextDecoder()
                const string = decoder.decode(buffer)
                console.stream(string)
                return buffer.length
            }
        })

        window.pyodide.setStderr({
            write: (buffer) => {
                const decoder = new TextDecoder()
                const string = decoder.decode(buffer)
                console.error(string)
                return buffer.length
            }
        })

        window.pyodide.setStdin({
            stdin: () => {
                return prompt("stdin")
            }
        })

        window.pyodide.loadPackagesFromImports(code).then(async () => {
            const rslt = await window.pyodide.runPython(code)

            if (typeof rslt === 'string') {
                send.lia(rslt)
            } else if (rslt && typeof rslt.toString === 'function') {
                send.lia(rslt.toString());
            }
        }, installPackagesManually_eval);

    } catch (e) {
        installPackagesManually_eval(e.message);
    }
    send.lia("LIA: stop")
    window.pyodide_running = false
}

if (window.pyodide_running) {
  setTimeout(() => {
    console.warn("Another process is running, wait until finished")
  }, 500)
  "LIA: stop"
} else {
  window.pyodide_running = true

  setTimeout(() => {
    run_eval(`@input`)
  }, 500)

  "LIA: wait"
}
</script>

<div id="target_@0"></div>
@end

-->

# "Data Detective Agency"- Escape the Data Exploration Course!
As members of an elite data detective agency, you are tasked with solving complex cases by analyzing various datasets. 
Your client, a world reknown Austrian bank, needs assistance in accessing their safe. 
The safe managers, who are all on a vacation, have left sets of clues and some datasets.

## 1. The Entrance Hall

You enter the Bank's entrance hall.
The door to your left has a screen on it.

``` python
import sys

for i in range(5):
	print("Hello", 'World #', i)

print(sys.version)
```
@Pyodide.eval


---

> Currently you can only plot one image, but this is shown within the terminal
> and also stored persitently. Thus, you can switch between different versions
> of your code and the resulting images will change too.

``` python
import numpy as np
import matplotlib.pyplot as plt

t = np.arange(0.0, 2.0, 0.01)
s = np.sin(2 * np.pi * t)

fig, ax = plt.subplots()
ax.plot(t, s)

ax.grid(True, linestyle='-.')
ax.tick_params(labelcolor='r', labelsize='medium', width=3)

plt.show()
```
@Pyodide.eval

> **NOTE:** You can resize the terminal and you can click onto the image to
> enlarge it. The additional loading of libraries is only required once, then
> they are stored locally...


``` python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_csv("https://raw.code.rbi.tech/WZHLIMB/teaching_tools_playground/main/EDA/tips.csv?token=GHSAT0AAAAAAAAAQ5MT5HGWQYXGBO64QWKGZ45ES4A")
df
```
@Pyodide.eval
