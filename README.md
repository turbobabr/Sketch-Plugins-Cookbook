Sketch Plugins Cookbook
=======================

A collection of recipes for Sketch App plugins developers.

I will be posting daily updates in my twitter. Follow me [@turbobabr](https://twitter.com/turbobabr) to stay tuned.

## Create Shared Style Programmatically

In order to create a shared style programmatically you use `-MSSharedLayerStyleContainer.addSharedStyleWithName:(NSString*)name firstInstance:(MSStyle*)style` method, where `name` is a name of shared style being created, `style` is a reference style used as a template for future shared style.

You can create a shared style from the existing style that is bound to some layer or create it from scratch with a custom `MSStyle` instance.

![Create Shared Style Programmically](./docs/create_shared_style_programmatically.png)

Create shared style from selected layers' style:
```JavaScript
var layer=selection.firstObject();
if(layer) {
    var sharedStyles=doc.documentData().layerStyles();
    sharedStyles.addSharedStyleWithName_firstInstance("Custom Style",layer.style());
}
```

Create shared style from scratch:
```JavaScript
var sharedStyles=doc.documentData().layerStyles();

var style=MSStyle.alloc().init();
var fill=style.fills().addNewStylePart();
fill.color = MSColor.colorWithSVGString("#B1C151");

sharedStyles.addSharedStyleWithName_firstInstance("Custom Style 2",style);

doc.reloadInspector();
```

Complete examples:
- [Create Shared Style From Selected Layer.sketchplugin](./Samples/Create Shared Style From Selected Layer.sketchplugin)
- [Create Shared Style Programmatically.sketchplugin](./Samples/Create Shared Style Programmatically.sketchplugin)

Works in:
- Sketch 3.1 +

## Missing 'MSColor.colorWithHex:alpha:'? :)

Prior to Sketch 3.2 there was a really nice and handy class method called `MSColor.colorWithHex:alpha:` that allowed to create instance of `MSColor` class with hex string, but unfortunately with the release of Sketch 3.2 version it was removed from the API.

Good news everyone! The replacement for this method does exist:
```JavaScript
// Create color without alpha.
var color = MSColor.colorWithSVGString("#FF0000");
print(color);
// -> (r:1.000000 g:0.000000 b:0.000000 a:1.000000)


// Create color with alpha.
var color = MSColor.colorWithSVGString("#FF0000");
color.alpha = 0.2;
print(color);
// -> (r:1.000000 g:0.000000 b:0.000000 a:0.200000)
```

Works in:
- Sketch 3.0 +

## Flatten Vector Layer

If you want to flatten a complex vector layer that contains several sub paths combined using different boolean operation into single layer, you can use `+MSShapeGroup.flatten` method.

![Flatten Vector Shape](./docs/flatten_vector_shape.png)

This sample code flattens a first selected vector layer:
```JavaScript
var layer=selection.firstObject();
if(layer && layer.isKindOfClass(MSShapeGroup)) {
    layer.flatten();
}
```

Complete examples:
- [Flatten Vector Layer.sketchplugin](./Samples/Flatten Vector Layer.sketchplugin)

Works in:
- Sketch 3.2 +

## Flatten Layers to Bitmap

In order to flatten one or several layers of any type to a single `MSBitmapLayer`, use `-MSLayerFlattener.flattenLayers:` method. It accepts one arguments which is an array of layers to be flattened.

![Flatten Layers to Bitmap](./docs/flatten_layers_to_bitmap.png)

The following example flattens all the selected layers to a bitmap layer:
```JavaScript
var flattener = MSLayerFlattener.alloc().init();
flattener.flattenLayers(selection);
```
Complete examples:
- [Flatten Selection to Bitmap.sketchplugin](./Samples/Flatten Selection To Bitmap.sketchplugin)

Works in:
- Sketch 3.2 +

## Convert Text Layer to Outlines

In order to convert an existing `MSTextLayer` to `MSShapeGroup` layer, you have to get texts' `NSBezierPath` representation and then convert it to a `MSShapeGroup` layer.

![Convert Text Layer to Outlines](./docs/convert_text_layer_to_outlines.png)

The following source code demonstrates how to get text layers' vector outline and use it to create a vector shape from it:
```JavaScript
function convertToOutlines(layer) {
    if(!layer.isKindOfClass(MSTextLayer)) return;

    var parent=layer.parentGroup();
    var shape=MSShapeGroup.shapeWithBezierPath(layer.bezierPathWithTransforms());

    shape.style = layer.style();
    var style=shape.style();
    if(!style.fill()) {
        var fill=style.fills().addNewStylePart();
        fill.color = MSColor.colorWithNSColor(layer.style().textStyle().attributes().NSColor);
    }

    var isSelected=layer.isSelected();
    shape.name = layer.name();
    shape.setIsSelected(isSelected);

    parent.removeLayer(layer);
    parent.addLayers([shape]);

    return shape;
}

var layer=selection.firstObject();
if(layer) {
    var vectorizedTextLayer=convertToOutlines(layer);
    print(vectorizedTextLayer);
}
```
Complete examples:
- [Convert Text Layer to Outlines.sketchplugin](./Samples/Convert Text Layer to Outlines.sketchplugin)

Works in:
- Sketch 3.1 +

## Get Points Coords Along the Shape Path

If you want to distribute some shapes along a path there is a convenient method `-pointOnPathAtLength:` implemented in `NSBezierPath_Slopes` class extension.

This method accepts a `double` value from 0 to 1 and represents a length at which you want to get a point coordinate. It returns a `CGPoint` struct with coordinates of the point.

![Ge points coords along shape path](./docs/getting_points_along_path.png)

The following example divides shape path into 15 segments and prints out their points coordinates:
```JavaScript
var layer=selection.firstObject();
if(layer && layer.isKindOfClass(MSShapeGroup)) {

    var count=15;
    var path=layer.bezierPathWithTransforms();

    var step=path.length()/count;
    for(var i=0;i<=count;i++) {
        var point=path.pointOnPathAtLength(step*i);
        print(point);
    }
}
```
Complete examples:
- [Get Points Coords Along Path.sketchplugin](./Samples/Get Points Coords Along Path.sketchplugin)
- [Create Dots Along Path.sketchplugin](./Samples/Create Dots Along Path.sketchplugin)

Works in:
- Sketch 3.2 +
