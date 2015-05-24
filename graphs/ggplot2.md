Summary of ggplot2
==================

Efficient summary of the ggplot2 book. 

#### TOC:

1. [Introduction](#chapter-1-introduction)
2. [qplot](#chapter-2-qplot)
3. [Mastering the Grammar](#chapter-3-mastering-the-grammar)
4. [Build a Plot Layer by Layer](#chapter-4-build-a-plot-layer-by-layer)
5. [Toolbox](#chapter-5-toolbox)
6. [Scales, Axes, and Legends](#chapter-6-scales-axes-and-legends)
7. [Positioning](#chapter-7-positioning)
8. [Polishing Your Plots for Publication](#chapter-8-polishing-your-plots-for-publication)
9. [Manipulating Data](#chapter-9-manipulating-data)
10. [Reducing Duplication](#chapter-10-reducing-duplication)

Chapter 1: Introduction
-------------------------

* For learning more about grammar of graphics, see Wilkinson 2005, Wickham 2009
* What is a statistical graphic? 
	- Mapping from data to aesthetic attributes (color, shape, size) of geometric objects (points, lines, bars)
	- Plot may contain statistical transformations, drawn to specific coordinate system
	- Faceting for displaying same plot for diff. subsets
* Basics:
	- Data: stuff you want to visualize, mapping describe how variables are mapped to aesthetic attributes
	- geom: represent what you see on the plot: points, lines, polygons, etc.
	- scale: map values in data space to values in aesthetic space (color, size, shape), draws legends, axes
	- coord: data coordinated mapped to plane of graphic
	- facet: how to break data into subsets
* How it fits into other R graphics:
	-  Base graphics: 
		- Pen and paper model - can only draw on top of the plot, cannot modify or delete existing content
		- No user accessible representation of graphics apart from appearance on screen
		- Includes tools for drawing primitives and entire plots
	- Grid:
		- Developed by Paul Murrell (1998 disseration)
		- Grid gobs (graphical objects) can be represented independently of the plot and modified later
		- System of viewports (each containing own coord. system) allows for complex graphics
		- Draws primitives but no way to produce statistical graphics
	- Lattice:
		- Sarkar 2008a
		- Implements trellis of Cleveland (1985, 1993)
		- Produce conditioned plots
		- Lack formal model

Chapter 2: qplot
------------------

#### Scatterplot + Smoothers

```
qplot(x,y, data=data)

" y can be a complex variable (a*b*c)
"
qplot(x, y, 
			data=data, 
			color=variable, # you can also do color=I("red") which isn't same as mapping
			shape=variable, 
			alpha= I(1/10), # alpha for overplotting
			geom = c("point", "smooth")) # geom="path" for assessing relation w/ time etc., paths

"
Smoothers: 
	- method="loess", span =.2 
	- method = "gam", formula = y ~ s(x)
	- method = "gam", formulat = y ~ s(x, bs = "cs")
	- example: qplot(x, y, data=data, geom=c("smooth"), method="loess", span=.2)
	- method = "lm"
	- method = "lm", formula = y ~ ns(x, 5)
	- method = "rlm" # robust linear model
"

```

#### Boxplot and jittered points

```
qplot(categorical, quantitative, data=data, geom="jitter", alpha = I(1/10)) 

"
for boxplots, geom="boxplot"
"
```

#### Histogram and Density

```
"
1d geoms: 
geom=histogram, freqpoly, density, bar
"
qplot(x, data=data, geom="histogram", 
					bindwith=.1, 
					fill=color, # color is a variable to which you want to map color
					xlim=c(a,b))

qplot(x, data=data, geom="density", color=color) # color is a variable to which you want to map color
qplot(x, data=data, geom="histogram", bindwith=.1, xlim=c(a,b))

qplot(x, data=data, geom="bar", weight=variable) # weighted bar chart 

```

#### Timeseries, paths

```
qplot(date, y, data=data, geom="line")
```

#### Faceting

```
qplot(x, data=data, facets = category ~ ., geom="histogram", binwidth=.1)

```
#### Other options

```
qplot(x, y, data=data, 
			xlab = "x axis label",
			ylab = "y axis label",
			main = "plot title",
			xlim = c(xmin, xmax),
			log = "xy") # logs x and y variables

```

Chapter 3: Mastering the Grammar
-----------------------------------

Basics of ggplot2: 

* Idea is to map aesthetics (position, size, shape, color) to variables or constants
* Basic constituents:
	- Geoms describe type of plot
		- scatterplot: geom = point
		- bubblechart: geom = point, size = mapped to a variable
		- barchart: geom = bar etc.
	- scaling: convert data units to pixels/spacing/colors
	- coord: use coordinate system for position (coord)
	- position adjustment
* End product: new dataset that records this information (x, y, color, size, shape)

* Layer 
	* layer = data (and mappings to it), stat, geom, position adjustment
	* Plot can have multiple layers, with diff. datasets

* Plot Object:
	- list with data, mapping, layers, scales, coordinates, and facet. and options (to store plot-specific theme options)
	- to rend the object: print()
	- to render to disk: ggsave()
	- to describe structure: summary()
	- to save cached copy to disk: save() (which can be brought back up using load())

Chapter 4: Build a Plot Layer by Layer
---------------------------------------

#### Get started

We start with data (always as a data.frame), and basic aesthetic mapping and add layers to it:

```
p <- ggplot(data, aes(x, y, color=z)) # nothing would be displayed as no layers yet
p + layer(geom = "point") # use + to add layers, this uses the mapping given and default values for stat and position adj.

```

#### Layer Options:

```
layer(geom, geom_params, stat, stat_params, data, mapping, position)
layer(geom = "bar", 
	  geom_params=list(fill= "red"),
	  stat="bin",
	  stat_params = list(bindwidth=2))
```

#### Shortcuts

* But that is verbose so we use shortcuts exploiting the fact that
	- every geom is associated w/ default stat
	- every stat w/ default geom
* All shortcut functions start either with geom_ or stat_

```
geom_histogram(binwidth=X, fill="red")
geom_point()
geom_smooth()

```

#### Each of the basics individually

* Data
	- Must be a data.frame
	- If you want to produce the same plot for different data frames
	- to see same plot with new data:

```
p <- ggplot(data, aes(x,y)) + geom_point()
p %+% new_data
```

* Aesthetic Mappings
	- aes function
	- the mappings can be modified later by doing + aes()
	- do summary(p) to see how the object changes
	- you can change the y-axis by doing p + geom_pont(aes(y=new_y))

* Setting Versus Mapping
	- color = "red" versus color = var

* Grouping

* Geoms
	- each geom has a set of aesthetics it understands. 
	- For instance, point needed x and y and understands color, size, shape
	- bar geom needs height (ymax), and understands width, border color, fill color
	- every geom has a default stat
	- list of all geoms:
		- abline, area, 
		- bar, blank (draws nothing), boxplot
		- countour (3d contours in 2d), crossbar (hollow bar with middle indicated by a line)
		- density, density_2d (2d density)
		- errorbar 
		- histogram, hline
		- interval, jitter, line (connect obs.)
		- linerange 
		- path (connect obs. in original order)
		- point, pointrange (vertical line, with point in the middle)
		- polygon
		- ribbon, rug
		- segment, smooth, step (connect obs. in stairs)
		- text, tile, vline

* Stat
	- stat takes dataset as input and outputs a dataset
	- stat_bin produces: count, density, center of the bin (x)
	- list of all stats:
		- bin, boxplot, contour, density, density_2d, function, identity, qq,
		- quantile, smooth, spoke (converts angle and radius to xend and yend),
		- sum, summary, unique

* Position Adjustments
	- dodge, fill, identity, jitter, stack
	
Chapter 5: Toolbox
-----------------------------------



Chapter 6: Scales, Axes, and Legends
-----------------------------------



Chapter 7: Positioning
-----------------------------------



Chapter 8: Polishing Your Plots for Publication
------------------------------------------------



Chapter 9: Manipulating Data
-----------------------------------



Chapter 10: Reducing Duplication
-----------------------------------
