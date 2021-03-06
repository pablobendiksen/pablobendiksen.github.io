# Michigan Uneployment Rate and Household Composition

This data visualization project pulled datasets from the Federal Reserve Economic Data (FRED) and the U.S. Census Bureau, both government run open data initiatives, in order to highlight the disproportionate representation of races in the context of unemployment rates for a few regions in Michigan. 

Why is it that Detroit and Flint, with their largest percentage of households categorized as Black, were hit harder by the economic recession of 2008 (as measured by the unemployment rate) than Ann Harbor, which carries a higher percentage of white households?

```markup

import pandas as pd
import numpy as np
import matplotlib as mpl
import matplotlib.style as style
import seaborn as sns
from functools import reduce
from matplotlib import ticker
import matplotlib.pyplot as plt
%matplotlib notebook

#Datasets pulled from government run open data initiatives such as the Federal Reserve
#Economic Data (FRED) and U.S. census data.
df_house = pd.read_csv('Household_Type.csv', skiprows=0)
df_Mich = pd.ExcelFile('Michigan_Unemployment.xls')
df_Ann = pd.ExcelFile('Ann_Arbor_Unemployment.xls')
df_Det = pd.ExcelFile('Detroit_Unemployment.xls')
df_Flin = pd.ExcelFile('Flint_Unemployment.xls')

# parsing first (index: 0) sheet
total_rows = df_Mich.book.sheet_by_index(0).nrows
#skiprows indexing starts at 1
skiprows = 10
#Set Column Labels for City TimeSeries Dataframes
michigan_areas_columns = ['Date', 'Unemployment Rate']
#header indexing starts at 0

df_Mich = df_Mich.parse('FRED Graph', skiprows=skiprows)
df_Mich.columns = michigan_areas_columns

df_Ann = df_Ann.parse('FRED Graph', skiprows=skiprows)
df_Ann.columns = michigan_areas_columns
#df_Ann.rename(columns = {'Unemployment Rate': 'Ann Arbor Unemployment Rate'}, inplace = True)
df_Det = df_Det.parse('FRED Graph', skiprows=skiprows)
df_Det.columns = michigan_areas_columns
#df_Det.rename(columns = {'Unemployment Rate': 'Detroit Unemployment Rate'}, inplace = True)

df_Flin = df_Flin.parse('FRED Graph', skiprows=skiprows)
df_Flin.columns = michigan_areas_columns
#df_Flin.rename(columns = {'Unemployment Rate': 'Flint Unemployment Rate'}, inplace = True)

#Fin = pd.concat([df_Ann, df_Det, df_Flin], axis= 0)
df_Mich['Region'] = 'Michigan'
df_Ann['Region'] = 'Ann Arbor'
df_Det['Region'] = 'Detroit'
df_Flin['Region'] = 'Flint'
Fin = pd.concat([df_Mich, df_Ann, df_Det, df_Flin], axis= 0)
#print(Fin)
Fin['Year'] = Fin['Date'].apply(lambda x : x.year)
df_mean_by_year=Fin.groupby(["Year", "Region"]).agg({'Unemployment Rate': 'mean'})
df_mean_by_year.reset_index(inplace=True)
df_mean_by_year = df_mean_by_year[df_mean_by_year['Year'] >= 2000]
df_house.rename(columns = {"5-year Estimate (2013-2017)": "Estimate(2013-2017)"}, inplace = True)
df_house["Estimate"] = df_house["Estimate(2013-2017)"].str.replace(",", "").apply(pd.to_numeric)
#print(df_mean_by_year)

# Create a figure instance, and the two subplots
fig = plt.figure()
plt.style.use('ggplot')
#plt.rcParams['axes.facecolor'] = 'white'
plt.rcParams['savefig.facecolor'] = 'white'
fig.suptitle("Unemployment Rate and Household Race Estimates, by Region")
#fig.patch.set_alpha(0.5)
fig.subplots_adjust(hspace = .5)
ax1 = fig.add_subplot(211)
ax2 = fig.add_subplot(212)
# Tell pointplot to plot on ax1 with the ax argument
plot1 = sns.pointplot(x="Year", y="Unemployment Rate", hue = "Region", data=df_mean_by_year, ax=ax1, size = 5)
#plot1.set(xlabel = [])
plot1.set(ylabel = "Unemployment Rate (%)")
for item in plot1.get_xticklabels():
    item.set_rotation(45)
#plot1.axes.get_xaxis().set_visible(False)
plot1.set_xlabel('')

# Tell the factorplot to plot on ax2 with the ax argument
# Also store the FacetGrid in 'g'
plot2 =sns.barplot(x="City", y="Estimate", hue = "Householder", order =['Ann Arbor', 'Detroit', 'Flint'], data=df_house, ax=ax2)
plot2.set(xlabel = '')
plot2.set_ylabel('Freq. of Households (2013-2017)', fontsize = 10)

yticks = ticker.MaxNLocator(4)
y_minorticks = ticker.MaxNLocator(6)
plot1.yaxis.set_major_locator(yticks)
plot1.yaxis.set_minor_locator(y_minorticks)

yticks2 = ticker.MaxNLocator(4)
y_minorticks2 = ticker.MaxNLocator(6)
plot2.yaxis.set_major_locator(yticks2)
plot2.yaxis.set_minor_locator(y_minorticks2)

plt.legend(loc='upper left')
plt.setp(plot2.get_legend().get_texts(), fontsize='5'); # for legend text
#leg = plt.legend()
#leg.get_frame().set_linewidth(0.0)
#leg(loc='upper left')

plt.savefig('Unemployment_Race.png', bbox_inches='tight')

```
