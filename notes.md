# How do I even get started with Data Visualization?

## Agenda

0. Why bother with programmatic visualization at all?
1. The types of visualizations
2. The types of data-viz tools
3. Demo & categorize data-viz tools
4. Your turn!
5. Final discusison & demonstrations

```python
from IPython.display import display

display("Let's Get Started")
```

## Google Colab: Install Missing Packages

Google Colab comes with many Python packages pre-installed, but we will need the
following pacakges as well.

```python
!pip install flexitext==0.2.0
```

## Types of Visualizations
- Exploratory
- Communicative

## Types of Data Visualization Tools
- Non-programmatic
    - low : hand/computer drawing
    - high: GUI chart builders* (excel, tableau, ...)
- Programmatic
    - low : programmatic drawing
    - high: declarative & convenience

```python
from IPython.display import display
from pandas import read_csv

anscombe = read_csv('https://raw.githubusercontent.com/dutc-io/agu-data-viz/main/data/anscombe.csv')

display(
    anscombe.head(),
    # anscombe.groupby('id')[['x', 'y']].agg(['mean', 'std']),
)
```

## Common Data Visualization APIs

### Drawing (non-declarative)

```python
from pandas import read_csv
from matplotlib.pyplot import subplots, show

anscombe = read_csv('https://raw.githubusercontent.com/dutc-io/agu-data-viz/main/data/anscombe.csv')
```

### High Level - Declarative

```python
from matplotlib.pyplot import show
from plotnine import ggplot, facet_wrap, geom_point, geom_smooth, labs, theme_minimal, aes
from pandas import read_csv

anscombe = read_csv('https://raw.githubusercontent.com/dutc-io/agu-data-viz/main/data/anscombe.csv')
```

### High Level - Convenience

```python
from matplotlib.pyplot import show
from seaborn import lmplot
from pandas import read_csv

anscombe = read_csv('https://raw.githubusercontent.com/dutc-io/agu-data-viz/main/data/anscombe.csv')
```

```python
from seaborn.objects import Plot, Dots, PolyFit, Line
from pandas import read_csv

anscombe = read_csv('https://raw.githubusercontent.com/dutc-io/agu-data-viz/main/data/anscombe.csv')
```

### Is it worth it to learn multiple data visualization libraries/languages?

## Let’s Make Some Viz!

### Static Data Visualizations (explore, communicative, fun)

**matplotlib**

```python
from matplotlib.pyplot import figure, show
from matplotlib.patches import Circle, Rectangle

fig = figure(figsize=(6,6))

c = Circle((.5, .8), .1)
fig.add_artist(c)

## uncomment below if running from Jupyter Notebook/Google Colab
# ax = fig.add_axes([0, 0, 0, 0])
# ax.set_visible(False)

# body_rect = Rectangle((.47, .75), .06, -.5)
# fig.add_artist(body_rect)

# arms_rect = Rectangle((.3, .55), .4, .05)
# fig.add_artist(arms_rect)

# lleg_rect = Rectangle((.5, .3), .3, .05, angle=225)
# fig.add_artist(lleg_rect)

# rleg_rect = Rectangle((.47, .26), .3, .05, angle=-45)
# fig.add_artist(rleg_rect)

show()
```

**Useful Things to draw for Data Viz**

```python
from numpy import linspace, pi, sin, cos
from matplotlib.pyplot import figure, show, plot, rc

rc('font', size=16)

xs = linspace(0, 2 * pi)

fig = figure()
# ax = fig.add_axes([.3, .3, .5, .5])

# ax.plot(xs, sin(xs))
# ax.plot(xs, cos(xs))

# ax.set_ylabel('this is my y label')
# fig.supylabel('this my figure y label')

show()
```

```python
from IPython.display import display

from numpy import linspace, pi, sin, cos
from matplotlib.pyplot import subplots, show

fig, ax = subplots()

# fig, axes = subplots(nrows=1, ncols=2)
# fig, axes = subplots(nrows=2, ncols=2)
# print(axes[:, 0])

xs = linspace(0, 2 * pi)

# display(axes)
# display(type(axes))

# axes[1, 0].plot(xs, sin(xs))

# display(axes, type(axes), axes[0])
# show()
```

