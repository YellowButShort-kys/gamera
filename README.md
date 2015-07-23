gamera.lua
==========

A camera for [LÖVE](http://love2d.org).

Initial setup
-------------

The first thing one needs to do to use gamera is to create it. You do so with `gamera.new`. This function requires 4 numbers (left, top, width and height) defining the "world boundaries" for the camera.

    local cam = gamera.new(0,0,2000,2000)

The left and top parameters are usually 0,0, but they can be anything, even negative numbers (see below).

You can update the world definition later on with `setWorld`:

    cam:setWorld(0,0,2000,2000)

By default gamera will use the whole screen to display graphics. You can restrict the amount of screen used with `setWindow`:

    cam:setWindow(0,0,800,600)

Moving the camera around
------------------------

You can move the camera around by using `setPosition`:

    cam:setPosition(100, 200)

`setPosition` takes into account the current window boundaries and world boundaries, and will keep the view inside the world. This means that if you try to look at something very close to the left border of the world, for example, the camera will not "scroll" to show empty space.

You can also zoom in and zoom out. This is done using the `setScale` method. When given two parameters, the first one is the x-scale, while the second one is the y-scale. If you pass only one parameter, it's used for both x and y. The default scale is 1.0 in both axis.

    cam:setScale(2.0) -- make everything twice as bigger. By default, scale is 1 (no change)

Take notice that gamera limits the amount of zoom out you can make; you can not "zoom out" to see the world edges. If you want to do this, make the world bigger first. For example, to give a 100-pixels border to a world defined as `0,0,2000,`, you can define it like `-100,100,2100,2100` instead.

Finally, you can modify the angle with `setAngle`:

    cam:setAngle(newAngle) -- newAngle is in radians, by default the angle is 0

`setAngle` will change *both* the scale and position of the camera to force you not to see the world borders. If you don't want this to happen, expand the world borders as mentioned above.

Drawing
-------

The camera has one method called "draw". It takes one function as a parameter, like this:

    cam:draw(function(l,t,w,h)
      -- draw camera stuff here
    end)

Anything drawn inside the function will be scaled, rotated, translated and cut so that it appears as it should in the screen window.

Notice that the function takes 4 optional parameters. These parameters represent the area that the camera "sees" (same as calling `cam:getVisible()`). They can be used to optimize the drawing, and not draw anything outside of those borders. Those borders are always axis-aligned. This means that when the camera is rotated, the area might include elements that are not strictly visible.


Querying the camera
-------------------

* `cam:getWorld()` returns the l,t,w,h of the world
* `cam:getWindow()` returns the l,t,w,h of the screen window
* `cam:getVisible()` returns the l,t,w,h of what is currently visible in the world, taking into account rotation, scale and translation. It coincides with the parameters of the callback function in `gamera.draw`. It can contain more than what is necessary due to rotation.
* `cam:getVisibleCorners()` returns the corners of the rotated rectangle that represent the exact region being seen by the camera, in the form `x1,y1,x2,y2,x3,y3,x4,y4`

* `cam:getPosition()` returns the coordinates the camera is currently "looking at", after it has been corrected so that the world boundaries are not visible, if possible.
* `cam:getScale()` returns the current scaleX and scaleY parameters
* `cam:getAngle()` returns the current rotation angle, in radians

Coordinate transformations
--------------------------

* `cam:toWorld(x,y)` transforms screen coordinates into world coordinates, taking into account the window, scale, rotation and translation. Useful for mouse interaction, for example.
* `cam:toScreen(x,y)` transforms given a coordinate in the world, return the real coords it has on the screen. Useul to represent icons in minimaps, for example.

FAQ
===

> Everything looks kindof "blurry" when I do zooms/rotations with this library. How do I prevent it?

LÖVE uses a default [filter mode](https://love2d.org/wiki/FilterMode) which makes images "blurry" when you make almost any transformation to them. To deactivate this behavior globally, you can add this at the beginning of your game, before you load anything (for example at the beginning of `love.load`):

``` lua
love.graphics.setDefaultFilter( 'nearest', 'nearest' )
```

You can also set the filter of each image you load individually:

``` lua
local img = love.graphics.newImage('imgs/player.png')
img:setFilter('nearest', 'nearest')
```

> I see "seams" around my tiles when I use this library. Why?

It is due to a combination of factors: how OpenGL textures work, how floating point numbers work and how LÖVE stores texture information in memory. 

The end result is that sometimes, when drawing an image, "parts of the area around it" are also drawn. So if you draw a Quad of "earth", and immediately above it in your image you have a "bright lava" tile, when you draw the earth tile sometimes "a bit of the lava" is drawn near the top. If you are using images instead of quads, you can get seams too (either black or from other colors, depending on how the image is stored in memory).

To prevent this:

* Always use quads for tiles, never images.
* Add a 2-pixel border to each of your quads (So for a 32x32 quad, you use 36x36 space, with the 32x32 quad in the center and a 2-pixel border)
* Fill up the borders with the colors of the 32x32 quad. For example, if the earth quad is all brown on its left side, color the left border brown. If on its upper side is gray in the center and brown on the sides, color the upper border gray in the center and brown on the sides.
* The quads will still be 32x32 - they will just have some border in the image now.
* The "seams" will still appear, but now they will show the "colored borders" of the quads, so they will not be noticeable.
* Calculating the coordinates of the quads is a bit more complex than before. You can use [anim8](https://github.com/kikito/anim8)'s "Grids" to simplify getting them:

```lua
local anim8 = require 'anim8'

...

local tiles = love.graphics.newImage('tiles.png')
local w,h = tiles:getDimensions()

local g = anim8.newGrid(32, 32, w, h, 0, 0, 2) -- tileWidth, tileHeight, imageW, imageH, left, top, border

local earth = g(1,1)[1] -- Get the first column, first row 32x32 quad, including border
local water = g(2,1)[1] -- Get the second column, first row quad
```

You can combine this with the previous FAQ and use a "nearest" filter instead of a linear one.

Installation
============

Just copy the gamera.lua file wherever you want it. Then require it where you need it:

    local gamera = require 'gamera'

Please make sure that you read the license, too (for your convenience it's included at the beginning of the gamera.lua file).

