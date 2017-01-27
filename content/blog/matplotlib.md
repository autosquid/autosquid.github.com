+++
title = "matplot"
date = "2016-11-01"
tags = [ "matplot", "python" ]
topics = [ "programming" ]
+++

- [Key Points:](#org9359ebc)
    - [multi-backends](#org94efc6a)
    - [Mostly used parts(Artist Layer)](#org3d0611d)
    - [Artist](#orgaca1a45)
      - [attributes](#orgf3d15c5)
      - [Types:](#org5471d77)
      - [Axis](#orgab0cf79)
    - [Scripting Layer (pyplot)](#org0214104)

I have to look up the manual every time I plot with **matplotlib**, which quite frustrating.

There are posts on the same topic both in English and Chinese<sup><a id="fnr.1" class="footref" href="#fn.1">1</a></sup>. This is a brief summarize of <sup><a id="fnr.1.100" class="footref" href="#fn.1">1</a></sup>.

I'll only list the key points, and only explain when necessary.


<a id="org9359ebc"></a>

# Key Points:


<a id="org94efc6a"></a>

## multi-backends

You want to visualize with Qt/GDK/Gtk+ etc. to generate ps, pdf, svg, png etc. These dirty works are done by the backends by abstracting of existing plotting tools into:

1.  `FigureCanvas`, which encapsulates the concept of a surface to draw onto (e.g. "the paper").
2.  `Renderer`, which does the drawing (e.g. "the paintbrush").
3.  `Event`, which handles user inputs such as keyboard and mouse events.

```python
from matplotlib.figure import Figure
from matplotlib.backends.backend_agg import FigureCanvasAgg as FigureCanvas

fig    = Figure()
canvas = FigureCanvas(fig)
ax     = fig.add_axes([0.1, 0.1, 0.8, 0.8])

line,  = ax.plot([0,1], [0,1])
ax.set_title("a straight line (OO)")
ax.set_xlabel("x value")
ax.set_ylabel("y value")

canvas.print_figure('demo.jpg')
```

In daily usage, only the `Canvas` we have to care about occasionally. (e.g. the code above)

Exceptionally, we may want to tune the `events`:

```python
import numpy as np
import matplotlib.pyplot as plt

def on_press(event):
    if event.inaxes is None: return
    for line in event.inaxes.lines:
        if event.key=='t':
            visible = line.get_visible()
            line.set_visible(not visible)
    event.inaxes.figure.canvas.draw()

fig, ax = plt.subplots(1)
fig.canvas.mpl_connect('key_press_event', on_press)

ax.plot(np.random.rand(2, 20))
plt.show()
```


<a id="org3d0611d"></a>

## Mostly used parts(Artist Layer)

The Artist hierarchy is the middle layer of the matplotlib stack, where the majority of the most important objects reside.

![img](//www.aosabook.org/images/matplotlib/artists_tree.png)


<a id="orgaca1a45"></a>

## Artist

The Artist is the object that knows how to take the Renderer (the paintbrush) and put ink on the canvas.

Everything in a `Figure` is an Artist instance: it's the base class (`matplotlib.artist.Artist`).


<a id="orgf3d15c5"></a>

### attributes

1.  the transformation from the artist coordinate system to the canvas coordinate system
2.  the visibility, the clip box
3.  the label, and the interface to handle user interaction such as "picking"
4.  others


<a id="org5471d77"></a>

### Types:

1.  Primitive artists: Line2D, Rectangle, Circle, and Text.
2.  Composite artists: Axis, Tick, Axes, and Figure.


<a id="orgab0cf79"></a>

### Axis

Axis is the most important artist we can never avoid: it is where most of the `matplotlib` API plotting methods are defined.

It contains most of the graphical elements that make up the background of the plot—the ticks, the axis lines, the grid, the patch of color which is the plot background, and contains numerous helper methods that create primitive artists and add them to the Axes instance.


<a id="org0214104"></a>

## Scripting Layer (pyplot)

The `pyplot` layer is for everyday purposes, particularly for interactive exploratory work by bench scientists who are not professional programmers.

Here we need to know what is going on behind:

1.  When the pyplot module is loaded, it parses a local configuration file in which the user states, among many other things, their preference for a default backend.
2.  For a `plotting` command, pyplot will check its internal data structures to see if there is a current Figure instance, and then re-use or create one accordingly.
3.  A `show` command, will force the `Figure` to render.

## Footnotes

<sup><a id="fn.1" class="footnum" href="#fnr.1">1</a></sup> <http://www.aosabook.org/en/matplotlib.html>
