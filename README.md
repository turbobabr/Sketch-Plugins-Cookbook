Sketch-Plugins-Cookbook
=======================

A collection of recipes for Sketch App plugins developers

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

In order to flatten one or several layers of any type to a single `MSBitmapLayer`, use `-MSLayerFlattener.flattenLayers` method. It accepts one arguments that is an array of layers to be flattened.

![Flatten Layers to Bitmap](./docs/flatten_layers_to_bitmap.png)

The following example flattens all the selected layers to a bitmap layer:
```JavaScript
var flattener = MSLayerFlattener.alloc().init();
flattener.flattenLayers(selection);
```
Complete examples:
- [Flatten Selection to Bitmap.sketchplugin](./Samples/Flatten Selection to Bitmap.sketchplugin)

Works in:
- Sketch 3.2 +