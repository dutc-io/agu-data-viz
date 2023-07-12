# How do I even get started with Data Visualization?

```python
print("Let's Get Started")
```

## If Running From Google Colab

Execute these cells to access the `data` for this workshop and install some additional packages.

```python
!git clone https://github.com/dutc-io/agu-data-viz.git
```

```python
%cd agu-data-viz
```

```python
!pip install ipyvizzu==0.15.0
```

## Types of Visualizations
- Exploratory
- Communicative
- Fun!

## Types of Data Visualization Tools
- Non-programmatic
    - low : hand/computer drawing
    - high: GUI chart builders* (excel, tableau, ...)
- Programmatic
    - low : programmatic drawing
    - high: declarative & convenience

```python
from pandas import read_csv

anscombe = read_csv('data/anscombe.csv')

print(
    anscombe.head(),
    anscombe.groupby('id')[['x', 'y']].agg(['mean', 'std']),
)
```

## Common Data Visualization APIs

### Drawing (non-declarative)

```python
from pandas import read_csv
from matplotlib.pyplot import subplots, show

anscombe = read_csv('data/anscombe.csv')
```

### High Level - Declarative

```python
from matplotlib.pyplot import show
from plotnine import ggplot, facet_wrap, geom_point, geom_smooth, labs, theme_minimal, aes
from pandas import read_csv

anscombe = read_csv('data/anscombe.csv')
```

### High Level - Convenience

```python
from matplotlib.pyplot import show
from seaborn import lmplot
from pandas import read_csv

anscombe = read_csv('data/anscombe.csv')
```

```python
from seaborn.objects import Plot, Dots, PolyFit, Line
from pandas import read_csv

anscombe = read_csv('data/anscombe.csv')
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
ax = fig.add_axes([.3, .3, .5, .5])

# ax.plot(xs, sin(xs))
# ax.plot(xs, cos(xs))

# ax.set_ylabel('this is my y label')
# fig.supylabel('this my figure y label')

show()
```

```python
from numpy import linspace, pi, sin, cos
from matplotlib.pyplot import subplots, show

fig, ax = subplots()

# fig, axes = subplots(nrows=1, ncols=2)
# fig, axes = subplots(nrows=2, ncols=2)

# xs = linspace(0, 2 * pi)

# print(axes)
# print(type(axes))

# axes[1, 0].plot(xs, sin(xs))

# print(axes, type(axes), axes[0])
show()
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
from pathlib import Path

from matplotlib.pyplot import subplots, show
from pandas import read_csv, to_datetime

df = (
    read_csv(
        Path('data') / 'failures.csv',
        index_col=['date', 'player','ship'],
        parse_dates=['date'],
    )
    .sort_index()
)

plot_data = (
    df.pivot_table(index='date', columns='player', values='faults', aggfunc='sum')
)
```

### Interactive Data Visualizations (explore, fun)

- System/function Exploration
- Data Exploration
- System Observability

*Limited Communicatve Ability unless STRONGLY guided*

**bokeh & panel** - a powerful way to share your data on the web!

```python
from panel import Column, extension
from bokeh.plotting import figure
from bokeh.models import ColumnDataSource

from numpy import linspace, zeros
from scipy.stats import skewnorm

# Connect `panel` application to notebook runtime
extension()

loc   = 0
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
from panel import Column, bind, extension
from panel.widgets import FloatSlider
from bokeh.plotting import figure
from bokeh.models import ColumnDataSource

from numpy import linspace, zeros
from scipy.stats import skewnorm

# Increase font size of widgets
css = '''
.bk-root .bk, .bk-root .bk:before, .bk-root .bk:after {
  font-size: 110%;
  }
'''
extension(raw_css=[css])

loc   = FloatSlider(name='mean', value=0, start=-10, end=10)
scale = FloatSlider(name='std. dev', value=1, start=.1, end=10)
skew  = FloatSlider(name='skew', value=0, start=-6, end=6)

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
    Column(loc, scale, skew),
    p,
    bind(update_plot, skew=skew, loc=loc, scale=scale)
).servable()
```

**Applied to our Star Trader Data - Planetary Weather**

```python
from pandas import read_csv

df = (
    read_csv(
        'data/weather_york.csv',
        usecols=['date', 'temperature_max', 'temperature_min'],
        parse_dates=['date'],
        index_col='date'
    )
).loc['1990':'2000']

# Long timeseries Zoom



# dropdown planet selector

```

### Animated Data Visualizations (communicative, fun)

**ipyvizzu**

