# Analysis

This section describes the analysis of wheather patterns in Silicon Valley.  Specifically, the analysis shows that the range of temperatures has widen in 2015 as compared to the previous 10-year period (from 2005 to 2014).

Data preparation is described in the [previous section](https://eagronin.github.io/sv-weather-prepare).

The results and visualization are described in the [next section](https://eagronin.github.io/sv-weather-report/).

The following code plots a line graph of the record high and record low temperatures by day of the year over the period 2005-2014. The area between the record high and record low temperatures for each day is shaded.  Then this range of daily temperatures is overlaied over a scatter of the 2015 data for any points (highs and lows) for which the ten year record (2005-2014) record high or record low was broken in 2015.

```python
# Read data
df_joined = transform_weather_data()

# Limit data to 2005-2014 period
df_2005_2014 = df_joined[df_joined.Date <= '2014-12-31'].copy()

x = df_2005_2014.Date.as_matrix()
y_min = df_2005_2014.TMIN.as_matrix()
y_max = df_2005_2014.TMAX.as_matrix()
index = pd.date_range(start='2015-1-1', end='2015-12-31', freq='D')
index = index[~((index.month == 2) & (index.day == 29))]

# Calculate record high and record low over 2005-2014
df_2005_2014['md'] = df_2005_2014.Date.dt.month*100 + df_2005_2014.Date.dt.day
df_2005_2014['year'] = df_2005_2014.Date.dt.year
df_min = df_2005_2014.pivot(index = 'md', columns = 'year', values = 'TMIN')
df_min['low'] = df_min.min(axis = 1)
df_min = df_min[['low']]
df_max = df_2005_2014.pivot(index = 'md', columns = 'year', values = 'TMAX')
df_max['high'] = df_max.max(axis = 1)
df_max = df_max[['high']]
df_record = df_min.merge(df_max, how = 'inner', left_index = True, right_index = True)

# Calculate record high and record low over 2015 which break the 2005-2014 records
df_2015 = df_joined[(df_joined.Date > '2014-12-31') & (df_joined.Date <= '2015-12-31')].copy()

df_2015 = df_2015[~((df_2015.Date.dt.month == 2) & (df_2015.Date.dt.day == 29))]       # remove 2/29
df_2015['md'] = df_2015.Date.dt.month*100 + df_2015.Date.dt.day
df_2015 = df_2015.set_index('md')

df_record = df_record.merge(df_2015, how = 'inner', left_index = True, right_index = True)
df_min = df_record[['Date', 'TMIN', 'low']]
df_min.TMIN[df_min.TMIN >= df_min.low] = np.nan
df_max = df_record[['Date', 'TMAX', 'high']]
df_max.TMAX[df_max.TMAX <= df_max.high] = np.nan

# 2015 higher than record high
x_high = df_max.Date.as_matrix()
y_high = df_max.high.as_matrix()
y_2015_high = df_max.TMAX.as_matrix()

# 2015 lower than record low
x_low = df_min.Date.as_matrix()
y_low = df_min.low.as_matrix()
y_2015_low = df_min.TMIN.as_matrix()

# Plot figure
fig, ax = plt.subplots(figsize = (15,10))

ax.plot(x_high, y_high, 'b', label = 'Record high and low temperature by day over 2005-2014')
ax.plot(x_high, y_2015_high, 'ro', label = 'Days in 2015 when record temperature was broken')
ax.plot(x_low, y_low, 'b', label = None)
ax.plot(x_low, y_2015_low, 'ro', label = None)

myFmt = DateFormatter("%m-%d")
ax.xaxis.set_major_formatter(myFmt)

plt.xlim(xmin = '2015-1-1', xmax = '2015-12-31')
plt.fill_between(index, y_low, y_high, facecolor = 'blue', alpha = .2)
plt.ylim(ymax = 550)
x = plt.gca().xaxis

for d in x.get_ticklabels():
    d.set_rotation(45)
    
for axis in ['bottom','left']:
    ax.spines[axis].set_linewidth(1.0)

plt.title('Silicon Valley Weather Patterns', fontsize = 18, fontname = 'Comic Sans MS', fontweight = 'bold', alpha=1)
plt.legend(loc = 'upper left', bbox_to_anchor=(0, 0.9), frameon = False)    #prop = {'weight':'bold'}
plt.xlabel('Date (month - day)', labelpad=15)
plt.ylabel('Tenths of Degrees C', labelpad=10)

# Remove all the ticks (both axes) and unnecessary borders
plt.tick_params(top='off', bottom='off', left='off', right='off', labelleft='on', labelbottom='on')
plt.gca().spines['top'].set_visible(False)
plt.gca().spines['right'].set_visible(False)

pylab.savefig('/Users/eagronin/Documents/Data Science/Portfolio/Project Output/sv-weather.png')
```

The results and visualization are described in [Results](https://eagronin.github.io/sv-weather-report/).
