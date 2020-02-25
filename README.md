Sketch Plugins Cookbook
=======================

A collection of recipes for Sketch App plugins developers.

I will be posting daily updates in my twitter. Follow me [@turbobabr](https://twitter.com/turbobabr) to stay tuned.

> NOTE: All the samples are tested against Sketch 51

## #20 Converting SVG Path Commands to Sketch Vector Shapes

![SVG Path Shrug](./docs/svg_path_shrug.png)

In case you want to create a `MSShapeGroup` layer out of set of [SVG Path Commands](https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorial/Paths), there is an easy way to do that by using
[+SVGPathInterpreter.bezierPathFromCommands:(NSString*)commands isPathClosed:(char *)isClosed](https://github.com/abynim/Sketch-Headers/blob/2b88a93f8749d7120a08cbe9d48f0b695d447a08/Headers/SVGPathInterpreter.h#L17) class method. I bet Sketch uses it internally to parse SVG files and convert them to vector shapes... let's give it a shot:
```js
var d = "m 18.015069,991.86506 -11.76393,0.092 -1.25114,0.01 0.0196,2.5023 1.25114,-0.01 9.80545,-0.076 4.63409,18.11514 11.76393,-0.092 1.25114,-0.01 -0.0196,-2.5019 -1.25114,0.01 -9.80545,0.077 -4.63409,-18.11514 z m 63.969859,0 -4.63409,18.11514 -9.80545,-0.077 -1.251136,-0.01 -0.01955,2.5019 1.251136,0.01 11.763935,0.092 4.634091,-18.11514 9.805451,0.076 1.251136,0.01 0.01955,-2.5023 -1.251136,-0.01 -11.763935,-0.092 z m -31.960659,1.3886 -2.38009,0.7725 0.38642,1.1902 c 0.42686,1.3151 0.59157,2.6771 0.74645,4.1278 l 0.13261,1.24404 2.48826,-0.2656 -0.13294,-1.24394 c -0.15653,-1.4663 -0.33069,-3.022 -0.85429,-4.6351 l -0.38642,-1.1899 z m -12.498,0.754 -0.66792,1.0579 c -1.39753,2.2137 -2.20708,5.14784 -2.20708,8.36864 0,3.2207 0.80955,6.1549 2.20708,8.3686 l 0.66792,1.0579 2.11586,-1.3358 -0.66793,-1.058 c -1.09082,-1.7278 -1.82066,-4.2388 -1.82066,-7.0327 0,-2.7939 0.72984,-5.30494 1.82066,-7.03284 l 0.66793,-1.0579 -2.11586,-1.3358 z m 24.94746,0 -2.11586,1.3358 0.66793,1.0579 c 1.09082,1.7279 1.820661,4.23894 1.820661,7.03284 0,2.7939 -0.729841,5.3049 -1.820661,7.0327 l -0.66793,1.058 2.11586,1.3358 0.667921,-1.0579 c 1.397528,-2.2137 2.207082,-5.1479 2.207082,-8.3686 0,-3.2208 -0.809554,-6.15494 -2.207082,-8.36864 l -0.667921,-1.0579 z m -18.45263,0.2681 -2.26703,1.0592 0.52978,1.1336 c 0.58528,1.2526 0.91584,2.5842 1.24788,4.00494 l 0.28476,1.2182 2.43678,-0.5695 -0.28476,-1.21824 c -0.33558,-1.4359 -0.6997,-2.9582 -1.41763,-4.4947 l -0.52978,-1.1335 z m 11.76687,0.1652 -0.23426,1.229 c -0.95694,5.01584 -3.50965,8.89534 -9.03425,11.48734 l -1.13253,0.5314 1.06281,2.2654 1.13286,-0.5314 c 6.15523,-2.8879 9.35583,-7.6584 10.42907,-13.28394 l 0.23426,-1.229 -2.45796,-0.4688 z";

var isPathClosedPtr = MOPointer.alloc().init();
var path = SVGPathInterpreter.bezierPathFromCommands_isPathClosed(d,isPathClosedPtr); // Returns an instance of `NSBezierPath`
print('isClosed: '+isPathClosedPtr.value()); // Indicates whether path is closed or not.

var shape = MSShapeGroup.layerWithPath(MSPath.pathWithBezierPath(path));
var fill = shape.style().addStylePartOfType(0);
fill.color = MSColor.blackColor();

shape.name = "Shrug";
shape.frame().constrainProportions = true;
shape.frame().origin = CGPointZero;
shape.frame().height = 100;

context.document.currentPage().addLayers([shape]);
```

> Note: Pay special attention to the required `isPathClosed` argument, that should be passed as pointer. You can read more about pointers in [#18 Using Obj-C Pointers + Handling Cocoa Errors](https://github.com/turbobabr/Sketch-Plugins-Cookbook#18-using-obj-c-pointers--handling-cocoa-errors) recipe.

## #19 Inserting Emoji as Bitmap Layers

![Emoji](./docs/emoji_circle.png)

Frankly speaking, this recipe doesn't show any new tricks related to Sketch itself. I just want to demonstrate the power of `AppKit/CoreGraphics` frameworks and how they could be used in Sketch plugins.

No more words, here is the code:
```js
function emojiToImage(str,sideSize) {
    var imageWidth = sideSize;
    var imageHeight = sideSize;
    var imageSize = CGSizeMake(imageWidth,imageHeight);

    var drawingContext = CGBitmapContextCreate(null,imageWidth,imageHeight,8,0,CGColorSpaceCreateDeviceRGB(),kCGImageAlphaPremultipliedFirst);

    var graphicsContext = NSGraphicsContext.graphicsContextWithCGContext_flipped(drawingContext,false);
    NSGraphicsContext.saveGraphicsState()
    NSGraphicsContext.setCurrentContext(graphicsContext);

    var targetRect = CGRectMake(0,0,imageWidth,imageHeight);

    var attrs = {};
    attrs[NSFontAttributeName] = NSFont.systemFontOfSize(sideSize);
    str.drawInRect_withAttributes(targetRect,attrs);

    NSGraphicsContext.restoreGraphicsState()

    var cgImage = CGBitmapContextCreateImage(drawingContext);
    return NSImage.alloc().initWithCGImage_size(cgImage,imageSize);
}

var emoji = ['😀', '😁', '😂', '🤣', '😃', '😄', '😅', '😆', '😉', '😊', '😋', '😎', '😍', '😘', '😗', '😙', '😚', '☺', '🙂', '🤗', '🤔', '😐', '😑', '😶', '🙄', '😏', '😣', '😥', '😮', '🤐', '😯', '😪', '😫', '😴', '😌', '🤓', '😛', '😜', '😝', '🤤', '😒', '😓', '😔', '😕', '🙃', '🤑', '😲', '☹', '🙁', '😖', '😞', '😟', '😤', '😢', '😭', '😦', '😧', '😨', '😩', '😬', '😰', '😱', '😳', '😵', '😡', '😠', '😇', '🤠', '🤡', '🤥', '😷', '🤒', '🤕', '🤢', '🤧', '😈', '👿', '👹', '👺', '💀', '☠', '👻', '👽', '👾', '🤖', '💩', '😺', '😸', '😹', '😻', '😼', '😽', '🙀', '😿', '😾', '🙈', '🙉', '🙊', '👦', '👦🏻', '👦🏼', '👦🏽', '👦🏾', '👦🏿'];

function randomPointAtCircle(centerX,centerY,radius) {
    var angle = Math.random()*Math.PI*2;
    return CGPointMake(centerX+Math.cos(angle)*radius,centerY+Math.sin(angle)*radius);
}

function layerWithImage(image) {
    var layer = MSBitmapLayer.alloc().init();
    layer.rect = CGRectMake(0,0,128,128);
    layer.image = MSImageData.alloc().initWithImage(image);
    layer.frame().mid = randomPointAtCircle(400,400,400)
    layer.multiplyBy(0.2 + 0.8*Math.random());
    layer.makeRectIntegral();

    return layer;
}

for(var i=0;i<emoji.length;i++) {
    var str = NSString.stringWithString(emoji[i]));
    var layer = layerWithImage(emojiToImage(str,128));

    context.document.currentPage().addLayers([layer]);
}
```

Credits:
- Idea and original code by Daniel Jalkut ([Evergreen Images](http://indiestack.com/2017/06/evergreen-images/))
- List of all emoji for JavaScript by @theraot ([emoji](https://github.com/theraot/emoji/blob/master/emoji.js))

## #18 Using Obj-C Pointers + Handling Cocoa Errors

Many AppKit and Sketch internal APIs have methods and functions that require passing so called `pointer to pointer` arguments. Whenever you see argument type that looks like `(id *)`, `NSError **`, `NSError * _Nullable *`, etc in any Objective-C API - it means that it wants pointer that points to a pointer thing :). It came to Objective-C from [C Programming Language](https://en.wikipedia.org/wiki/C_(programming_language)), since Objective-C is a layer build atop of `C Language`. You can read more about that pointer to pointer concept here - [TurorialsPoint: C - Pointer to Pointer](https://www.tutorialspoint.com/cprogramming/c_pointer_to_pointer.htm). Also, there's another case when we need to pass pointers to C value variables, e.g. `(char*)` to let method populate the value of that variable during it's execution.

Too bad that JavaScript language and engines don't support such thing as pointer at all, but we need them! Good news everyone! Sketch uses [Mocha Framework](https://github.com/ccgus/CocoaScript/tree/master/src/framework/mocha) for bridging Objective-C objects to JavaScript context and this framework has a special class to deal with pointers called [MOPointer](https://github.com/ccgus/CocoaScript/blob/master/src/framework/mocha/Objects/MOPointer.h).

#### Using MOPointer class to call methods requiring pointer as one of arguments

Suppose we want to convert [SVG Path Commands](https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorial/Paths) to `NSBezierPath` for further use in our plugin. Sketch has a handy class named [SVGPathInterpreter](https://github.com/abynim/Sketch-Headers/blob/2b88a93f8749d7120a08cbe9d48f0b695d447a08/Headers/SVGPathInterpreter.h) and a super useful [+SVGPathInterpreter.bezierPathFromCommands:(NSString*)commands isPathClosed:(char *)isClosed](https://github.com/abynim/Sketch-Headers/blob/2b88a93f8749d7120a08cbe9d48f0b695d447a08/Headers/SVGPathInterpreter.h#L17) class method that takes a set of svg path commands as string and returns and instance of `NSBezierPath` representing the path defined by these commands.

But, there is a second argument named `isPathClosed` of `(char *)` type which is a pointer to a char(basically a single byte):

![SVGPathInterpreter](./docs/svg_interpreter_header.png)

Let's try to pass `null` value just to call the method, we don't really care about whether path is closed or not, we just need to convert it to `NSBezierPath` instance:
```js
var path = SVGPathInterpreter.bezierPathFromCommands_isPathClosed("M10 80 Q 95 10 180 80",null);
// -> Sketch crashes 💥💥💥
```

Another attempt... let's feed an arbitrary variable to the method:
```js
var isClosed;
var path = SVGPathInterpreter.bezierPathFromCommands_isPathClosed("M10 80 Q 95 10 180 80",isClosed);
// -> Sketch crashes 💥💥💥
```

The correct way to call this method is to create `MOPointer` class instance and feed it to the method:
```js
function svgPathToBezierPath(svgPath) {
    var isClosedPtr = MOPointer.alloc().init();
    var path = SVGPathInterpreter.bezierPathFromCommands_isPathClosed(svgPath,isClosedPtr);

    return {
        path: path,
        isClosed: isClosedPtr.value()
    };
}

print(svgPathToBezierPath("M10 10 H 90 V 90 H 10 Z").isClosed);
// -> 1

print(svgPathToBezierPath("M10 10 H 90 V 90 H 10").isClosed);
// -> 0

// No crashes! 😅
```

#### Dealing with NSErrors and handling errors in AppKit APIs

There are dozens of APIs both in Sketch and AppKit that throws various errors, usually errors handling is optional and we can omit it. For example, let's take [+NSString.stringWithContentsOfFile:encoding:error:](stringWithContentsOfFile:encoding:error:) class method that can read text file to a string. In case this method fails to read file, it returns `null` as a result and indicates that something went wrong.

Just run the following code in custom script editor without any changes:
```js
var path = "/some/non/existing/file/path.txt"
var str = NSString.stringWithContentsOfFile_encoding_error(path, NSUTF8StringEncoding, null);
print(str);
// -> (null)
```

As you can see, it fails because the provided file isn't exist in your file system. It's not a big deal, since we receive `null` as a result and can check it making an assumption that `null` means that file isn't exist, but in reality the method might fail for a number of reasons like insufficient permissions, wrong encoding, etc.

In order to check for exact reason why it fails, we have to use `MOPointer` class again:
```js
var path = "/some/non/existing/file/path.txt"
var errorPtr = MOPointer.alloc().init()

var str = NSString.stringWithContentsOfFile_encoding_error(path, NSUTF8StringEncoding, errorPtr);
print(str);
print(errorPtr.value())
```

As you can see, by calling `errorPtr.value()`, we get an instance of [NSError](https://developer.apple.com/documentation/foundation/nserror?language=objc) class describing the real reason why method failed:
> Error Domain=NSCocoaErrorDomain Code=260 "The file “path.txt” couldn’t be opened because there is no such file." UserInfo={NSFilePath=/some/non/existing/file/path.txt, NSUnderlyingError=0x60800204c600 {Error Domain=NSPOSIXErrorDomain Code=2 "No such file or directory"}}

Handling errors correctly might help a lot during development phase as well as make your plugin more stable. I strongly recommend to check out [Dealing with Errors](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/ErrorHandling/ErrorHandling.html) and [Error Handling Programming Guide](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ErrorHandlingCocoa/ErrorHandling/ErrorHandling.html#//apple_ref/doc/uid/TP40001806) docs from Apple to get familiar with the way errors work and handled in Cocoa apps.


## #17 Working with files
Author: [@pravdomil](https://pravdomil.com/#sketch)

Save string to file.

```js
function readFile(path) {
    return NSString.stringWithContentsOfFile_encoding_error(path, NSUTF8StringEncoding, null);
}

function writeFile(path, content) {
    const string = NSString.stringWithFormat("%@", content);
    return string.writeToFile_atomically(path, true);
}
```
More can be found in [sskyy/blade](https://github.com/sskyy/blade/blob/master/src/plugin/common.js) repo.

## #16 Using Image Fills

Sketch supports a bunch of various modes to fill style based layers(`MSShapeGroup`, `MSTextLayer`, etc) with images. Currently, we can use for modes that illustrated below:

![Image Pattern Fill](./docs/fill_pattern_type_results.png)

To get started, we have to grab a couple of useful definitions of Sketch constants from [sketch-constants](https://github.com/turbobabr/sketch-constants/blob/master/src/index.js) repo. They will be used in all the examples below:
```JavaScript
const FillType = {
    Solid: 0,
    Gradient: 1,
    Pattern: 4,
    Noise: 5
};

const PatternFillType = {
    Tile: 0,
    Fill: 1,
    Stretch: 2,
    Fit: 3
};

```

#### Using local image files

The following sample code demonstrates how to load image from the local file system and set it as a pattern fill for selected layer:
```JavaScript
const FillType = { Solid: 0, Gradient: 1, Pattern: 4, Noise: 5 };
const PatternFillType = { Tile: 0, Fill: 1, Stretch: 2, Fit: 3};

function loadLocalImage(filePath) {
    if(!NSFileManager.defaultManager().fileExistsAtPath(filePath)) {
        return null;
    }

    return NSImage.alloc().initWithContentsOfFile(filePath);
}

var layer = context.selection.firstObject();
if(layer && layer.style().firstEnabledFill()) {
   const fill = layer.style().firstEnabledFill()
    var image = loadLocalImage("/Applications/Sketch.app/Contents/Resources/app.icns");
    if(image) {
        fill.fillType = FillType.Pattern;
        fill.patternFillType = PatternFillType.Fill;
        fill.image = MSImageData.alloc().initWithImage(image);

    } else {
        context.document.showMessage("[erorr]: Can't load image!");
    }
} else {
    context.document.showMessage("[error]: Select a layer that has at least one fill style");
}
```

#### Using remote image files

In order to use remote image file we have to download it first and convert to an instance of `NSImage` first using a helper function:
```JavaScript
const FillType = { Solid: 0, Gradient: 1, Pattern: 4, Noise: 5 };
const PatternFillType = { Tile: 0, Fill: 1, Stretch: 2, Fit: 3};

function fetchImage(url, ignoreCache) {
    var request = ignoreCache ?NSURLRequest.requestWithURL_cachePolicy_timeoutInterval(NSURL.URLWithString(url),NSURLRequestReloadIgnoringLocalCacheData,60) : NSURLRequest.requestWithURL(NSURL.URLWithString(url));
    var responsePtr = MOPointer.alloc().init();
    var errorPtr = MOPointer.alloc().init();

    var data = NSURLConnection.sendSynchronousRequest_returningResponse_error(request, responsePtr, errorPtr);
    if(errorPtr.value() != null) {
        print(errorPtr.value());
        return null;
    }

    var response = responsePtr.value();
    if(response.statusCode() != 200) {
        return null;
    }

    var mimeType = response.allHeaderFields()["Content-Type"];
    if(!mimeType || !mimeType.hasPrefix("image/")) {
        return null;
    }

    return NSImage.alloc().initWithData(data);
}

var layer = context.selection.firstObject();
if(layer && layer.style().firstEnabledFill()) {
   const fill = layer.style().firstEnabledFill()
    var image = fetchImage("https://s3.amazonaws.com/sketch-plugins-cookbook/jake_the_dog.png");
    if(image) {
        fill.fillType = FillType.Pattern;
        fill.patternFillType = PatternFillType.Fill;
        fill.image = MSImageData.alloc().initWithImage(image);

    } else {
        print("Can't load image!");
    }
} else {
    print("Select a layer that has at least one fill style");
}
```
> Caution: Sketch does not support HTTP requests using plain `http` protocol due to [App Transport Security (ATS)](https://developer.apple.com/library/content/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW33) networking permissions defined in SketchApp. Be sure that you use `https` protocol and all your urls has `https://` perfix.

#### Using tiled fill

There's a special case for `Tile` mode, since it has additional `patternTileScale` property involved to control scaling of the image pattern:
```JavaScript
const FillType = { Solid: 0, Gradient: 1, Pattern: 4, Noise: 5 };
const PatternFillType = { Tile: 0, Fill: 1, Stretch: 2, Fit: 3};

function loadLocalImage(filePath) {
    if(!NSFileManager.defaultManager().fileExistsAtPath(filePath)) {
        return null;
    }

    return NSImage.alloc().initWithContentsOfFile(filePath);
}

var layer = context.selection.firstObject();
if(layer && layer.style().firstEnabledFill()) {
   const fill = layer.style().firstEnabledFill()
    var image = loadLocalImage("/Applications/Sketch.app/Contents/Resources/app.icns");
    if(image) {
        fill.fillType = FillType.Pattern;

        // Setting up tiled fill
        fill.patternFillType = PatternFillType.Tile;
        fill.patternTileScale = 0.1; // This property is just a multiplier of original image size

        fill.image = MSImageData.alloc().initWithImage(image);

    } else {
        context.document.showMessage("[erorr]: Can't load image!");
    }
} else {
    context.document.showMessage("[error]: Select a layer that has at least one fill style");
}
```

As a result of running the sample above, you might get something like this:

![Image Pattern Fill](./docs/fill_pattern_tiled_results.png)


## #15 Working with Constraints

All edge constraints are always anchored to a parent group (e.g, artboard, symbol or layer group) of the layer:

![Controlling properties](./docs/constraints-and-anchors.png)

To control certain constraints, `MSLayer` class has a bunch of get/set properties named `hasFixed[Edge/Size]` that allow to get and update information about constraints for a given layer:

![Controlling properties](./docs/constraining-props.png)

Checking current constraints:
```JavaScript
var layer = context.selection.firstObject();
if(layer) {
    // Edges
    print("left:" + layer.hasFixedLeft())
    print("top:" + layer.hasFixedTop())
    print("right:" + layer.hasFixedRight())
    print("bottom:" + layer.hasFixedBottom())

    // Size
    print("height:" + layer.hasFixedWidth())
    print("width:" + layer.hasFixedHeight())
}
```

Setting all edge constraints to `fixed`:
```JavaScript
var layer = context.selection.firstObject();
if(layer) {
    layer.hasFixedHeight = false;
    layer.hasFixedWidth = false;

    layer.hasFixedLeft = true;
    layer.hasFixedTop = true;
    layer.hasFixedRight = true;
    layer.hasFixedBottom = true;
}
```


## #14 CocoaScript: Don't use '===' operator

At first glance CocoaScript seems to be a just fancy name for JavaScript with some syntactic sugar, but in reality many things work differently and it's better to know them.

![Don't use strict equal operator](./docs/cocoascript_do_not_use_scrict_equal_operator.png)

One of the caveats are `===` and `!==` operators best known as `strict equality` and `strict not equal`. The very brief suggestion about them is:
- **AVOID USING THEM AT ANY COST IN SKETCH PLUGINS!**

To understand the problem, try to run the following script:
```JavaScript
var strA = "hello!";
var strB = @"hello!";

if(strA == strB) {
    print("They are EQUAL!");
} else {
    print("NOT EQUAL!")
}
// -> "They are EQUAL!"

if(strA === strB) {
    print("They are EQUAL!");
} else {
    print("NOT EQUAL!")
}
// -> "NOT EQUAL!"
```

It will produce `true` for `==` operator and `false` for `===`. The string values are equal, both are assigned with `"hello!"` string but their types are different. Now run this script to check their types:
```JavaScript
function typeOf(obj) {
    print(toString.call(obj));
}

var strA = "hello!";
var strB = @"hello!";

typeOf(strA);
// -> [object String]

typeOf(strB);
// -> [object MOBoxedObject]
```

As you can see, variables `strA` & `strB` are of different types. `strA` is a JavaScript string, but `strB` is a mysterious `MOBoxedObject`. The problem is in definition of `strB` - `@"hello!"` is equal to `NSString.stringWithString("hello!")` and it produces boxed instance of NSString class instead of JS string.

When developing Sketch plugins, you usually deal with the data that is produced on `Sketch Runtime` side. And most of the property getters and class methods return boxed Objective-C objects instead of native JS objects.

To demonstrate a real world problem you can easily encounter with: (1) Create a rectangle shape, (2) Select it, (3) Run the following script:
```JavaScript
var layer=selection.firstObject();
if(layer) {
    print(layer.name());
    // -> Rectangle 1

    var isNameEqual = layer.name() === "Rectangle 1";
    print(isNameEqual);
    // -> false
}
```

> Note: The usage of `===` and `!==` isn't forbidden, you can use them whenever you want to, but always pay attention to types of variables you compare. It's especially important when you try to port an existing JavaScript library or framework to CocoaScript. But anyway, I insist to forget strict equal/not equal operators and use '==' and '!=' + manual type check if needed.

## #13 Playing Sounds

Usually sounds bound to commands are annoying and useless, but sometimes they are very helpful when used with care.

![Play Sound](./docs/play_sound.png)

Since Sketch plugins have access to all the APIs of [AppKit Framework](https://developer.apple.com/documentation/appkit?language=objc), we are able to do really crazy & cool things with plugins.. for example play a `beep!` sound when plugin shows an error message using `-MSDocument.showMessage:` method to make the message more noticeable to a user.

Here is how we can play `beep` sound to indicate some sort of error:
```JavaScript
context.document.showMessage("Hey! We have a problem!")
NSBeep()
```

To play a custom audio file we can use a simple interface of [NSSound](https://developer.apple.com/documentation/appkit/nssound?language=objc) class. Here is the sample code how to use it:
```JavaScript
var filePath = "/System/Library/Sounds/Pop.aiff"

var sound = NSSound.alloc().initWithContentsOfFile_byReference(filePath,true);
sound.play();
```



Alternatively, you can play any system sound using a handy [+NSSound.soundNamed:](https://developer.apple.com/documentation/appkit/nssound/1477318-soundnamed?language=objc) class method:
```JavaScript
const SystemSounds = {
  Basso: "Basso",
  Blow: "Blow",
  Bottle: "Bottle",
  Frog: "Frog",
  Funk: "Funk",
  Glass: "Glass",
  Hero: "Hero",
  Morse: "Morse",
  Ping: "Ping",
  Pop: "Pop",
  Purr: "Purr",
  Sosumi: "Sosumi",
  Submarine: "Submarine",
  Tink: "Tink"
};

var sound = NSSound.soundNamed(SystemSounds.Glass)
sound.play()
```

## #12 Centering Rectangle on Canvas

To center canvas on a certain point or region, you can use a handy `-(void)MSContentDrawView.centerRect:(CGRect)rect animated:(BOOL)animated` instance method, where `rect` is a rectangle to be centered, `animated` is a flag that turns on/off animation during the scrolling process.

![Create Custom Shape](./docs/center_rect.png)

The origin and size of the rectangle you provide to this method should be in absolute coordinates.

The following example centers viewport by `x:200,y:200` point:
```JavaScript
var canvasView = context.document.contentDrawView(); // Getting canvas view
canvasView.centerRect_animated(CGRectMake(200,200,1,1),true);
```
The example below shows how to center on the first selected layer using the same method without animation:
```JavaScript
var layer = context.selection.firstObject()
if(layer) {
    var view = context.document.contentDrawView();
    view.centerRect_animated(layer.absoluteRect().rect(),false);
}
```

## #11 Creating Custom Shapes

To create a custom vector shape programmatically, you have to create an instance of [NSBezierPath](https://developer.apple.com/documentation/appkit/nsbezierpath?language=objc) class and draw whatever shape or combination of shapes you want to. Then convert the path to `MSPath` class instance and create a shape group out of it using `+(MSShapeGroup*)MSShapeGroup.layerWithPath:(MSPath*)path` class method.

![Create Custom Shape](./docs/create_custom_shape.png)

This technique is very similar to creation of custom paths described in previous recipe. The only difference is that you have to close the path before converting it to the shape group.

The following example create a simple arrow shape:
```JavaScript
// All coordinates are provided in pixels
var path = NSBezierPath.bezierPath();
path.moveToPoint(NSMakePoint(10,10));
path.lineToPoint(NSMakePoint(100,10));
path.lineToPoint(NSMakePoint(100,0));
path.lineToPoint(NSMakePoint(120,15));
path.lineToPoint(NSMakePoint(100,30));
path.lineToPoint(NSMakePoint(100,20));
path.lineToPoint(NSMakePoint(10,20));
path.closePath();

var shape = MSShapeGroup.layerWithPath(MSPath.pathWithBezierPath(path));
var fill = shape.style().addStylePartOfType(0); // `0` constant indicates that we need a `fill` part to be created
fill.color = MSColor.colorWithRGBADictionary({r: 0.8, g: 0.1, b: 0.1, a: 1});

var documentData = context.document.documentData();
var currentParentGroup = documentData.currentPage().currentArtboard() || documentData.currentPage()
currentParentGroup.addLayers([shape]);
```

## #10 Creating Line Shapes

In order to create a line shape programmatically, you have to create an instance of [NSBezierPath](https://developer.apple.com/documentation/appkit/nsbezierpath?language=objc) class and add two points to it. Then convert the path to `MSPath` class instance and create a shape group out of it using `+(MSShapeGroup*)MSShapeGroup.layerWithPath:(MSPath*)path` class method.

![Create Line Shape](./docs/create_line_shape.png)

To make Sketch recognize the provided path as a line shape, you have to add only two points using `moveToPoint` & `lineToPoint` methods of `NSBezierPath`.

The following example creates a simple line shape with two points:
```JavaScript
var path = NSBezierPath.bezierPath();
path.moveToPoint(NSMakePoint(10,10));
path.lineToPoint(NSMakePoint(200,200));

var shape = MSShapeGroup.layerWithPath(MSPath.pathWithBezierPath(path));
var border = shape.style().addStylePartOfType(1);
border.color = MSColor.colorWithRGBADictionary({r: 0.8, g: 0.1, b: 0.1, a: 1});
border.thickness = 3;

context.document.currentPage().addLayers([shape]);
```

The same way, you can easily create a multi segment line using methods provided by [NSBezierPath](https://developer.apple.com/documentation/appkit/nsbezierpath?language=objc) class. Whenever you add more than two points into the path, Sketch treats such shape as a vector path similar to what can be created using standard `V - Vector` tool.

The following example demonstrates how to create a curved path with four points:
```JavaScript
var path = NSBezierPath.bezierPath();
path.moveToPoint(NSMakePoint(84.5,161));
[path curveToPoint:NSMakePoint(166,79.5) controlPoint1:NSMakePoint(129.5,161) controlPoint2:NSMakePoint(166,124.5)];
[path curveToPoint:NSMakePoint(84.5,-2) controlPoint1:NSMakePoint(166,34.5) controlPoint2:NSMakePoint(129.5,-2)];
[path curveToPoint:NSMakePoint(3,79.5) controlPoint1:NSMakePoint(39.5,-2) controlPoint2:NSMakePoint(3,34.5)];

var shape = MSShapeGroup.layerWithPath(MSPath.pathWithBezierPath(path));
var border = shape.style().addStylePartOfType(1);
border.color = MSColor.colorWithRGBADictionary({r: 0.8, g: 0.1, b: 0.1, a: 1});
border.thickness = 2;

context.document.currentPage().addLayers([shape]);
```

## #9 Setting Border Radius for Specific Corners

Starting from version 3.2 Sketch allows to set custom border radius for specific corner of rectangle shape. It was possible prior to 3.2, but there was no direct API.

![Set Custom Border Radius](./docs/set_custom_border_radius_for_specific_corner.png)

In order to set custom radiuses you use `-MSRectangleShape.setCornerRadiusFromComponents:(NSString*)compoents` instance method, where `components` is a string that represents radius values for every corner separated by `;` character. The sequence is following: `top-left/top-right/bottom-right/bottom-left`.

The following sample sets left-top and right-top corners of a selected rect shape to 15 points:
```JavaScript
var layer = context.selection.firstObject();
if(layer && layer.isKindOfClass(MSShapeGroup)) {
    var shape=layer.layers().firstObject();
    if(shape && shape.isKindOfClass(MSRectangleShape)) {
        shape.setCornerRadiusFromComponents("15;15;0;20");
    }
}
```

## #8 Scaling Layers

You can scale any layer using `-MSLayer.multiplyBy:(double)scaleFactor` instance method, where `scaleFactor` is a floating-point value that is used to multiple all the layers' properties including position, size, and all the style attributes such as border thickness, shadow, etc. Here are some example scale factors: `1.0 = 100%`, `2.5 = 250%`, `0.5 = 50%`, etc.

This method produces the same result as a standard [Scale](http://bohemiancoding.com/sketch/support/documentation/03-layer-basics/4-resizing-layers.html) tool. Since all the layer type classes are inherited from `MSLayer` class, you can use this method to scale any type of layer including Pages and Artboards.

> Note: After the call of the method, `x` and `y` position values will also be multiplied. If you need the layer to remain in the same position after scaling, you'll have to change its position to the appropriate values.

![Finding Selection Bounds](./docs/scale_layers.png)


The following sample demonstrates how to scale first selected layer:
```JavaScript
var layer = context.selection.firstObject()
if(layer) {
    // Preserve layer center point.
    var midX=layer.frame().midX();
    var midY=layer.frame().midY();

    // Scale layer by 200%
    layer.multiplyBy(2.0);

    // Translate frame to the original center point.
    layer.frame().midX = midX;
    layer.frame().midY = midY;
}
```

## #7 Finding Bounds For a Set of Layers

If you want to quickly find a bounding rectangle for selected layers or any set of layers, there is a very handy class method for that `+(CGRect)MSLayerGroup.groupBoundsForContainer:(MSLayerArray*)container`. It accepts an instance of `MSLayerArray` class, that represents a list of layers.

![Scaling Layers](./docs/find_selection_bounds.png)

A quick sample that demonstrate how to use it:
```JavaScript
var selection = context.selection;
var bounds= MSLayerGroup.groupBoundsForContainer(MSLayerArray.arrayWithLayers(selection));

print("x: "+bounds.origin.x);
print("y: "+bounds.origin.y);
print("width: "+bounds.size.width);
print("height: "+bounds.size.height);
```

## #6 Creating Oval Shapes

In order to create an oval shape programmatically, you have to create an instance of `MSOvalShape` class, set its frame and wrap with `MSShapeGroup` container.

![Create Oval Shape](./docs/create_oval_shape.png)

The following sample demonstrates how to do it:
```JavaScript
var ovalShape = MSOvalShape.alloc().init();
ovalShape.frame = MSRect.rectWithRect(NSMakeRect(0,0,100,100));

var shapeGroup=MSShapeGroup.shapeWithPath(ovalShape);
var fill = shapeGroup.style().addStylePartOfType(0);
fill.color = MSColor.colorWithRGBADictionary({r: 0.8, g: 0.1, b: 0.1, a: 1});

context.document.currentPage().addLayers([shapeGroup]);
```


## #5 Creating MSColor instances from CSS color strings (hex, rgba, etc)

There is no way to create instance of `MSColor` model class from CSS string directly, but it's possible to do so via it's immutable counterpart class named `MSImmutableColor`. It has a class method `+MSImmutableColor.colorWithSVGString:(NSString*)string` that accepts any value supported by [CSS Color](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value) data type.

Here is a super handy helper function and a bunch of calls with various supported formats of providing color as string:
```JavaScript
function MSColorFromString(color) {
    return MSImmutableColor.colorWithSVGString(color).newMutableCounterpart()
}

// Hex
MSColorFromString("#33AE15")
MSColorFromString("#333")
MSColorFromString("FF0000")
MSColorFromString("#145515FF")

// rgb/rgba
MSColorFromString("rgb(255,0,0)")
MSColorFromString("rgba(255,0,0,0.5)")

// Color keywords
MSColorFromString("red")
MSColorFromString("blue")
MSColorFromString("magenta")
MSColorFromString("darkviolet")

// hls
MSColorFromString("hsl(270, 60%, 50%, .15)")
MSColorFromString("hsl(270deg, 60%, 70%)")
MSColorFromString("hsl(4.71239rad, 60%, 70%)")
MSColorFromString("hsla(240, 100%, 50%, .4)")
```

The following example demonstrates how to set fill color for a selected layer using `MSColorFromString` helper function:
```JavaScript
function MSColorFromString(color) {
    return MSImmutableColor.colorWithSVGString(color).newMutableCounterpart()
}

var layer = context.selection.firstObject();
if(layer) {
    layer.style().firstEnabledFill().color = MSColorFromString("#FA1010");
}
```

## #4 Flattening Complex Vector Layers

If you want to flatten a complex vector layer that contains several sub paths combined using different boolean operations into single layer, you can use `+MSShapeGroup.flatten` method.

![Flatten Vector Shape](./docs/flatten_vector_shape.png)

This sample code flattens a first selected vector layer:
```JavaScript
var layer = context.selection.firstObject();
if(layer && layer.isKindOfClass(MSShapeGroup)) {
    layer.flatten();
}
```

## #3 Flattening Layers to Bitmap

In order to flatten one or several layers of any type to a single `MSBitmapLayer`, use `-MSLayerFlattener.flattenLayers:` method. It accepts one arguments which is a container of layers to be flattened.

![Flatten Layers to Bitmap](./docs/flatten_layers_to_bitmap.png)

The following example flattens all the selected layers to a bitmap layer:
```JavaScript
var flattener = MSLayerFlattener.alloc().init();
flattener.flattenLayers(MSLayerArray.arrayWithLayers(context.selection));
```

## #2 Converting Text Layer to Vector

In order to convert an existing `MSTextLayer` to `MSShapeGroup` layer, you have to get texts' `NSBezierPath` representation and then convert it to a `MSShapeGroup` layer.

![Convert Text Layer to Outlines](./docs/convert_text_layer_to_outlines.png)

The following source code demonstrates how to get text layers' vector outline and use it to create a vector shape from it:
```JavaScript
function convertToOutlines(layer) {
    if(!layer.isKindOfClass(MSTextLayer)) return;

    var parent = layer.parentGroup();
    var shape = MSShapeGroup.layerWithPath(layer.pathInFrameWithTransforms());

    var style = shape.style();
    var fill = style.addStylePartOfType(0);
    fill.color = MSColor.colorWithRGBADictionary(layer.textColor().RGBADictionary());

    shape.name = layer.name();

    parent.removeLayer(layer);
    parent.addLayers([shape]);

    return shape;
}

var layer= context.selection.firstObject();
if(layer) {
    var vectorizedTextLayer = convertToOutlines(layer);
    if(!vectorizedTextLayer) {
        context.document.showMessage("Select text layer to convert");
    } else {
        vectorizedTextLayer.select_byExpandingSelection(true,false)
        print(vectorizedTextLayer);
    }
}
```

## #1 Getting Points Coordinates Along a Shape Path

If you want to distribute some shapes along a path there is a convenient method `-pointOnPathAtLength:` implemented in `NSBezierPath_Slopes` class extension.

This method accepts a `double` value that represents a position on path at which you want to get a point coordinate, where `0.0` value means start of the path and `1.0` end of the path. It returns a `CGPoint` struct with coordinates of the point.

![Ge points coords along shape path](./docs/getting_points_along_path.png)

The following example divides shape path into 15 segments and prints out their points coordinates:
```JavaScript
var layer = context.selection.firstObject();
if(layer && layer.isKindOfClass(MSShapeGroup)) {
    var count = 15;
    var path = NSBezierPath.bezierPathWithCGPath(layer.pathInFrameWithTransforms().CGPath());

    var step = path.length()/count;
    for(var i=0;i<=count;i++) {
        var point = path.pointOnPathAtLength(step*i);
        print(point);
    }
}
```