Matplotlib is object oriented
- Containers → Artists
- Figure → Axes →
    - X/YAxis
        - ticks
        - ticklabels
        - axis label
    - Primitives
        - Patches (Circle, Rectangle)
        - Line2d
        - Annotations/Text
        - ...
    - Legend
        - Primitives
        - Text (label & title)

- Coordinate Spaces: values → ... → screen
    - Proportional coordinate space (Figure & Axes)
    - Data coordinate space         (Axes)
    - Identity/point space          (Figure)

**Applied to Star Trader Data - Tracking Ship Failures**

```python
from matplotlib.pyplot import subplots, show
from pandas import read_csv, to_datetime

df = (
    read_csv(
        'https://raw.githubusercontent.com/dutc-io/agu-data-viz/main/data/failures.csv',
        index_col=['date', 'player','ship'],
        parse_dates=['date'],
    )
    .sort_index()
)


plot_data = (
    df.pivot_table(index='date', columns='player', values='faults', aggfunc='sum')
    .rolling('90D').mean()
)

ax = plot_data.plot(legend=False)
for line in ax.lines:
    x, y = line.get_data()
    ax.annotate(
        line.get_label(), xy=(x[-1], y[-1]),
        xytext=(5, 0), textcoords='offset points',
        color=line.get_color()
    )

show()
```

### Interactive Data Visualizations (explore, fun)

- System/function Exploration
- Data Exploration
- System Observability

*Limited Communicatve Ability unless STRONGLY guided*

**bokeh & panel** - a powerful way to share your data on the web!

```python
from panel import extension

# Increase font size of widgets
css = '''
.bk-root .bk, .bk-root .bk:before, .bk-root .bk:after {
  font-size: 110%;
}
'''
extension(raw_css=[css]) # Connect `panel` application to notebook runtime
```

```python
from panel import Column
from bokeh.plotting import figure
from bokeh.models import ColumnDataSource

from numpy import linspace, zeros
from scipy.stats import skewnorm

loc   = 5
scale = 1
skew  = 0

cds = ColumnDataSource({
    'x': linspace(-10, 10, 500),
    'y1': zeros(shape=500),
})
cds.data['y2'] = skewnorm(loc=loc, scale=scale, a=skew).pdf(cds.data['x'])

def update_plot(loc, scale, skew):
    cds.data['y2'] = skewnorm.pdf(x=cds.data['x'], a=skew, loc=loc, scale=scale)

p = figure(y_range=(0, .5), width=500, height=300)
p.varea(x='x', y1='y1', y2='y2', source=cds, alpha=.3)
p.line(x='x', y='y2', source=cds, line_width=4)
p.yaxis.major_label_text_font_size = "20pt"
p.xaxis.major_label_text_font_size = "20pt"

Column(p).servable()
```

adding interactivity

```python
from panel import Column, bind
from panel.widgets import FloatSlider
from bokeh.plotting import figure
from bokeh.models import ColumnDataSource

from numpy import linspace, zeros
from scipy.stats import skewnorm

loc   = FloatSlider(name='mean', value=0, start=-10, end=10)
scale = FloatSlider(name='std. dev', value=1, start=.1, end=10)
skew  = FloatSlider(name='skew', value=0, start=-6, end=6)

# Data abstraction
cds = ColumnDataSource({
    'x': linspace(-10, 10, 500),
    'y1': zeros(shape=500),
    'y2': zeros(shape=500),
})

def update_plot(loc, scale, skew):
    cds.data['y2'] = skewnorm.pdf(x=cds.data['x'], a=skew, loc=loc, scale=scale)

p = figure(y_range=(0, .5), width=500, height=300)
p.varea(x='x', y1='y1', y2='y2', source=cds, alpha=.3)
p.line(x='x', y='y2', source=cds, line_width=4)
p.yaxis.major_label_text_font_size = "20pt"
p.xaxis.major_label_text_font_size = "20pt"

Column(
    Column(loc, scale, skew), # render widgets
    p,                        # render plot
    bind(update_plot, skew=skew, loc=loc, scale=scale) # bind widgets to `update_plot` function
).servable()
```

