
#ArcGIS API for Javascript

Justin Colville

---

## Javascript
 - [3.x Developer's Site](https://developers.arcgis.com/javascript/)
 - [4.0 Beta Developer's Site](https://developers.arcgis.com/javascript/beta/)

---

## Visualization Examples
 - ...

---

## 3.x
 - [Geometry Engine](http://developers.arcgis.com/javascript/samples/ge_geodesic_buffers/)
 - [Smart Mapping](http://developers.arcgis.com/javascript/samples/smartmapping_bycolor/)
 - [Image Server](http://developers.arcgis.com/javascript/samples/layers_imageservicevector/)
 - [Quantization](demos/quantization/PIXELATE_ALL_THE_POLYGONS.html) vs. [Generalization](demos/quantization/TRIANGULATE_ALL_THE_POLYGONS.html)

---

## 4.0 - highlights
 - Promises-based
 - 2D/3D
 - 4.0beta2
 - Additional betas coming
 - API 4.0: new concepts & changes
 - IE9+ for 2D, IE11+ for 3D

---

## Promises

 - 4.0 is a promises-based architecture!
 
 - The basic pattern looks like this:

  ```javascript
  someAsyncFunction.then(callback, errback);  
  ```
  
 - And, it's implemented like this:

  ```javascript
  someAsyncFunction().then(function(resolvedVal){
    //This is called when the promise resolves
    console.log(resolvedVal);  //logs the value the promise resolves to
  }, function(error){
    //This function is called when the promise is rejected
    console.error(error);  //logs the error message
  });
  ```

---

## 2D/3D
 - Starting point of 4.0: 3D is coming!
 - currently in [3.x](http://developers.arcgis.com/javascript/samples/map_simple/):
   - Map, many DOM nodes
   - Each Layer, 1 DOM Node
 - Can't work, WebGL renders in one Canvas
 - Solution?

---

## 2D/3D
 - Separate the business logic from the drawing logic.
![New model: Map/Layers + View(s)/LayerViews](images/architecture.png)
 - Communication model by __events__ and __properties watching__
   - clean decoupling
   - clearer about what's going on when something changes

---

## 2D/3D
 - For the rest, one API
 - [demo](demos/visualization/epic-citadel.html)

---

## 2D
 - new "engine" in the work.
 - faster, more future proof
   - abstraction to draw tiles and dynamic images to ease custom layers/layerviews
   - abstraction to draw in DOM or Canvas, possibly webgl ;-)
 - display graphics while zooming.
 - rotation
 - continous zoom
 - [vector map tiles](http://blogs.esri.com/esri/arcgis/2015/07/20/vector-tiles-preview/), [basemaps](http://basemapsbeta.arcgis.com/preview/app/index.html)

---

## 3D
 - webgl engine to display the earth.
 - [z/m support](http://maps.esri.com/rc/sat/index.html) in the API, tasks, layers...
 - support for simple symbols
 - new 3D Symbols

---

Experiment - Map running in node
![Map running in node](images/map-node.png)

---

## 3D

```javascript
// create the map and its layers
var map = new Map({
  basemap: "topo"
});

map.add(new FeatureLayer(...));

// create a 3D view for the Map
var view = new SceneView({
  map: map,
  container: "viewDiv"
});
```

---

## `esri/core/Accessor`
 - Mixin similar to `dojo/Stateful`
 - Consistent pattern for `get()`, `set()`, `watch()`
 - Single object constructor
 - Support for ES7 `Object.observe()`

---

## Properties watching

 - Direct benefits:
   - remove inconsistancies between constructor, getter, setter functions, events
   - one convention everywhere. _"just need to know what properties for a class"_
   - Single object constructor, no more 3+ constructors
   - Leaner SDK: we doc only the properties, the rest is convention

 - Changes:
   - no more **_property_**-change events, use `watch()`
   - in 3.x, listen for `extent-change` event.
   - in 4.0 `extent` watchers will be call very often
   - new events and properties for animation. 

---


## Properties watching


```javascript

	featureLayer = new featureLayer({url});

	map.add(featureLayer);

	featureLayer.watch("loaded", function(newValue, oldValue, property, object) {
		if (property === "loaded") {
			//Do something
		}
	});


```

---

## Properties watching

 - Frameworks integration
   - properties are framework agnostic
   - better/easier integration

 - Examples
   - [side by side views](demos/accessor/side-by-side.html)
   - [dbind](demos/integration/dbind.html)
   - [React](http://jsbin.com/togemadodo/1/edit?js,output)
   - [camera recorder](http://output.jsbin.com/donujo)

---

## Layers

 - `map.layers`, a collection of the operational layers
   - mix of image AND graphics
 - Shorter names: `ArcGISTiledLayer`, `ArcGISDynamicLayer`
 - new ones:
   - `ArcGISElevationLayer`
   - `SceneLayer`
   - `GroupLayer`

---

## GroupLayer

 - New layer: GroupLayer
 - group layers together
 - structure your data visualization     
 - visibility mode: `exclusive`, `independent`, `inherit`
 - listMode: `hide-children`, `hidden`
 - [demo](demos/grouplayer/index.html)  

---

## GroupLayer

  ```javascript
    map = new Map({
      layers: [
        new ArcGISTiledLayer({
          title: 'Dark Gray Canvas',
          url: '//services.arcgisonline.com/arcgis/rest/services/Canvas/World_Dark_Gray_Base/MapServer',
          //listMode: 'hide',
        }),
    
        new GroupLayer({
          title: 'USA Tiled Services',
          visibilityMode: 'exclusive',
          //listMode: 'hide-children',
          layers: [
            new ArcGISTiledLayer({
              url: '//server.arcgisonline.com/ArcGIS/rest/services/Demographics/USA_Median_Household_Income/MapServer',
              title: 'Median Household Income',
              visible: false
            }),
            new ArcGISTiledLayer({
              "url": '//services.arcgisonline.com/ArcGIS/rest/services/Demographics/USA_Tapestry/MapServer',
              "title": "Tapestry Segmentation",
              visible: true
            }),
            new ArcGISTiledLayer({
              url: '//server.arcgisonline.com/ArcGIS/rest/services/Demographics/USA_Population_Density/MapServer',
              title: 'Population Density',
              visible: false
            })
          ]
        })
      ]
    });
  ``` 

---

## Collection

 - More or less like an Array
 - `add` / `remove` / `forEach` / `map` / `find` / `findIndex`...
 - emit "change" events when something is added/removed/moved
 - used for layers, used for layers in Basemap, used for graphics...

---

## Basemap

- full fledge class `esri/Basemap`
- basemap's layers are _not_ part of the `map.layers`, but from `map.basemap`
- contains 3 Collections: baseLayers, referenceLayers, elevationLayers
- can be set with
  - [string for esri's basemap](demos/basemap/2d.html)
  - or custom [Basemap instance](demos/basemap/2d-custom.html)
  - in 2D and [3D](demos/basemap/3d.html)

---

## Basemap

 - `basemap` as a string, creation of the appropriated Basemap instance

  ```javascript
  var map = new Map({
    basemap: 'topo'
  });

  map.basemap = 'streets';
  ```

 - `basemap` as an instance of `Basemap`

  ```javascript
  var map = new Map({/*...*/});

  var toner = new Basemap({
    baseLayers: [
      new WebTiledLayer({
        urlTemplate: '...'
      })
    ]
  })

  map.basemap = toner;
  ```

---

## Resizing logic
 - automatically measure and position the view
 - [resize by center, or not](demos/resizing/manual-resize.html)
 - better integration with responsive design pages
 - and [frameworks](demos/resizing/responsive-bootstrap.html)

---

## Padding
 - easier fullscreen view application.
 - defines inner margin to make space for UI.
 - [2D](demos/padding/2d.html)
 - [3D](demos/padding/3d.html)

---

## Animation
 - generic function `animateTo(target, options):Promise`
 - customize [easing, duration, chaining](demos/animation/random.html)
 - DIY using [other libs](demos/animation/tweenjs.html)
 - `esri/Viewpoint`: common way to share between 2D/3D

---

## Widgets
 - `ui` property on view to quickly place components
 - widgets designed as MVVM
   - separates the logic from the UI implementation
   - easier to create new versions using other frameworks
 - ported to 4.0beta1: Search, Zoom, Attribution
 - new ones: Compass

---

## WebMap & WebScene APIs
 - read
 - save / save as
 - easier portal / arcgis.com interaction

---

## SDK
 - new SDK, built from scratch
 - simpler, focused samples
 - user experience
 - more code snippets

---

## Other
 - legacy dojo loader removed - AMD only
 - classes properly cased: esri/Map, esri/Graphic, esri/layers/Layer
 - new folder structure.

---

## Beta3 ????
 - ?????????????


---

<!-- .slide: data-background="images/atmosphere.png" -->

---

<!-- .slide: data-background="images/atmosphere-realistic.png" -->

---

<!-- .slide: data-background="images/subsurface-data.png" -->

---

<!-- .slide: data-background="images/subsurface-data-wireframe.png" -->

---

## Conclusion
 - One API
 - Promises, promises
 - 3D, and better 2D
 - simplified and consistent API

---


## Questions


--- 

# 
