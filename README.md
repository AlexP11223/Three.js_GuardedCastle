*A simple scene implemented with Three.js several years ago during Computer Graphics course in university following J. Dirksen book "Learning Three.js". Uses old version of Three.js and other libs, I don't know if it is compatible with modern versions.*

It is the castle from [Island Castle](https://github.com/AlexP11223/Three.js_IslandCastle) but with textures and some guards: archers on the walls and knights patrolling around. It is also possible to add new knights, they will walk through the gate (not forgetting to open it) and will start patrolling too.

![](https://i.imgur.com/IOp6pNa.png)

![](https://i.imgur.com/WA14kGQ.png)

![](https://i.imgur.com/ZLCJ63T.png)

Requires HTTP server to serve the files, cannot be run by just opening local HTML file (without changing security settings in most web browsers). At the time I was using [Mongose](https://cesanta.com/binary.html) web server suggested in the book: it is free for personal use and very easy to use (on Windows just put .exe in the directory with files, run the .exe and it will open a web page).

Can be run here: [https://alexp11223.github.io/Three.js_GuardedCastle/index.html](https://alexp11223.github.io/Three.js_GuardedCastle/index.html)

# Implementation details

## World

All textures and models used here are from [http://opengameart.org](http://opengameart.org).

The ground is a big plane with grass texture:

```javascript
function createGround() {
    var size = 2000;

    var groundGeometry = new THREE.PlaneGeometry(size, size);
    var groundMaterial = new THREE.MeshLambertMaterial({
            map: createRepeatingTexture("assets/grass.jpg", 5, 5)
    });
    var ground = new THREE.Mesh(groundGeometry, groundMaterial);
    ground.receiveShadow  = true;
    ground.rotation.x = -0.5 * Math.PI;

    return ground;
}
```

Fog is added at its borders:

```javascript
scene.fog = new THREE.Fog(0xcccccc, 400, 900);
 ```
 
3 cameras are available to look at the scene from different positions (as well as change it using `TrackballControls`). At first I was thinking about creating 3 camera objects and `activeCamera` variable that will be passed to the render method, but it did not work well with `TrackballControls`, so I ended up simply changing the position on buttons click like this:

```javascript
gui.add(new function() {
    this.camera2 = function() {
        cameraControls.reset();

        camera.position.x = -90;
        camera.position.y = 80;
        camera.position.z = -50;

        cameraControls.target = castle.getObjectByName("gateBuilding").position.clone();
    }
},'camera2');
```

## Castle

The castle structure is mostly the same as in my [Island Castle](https://github.com/AlexP11223/Three.js_IslandCastle), but all materials (except windows) use textures. Most of the textures (walls, floor, ...) are set to `RepeatWrapping`.

```javascript
function createRepeatingTexture(fileName, repeatX, repeatY) {
    var texture = THREE.ImageUtils.loadTexture(fileName);
    texture.wrapS = texture.wrapT = THREE.RepeatWrapping;
    texture.repeat.set(repeatX, repeatY);

    return texture;
}
var wallTextureName = "assets/wall.jpg";
var roofTextureName = "assets/roof.jpg";
var gateTextureName = "assets/gate.png";
var doorTextureName = "assets/door.png";
var floorTextureName = "assets/floor.jpg";
var roadTextureName = "assets/road.png";

var wallMaterial = new THREE.MeshLambertMaterial({map: createRepeatingTexture(wallTextureName, 4, 0.8)});

var floorMaterial = new THREE.MeshLambertMaterial({map: createRepeatingTexture(floorTextureName, 4, 0.6)});

var battlementTexture = createRepeatingTexture(wallTextureName, 0.22, 0.33);
battlementTexture.offset.x = 0.1;

var battlementMaterial = new THREE.MeshLambertMaterial({map: battlementTexture});

var towerWallMaterial = new THREE.MeshLambertMaterial({map: createRepeatingTexture(wallTextureName, 6, 1.5)});

var roofMaterial = new THREE.MeshLambertMaterial({map: createRepeatingTexture(roofTextureName, 8, 1.5)});

var gateBuildingWallMaterial = new THREE.MeshLambertMaterial({map: createRepeatingTexture(wallTextureName, 1, 1.5)});
var gateMaterial = new THREE.MeshLambertMaterial({map: THREE.ImageUtils.loadTexture(gateTextureName), side: THREE.DoubleSide});

var buildingWallMaterial = new THREE.MeshLambertMaterial({map: createRepeatingTexture(wallTextureName, 2, 1.3)});
var doorMaterial = new THREE.MeshLambertMaterial({map: THREE.ImageUtils.loadTexture(doorTextureName), transparent: true});
var buildingRoofMaterial = new THREE.MeshLambertMaterial({map: createRepeatingTexture(roofTextureName, 4, 2), side: THREE.DoubleSide});

var innerRoadMaterial = new THREE.MeshLambertMaterial({map: createRepeatingTexture(floorTextureName, 2, 0.8)});

var roadMaterial = new THREE.MeshLambertMaterial({map: createRepeatingTexture(roadTextureName, 15, 1)});
```

In the previous work I implemented the upper part of the building as 2D triangles using `THREE.Geometry` with manually added vertices and faces. To make it work with texture we need to calculate UV coordinates. For this I have found a function ([https://stackoverflow.com/questions/20774648/three-js-generate-uv-coordinate](https://stackoverflow.com/questions/20774648/three-js-generate-uv-coordinate)) that can generate it automatically for planar surfaces.

The gate building now has gate only on one side and the space from the other side is subtracted (using `ThreeCSG` library).

```javascript
var gateBuilding = new THREE.Mesh(new THREE.BoxGeometry(gateBuildingWidth, gateBuildingHeight, gateBuildingDepth), gateBuildingWallMaterial);

gateBuilding.position.y = gateBuildingHeight / 2;

// create gate (2D plane with texture)

var gateWidth = 24;
var gateHeight = 36;

var gateGeometry = new THREE.PlaneGeometry(gateWidth, gateHeight);

var gate = new THREE.Mesh(gateGeometry, gateMaterial);
gate.receiveShadow = true;

gate.position.set(0, -2, gateBuildingDepth / 2);

gate.name = "gate";

// create roof

var roof = new THREE.Mesh(new THREE.CylinderGeometry(0, 25, 16, 4), roofMaterial);
roof.castShadow = true;

roof.rotation.y = 0.25*Math.PI;
roof.position.y = gateBuildingHeight / 2 + 8;

// extract space for the gate from the building

var gateMesh = new THREE.Mesh(new THREE.BoxGeometry(gateWidth, gateHeight, gateBuildingDepth), new THREE.MeshLambertMaterial());
gateMesh.position.y = gateHeight / 2;

var subtractedBsp = new ThreeBSP(gateBuilding).subtract(new ThreeBSP(gateMesh));

gateBuilding = subtractedBsp.toMesh(gateBuildingWallMaterial);
gateBuilding.geometry.computeVertexNormals();
gateBuilding.castShadow = true;
gateBuilding.receiveShadow = true;

gateBuilding.add(gate);

gateBuilding.add(roof);
```

![](https://i.imgur.com/iKAGEhc.png)

Because of that the front wall is split into two parts.

```javascript
function createWall(wallWidth, wallDepth, withoutTower) {
    ...
    if (!withoutTower) {
        // add tower to the right end of the wall    }
}

var frontWallLeftPart = createWall((wallWidth - gateSize) / 2, wallDepth, true);
var frontWallRightPart = createWall((wallWidth - gateSize) / 2, wallDepth); 

frontWallLeftPart.position.x = -(wallWidth - gateSize) / 2 / 2;
castle.add(frontWallLeftPart);

frontWallRightPart.position.x = (wallWidth - gateSize) / 2 / 2;
castle.add(frontWallRightPart);
The castle also contains a path from the building to the gate and a road with intersection outside:
function createRoad(length, width, material) {
    var road = new THREE.Mesh(new THREE.PlaneGeometry(length, width), material);
    road.receiveShadow = true;
    road.rotation.x = -0.5 * Math.PI;
    road.position.y = 0.2;
    return road;
}
// add road inside the castle

var innerRoad = createRoad(wallWidth / 2, 25, innerRoadMaterial);
innerRoad.rotateZ(0.5 * Math.PI);
innerRoad.position.z = wallWidth / 2 / 2 + gateSize / 2;
castle.add(innerRoad);

// add roads with intersection outside

var outerRoad = createRoad(850, 35, roadMaterial);
outerRoad.rotateZ(0.5 * Math.PI);
outerRoad.position.z = wallWidth / 2 + gateSize / 2 + 425;
castle.add(outerRoad);

var outerRoad2 = createRoad(1900, 55, roadMaterial);
outerRoad2.position.z = 600;
castle.add(outerRoad2);
```

## Guards

In order to be able to export Blender model to JSON I updated `Three.js` to the latest version at that time (r71) because the old exporter (`io_three_mesh`) was replaced with new `io_three` and the old one was not included in the Three.js distribution provided with the book examples (and it was hard to find where it can be downloaded, besides I was not sure if it works with the current Blender version).
