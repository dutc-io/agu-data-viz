# How do I even get started with Data Visualization?

```python
print("Let's Get Started")
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

fig, axes = subplots(2, 2, sharex=True, sharey=True)
for (label, group), ax in zip(anscombe.groupby('id'), axes.flat):
    ax.scatter(group['x'], group['y'], s=8)
    ax.set_title(label)

fig.suptitle('Anscombe’s Quartet', size='x-large')
show()
```

### High Level - Declarative

```python
from matplotlib.pyplot import show
from plotnine import ggplot, facet_wrap, geom_point, geom_smooth, labs, theme_minimal, aes
from pandas import read_csv

anscombe = read_csv('data/anscombe.csv')

(
    ggplot(anscombe, aes(x='x', y='y'))
    + facet_wrap('id', ncol=2)
    + geom_point()
    + geom_smooth(method='ols')
    + labs(x='x variable', y='y variable', title='Anscombe’s Quartet')
    + theme_minimal()
).draw()

show()
```

### High Level - Convenience

```python
from matplotlib.pyplot import show
from seaborn import lmplot
from pandas import read_csv

anscombe = read_csv('data/anscombe.csv')

lmplot(
    anscombe, x='x', y='y', col='id', col_wrap=2,
)

show()
```

```python
from seaborn.objects import Plot, Dots, PolyFit, Line
from pandas import read_csv

anscombe = read_csv('data/anscombe.csv')

(
    Plot(anscombe, x='x', y='y')
    .facet(col='id', wrap=2)
    .add(Dots(color='black'))
    .add(Line(), PolyFit(1))
).show()
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

- Object oriented
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

smooth_data = (
    df
    .groupby(['date', 'player'])['faults'].sum(numeric_only=True)
    .unstack(['player'])
    .rolling('90D').mean()
)

ax = smooth_data.plot(legend=False)
ax.set_title('Galaxy Wide Fuel Quality Issues Spiked Engine Faults', loc='left', size='x-large')

for line in ax.lines:
    x, y = line.get_data()
    ax.annotate(
        line.get_label(), xy=(x[-1], y[-1]),
        xytext=(5, 0), textcoords='offset points',
        color=line.get_color(),
        va='center'
    )

ax.spines[['right', 'top']].set_visible(False)

from matplotlib.ticker import NullLocator
ax.xaxis.set_minor_locator(NullLocator())
ax.set_ylabel('Engine Faults', size='large')
ax.set_xlabel('')

from matplotlib.dates import date2num
rect = ax.axvspan(xmin=to_datetime('2024-04-01'), xmax=to_datetime('2024-07-04'), ymin=0, ymax=1, color='gainsboro', alpha=.9, zorder=0)
ax.annotate(
    'Galaxy-wide fuel quality issue',
    xy=(date2num(to_datetime('2024-07-04')), .95), xycoords=ax.get_xaxis_transform(),
	xytext=(5, 0), textcoords='offset points'
)

ax.figure.tight_layout()
show()
```

### Interactive Data Visualizations (explore, fun)

- System/function Exploration
- Data Exploration
- System Observability

*Limited Communicatve Ability unless STRONGLY guided*

**bokeh & panel** - a powerful way to share your data on the web!

