Efficient summary of the ggplot2 book
======================================

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

**What is a statistical graphic?**
- Mapping from data to aesthetic attributes (color, shape, size) of geometric objects (points, lines, bars)

**Basic components of such a mapping:**
- data: stuff we want to visualize, mapping describes how variables are mapped to aesthetic attributes
- geom: represents what you see on the plot: points, lines, polygons, etc.
- scale: map values in data space to values in aesthetic space (color, size, shape), draws legends, axes
- coord: data coordinates mapped to the plane of the graphic
- facet: how to break data into subsets

**How ggplot fits into other R graphics:**
- **Base graphics**: 
	- Pen and paper model: can only draw on top of the plot, cannot modify or delete existing content
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
	- Produces conditioned plots
	- Lacks formal model

Chapter 2: qplot
------------------

Basic plotting function.

#### Scatterplot + Smoothers
```{r }

# Assume x and y are quant. variables
qplot(x,y, data=data)

qplot(x, y,  # x or y can be a complex variable (a*b*c)
			data=data, 
			color=variable, # you can also do color=I("red") which isn't same as mapping
			shape=variable, 
			alpha= I(1/10), # alpha for overplotting
			geom = c("point", "smooth")) # geom="path" for assessing relation w/ time etc., paths

"
Smoothers: 
	- method="loess", span =.2 
	- method = "gam", formula = y ~ s(x), formula = y ~ s(x, bs = "cs")
	- method = "lm"
	- method = "lm", formula = y ~ ns(x, 5)
	- method = "rlm" # robust linear model
"

qplot(x, y, 
			data=data, 
			geom=c("smooth"), 
			method="loess", 
			span=.2)
```

#### Boxplot and jittered points
```{r }

# Scatter plot for quant vars. 
# To prevent overplotting - jitter, and alpha
qplot(categorical, quantitative, data=data, geom="jitter", alpha = I(1/10)) 

# for boxplots, geom="boxplot"
```

#### Histogram and Density
```{r }
# 1d geoms:  geom=histogram, freqpoly, density, bar 

qplot(x, data=data, geom="histogram", # or geom = "density"
					bindwith=.1, 
					fill=color, # color is a variable to which you want to map color
					xlim=c(a,b))

# weighted bar chart 
qplot(x, data=data, geom="bar", weight=variable) 

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

#### Basic components of ggplot2:
* Geoms describe type of plot
	- scatterplot: geom = point
	- bubblechart: geom = point, size = mapped to a variable
	- barchart: geom = bar etc.
* scaling: convert data units to pixels/spacing/colors
* coord: use coordinate system for position (coord)
* position adjustment
* faceting
* End product: new dataset that records this information (x, y, color, size, shape)

#### Layer 
* layer = data (and mappings to it), stat, geom, position adjustment
* Plot can have multiple layers, with diff. datasets

#### Plot object:
* a list with data, mapping, layers, scales, coordinates, and facet. and options (to store plot-specific theme options)
* to render to screen: print()
* to render to disk: ggsave()
* to describe structure: summary()
* to save cached copy to disk: save() (which can be brought back up using load())

Chapter 4: Build a Plot Layer by Layer
---------------------------------------

ggplot always takes data as a data.frame. Start with mapping basic aesthetics to data, and then add layers to it:

```{r }
p <- ggplot(data, aes(x, y, color=z)) # nothing would be displayed as no layers yet
p + layer(geom = "point") # use + to add layers, this uses the mapping given and default values for stat and position adj.
```

#### Layer:
```{r }
layer(geom, geom_params, stat, stat_params, data, mapping, position)
layer(geom = "bar", 
	  geom_params=list(fill= "red"),
	  stat="bin",
	  stat_params = list(bindwidth=2))
