## Integration of Athens in Spec

status: Did a first pass on the material of renaud

This chapter has been originally written by renaud villemeur. We thank him for this contribution. It shows how you can integrate vector graphic drawing within Spec component.

### Introduction

There are two different computer graphics: vector and raster graphics. 
Raster graphics represents images as a collection of pixels. Vector graphics 
is the use of geometric primitives such as points, lines, curves, or polygons 
to represent images. These primitives are created using mathematical equations.

Both types of computer graphics have advantages and disadvantages. 
The advantages of vector graphics over raster are:
- smaller size, 
- ability to zoom indefinitely, 
- moving, scaling, filling, and rotating do not degrade the quality of an image.

Ultimately, picture on a computer are displayed on a screen with a specific 
display dimension. However, while raster graphic doesn't scale very well when
the resolution differ too much from the picture resolution, vector graphics
are rasterized to fit the display they will appear on. Rasterization is the 
technique of taking an image described in a vector graphics format and 
transform it into a set of pixels for output on a screen.

###### Note. 
You have the same concept when doing 3D programming with an API like openGL. You describe your scene with point, vertices, etc..., and in the end, you rasterize your scene to display it on your screen.

Morphic is the way to do graphics with Pharo.
However, most existing canvas are pixel based, and not vector based. 
This can be an issue with current IT ecosystems, where the resolution can differ from machine to machine (desktop, tablet, phones, etc...)

Enter Athens, a vector based graphic API. Under the scene, it can either use
balloon Canvas, or the cairo graphic library for the rasterization phase.

When you integrate Athens with Spec, you'll use its rendering engine to 
create your picture. 
It's then transformed in a `Form` and displayed on the screen.

### Hello-world in Athens

We'll see how to use Athens directly integrated with Morphic. 
This is why we first start to create a `Morph` subclass. 
Figure *@figathens@* shows the display of such a morph.
It will be the class we'll use after for all our experiment.

![AthensHello new openInWindow](figures/athens.png width=80&label=figathens)



First, we define a class, which inherit from `Morph`:
```language=Smalltalk
Morph << #AthensHello
    slots: { #surface };
    package: 'CodeOfSpec20BookAthens'
```


During the initialization phase, we'll create our Athens surface:
```language=Smalltalk
AthensHello >> initialize
    super initialize.
    self extent: self defaultExtent.
    surface := AthensCairoSurface extent: self extent
```

where `defaultExtent` is simply defined as
```language=Smalltalk
AthensHello >> defaultExtent
    ^ 400@400
```


The `drawOn:` method, mandatory in Morph subclasses, asks Athens to render
its drawing, and it'll then display it in a Morphic canvas as a Form (a bitmap 
pictures)

```language=Smalltalk
AthensHello >> drawOn: aCanvas
    self renderAthens.
    surface displayOnMorphicCanvas: aCanvas at: bounds origin
```


Our actual Athens code is located into `renderAthens` method:, and the result is
stored in the surface instance variable.

```language=Smalltalk
AthensHello >> renderAthens
    | font |
    font := LogicalFont familyName: 'Arial' pointSize: 10.
    surface drawDuring: [:canvas | 
        surface clear. 
        canvas setPaint: ((LinearGradientPaint from: 0@0  to: self extent) colorRamp: {  0 -> Color white. 1 -> Color black }).
        canvas drawShape: (0@0 extent: self extent). 
        canvas setFont: font. 
        canvas setPaint: Color pink.
        canvas pathTransform translateX: 20 Y: 20 + (font getPreciseAscent); scaleBy: 2; rotateByDegrees: 25.
        canvas drawString: 'Hello Athens in Pharo/Morphic' ]
```
Note that recreating the paint and the font is not the best way to have efficient code, but this is not the purpose of this example. 

To test your code, let's add an helper method. This will add a button on the left
of the method name. When you click on it, it'll execute the content of the 
script instruction.

```language=Smalltalk
AthensHello >> open
    <script: 'self new openInWindow'>
```





### One last thing: Handling resizing

You can already create the window, and see a nice gradient, with 
a greeting text. However, you'll notice, if you resize your window, that the 
Athens content is not resized. To fix this, we'll need one last method.

```language=Smalltalk
AthensHello >> extent: aPoint
    | newExtent |
    newExtent := aPoint rounded.
    (bounds extent closeTo: newExtent)
        ifTrue: [ ^ self ].
    bounds := bounds topLeft extent: newExtent.
    surface := AthensCairoSurface extent: newExtent.
    self layoutChanged.
    self changed
```


Congratulation, you have now created your first morphic windows where content
is rendered using Athens. 


### Using your morph with Spec

First we create a presenter to illustrate it but you could do it in one of yours. 

```
SpPresenter << #SpAthensHelloPresenter
    slots: { #morphPresenter };
    package: 'CodeOfSpec20BookAthens'
```

We define a basic layout so that Spec knows where to place it. 

```
SpAthensHelloPresenter >> defaultLayout
    ^ SpBoxLayout newTopToBottom
          add: morphPresenter;
          yourself
```

```
SpAthensHelloPresenter >> initializePresenters

    morphPresenter := self instantiate: SpMorphPresenter.
    morphPresenter morph: AthensHello new
```

Now we are done. When we open the component it will display the morph:
`SpAthensHelloPresenter new open`.


### Direct integration of Athens with Spec

We can also get a direct integration without relying on a specific Morph creation. 

We first create a presenter named `SpAthensExamplePresenter`.
This is this presenter that will support the actual rendering drawn using Athens. 


```
SpPresenter << #SpAthensExamplePresenter
    slots: { #paintPresenter };
    package: 'CodeOfSpec20BookAthens'
```
We define a simple layout to place the painPresenter.

```language=Smalltalk
SpAthensExamplePresenter >> idefaultLayout

    ^ SpBoxLayout newTopToBottom
          add: paintPresenter;
          yourself
```

This presenter wraps an `SpAthensPresenter` as follows: 
```language=Smalltalk
SpAthensExamplePresenter >> initializePresenters

    paintPresenter := self instantiate: SpAthensPresenter.
    paintPresenter surfaceExtent: 600 @ 400.
    paintPresenter drawBlock: [ :canvas | self render: canvas ]
```
It configures the `SpAthensPresenter` to call draw the `render:` message. 


Now we define `render:` method:

```language=Smalltalk
MyPresenter >> render: canvas
    canvas
        setPaint:
            (canvas surface
                createLinearGradient:
                    {(0 -> Color white).
                    (1 -> Color black)}
                start: 0@0
                stop: canvas surface extent).
    canvas drawShape: (0 @ 0 extent: canvas surface extent).
```
We could decorate the window as with any presenters.

Executing `SpAthensExamplePresenter new open` produces Figure *@figathens2@*.

![SpAthensExamplePresenter new open](figures/athens2.png width=80&label=figathens2)

This example is simple because the rendering may have to be invalidated if something changes but it shows the key aspect of the architecture.

### Conclusion

This chapter illustrates clearly that Spec can take advantage of canvas related operations such as proposed by Athens to open 
the door to specific visuals. 


