# Data_Cleaning_Exploratory_Data_Analysis

import numpy as np
import re
import pandas as pd
import matplotlib.pyplot as plt
%matplotlib inline
import seaborn as sns
from matplotlib import cm
from datetime import datetime
import glob
import os
import json
import pickle
import six
import re

sns.set()
pd.set_option('display.max_columns', None)
pd.set_option('display.max_rows', None)
pd.options.mode.chained_assignment = None

path = 'C:\\Users\\User\\Data'
allcsv = glob.glob(path + '\*.csv')

all_df = []

for csv in allcsv:
    df = pd.read_csv(csv)
    df['Country'] = csv[19:21]
    all_df.append(df)
    print(df['Country'])

all_df[0].head()

for df in all_df:
    df['video_id'] = df['video_id'].astype('str')
    df['trending_date'] = df['trending_date'].astype('string')
    date_pieces = df['trending_date'].str.split('.')
    
    df['Year'] = date_pieces.str[0].astype(int)
    df['Day'] = date_pieces.str[1].astype(int)
    df['Month'] = date_pieces.str[2].astype(int)
    
    updatedyear = []
    for i in range(len(df)):
        y = df.loc[i,'Year']
        newy = y + 2000
        updatedyear.append(newy)
        
    for i in range(len (df)):
        newy = updatedyear[i]
        tr = df.loc[i,'Year']
        df['Year'].replace(tr,newy,inplace=True)
        
    del df['trending_date']
    df['trending_date'] = pd.to_datetime(df[['Year','Month','Day']],format='%Y%m%d')
    del df['Year']
    del df['Day']
    del df['Month']
    
    df['title'] = df['title'].astype('str')
    df['channel_title'] = df['channel_title'].astype('str')
    df['category_id'] = df['category_id'].astype(str) 
    
    df['tags'] = df['tags'].astype('str')
    
    df['thumbnail_link'] = df['thumbnail_link'].astype('str') 
    
    df['description'] = df['description'].astype('str')
    df['comments_disabled'] = df['comments_disabled'].astype('category') 
    df['ratings_disabled'] = df['ratings_disabled'].astype('category') 
    df['video_error_or_removed'] = df['video_error_or_removed'].astype('category') 
    
    df['publish_time'] = pd.to_datetime(df['publish_time'],errors='coerce',format='%Y-%m-%dT%H:%M:%S.%fz')
    df['publish_time'] = df['publish_time'].dt.strftime('%Y-%m-%d %H:%M:%S')
 
    date_piece = df['publish_time'].str.split(' ')
    
    df['Publish_Date'] = pd.to_datetime(date_piece.str[0])
    
    df['Publish_Time'] = date_piece.str[1]
    
    del df['publish_time']
    
    
    df = df[['video_id','trending_date','title','channel_title','category_id','Publish_Date',
                 'Publish_Time','tags','views','likes','dislikes','comment_count','thumbnail_link',
                 'comments_disabled','ratings_disabled','video_error_or_removed','description']]
             

    df.set_index('video_id',inplace=True)


all_df[1].dtypes




for df in all_df: #to identify null values, since no color beside black one, hence no missing valus
    sns.heatmap(df.isnull(),cbar=False)
    plt.figure() 
    
    
    
    comb_df = pd.concat(all_df)

backup_df = comb_df.reset_index().sort_values('trending_date',ascending=False).set_index('video_id')

comb_df = comb_df.reset_index().sort_values('trending_date',ascending=False).drop_duplicates('video_id',
          keep='first').set_index('video_id')

for df in all_df:
    df = df.reset_index().sort_values('trending_date',ascending=False).set_index('video_id')
    
comb_df.head()





# read file
with open('US_category_id.json', 'r') as f:  # reading one randomly selected json files to make sense of its contents
    data = f.read()
# parse file
obj = json.loads(data)
# printing
obj


