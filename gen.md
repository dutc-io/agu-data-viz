# failures.csv

```python
from pandas import DataFrame, date_range, MultiIndex, IndexSlice
from functools import reduce
from numpy.random import default_rng
from pathlib import Path

rng = default_rng(0)

players = ['Alice', 'Bob', 'Charlie']
stars = '''
    stan
    ivan
    york
    quin
    task
    reef
    fate
    sol
    kirk
    kris
    boyd
    gaol
    hook
'''.title().split()

black_holes = rng.choice(stars, size=4)
dates = date_range('2020-01-01', periods=365 * 10, freq='D')

df = (
    DataFrame(
        index=(idx := MultiIndex.from_product([
            dates,
            players,
            range(7),
        ], names=['date', 'player', 'ship'])),
        data={
            'location': rng.choice(stars, size=len(idx)),
            'failure': rng.choice([True, False], p=[p := .05, 1-p], size=len(idx)),
        },
    )
)
df = df.loc[
    reduce(
        lambda acc, x: acc.union(x),
        [
            df.loc[IndexSlice[:, ['Alice'], :3]].index,
            df.loc[IndexSlice[:, ['Bob'], :]].index,
            df.loc[IndexSlice[:, ['Charlie'], :5]].index,
        ]
    )
]
df.loc[df['location'].isin(black_holes), 'failure'] = (
    df['location'].isin(black_holes).pipe(
        lambda df: df | rng.choice([True, False], p=[p := .25, 1-p], size=len(df))
    )
)
player_ships = df.reset_index('ship', drop=False)['ship'].groupby('player').max()
# too_many = (
#     df.groupby(['date', 'player'])['location'].transform(
#         lambda g: g.value_counts().loc[g.value_counts().idxmax()] >= 2
#     )
# )
def too_many_ships(g):
    vc = g.value_counts()
    if vc.max() > 1:
        return g == vc.idxmax()
    return g.isna()
too_many = (
    df.groupby(['date', 'player'], group_keys=False)['location'].apply(too_many_ships)
)
df.loc[:, 'failure'] = df.loc[:, 'failure'] | too_many

df.loc[:, 'faults'] = df.loc[:, 'failure'] * rng.normal(loc=1000, scale=100, size=len(df))
df.loc[:, 'faults'] += rng.normal(loc=250, scale=75, size=len(df))
# df.loc[:, 'faults'] *= rng.normal(loc=1, scale=.25, size=len(df)) > 1.15
df['faults'] = df.loc[:, 'faults'].astype(int).clip(0, 2500)

df.loc[IndexSlice['2024-04-01':'2024-07-04', :, :], 'faults'] += 500

df = df.sample(frac=.8, random_state=rng).sort_index()

data_dir = Path('data')
data_dir.mkdir(exist_ok=True, parents=True)
df[['location', 'faults']].to_csv(data_dir / 'failures.csv')

print(
    df,
    # black_holes,
    # too_many,
    # player_ships,
    sep='\n',
)
```
