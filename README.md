# push2-python

Utils to interface with [Ableton's Push 2](https://www.ableton.com/en/push/) from Python.

These utils follow Ableton's [Push 2 MIDI and Display Interface Manual](https://github.com/Ableton/push-interface/blob/master/doc/AbletonPush2MIDIDisplayInterface.asc) for comunicating with Push 2. I recommend reading Ableton's manual before using this tool.

So far I only implemented some utils to **interface with the display** and some utils for **basic interaction with pads, buttons, encoders and the touchstrip**. More detailed interaction with each of these elements (e.g. changing color palettes, support for led blinking, advanced touchstrip configuration, etc.) has not been implemented. Contributions are welcome :)

I only testd the package in **Python 3** and **macOS**. Some things will not work on Python 2 but it should be easy to port. I don't know how it will work on Windows/Linux. It is possible that MIDI port names (see [push2_python/constants.py](https://github.com/ffont/push2-python/blob/master/push2_python/constants.py#L12-L13)) need to be changed to correctly reach Push2 in Windows/Linux.


## Table of Contents

* [Install](#install)
* [Documentation](#documentation)
    * [Initializing Push](#initializing-push)
    * [Setting action handlers for buttons, encoders, pads and the touchstrip](#setting-action-handlers-for-buttons--encoders--pads-and-the-touchstrip)
    * [Button names, encoder names, pad numbers and coordinates](#button-names--encoder-names--pad-numbers-and-coordinates)
    * [Set pad and button colors](#set-pad-and-button-colors)
    * [Interface with the display](#interface-with-the-display)
* [Code examples](#code-examples)
    * [Set up handlers for pads, encoders, buttons and the touchstrip...](#set-up-handlers-for-pads-encoders-buttons-and-the-touchstrip)
    * [Light up buttons and pads](#light-up-buttons-and-pads)
    * [Interface with the display (static content)](#interface-with-the-display-static-content)
    * [Interface with the display (dynamic content)](#interface-with-the-display-dynamic-content)


## Install

You can install using `pip` and pointing at this repository:

```
pip install git+https://github.com/ffont/push2-python
```

This will install Python requirements as well. Note however that `push2-python` requires [pyusb](https://github.com/pyusb/pyusb) which is based in [libusb](https://libusb.info/). You'll most probably need to manually install `libusb` for your operative system if `pip` does not do it for you.

## Documentation

Well, to be honest there is no proper documentation. However the use of this package is so simple that I hope it's going to be enough with the [code examples below](#code-examples) and the simple notes given here.

### Initializing Push
 
 To interface with Push2 you'll first need to import `push2_python` and initialize a Python object as follows:

```python
import push2_python
push = push2_python.Push2() 
```

**NOTE**: all code snippets below assume you import `push2_python` and initialize the `Push2` like in the snippet above.

You can pass the optional argument `use_user_midi_port=True` when initializing `push` to tell it to use User MIDI port instead of Live MIDI port. Check [MIDI interface access](https://github.com/Ableton/push-interface/blob/master/doc/AbletonPush2MIDIDisplayInterface.asc#midi-interface-access) and [MIDI mode](https://github.com/Ableton/push-interface/blob/master/doc/AbletonPush2MIDIDisplayInterface.asc#MIDI%20Mode) sections of the [Push 2 MIDI and Display Interface Manual](https://github.com/Ableton/push-interface/blob/master/doc/AbletonPush2MIDIDisplayInterface.asc) for more information.

### Setting action handlers for buttons, encoders, pads and the touchstrip

You can easily set action handlers that will trigger functions when the physical pads, buttons, encoders or the touchtrip are used. You do that by **decorating functions** that will be triggered in response to the physical actions. For example, you can set up an action handler that will be triggered when
the left-most encoder is rotated in this way:

```python
@push2_python.on_encoder_rotated(push2_python.constants.ENCODER_TEMPO_ENCODER)
def on_left_encoder_rotated(push, incrememnt):
    print('Left-most encoder rotated with increment', increment)
```

Similarly, you can set up an action handler that will trigger when play button is pressed in this way:

```python
@push2_python.on_button_pressed(push2_python.constants.BUTTON_PLAY)
def on_play_pressed(push):
    print('Play!')
```

These are all available decorators for setting up action handlers:

* `@push2_python.on_button_pressed(button_name=None)`
* `@push2_python.on_button_released(button_name=None)`
* `@push2_python.on_touchstrip()`
* `@push2_python.on_pad_pressed(pad_n=None, pad_ij=None)`
* `@push2_python.on_pad_released(pad_n=None, pad_ij=None)`
* `@push2_python.on_pad_aftertouch(pad_n=None, pad_ij=None)`
* `@push2_python.on_encoder_rotated(encoder_name=None)`
* `@push2_python.on_encoder_touched(encoder_name=None)`
* `@push2_python.on_encoder_released(encoder_name=None)`

Full documentation for each of these can be found in their docstrings [starting here](https://github.com/ffont/push2-python/blob/master/push2_python/__init__.py#L128). 
Also have a look at the [code examples](#code-examples) below to get an immediate idea about how it works.


### Button names, encoder names, pad numbers and coordinates

Buttons and encoders can de identified by their name. You can get a list of avialable options for  `button_name` and `encoder_name` by checking the
contents of [push2_python/constants.py](https://github.com/ffont/push2-python/blob/master/push2_python/constants.py)
or by using the following properties after intializing the `Push2` object:

```python
print(push.buttons.available_names)
print(push.encoders.available_names)
```

Pads are identified either by their number (`pad_n`) or by their coordinates (`pad_ij`). Pad numbers correspond to the MIDI note numbers assigned
to each pad as defined in [Push 2 MIDI and Display Interface Manual](https://github.com/Ableton/push-interface/blob/master/doc/AbletonPush2MIDIDisplayInterface.asc#23-midi-mapping) (see MIDI mapping diagram). Pad coordinates are specified as a `(i,j)` tuples where `(0,0)` corresponds to the top-left pad and `(7, 7)` corresponds to the bottom right pad.

### Set pad and button colors

Pad and button colors can be set using methods provided by the `Push2` object. For example you can set pad colors using the following code:

```python
push = push2_python.Push2() 
pad_ij = (0, 3)  # Fourth pad of the top row
push.pads.set_pad_color(pad_ij, 'green')
```

You set button colors in a similar way:

```python
push = push2_python.Push2() 
push.buttons.set_button_color(push2_python.constants.BUTTON_PLAY, 'green')
```

All pads support RGB colors, and some buttons do as well. However, some buttons only support black and white. Checkout the MIDI mapping diagram in the 
[Push 2 MIDI and Display Interface Manual](https://github.com/Ableton/push-interface/blob/master/doc/AbletonPush2MIDIDisplayInterface.asc#23-midi-mapping) to see which buttons support RGB and which ones only support black and white. In both cases colors are set using the same method, but the list of available colors for black and white buttons is restricted.

For a list of avilable RGB colors check the keys of the `RGB_COLORS` dictionary in [push2_python/constants.py](https://github.com/ffont/push2-python/blob/master/push2_python/constants.py). Similarly, black and white available colors are defined in the `BLACK_WHITE_COLORS` dictionary in the same file. You can also list available colors in code like this:

```python
print(list(push2_python.constants.RGB_COLORS.keys()))
print(list(push2_python.constants.BLACK_WHITE_COLORS.keys()))
```

**NOTE**: The Push 2 Interface Manual provides a way to configure custom color palettes which has not yet been implemented in `push2-python`. Also, only a limited number of colors is included here but many more are avilable in the default color palette.


### Interface with the display

You interface with Push2's display by senidng frames to be display using the `push.display.display_frame` method as follows:

```python
img_frame = ...  # Some existing valid img_frame
push.display.display_frame(img_frame, input_format=push2_python.constants.FRAME_FORMAT_BGR565)
```

`img_frame` is expected to by a `numpy` array. Depending on the `input_format` argument, `img_frame` will need to have the following characteristics:
        
* for `push2_python.constants.FRAME_FORMAT_BGR565`: `numpy` array of shape 910x160 and of type `uint16`. Each `uint16` element specifies rgb 
    color with the following bit position meaning: `[b4 b3 b2 b1 b0 g5 g4 g3 g2 g1 g0 r4 r3 r2 r1 r0]`.

* for `push2_python.constants.FRAME_FORMAT_RGB565`: `numpy` array of shape 910x160 and of type `uint16`. Each `uint16` element specifies rgb 
    color with the following bit position meaning: `[r4 r3 r2 r1 r0 g5 g4 g3 g2 g1 g0 b4 b3 b2 b1 b0]`.

* for `push2_python.constants.FRAME_FORMAT_RGB`: numpy array of shape 910x160x3 with the third dimension representing rgb colors
    with separate float values for rgb channels (float values in range `[0.0, 1.0]`).

The preferred format is `push2_python.constants.FRAME_FORMAT_BGR565` as it requires no conversion before sending to Push2 (that is the format that Push2 expects). Using `push2_python.constants.FRAME_FORMAT_BGR565` it should be possible to achieve frame rates of more than 36fps (depending on the speed of your computer). 
With `push2_python.constants.FRAME_FORMAT_RGB565` we need to convert the frame to `push2_python.constants.FRAME_FORMAT_BGR565` before sending to Push2. This will reduce frame rates to ~14fps (allways depending on the speed of your computer). Sending data in `push2_python.constants.FRAME_FORMAT_RGB` will result in very long frame conversion times that can take seconds. This format should only be used for displaying static images that are prepared offline using the `push.display.prepare_frame` method. The code examples below ([here](#interface-with-the-display-static-content) and [here](#interface-with-the-display-dynamic-content)) should give you an idea of how this works. It's easy!

**NOTE 1**: According to Push2 display specification, when you send a frame to Push2, it will stay on screen for two seconds. Then the screen will go to black.

**NOTE 2**: Interfacing with the display using `push2-python` won't allow you to get very high frame rates, but it should be enough for most applications. If you need to make more hardcore use of the display you should probably implement your own funcions directly in C or C++. Push2's display theoretically supports up to 60fps. More information in the [Push 2 MIDI and Display Interface Manual](https://github.com/Ableton/push-interface/blob/master/doc/AbletonPush2MIDIDisplayInterface.asc#32-display-interface-protocol).


## Code examples

### Set up action handlers for pads, encoders, buttons and the touchstrip...

```python
import push2_python

# Init Push2
push = push2_python.Push2()

# Now set up some action handlers that will trigger when interacting with Push2
# This is all done using decorators.
@push2_python.on_pad_pressed()
def on_pad_pressed(push, pad_n, pad_ij, velocity):
    print('Pad', pad_ij, 'pressed with velocity', velocity)

@push2_python.on_encoder_rotated()
def on_encoder_rotated(push, encoder_name, increment):
    print('Encoder', encoder_name, 'rotated', increment)

@push2_python.on_touchstrip()
def on_touchstrip(push, value):
    print('Touchstrip touched with value', value)

# You can also set handlers for specic encoders or buttons by passing argument to the decorator
@push2_python.on_encoder_rotated(push2_python.constants.ENCODER_TRACK1_ENCODER)
def on_encoder1_rotated(push, incrememnt):
    print('Encoder for Track 1 rotated with increment', increment)

@push2_python.on_button_pressed(push2_python.constants.BUTTON_1_16)
def on_button_pressed(push):
    print('Button 1/16 pressed')

# Now start infinite loop so the app keeps running
print('App runnnig...')
while True:
    pass
```

### Light up buttons and pads

```python
import push2_python

# Init Push2
push = push2_python.Push2()

# Start by setting all pad colors to white
push.pads.set_all_pads_to_color('white')

@push2_python.on_button_pressed()
def on_button_pressed(push, button_name):
    # Set pressed button color to white
    push.buttons.set_button_color(button_name, 'white')

@push2_python.on_button_released()
def on_button_released(push, button_name):
    # Set released button color to black (off)
    push.buttons.set_button_color(button_name, 'black')

@push2_python.on_pad_pressed()
def on_pad_pressed(push, pad_n, pad_ij, velocity):
    # Set pressed pad color to green
    push.pads.set_pad_color(pad_ij, 'green')

@push2_python.on_pad_released()
def on_pad_released(push, pad_n, pad_ij, velocity):
    # Set released pad color back to white
    push.pads.set_pad_color(pad_ij, 'white')

# Start infinite loop so the app keeps running
print('App runnnig...')
while True:
    pass
```

### Interface with the display (static content)

Here you have some example code for interfacing with Push2's display. Note that this code example requires [`pillow`](https://python-pillow.org/) Python package, install it with `pip install pillow`.

```python
import push2_python
import random
import numpy
from PIL import Image

# Init Push2
push = push2_python.Push2()

# Define util function to generate a frame with some colors to be shown in the display
# Frames are created as matrices of shape 960x160 and with colors defined in bgr565 format
# This function is defined in a rather silly way, could probably be optimized a lot ;)
def generate_3_color_frame():
    colors = ['{b:05b}{g:06b}{r:05b}'.format(
                  r=int(32*random.random()), g=int(64*random.random()), b=int(32*random.random())),
              '{b:05b}{g:06b}{r:05b}'.format(
                  r=int(32*random.random()), g=int(64*random.random()), b=int(32*random.random())),
              '{b:05b}{g:06b}{r:05b}'.format(
                  r=int(32*random.random()), g=int(64*random.random()), b=int(32*random.random()))]
    line_bytes = []
    for i in range(0, 960):  # 960 pixels per line
        if i <= 960 // 3:
            line_bytes.append(colors[0])
        elif 960 // 3 < i <= 2 * 960 // 3:
            line_bytes.append(colors[1])
        else:
            line_bytes.append(colors[2])
    frame = []
    for i in range(0, 160):  # 160 lines
        frame.append(line_bytes)
    return numpy.array(frame, dtype=numpy.uint16).transpose()

# Pre-generate different color frames
color_frames = list()
for i in range(0, 20):
    color_frames.append(generate_3_color_frame())

# Now crate an extra frame which loads an image from a file. Image must be 960x160 pixels.
img = Image.open('test_img_960x160.png')
img_array = numpy.array(img)
frame = img_array/255  # Convert rgb values to [0.0, 1.0] floats

# Because the pixel format returned by Image.open is not the one required for Push2's display,
# this frame needs to be prepared before sending it to Push. This conversion takes a bit of 
# time so we do it offline. Some formats can be converted on the fly by `push2-python` but not 
# the RGB format retruned by PIL.
prepared_img_frame = \
    push.display.prepare_frame(frame, input_format=push2_python.constants.FRAME_FORMAT_RGB)

# Now lets configure some action handlers which will display frames in Push2's display in 
# reaction to pad and button presses
@push2_python.on_pad_pressed()
def on_pad_pressed(push, pad_n, pad_ij, velocity):
    # Display one of the three color frames on the display
    random_frame = random.choice(color_frames)
    push.display.display_frame(random_frame)

@push2_python.on_button_pressed()
def on_button_pressed(push, button_name):
    # Display the frame with the loaded image
    push.display.display_prepared_frame(prepared_img_frame)

# Start infinite loop so the app keeps running
print('App runnnig...')
while True:
    pass
```

### Interface with the display (dynamic content)

And here is a more advanced example of interfacing with the display. In this case display frames are generated dynamically and show some values that can be modified by rotating the encoders. Note that this code example requires [`pycairo`](https://github.com/pygobject/pycairo) Python package, install it with `pip install pycairo` (you'll most probably also need to install [`cairo`](https://www.cairographics.org/) before that, see [this page](https://pycairo.readthedocs.io/en/latest/getting_started.html) for info on that).

```python
import push2_python
import cairo
import numpy
import random
import time

# Init Push2
push = push2_python.Push2()

# Init dictionary to store the state of encoders
encoders_state = dict()
max_encoder_value = 100
for encoder_name in push.encoders.available_names:
    encoders_state[encoder_name] = {
        'value': int(random.random() * max_encoder_value),
        'color': [random.random(), random.random(), random.random()],
    }
last_selected_encoder = list(encoders_state.keys())[0]

# Function that generates the contents of the frame do be displayed
def generate_display_frame(encoder_value, encoder_color, encoder_name):

    # Prepare cairo canvas
    WIDTH, HEIGHT = push2_python.constants.DISPLAY_LINE_PIXELS, push2_python.constants.DISPLAY_N_LINES
    surface = cairo.ImageSurface(cairo.FORMAT_RGB16_565, WIDTH, HEIGHT)
    ctx = cairo.Context(surface)

    # Draw rectangle with width proportional to encoders' value
    ctx.set_source_rgb(*encoder_color)
    ctx.rectangle(0, 0, WIDTH * (encoder_value/max_encoder_value), HEIGHT)
    ctx.fill()

    # Add text with encoder name and value
    ctx.set_source_rgb(1, 1, 1)
    font_size = HEIGHT//3
    ctx.set_font_size(font_size)
    ctx.select_font_face("Arial", cairo.FONT_SLANT_NORMAL, cairo.FONT_WEIGHT_NORMAL)
    ctx.move_to(10, font_size * 2)
    ctx.show_text("{0}: {1}".format(encoder_name, encoder_value))

    # Turn canvas into numpy array compatible with push.display.display_frame method
    buf = surface.get_data()
    frame = numpy.ndarray(shape=(HEIGHT, WIDTH), dtype=numpy.uint16, buffer=buf)
    frame = frame.transpose()
    return frame

# Set up action handlers to react to encoder touches and rotation
@push2_python.on_encoder_rotated()
def on_encoder_rotated(push, encoder_name, increment):
    def update_encoder_value(encoder_idx, increment):
        updated_value = int(encoders_state[encoder_idx]['value'] + increment)
        if updated_value < 0:
            encoders_state[encoder_idx]['value'] = 0
        elif updated_value > max_encoder_value:
            encoders_state[encoder_idx]['value'] = max_encoder_value
        else:
            encoders_state[encoder_idx]['value'] = updated_value

    update_encoder_value(encoder_name, increment)
    global last_selected_encoder
    last_selected_encoder = encoder_name

@push2_python.on_encoder_touched()
def on_encoder_touched(push, encoder_name):
    global last_selected_encoder
    last_selected_encoder = encoder_name

# Draw method that will generate the frame to be shown on the display
def draw():
    encoder_value = encoders_state[last_selected_encoder]['value']
    encoder_color = encoders_state[last_selected_encoder]['color']
    frame = generate_display_frame(encoder_value, encoder_color, last_selected_encoder)
    push.display.display_frame(frame, input_format=push2_python.constants.FRAME_FORMAT_RGB565)

# Now start infinite loop so the app keeps running
print('App runnnig...')
while True:
    draw()
    time.sleep(1.0/30)  # Sart drawing loop, aim at ~30fps
```