category_id = {}
with open('DE_category_id.json', 'r') as f:
    d = json.load(f)
    for category in d['items']:
        category_id[category['id']] = category['snippet']['title']
comb_df.insert(2, 'category', comb_df['category_id'].map(category_id),allow_duplicates=True)
backup_df.insert(2, 'category', backup_df['category_id'].map(category_id),allow_duplicates=True)
for df in all_df:
    df.insert(2, 'category', df['category_id'].map(category_id),allow_duplicates=True)
    
    
    
comb_df['category'].unique()




# Ratio of likes-dislikes in different categories
likesdf = comb_df.groupby('category')['likes'].agg('sum')
dislikesdf = comb_df.groupby('category')['dislikes'].agg('sum')
ratiodf = likesdf/dislikesdf 
ratiodf = ratiodf.sort_values(ascending=False).reset_index()
ratiodf.columns = ['category','ratio']
plt.subplots(figsize=(10, 15))
sns.barplot(x="ratio", y="category", data=ratiodf,
            label="Likes-Dislikes Ratio", color="b")
comb_df.head()



# Users like videos from which category more?
path = 'C:\\Users\\User\\Data'
allcsv = glob.glob(path + '\*.csv')
countries = []

for csv in allcsv:
    c = csv[19:21]
    countries.append(c)
    
for country in countries:
    if country == 'US':
        tempdf = comb_df[comb_df['Country'] == country]

for country in countries:
    if country == 'IN':
        tempdf = comb_df[comb_df['Country'] == country]['category'].value_counts().reset_index()

        ax = sns.barplot(y=tempdf['index'], x=tempdf['category'], data=tempdf, orient='h')
        plt.xlabel("Number of Videos")
        plt.ylabel("Categories")
        plt.title("Catogories of trend videos in " + country)
    else:
        tempdf = comb_df[comb_df['Country']==country]['category'].value_counts().reset_index()
        ax = sns.barplot(y=tempdf['index'], x=tempdf['category'], data=tempdf, orient='h')
        plt.xlabel("Number of Videos")
        plt.ylabel("Categories")
        plt.title("Catogories of trend videos in " + country)
        plt.show()
        
        
        
        
        
        # Top 5 videos that are on trending in each country
temporary = []
for df in all_df:
    temp= df
    temp = temp.reset_index().sort_values('views',ascending=False)
    temp.drop_duplicates('video_id',keep='first',inplace=True)
    temp.set_index('video_id',inplace=True)
    temporary.append(temp)
temporary[2].head()




# Is the most liked video also the most trending video
temporary = []

for df in all_df:
    temp=df
    temp = temp.reset_index().sort_values('likes',ascending=False)
    temp.drop_duplicates('video_id',keep='first',inplace=True)
    temp.set_index('video_id',inplace=True)
    temporary.append(temp)
    
    
    
    
    # Maximum number of days to trending status for a video
comb_df['timespan'] = (comb_df['trending_date'] - comb_df['Publish_Date']).dt.days
to_trending = df.sample(1000).groupby('video_id')['timespan'].max()

#sns_ax = sns.boxplot(y = to_trending)
sns_ax.set(yscale = "log")
#plt.show()
#sns.distplot(to_trending.value_counts(),bins='rice',kde=False)




# Users like videos from which category more

temp = comb_df

temp = temp.groupby('category')['likes','views'].sum().sort_values(by='likes',ascending=False)
temp.head()




# Users comment on which category the most

temp = comb_df

temp = temp.groupby('category')['likes','views','comment_count'].sum().sort_values(by='comment_count',ascending=False)
temp.head()




# Correlation between views, likes, dislikes, and commentscol = ['views', 'likes', 'dislikes', 'comment_count']
corr = comb_df[col].corr()
corr
plt.figure(figsize=(7,7))
sns.heatmap(corr,annot=True,cmap='coolwarm',linewidth=1,linecolor='w',square=True)






    
    
