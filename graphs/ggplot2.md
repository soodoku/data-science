Summary of ggplot2
==================

Efficient summary of the ggplot2 book. 

TOC:
--------
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

**Note**: To learn more about grammar of graphics, see [Wilkinson 2005](http://www.amazon.com/The-Grammar-Graphics-Statistics-Computing/dp/0387245448), [Wickham 2009 (pdf)](http://byrneslab.net/classes/biol607/readings/wickham_layered-grammar.pdf)

* What is a statistical graphic?
  - Mapping from data to aesthetic attributes (color, shape, size) of geometric objects (points, lines, bars)
* Basics:
  - data: stuff we want to visualize, mapping describes how variables are mapped to aesthetic attributes
  - geom: represents what you see on the plot: points, lines, polygons, etc.
  - scale: map values in data space to values in aesthetic space (color, size, shape), draws legends, axes
  - coord: data coordinates mapped to the plane of the graphic
  - facet: how to break data into subsets
* How ggplot fits into other R graphics:
  - **Base graphics**: 
	- Pen and paper model - can only draw on top of the plot, cannot modify or delete existing content
	- No user accessible representation of graphics apart from appearance on screen
	- Includes tools for drawing primitives and entire plots
  - **Grid**:
	- Developed by Paul Murrell (1998 disseration)
	- Grid gobs (graphical objects) can be represented independently of the plot and modified later
	- System of viewports (each containing own coord. system) allows for complex graphics
	- Draws primitives but no way to produce statistical graphics
  - **Lattice**:
	- Sarkar 2008a
	- Implements trellis of Cleveland (1985, 1993)
	- Produce conditioned plots
	- Lack formal model

Chapter 2: qplot
------------------

#### Scatterplot + Smoothers

```{r }
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

```{r }
qplot(categorical, quantitative, data=data, geom="jitter", alpha = I(1/10)) 

"
for boxplots, geom="boxplot"
"
```

#### Histogram and Density

```{r }
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

```{r }
qplot(date, y, data=data, geom="line")
```

#### Faceting

```{r }
qplot(x, data=data, facets = category ~ ., geom="histogram", binwidth=.1)

```
#### Other options

```{r }
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

```{r }
p <- ggplot(data, aes(x, y, color=z)) # nothing would be displayed as no layers yet
p + layer(geom = "point") # use + to add layers, this uses the mapping given and default values for stat and position adj.

```

#### Layer Options:

```{r }
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

```{r }
geom_histogram(binwidth=X, fill="red")
geom_point()
geom_smooth()

```

#### Each of the basics individually

* Data
	- Must be a data.frame
	- If you want to produce the same plot for different data frames
	- to see same plot with new data:

```{r }
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
	- break by a categorical variable for instance

* Geoms
	- each geom has a set of aesthetics it needs and a larger set it understands. 
	- For instance, point needs x and y and understands color, size, shape
	- Bar geom needs height (ymax) and understands width, border color, fill color
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

* Growth Trajectory tip
	- Plots of long. data are called spaghetti plots
	- Plot estimated trajectories + actual data and can't see the issues
	- Plot residuals

Chapter 5: Toolbox
-----------------------------------

#### Three purposes of a layer:
1. Display the data
2. Display stat summaries
3. Display metadata

#### Basic plot types (shortcuts) (all these understand size and color aesthetic):
* geom_area()
* geom_bar()
* geom_line() (also understands linetype), geom_path() (same as geom_line but lines connected in order)
* geom_point()
* geom_polygon() (filled path, each vertex = separate row)
* geom_text() (also has hjust, vjust)
* geom_tile()

#### 1d:
* geom_histogram()
* geom_freqpoly
* geom_boxplot()
* geom_density()
* geom_jitter() (crude way of looking at categorical data, prevents overplotting)


#### Dealing with overplotting
* Make points smaller (shape =".") or using hollow glyphs (shape =1)
* alpha
* jitter

```{r }
p <- ggplot(data, aes(x, y))
j <- position_jitter(width=.5)
p + geom_jitter(position = j, alpha("color", 1/10))

```

* binning 
	- but breaking in squares can produce artifacts so use hexagon (geom_hexagon)
	- stat_bin2d
	- stat_binhex(bins = 10)
	- stat_binhex(bindiwth=c(.02, 200))

* summaries

#### Surface Plots + Maps

#### Revealing Uncertainty

Geoms that can do intervals:
* geom_ribbon -> geom_smooth
* geom_errorbar -> geom_crossbar
* geom_linerange -> geom_pointrange

```{r }
library(effects)
a <- as.data.frame(effect(...))
```


Chapter 6: Scales, Axes, and Legends
-----------------------------------

#### Common Arguments
* name 
	- label for axis/legend 
	- use \n for a new line
	- math expressions using plotmath
	- common shortcuts: xlab, ylab, labs()

* limits
	- fixes limits

* breaks, labels
	- breaks control which vals appear on axis/legend
	- labels - labels to appear on those breakpoints

* formatter
	- if no labels specified, this is called. 
	- useful formatters: comma, percent, dollar, scientific, and abbreviate (for categorical scales)

#### Position Scales

* Shorcuts
	- xlim(10, 20) # produces cont. scale
	- xlim("a", "b", "c")
* If you don't want graph to extend beyond range of data: expand = c(0,0)
* Continuous Scales:
	- scale_x_continuous
	- takes trans argument to transform: pow10, probit, recip (x^-1), reverse, sqrt, log, exp, identity, etc.
	- scale_x_log10() = scale_x_continuous(trans = "log10")
* Date and Time
	- takes as.Date or as.POSIXct
	- takes three things: major (breaks), minor (breaks), format (formatting of labels)
	- e.g. major = "2 weeks", format = "%d/%m/%y" etc.

#### Color
* Read more about colour at http://www.handprint.com/HP/WCL/color7.html
* hcl color space: hue (color), chroma (purity of color), luminance (lightness of color) 
* continuous:
	- scale_colour_gradient() # 2 color gradient
	- scale_color_gradient2() # 3 color gradient
	- scale_color_gradientn() # custom n gradient
	- scale_fill_gradient() etc. also...
	- RColorBrewer::display.brewer.all, can do fill = vore (where vore is from color brewer)
* discrete:
	- scale_linetype
	- scale_shape
	- scale_size_discrete
	- scale_shape_manual


Chapter 7: Positioning
-----------------------------------



Chapter 8: Polishing Your Plots for Publication
------------------------------------------------



Chapter 9: Manipulating Data
-----------------------------------



Chapter 10: Reducing Duplication
-----------------------------------