```python
from panel import Column, bind, Row, extension, Spacer
from panel.widgets import FloatSlider, Button
from bokeh.plotting import figure
from bokeh.models import ColumnDataSource

from numpy import linspace, zeros
from scipy.stats import skewnorm

css = '''
.bk-root .bk, .bk-root .bk:before, .bk-root .bk:after {
  font-size: 110%;
  }
'''
extension(raw_css=[css])

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
from panel import Column, bind, Row, extension, Spacer
from panel.widgets import FloatSlider, Button
from bokeh.plotting import figure
from bokeh.models import ColumnDataSource

from numpy import linspace, zeros
from scipy.stats import skewnorm

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

from bokeh.plotting import figure, ColumnDataSource
from panel import Column, bind
from panel.widgets import IntSlider
from pandas import to_datetime, DateOffset

cds = ColumnDataSource(df)

p = figure(width=1000, height=250, x_axis_type='datetime', y_range=[0, 110], x_range=[df.index.min(), df.index.min() + DateOffset(years=1, days=-1)])
p.vbar(x='date', bottom='temperature_min', top='temperature_max', source=cds, width=24 * 60 * 60 * 900)
# p.varea(x='date', y1='temperature_min', y2='temperature_max', source=cds)

# slider = IntSlider(start=int(df.index.year.min()), end=int(df.index.year.max()))
# def update_data(year):
#     # new_data = df.loc[f'{year}'].reset_index().to_dict('list')
#     # cds.data.update(new_data)
#     p.x_range.start = to_datetime(f'{year}-01-01')
#     p.x_range.end = to_datetime(f'{year}-12-31')

from bokeh.models import RangeTool

range_p = figure(
    height=p.height // 4, width=p.width, y_range=p.y_range, x_range=[df.index.min(), df.index.max()],
    x_axis_type="datetime", toolbar_location=None
)
range_p.vbar(x='date', bottom='temperature_min', top='temperature_max', source=cds, width=24 * 60 * 60 * 900)

from bokeh.models import AdaptiveTicker
range_p.yaxis.ticker = AdaptiveTicker(desired_num_ticks=2, num_minor_ticks=0)
# range_p.varea(x='date', y1='temperature_min', y2='temperature_max', source=cds)

rangetool = RangeTool(x_range=p.x_range)
range_p.add_tools(rangetool)

Column(
    p,
    range_p,
    # bind(update_data, slider),
) .servable()
```

### Animated Data Visualizations (communicative, fun)

**ipyvizzu**

```python
import panel as pn
from panel.pane import Markdown, HTML
from pandas import read_csv, MultiIndex
from ipyvizzu import Chart, Data, Config, Style, DisplayTarget

from numpy import arange, linspace, concatenate

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

filt = '||'.join(
    f"record.Country == '{c}'"
    for c in countries_min
)

chart.animate(
    Config({
        'title': 'Countries Winning > 80 Summer Olympic Medals',
    }),
    Data.filter(filt),
    delay=2,
	duration=4,
)

for i, (year, group) in enumerate(medals_count.groupby('Year')):
    title = 'Summer Olympic Medals 1896'
    if year != '1896':
        title += f' - {year}'
    chart.animate(
        Data.filter(
            f'record.Year == {year} && ({filt})'
        ),
        Config(
            config |
            {'title': title, 'x': 'Cumulative Medals'}
        ),
		delay=4 if i == 0 else 0,
        duration=1,
        x={"easing": "linear", "delay": 0},
        y={"delay": 0},
        show={"delay": 0},
        hide={"delay": 0},
        title={"duration": 0, "delay": 0},
    )

# Zoom Out
chart.animate(
    Data.filter(None),
    Config({
        'title': 'Summer Olympic Medals up to 2012',
        'x': 'Total Medals',
    }),
    duration=3
)

chart.animate(
    Data.filter('''
        record.Country == 'United States'
        || record.Country == 'United Kingdom'
        || record.Country == 'France'
        || record.Country == 'Italy'
    '''),
    Config({'title': 'Select Countries'}),
)

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
).groupby('year')['temperature_max'].mean().reset_index()

yr_start, yr_stop = df.year.min(), df.year.max()
df = df.astype({'year': str})

data = Data()
data.add_data_frame(df)

config = {
	'channels':{
    	'y': {'set': 'temperature_max', 'range': {'min': 50, 'max': '75'}},
    	# 'x': {'set': 'doy', 'range': {'min': '0', 'max': '366'},},
    	'x': {'set': 'year'} #, 'range': {'min': str(yr_start), 'max': str(yr_stop)},},
	},
	'geometry': 'line'
}

chart = Chart(
    width="800px", height="600px",
    display=DisplayTarget.MANUAL
)

method = """
	console.log(event.data);
	let doy = parseFloat(event.data.text);
	if (!event.data.text.includes("$") && !isNaN(doy) && doy % 5 != 0)
		event.preventDefault();
"""
handler = chart.on("plot-axis-label-draw", method)
chart.on('logo-draw', 'event.preventDefault();')
chart.animate(
    data,
    Config(config | {'title': 'Slight Increase in Average Temperature'}),
)
for year in range(yr_start, yr_stop+1):
	# chart.animate(
	# 	data.filter(f'record.year == {year}'),
	# 	Config({'title': f'York Temperatures {year}'}),
        # duration=4,
	# )
	chart.animate(
		data.filter(f'parseInt(record.year) <= {year}'),
		# Config({'title': f'York Temperatures {year}'}),
		duration=.3
	)

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


