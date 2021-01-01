<!-- embrava blynclight agile innovations blinkstick kuando busylight luxafor flag thingM blink(1) -->
![BusyLight Project Logo][1]

![Python 3.7 Test][37] ![Python 3.8 Test][38] ![Python 3.9 Test][39]

[BusyLight for Humans™][0] gives you control of USB attached LED
lights from a variety of vendors. Lights can be controlled via
the command-line, using a HTTP API or imported directly into your own
python project.

![All Supported Lights][DemoGif]

<em>Back to Front, Left to Right</em> <br>
<b>BlyncLight, BlyncLight Plus, Busylight</b> <br>
<b>Blink(1), Flag, BlinkStick</b>

## Features
- Control Lights via [Command-Line][BUSYLIGHT.1]
- Control Lights via [Web API][WEBAPI]
- Five (5!) Supported Vendors:
  * [**Agile Innovations** BlinkStick ][2]
  * [**Embrava** Blynclight][3]
  * [**Kuando** BusyLight][4]
  * [**Luxafor** Flag][5]
  * [**ThingM** Blink1][6]
- Supported on MacOS, Linux, probably Windows and BSD too!
- Tested extensively on Raspberry Pi 3b+, Zero W and 4

## Basic Install 

```console
$ python3 -m pip install busylight-for-humans 
```

## Web API Install

Install `uvicorn` and `FastAPI` in addition to `busylight`:

```console
$ python3 -m pip install busylight-for-humans[webapi]
```

## Linux Post-Install Activities
Linux controls access to USB devices via the udev subsystem and by default denies non-root users access to devices it doesn't recognize. I've got you covered!

You'll need root access to configure the udev rules:

```console
$ busylight udev-rules -o 99-busylights.rules
$ sudo cp 99-busylights.rules /etc/udev/rules.d
$ sudo udevadm control -R
$ # unplug/plug your light
$ busylight on
```

## Command-Line Examples

```console
$ busylight on               # light turns on green
$ busylight on red           # now it's shining a friendly red
$ busylight on 0xff0000      # still red
$ busylight on #00ff00       # now it's blue!
$ busylight blink            # it's slowly blinking on and off with a red color
$ busylight blink green fast # blinking faster green and off
$ busylight --all on         # turn all lights on green
$ busylight --all off        # turn all lights off
```

## HTTP API Examples

First start the `busylight` API server:
```console
$ busylight serve
INFO:     Started server process [20189]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8888 (Press CTRL+C to quit)
```

The API is fully documented and available @ `https://localhost:8888/redoc`


Now you can use the web API endpoints which return JSON payloads:

```console
  $ curl http://localhost:8888/1/lights
  $ curl http://localhost:8888/1/lights/on
  $ curl http://localhost:8888/1/lights/off
  $ curl http://localhost:8888/1/light/0/on/purple
  $ curl http://localhost:8888/1/light/0/off
  $ curl http://localhost:8888/1/lights/on
  $ curl http://localhost:8888/1/lights/off
  $ curl http://localhost:8888/1/lights/rainbow
```

## Code Examples


### Simple Case: Turn On a Single Light

In this example, we pick an Embrava Blynclight to activate with
the color white. 

```python
from busylight.lights.embrava import Blynclight

light = Blynclight.first_light()

light.on((255, 255, 255))
```

### Slightly More Complicated

The `busylight` package includes a manager class that great for managing
multiple lights or lights that are currently being animated:

```python
from busylight.manager import LightManager, ALL_LIGHTS
from busylight.effects import rainbow

manager = LightManager()
for light in manager.lights:
   print(light.name)

manager.apply_effect_to_light(ALL_LIGHTS, rainbow)
...
manager.lights_off(ALL_LIGHTS)
```

[0]: https://pypi.org/project/busylight-for-humans/
[1]: https://github.com/JnyJny/busylight/blob/master/docs/assets/BusyLightLogo.png
[BUSYLIGHT.1]: https://github.com/JnyJny/busylight/blob/master/docs/busylight.1.md
[WEBAPI]: https://github.com/JnyJny/busylight/blob/master/docs/busylight_api.pdf
[2]: https://github.com/JnyJny/busylight/blob/master/docs/devices/agile_innovations.md
[3]: https://github.com/JnyJny/busylight/blob/master/docs/devices/embrava.md
[4]: https://github.com/JnyJny/busylight/blob/master/docs/devices/kuando.md
[5]: https://github.com/JnyJny/busylight/blob/master/docs/devices/luxafor.md
[6]: https://github.com/JnyJny/busylight/blob/master/docs/devices/thingm.md

[37]: https://github.com/JnyJny/busylight/workflows/Python%203.7/badge.svg
[38]: https://github.com/JnyJny/busylight/workflows/Python%203.8/badge.svg
[39]: https://github.com/JnyJny/busylight/workflows/Python%203.9/badge.svg

[DemoGif]: https://github.com/JnyJny/busylight/raw/master/demo/demo.gif