```

#### Shortcuts
* Specifying layer is verbose so we use shortcuts exploiting the fact that
	- every geom is associated w/ default stat
	- every stat w/ default geom
* All shortcut functions start either with geom_ or stat_

```{r }
geom_histogram(binwidth=X, fill="red")
geom_point()
geom_smooth()
```

#### Each basic component in detail

**Data**
- Must be a data.frame
- If you want to produce the same plot for different data frames
- to see same plot with new data:
```{r }
p <- ggplot(data, aes(x,y)) + geom_point()
p %+% new_data
```

**Aesthetic Mappings**
- aes function
- the mappings can be modified later by doing + aes()
- do summary(p) to see how the object changes
- you can change the y-axis by doing p + geom_pont(aes(y=new_y))
- **Setting Versus Mapping**
	- color = "red" versus color = var

**Grouping**
- break by a categorical variable for instance

**Geoms**
- each geom has a set of aesthetics it needs and a larger set it understands. 
- For instance, point needs x and y and understands color, size, shape
- Bar geom needs height (ymax) and understands width, border color, fill color
- every geom has a default stat
- list of all geoms:
	- abline, area, bar, 
	- blank (draws nothing), 
	- boxplot
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

**Stat**
- stat takes dataset as input and outputs a dataset
- stat_bin produces: count, density, center of the bin (x)
- list of all stats:
	- bin, boxplot, contour, density, density_2d, function, identity, qq,
	- quantile, smooth, spoke (converts angle and radius to xend and yend),		
	- sum, summary, unique

**Position Adjustments**
- dodge, fill, identity, jitter, stack

**Growth Trajectory tip**
- Plots of longitudinal data are called spaghetti plots
- Plot estimated trajectories + actual data and can't see the issues
- Plot residuals

Chapter 5: Toolbox
-----------------------------------

#### Three purposes of a layer:
1. Display the data
2. Display stat summaries
3. Display metadata

#### Basic plot types (shortcuts) (all these understand size and color aesthetic):
* geom_area, geom_bar()
* geom_line() (also understands linetype), geom_path() (same as geom_line but lines connected in order)
* geom_point(), geom_polygon() (filled path, each vertex = separate row)
* geom_text() (also has hjust, vjust)
* geom_tile()
* geom_histogram()
* geom_freqpoly
* geom_boxplot()
* geom_density()
* geom_jitter() (crude way of looking at categorical data, prevents overplotting)

#### Dealing with Overplotting
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

* stat summaries

#### Maps
* add map borders using borders() function 
```{r }
library(maps)
data(us.cities)
ggplot(data, aes(long, lat)) + borders("state", size=.5)

