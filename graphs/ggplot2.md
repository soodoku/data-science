Summary of ggplot2
==================

### Efficient Summary of ggplot2 by Hadley Wickham

#### TOC:

1. [Introduction](#chapter-1-introduction)
2. [qplot]()
3. [Mastering the Grammar]()
4. [Build a Plot Layer by Layer]()
5. [Toolbox]()
6. [Scales, Axes, and Legends]()
7. [Positioning]()
8. [Polishing Your Plots for Publication]()
9. [Manipulating Data]()
10. [Reducing Duplication]()

Chapter 1: Introduction
-------------------------

* For Grammar of G, See Wilkinson 2005, Wickham 2009
* What is a Statistical Graphic? 
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

Basics:

* Map aesthetics (position, size, shape, color) to variables or to constants
* Once mapping is done, create a new dataset that records this information
* Geoms describe type of plot
	- scatterplot: geom = point
	- bubblechart: geom = point, size = mapped to a variable
	- barchart: geom = bar etc.
* use coordinate system for position using coord



Chapter 4: Build a Plot Layer by Layer
---------------------------------------



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