**Applied to our Star Trader Data - Planetary Weather**

```python
from pandas import read_csv

df = (
    read_csv(
        'https://raw.githubusercontent.com/dutc-io/agu-data-viz/main/data/weather_york.csv',
        usecols=['date', 'temperature_max', 'temperature_min'],
        parse_dates=['date'],
        index_col='date'
    )
).loc['1990':'2000']

# Long timeseries Zoom
from bokeh.models import RangeTool
from bokeh.plotting import figure, ColumnDataSource
from panel import Column
from pandas import to_datetime, DateOffset

cds = ColumnDataSource(df)
p = figure(
    width=1000, height=500, x_axis_type='datetime', y_range=[0, 110],
    x_range=[df.index.min(), df.index.min() + DateOffset(years=1, days=-1)],
)
p.vbar(x='date', bottom='temperature_min', top='temperature_max', source=cds, width=(24 * 60 * 60 * 1000))

range_p = figure(
    width=p.width, height=p.height // 2, x_axis_type='datetime', y_range=[0, 110],
    x_range=[df.index.min(), df.index.max()],
)
range_p.vbar(x='date', bottom='temperature_min', top='temperature_max', source=cds, width=(24 * 60 * 60 * 1000))

rangetool = RangeTool(x_range=p.x_range)
range_p.add_tools(rangetool)

Column(p, range_p).servable()
```

**Looking for more resources to help get you started?** - try recreating examples from the documentation for these tools
(see **Useful Links** below), or follow along with a tutorial for a tool of your choice!

## Useful Links

### Conceptual Guides

[Data to Viz](https://www.data-to-viz.com/) provides a flowchart-style to the
types of charts one can create given various types of data.

[R Graph Gallery](https://r-graph-gallery.com/) and its counter part [Python Graph Gallery](https://python-graph-gallery.com/) provide a vast number of high quality charts written in either R or Python.

### Tools

**Matplotlib**
- Tutorial: https://matplotlib.org/stable/tutorials/index.html
- Cheatsheets: https://matplotlib.org/cheatsheets/
- Examples: https://matplotlib.org/stable/gallery/index.html

**Plotnine**
- Tutorial: http://r-statistics.co/Complete-Ggplot2-Tutorial-Part1-With-R-Code.html (note that plotnine does not have official tutorials, so please refer to ggplot2)
- Examples: https://plotnine.readthedocs.io/en/stable/gallery.html#

**Bokeh**
- Tutorial: https://docs.bokeh.org/en/latest/docs/first_steps.html#first-steps
- Examples: https://docs.bokeh.org/en/latest/docs/gallery.html#gallery

**Seaborn**
- Tutorial: https://seaborn.pydata.org/tutorial/introduction.html
- Examples: https://seaborn.pydata.org/examples/index.html

## Your Turn…

Take any of the tools we have discussed today and make 1 static or 1 interactive (web) chart.
Remember before you start, think about what you want to create? Something exploratory, communicative?

### Suggested Starting Points

*static* using data/weather_york.csv datasets
- Explore: visualize as much of the data as possible, do you notice any trends?
- Communicate: Select one interesting feature to highlight and create a chart
  that communiates that feature.

```python
from pandas import read_csv

df = (
    read_csv(
        'https://raw.githubusercontent.com/dutc-io/agu-data-viz/main/data/weather_york.csv',
        usecols=['date', 'temperature_max', 'temperature_min'],
        parse_dates=['date'],
    )
)

print(df.head())
```

*interactive* Using the failures.csv dataset,
  - Plot the total failures for each 'player' for each day.
    - Apply a smoothing factor (rolling average) of 90 days prior to plotting the data.
  - Create a slider widget that control the number of days involved in the smoothing.
      - e.g. this slider should allow me to apply 0 days of smoothing all the way up to 90 days of smoothing

```python
from matplotlib.pyplot import subplots, show
from pandas import read_csv, to_datetime

df = (
    read_csv(
        'https://raw.githubusercontent.com/dutc-io/agu-data-viz/main/data/failures.csv',
        parse_dates=['date'],
    )
)

print(df.head())
```