"
choropleth maps:
start by merging map data with whatever else
reorder the merged
geom = polygon 
"
```
#### Revealing Uncertainty

Geoms that can do intervals:
* geom_ribbon -> geom_smooth
* geom_errorbar -> geom_crossbar
* geom_linerange -> geom_pointrange

```{r }
library(effects)
a <- as.data.frame(effect(...))
```

#### Stat Summaries
* stat_summary()
* individual summary funcs like fun.y, fun.ymin, fun.ymax - take functions that take a vector and return single numeric
```{r }
funs <- function(x) mean(x, trim=.5)
stat_summary(fun.y = funs, geom="point")
```
* more complex data func = fun.data (return named vector as output)
```{r }
stat_summary(fun.data=exoticfunc, geom="ribbon")
```

#### Annotating a Plot
* Key thing: Annotations are extra data
```{r }
geom_vline(), geom_hline # use it to highlight certain points on say a timeline
geom_rect() to highlight interesting rect regions (takes xmin, xmax, ymin, ymax)
geom_text(aes(x, y, label="caption")) # x, y are coords, label can be a single label, list of labels
```
* Arrows
	- geom_line, geom_path, and geom_segment all have an arrow param
	- use arrow() function to tweak the arrow. arrow takes angle, length, ends, type

Chapter 6: Scales, Axes, and Legends
-----------------------------------

**Scales control mapping from data to aesthetics. They also provide viewers a guide to map back from aesthetics to data.**

#### How Scales work:
* Input can be discrete or continuous. 
* Process of mapping domain to range:
	- transformation: only for continuous. log etc.
	- training: domain is learned. complicated when graph has multiple layers
	- mapping: function that translates domain to range (which we knew before)

#### Usage
```{r }
plot <- qplot(quant, quant, data)
plot + aes(x=categorical) # doesn't work as scale of x is still cont.
plot + aes(x=categorical) + scale_x_discrete()
```

To modify default_scale:
* Must construct a new scale and add with +
* Scale constructors 
	- start with scale_ 
	- followed by aesthetic (color_, shape_, x_)
	- followed by name of scale (gradient, hue, manual)
	- for e.g. scale_color_hue
* Scale for each aesthetic 
	- Colour and fill: 
		- discrete: brewer, grey, hue, identity, manual (e.g. scale_color_brewer())
		- cont: gradient, gradient2, gradientn
	- position:
		- discrete
		- continuous, date
	- shape:
		- discrete only: shape, identity, manual
	- line type:
		- discrete only: linetype, identity, manual
	- size:
		- discrete: identity, manual
		- cont: size
* Types of Scales
	- Position Scales: for axes
	* Color Scales: map to colors
	* Manual Scales: map manually to size, shape, color, line type etc.
	* Identity Scale

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
```{r }
ggplot(data, aes(x)) +
geom_line(aes(y = y + 5, color = "above")) +
geom_line(aes(y = y -5, color = "below")) +
scale_color_manual("Legend Label", c("below" = "blue", "above" ="red"))
```
* scale_identity: data are colours

#### Legends and Axes
* Axis: axis label, tick mark, tick label
* Legend: legend title, key, key label
* Scale name controls axis label, legend title
* breaks and labels (see above)
* for visual appearance, see axis.*, legend.*
* internal grid: panel.grid.major/minor
* legend.position, legend.justification

Chapter 7: Positioning
-----------------------------------
#### Faceting:

**facet_grid: 2d grid of panels**
* facet_grid(. ~ .) means neither rows nor columns are faceted
* facet_grid(. ~ var) produces single row with multiple cols

**facet_wrap: 1d ribbon of panels wrapped into 2d**

** Controlling Scales**
* scales="fixed", x and y scales are fixed across all panels
* scales = "free", x and y vary across panels
* scales = "free_x" only x varies, y is fixed

** Dodging vs. Faceting**
*  Can be pretty similar except for the labeling.
```{r }
qplot(xaxis, data=data, geom="bar", fill=diff_color_bars_same_x, position ="dodge")
# similar to
qplot(diff_color_bars_same_x, data=data, geom = "bar", fill=diff_color_bars_same_x) + 
facet_grid(. ~ xaxis)
```
* To facet by continuous vars, categorize it: cut_interval(x, n=10) etc.

** Coordinate Systems**
* Diff coordinate systems built in:
	* coord_flip: x is y now
	* coord_equal: equal scale coords
	* coord_trans: transform/log etc.
	* coord_map
	* coord_polar
```{r }
plot + coord_flip()
plot + coord_trans(x="pow10")
plot + coord_cartesian(xlim=c(0,20))
# coord_polar takes theta - position variable that is mapped to angle
plot  + coord_polar(theta="y") # default is x (creates pie chart from a stacked bar chart)
```

Chapter 8: Polishing Your Plots for Publication
------------------------------------------------
**Themes**
* Themes control non data elements of plot
* Themese control: font (title, axis, tick labels, strips, legend labels, legend key labels), color (ticks, grid lines, backgrounds (panel, plot, strip, legend))
* Built in themes
	* theme_gray() (Based on advice by Tufte, Brewer, Carr etc.)
	* theme_bw()
```
previous_theme <- theme_set(theme_bw())
plot + previous_theme
theme_set(previous_theme) # permanently restores previous theme
```
* Theme elements
	* theme_text() for labels, headings. Control font family, color, face, size, hjust, vjust, angle, lineheight
	* theme_line(), theme_segment() take color, size, and linetype. e.g. opts(panel.grid.major = theme_line(color="red"))
	* theme_rect() for background etc. and takes color, size, and linetype (changes the border)
	* theme_blank() draws nothing. e.g. opts(panel.grid.major = theme_blank())
	* you can alter old theme with theme_update
* Save
	* ggsave()
	* pdf() qplot() dev.off()
* Subplots
	* viewport(x, y, width, height)
	* pdf(); vp <- viewport(); small_graph; print(big_graph, vp=vp); dev.off()
	* Grids
		* grid.layout()
		* grid.newpage()
		* pushViewport(viewport(layout = grid.layout(2,2)))

Chapter 9: Manipulating Data
-----------------------------------
* ddply(.data, .variables, .fun, ...)
	* .data: dataset to break up
	* .variables: grouping var
	* .fun: summary function to apply
* colwise() to apply a function to each column of data
	* colwise(function)(data)

Chapter 10: Reducing Duplication
-----------------------------------
* last plot can be accessed via last_plot()
* you can save plot templates
        ```{r }
        xquiet <- scale_x_continuous("", breaks=NA)
        yquiet <- scale_y_continuous("", breaks=NA)
        quiet  <- c(xquiet, yquiet)
        plot + quiet 
        ```