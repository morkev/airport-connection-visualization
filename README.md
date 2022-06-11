# Airport Connection Visualization

```python
Pseudoalgorithm:
- Import data (airport :: instances and connections).
– Convert from Lat/Lon to Cartesian (X/Y/Z).
– Code the flight routes and connect the IDs.
– Project the routes on a sphere.
– Create curvature for the paths between nodes.
```

## Cartesian System of Coordinates
The position is stored as latitute and longitute. Because the built-in functions that allow us to translate latitute and longitude values did not yield correct results with our ID parameters (integers stored in columns 3 and 5 from <b><i>airports.dat</i></b>), we decided to write our own program to translate these two parameters into a cartesian system of coordinates (X, Y, Z).

Create a `pointwrangle node` with following code:

```python
vector spherToCart(float lat, lon, rad){
	return rad * set(
		-cos(lat) * cos(lon),
		sin(lat),
		cos(lat) * sin(lon)
	);
}

v@P = spherToCart(
	radians(@lat),
	radians(@lon),
	1.0
);

v@N = normalize(@P);
```

## Building Flight Routes
Add a second `pointwrangle node`, and connect it to the route data. The output from the first `pointwrangle node` should point to the second input slot of `pointwrangle-2 node`. We essentially want to search for airport IDs and trace the path between two airport instances.

```python
int src = findattribval(1, "point", "id", i@src);
int dst = findattribval(1, "point", "id", i@dst);

if (src >= 0 && dst >= 0) {
  int newprim = addprim(0, "polyline");
	setprimgroup (0, "new", newprim, 1) ;
	vector v1 = point (1, "p", src);
	vector v2 = point (1, "p", dst):
	int p1 = addpoint (0, v1);
	int p2 = addpoint (0, v2) :
	float distance = distance (v1, v2);
	setpointattrib(0, "dist", p1, distance) ;
	setpointattrib(0, "dist", p2, distance) ;
	addvertex(0, newprim, p1);
	addvertex(0, newprim, p2);
}
```

## Solving a Few Challenges
At this point, you will notice a couple sets of 3 or more red edges – we don't need this data. In order to fix these red lines, we must connect the output the last node (i.e., `pointwrangle-2`) to a new `blast node` of <i>Primitive Group</i> type.

Finally, all our connections are being represented by a straight line, and it is hard to distinguish any path. First, make sure you have the `Curve U Attribute` enabled in the `resample node`. To create a curvature between the path from the airport source and the airport destination, we will use the following `VEXpression`: 

```python
float scale = chf("Lift_Scale");
float lift = chramp("Lift_Amount", @curveu);
float width = chramp("Width", @curveu);

vector liftpos = v@P +(v@N * lift * scale * @dist);

v@P = liftpos;
f@width = width * .0005;
```

<img src="https://user-images.githubusercontent.com/83437383/173165419-5e55535c-9eaf-482a-b175-bb1865cfbffe.jpg" alt="Visualization of airport connections" width="450"/>

---

Data provided by this brilliant open source project: https://openflights.org
