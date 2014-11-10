Sketch-Plugins-Cookbook
=======================

A collection of recipes for Sketch App plugins developers

## Flatten Vector Layer

If you want to flatten a complex vector layer that contains several sub paths combined using different boolean operation into single layer, you can use `MSShapeGroup.flatten` method.

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

Available in:
- Sketch 3.2 +
