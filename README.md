Sketch-Plugins-Cookbook
=======================

A collection of recipes for Sketch App plugins developers

## Flatten Vector Layer

If you want to flatten a complex vector layer that contains several sub paths combined using different boolean operation into single layer, you can use `MSShapeGroup.flatten` method.

![Flatten Vector Shape](./docs/flatten_vector_shape.png)

```JavaScript
var layer=selection.firstObject();
if(layer && layer.isKindOfClass(MSShapeGroup)) {
    layer.flatten();
}
```
