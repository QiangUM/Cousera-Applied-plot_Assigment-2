# Cousera-Applied-plot_Assigment-2
This is UM Cousera Applied plot assigment 2 code. 


import matplotlib.pyplot as plt
import mplleaflet
import pandas as pd

def leaflet_plot_stations(binsize, hashid):

    df = pd.read_csv('data/C2A2_data/BinSize_d{}.csv'.format(binsize))
    
    station_locations_by_hash = df[df['hash'] == hashid]
 

    lons = station_locations_by_hash['LONGITUDE'].tolist()
    lats = station_locations_by_hash['LATITUDE'].tolist()

    plt.figure(figsize=(8,8))

    plt.scatter(lons, lats, c='r', alpha=0.7, s=200)

    return mplleaflet.display()

leaflet_plot_stations(400,'fb441e62df2d58994928907a91895ec62c2c42e6cd075c2700843b89')


## this is the code 

# from datetime import datetime
import datetime as dt
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
#read data from a csv
df = pd.read_csv('data/C2A2_data/BinnedCsvs_d400/fb441e62df2d58994928907a91895ec62c2c42e6cd075c2700843b89.csv')
#change columns Date to datetime
df['Date']=pd.to_datetime(df['Date'])

#del the leap year from the dataframe

leaplist=['2008229', '2012229']

leapdates = [dt.datetime.strptime(i, "%Y%m%d").date()  for i in leaplist]

df= df[~df['Date'].isin(leapdates)]

df['Month']=pd.DatetimeIndex(df['Date']).month

#cutoff the df before and after 20141231 as df2 and df3

cutoffdate = '20141231'
cutoff_date=dt.datetime.strptime(cutoffdate,"%Y%m%d").date()

df2 = df[df['Date']<=cutoff_date]
df3=df[df['Date']>cutoff_date]
# print(df2.head())

# get the columns Element equal TMIN and TMAX, then groupby Month and aggregate the columns 

df2_max = df2[df2['Element']=='TMAX'].groupby('Month').agg({"Data_Value":np.max}).copy(deep=True)
df2_min = df2[df2['Element']=='TMIN'].groupby('Month').agg({'Data_Value':np.min}).copy(deep=True)

# look df3 if has Temperature higher than df2_max and lower than df2_min


df3_max = df3[df3['Data_Value']>df3['Month'].map(df2_max['Data_Value'])].copy(deep=True)
df3_min = df3[df3['Data_Value']<df3['Month'].map(df2_min['Data_Value'])].copy(deep=True)


# make tenths of degrees C of original degrees C


df2_max['Data_Value']*=0.1
df2_min['Data_Value']*=0.1
df3_max['Data_Value']*=0.1
df3_min['Data_Value']*=0.1

#plot the df2_max,df2_min,  and xlabel, ylabel, xticks,  and fill_between.  

plt.xlabel("Years", fontsize =25)
plt.ylabel("Temperature", fontsize =25)
plt.xticks(df2_max.index,['Jan','Feb','Mar','Apr','May','Jun','Jul','Aug','Sept','Oct','Nov','Dec'],
           fontsize ='x-large',rotation =20)

[a,b]=plt.plot(df2_max, '-bo',df2_min,'-go',linewidth=2, markersize=8,alpha =0.5)
plt.fill_between(df2_max.index, df2_max.values.ravel(), df2_min.values.ravel(),color = 'gray',alpha = 0.6)

# scatter the df3_max, df3_min, and set the title 

c=plt.scatter(df3_max.Month.tolist(),df3_max['Data_Value'],s=25, c='red')
d=plt.scatter(df3_min.Month.tolist(),df3_min['Data_Value'],s =25, c='black')
plt.title('Min and Max temperatures in Ann Arbor Michigan 2005-2015', fontsize=25)

#get curent figure and set size, final show thi figure
plt.legend([a,b,c,d],['2005-2014 max Temp','2005-2014 min Temp','2015 Temp higher','2015 Temp lower'])

fig= plt.gcf()
fig.set_size_inches(20,10)
plt.savefig('Ann Arbor Temperature.png')
plt.show()