```python
from panel.pane import Markdown, HTML
from pandas import read_csv, MultiIndex
from ipyvizzu import Chart, Data, Config, Style, DisplayTarget

countries = (
    read_csv('data/dictionary.csv')
    .set_index('Code')['Country']
)
countries['URS'] = 'Soviet Union'

medals_count = (
    read_csv('data/summer.csv')
    .drop_duplicates(['Year', 'Country', 'Event', 'Medal'])
    .groupby(['Year', 'Country']).size()
    .rename('Total Medals')
)

medals_count = (
    medals_count.reindex(
        MultiIndex.from_product(
            medals_count.index.levels),
        fill_value=0
    )
    .reset_index()
    .astype({'Year': str})
    .assign(Country=lambda d: d['Country'].map(countries))
)

medals_count['Cumulative Medals'] = medals_count.groupby(['Country'])['Total Medals'].cumsum()

countries_min = (
    medals_count.groupby('Country')
    ['Total Medals'].sum()
    .gt(80)
    .loc[lambda s: s].index
)

data = Data()
data.add_data_frame(medals_count)

config = {
	'y': 'Country',
	'x': 'Total Medals',
    'sort': 'byValue'
}

style = Style(
    {'plot': {'paddingTop': 40, 'paddingLeft': 150}}
)

chart = Chart(
    width="800px", height="600px",
    display=DisplayTarget.MANUAL
)
chart.on('logo-draw', 'event.preventDefault();')
chart.animate(
    data,
    style,
    Config(config | {'title': 'United States Leads Summer Olympic Medals'}),
)

# filt = '||'.join(
#     f"record.Country == '{c}'"
#     for c in countries_min
# )
# chart.animate(
#     Config({
#         'title': 'Countries Winning > 80 Summer Olympic Medals',
#     }),
#     Data.filter(filt),
#     delay=2,
# 	duration=4,
# )

# for i, (year, group) in enumerate(medals_count.groupby('Year')):
#     title = 'Summer Olympic Medals 1896'
#     if year != '1896':
#         title += f' - {year}'
#     chart.animate(
#         Data.filter(
#             f'record.Year == {year} && ({filt})'
#         ),
#         Config(
#             config |
#             {'title': title, 'x': 'Cumulative Medals'}
#         ),
# 		delay=4 if i == 0 else 0,
#         duration=1,
#         x={"easing": "linear", "delay": 0},
#         y={"delay": 0},
#         show={"delay": 0},
#         hide={"delay": 0},
#         title={"duration": 0, "delay": 0},
#     )

# # Zoom Out
# chart.animate(
#     Data.filter(None),
#     Config({
#         'title': 'Summer Olympic Medals up to 2012',
#         'x': 'Total Medals',
#     }),
#     duration=3
# )

# chart.animate(
#     Data.filter('''
#         record.Country == 'United States'
#         || record.Country == 'United Kingdom'
#         || record.Country == 'France'
#         || record.Country == 'Italy'
#     '''),
#     Config({'title': 'Select Countries'}),
# )

HTML(chart).servable()
```

**Applied to Star Trader Data - **

```python
from panel.pane import Markdown, HTML
from pandas import read_csv
from ipyvizzu import Chart, Data, Config, Style, DisplayTarget

df = (
    read_csv(
        'data/weather_york.csv',
        usecols=['date', 'temperature_max', 'temperature_min'],
        parse_dates=['date'],
        index_col='date'
    )
    .assign(
        year=lambda d: d.index.year,
        doy=lambda d: d.index.dayofyear.astype(str),
    )
    .sort_index()
).loc['1990':'2000']

# Animate the annual weather curve

HTML(chart).servable()
```

**Animation as Automated Interactivity**
- Drill Down    : provides context
- Transformation: insight to dynamic metrics
- Movement      : pre-attentive cues to draw eyes

## Your Turn…
- Take any of the tools we have discussed today and make 1 static, 1 interactive (web), or 1 animated
- Remember before you start, think about what you want to create? Something exploratory, communicative?

**Suggested Starting Points**
1. *static* Using all `data/weather_*.csv` datasets
	- Visualize the average yearly `temperature_max` and `temperature_min` for a single planet (york, sol, kirk, …).
	- Visualize the average yearly `temperature_max` and `temperature_min` for ALL planet (york, sol, kirk, …)
		- Prioritize the comparison of the stars within a given year.
		- Think: should the values be overlaid onto a single chart? Or spread across multiple?

```python
# …
```

2. *interactive* Using the `data/failures.csv` dataset,
	- Plot the total failures for each 'player' for each day.
		- Apply a smoothing factor (`rolling average`) of 90 days prior to plotting the data.
	- Create a slider widget that control the number of days involved in the smoothing.
		- e.g. this slider should allow me to apply 0 days of smoothing all the way up to 90 days of smoothing

```python
# …
```

3. *animated* Using the `data/weather_*.csv` datasets
	- Plot ALL of the 'temperature_max' data for EACH of the planets.
	- Animate: drill-down to the planet with the HIGHEST average temperature (across all datapoints)
	- Animate: reintroduce the other planet’s 'temperature_max' to the chart while maintaining the year 2010 zoom.

```python
# …
```

**Can’t Get Started With The Above?** - try recreating examples from the documentation for these tools
(see **Useful Links** below), or follow along with a tutorial for a tool of your choice!

## Useful Links

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

**IPyvizzu**
- Tutoriall: https://ipyvizzu.vizzuhq.com/latest/tutorial/
- Examples: https://ipyvizzu.vizzuhq.com/latest/examples/analytical_operations/


