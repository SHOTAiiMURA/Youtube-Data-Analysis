# Youtube-Data-Analysis
![image](https://github.com/user-attachments/assets/b467f889-56c4-4300-9a1b-1f45db19c61b)

# Contents

# Introduction

# Requirement

```python
import pandas as pd
import seaborn as se
import matplotlib.pyplot as plt
from googleapiclient.discovery import build
```
```python
# replace with your own API key
API_KEY = 'AIzaSyAF94pq5ThP1lE4GO5bLcknPRsQRPP0a0Y'

def get_trending_videos(api_key, max_results=200):
    # build the youtube service
    youtube = build('youtube', 'v3', developerKey=api_key)

    # initialize the list to hold video details
    videos = []

    # fetch the most popular videos
    request = youtube.videos().list(
        part='snippet,contentDetails,statistics',
        chart='mostPopular',
        regionCode='US',  
        maxResults=50
    )

    # paginate through the results if max_results > 50
    while request and len(videos) < max_results:
        response = request.execute()
        for item in response['items']:
            video_details = {
                'video_id': item['id'],
                'title': item['snippet']['title'],
                'description': item['snippet']['description'],
                'published_at': item['snippet']['publishedAt'],
                'channel_id': item['snippet']['channelId'],
                'channel_title': item['snippet']['channelTitle'],
                'category_id': item['snippet']['categoryId'],
                'tags': item['snippet'].get('tags', []),
                'duration': item['contentDetails']['duration'],
                'definition': item['contentDetails']['definition'],
                'caption': item['contentDetails'].get('caption', 'false'),
                'view_count': item['statistics'].get('viewCount', 0),
                'like_count': item['statistics'].get('likeCount', 0),
                'dislike_count': item['statistics'].get('dislikeCount', 0),
                'favorite_count': item['statistics'].get('favoriteCount', 0),
                'comment_count': item['statistics'].get('commentCount', 0)
            }
            videos.append(video_details)

        # get the next page token
        request = youtube.videos().list_next(request, response)

    return videos[:max_results]

def save_to_csv(data, filename):
    df = pd.DataFrame(data)
    df.to_csv(filename, index=False)

def main():
    trending_videos = get_trending_videos(API_KEY)
    filename = 'trending_videos.csv'
    save_to_csv(trending_videos, filename)
    print(f'Trending videos saved to {filename}')

if __name__ == '__main__':
    main()
```
```python
trending_youtube=pd.read_csv("trending_videos.csv")
print(trending_youtube.head())
```
<img width="654" alt="Screenshot 2024-09-23 at 20 42 52" src="https://github.com/user-attachments/assets/6077529a-1b6b-4373-8d44-8bf42fa290c7">

```python
s#checking for missing values
missing_values = trending_youtube.isnull().sum()
#display data types
data_types = trending_youtube.dtypes

missing_values, data_types
```
<img width="238" alt="Screenshot 2024-09-23 at 20 43 17" src="https://github.com/user-attachments/assets/8d41612f-e854-4c67-86d2-df7df9057e9e">

```python
# fill missing descriptions with "No description"
trending_youtube['description'].fillna('No description', inplace=True)

# convert `published_at` to datetime
trending_youtube['published_at'] = pd.to_datetime(trending_youtube['published_at'])

# convert tags from string representation of list to actual list
trending_youtube['tags'] = trending_youtube['tags'].apply(lambda x: eval(x) if isinstance(x, str) else x)
```
```python
# descriptive statistics
descriptive_stats = trending_youtube[['view_count', 'like_count', 'dislike_count', 'comment_count']].describe()

descriptive_stats
```
<img width="486" alt="Screenshot 2024-09-23 at 20 43 57" src="https://github.com/user-attachments/assets/d83db366-2651-4418-924e-5ac3b16541ff">

```python
#look at the distribution of views, likes and comments of all the videos in the data:
se.set(style="whitegrid")

fig, axes = plt.subplots(1, 3, figsize=(18, 5))

# view count distribution
se.histplot(trending_youtube['view_count'], bins=30, kde=True, ax=axes[0], color='blue')
axes[0].set_title('View Count Distribution')
axes[0].set_xlabel('View Count')
axes[0].set_ylabel('Frequency')

# like count distribution
se.histplot(trending_youtube['like_count'], bins=30, kde=True, ax=axes[1], color='green')
axes[1].set_title('Like Count Distribution')
axes[1].set_xlabel('Like Count')
axes[1].set_ylabel('Frequency')

# comment count distribution
se.histplot(trending_youtube['comment_count'], bins=30, kde=True, ax=axes[2], color='red')
axes[2].set_title('Comment Count Distribution')
axes[2].set_xlabel('Comment Count')
axes[2].set_ylabel('Frequency')

plt.tight_layout()
plt.show()
```
<img width="740" alt="Screenshot 2024-09-23 at 20 44 19" src="https://github.com/user-attachments/assets/d9de5dc4-0b11-4aae-bb1f-63a7720a62ce">

```python
# correlation matrix
correlation_matrix = trending_youtube[['view_count', 'like_count', 'comment_count']].corr()

plt.figure(figsize=(8, 6))
se.heatmap(correlation_matrix, annot=True, cmap='coolwarm', linewidths=0.5, linecolor='black')
plt.title('Correlation Matrix of Engagement Metrics')
plt.show()
```
<img width="640" alt="Screenshot 2024-09-23 at 20 44 41" src="https://github.com/user-attachments/assets/d681ae64-33e0-46d7-8ca3-4759d4080258">

```python
#collect the category names as well to analyze the category of the trending videos
API_KEY = 'AIzaSyAF94pq5ThP1lE4GO5bLcknPRsQRPP0a0Y'
youtube = build('youtube', 'v3', developerKey=API_KEY)

def get_category_mapping():
    request = youtube.videoCategories().list(
        part='snippet',
        regionCode='US'
    )
    response = request.execute()
    category_mapping = {}
    for item in response['items']:
        category_id = int(item['id'])
        category_name = item['snippet']['title']
        category_mapping[category_id] = category_name
    return category_mapping

# get the category mapping
category_mapping = get_category_mapping()
print(category_mapping)
```
<img width="733" alt="Screenshot 2024-09-23 at 20 45 06" src="https://github.com/user-attachments/assets/e3928896-600e-4bbe-b160-a7e170b91fba">

```python
#analyze the number of trending videos by category
trending_youtube['category_name'] = trending_youtube['category_id'].map(category_mapping)

# Bar chart for category counts
plt.figure(figsize=(12, 8))
se.countplot(y=trending_youtube['category_name'], order=trending_youtube['category_name'].value_counts().index, palette='viridis')
plt.title('Number of Trending Videos by Category')
plt.xlabel('Number of Videos')
plt.ylabel('Category')
plt.show()
```
<img width="739" alt="Screenshot 2024-09-23 at 20 45 25" src="https://github.com/user-attachments/assets/95309618-4b46-43f6-9143-373e5266fa18">

```python
#look at the average engagement metrics by category:
# average engagement metrics by category
category_engagement = trending_youtube.groupby('category_name')[['view_count', 'like_count', 'comment_count']].mean().sort_values(by='view_count', ascending=False)

fig, axes = plt.subplots(1, 3, figsize=(18, 10))

# view count by category
se.barplot(y=category_engagement.index, x=category_engagement['view_count'], ax=axes[0], palette='viridis')
axes[0].set_title('Average View Count by Category')
axes[0].set_xlabel('Average View Count')
axes[0].set_ylabel('Category')

# like count by category
se.barplot(y=category_engagement.index, x=category_engagement['like_count'], ax=axes[1], palette='viridis')
axes[1].set_title('Average Like Count by Category')
axes[1].set_xlabel('Average Like Count')
axes[1].set_ylabel('')

# comment count by category
se.barplot(y=category_engagement.index, x=category_engagement['comment_count'], ax=axes[2], palette='viridis')
axes[2].set_title('Average Comment Count by Category')
axes[2].set_xlabel('Average Comment Count')
axes[2].set_ylabel('')

plt.tight_layout()
plt.show()
```
<img width="737" alt="Screenshot 2024-09-23 at 20 45 49" src="https://github.com/user-attachments/assets/36992fde-fc9a-4567-b3bc-b70f44dd57f1">

```python
!pip install isodate
import isodate

# convert ISO 8601 duration to seconds
trending_youtube['duration_seconds'] = trending_youtube['duration'].apply(lambda x: isodate.parse_duration(x).total_seconds())

trending_youtube['duration_range'] = pd.cut(trending_youtube['duration_seconds'], bins=[0, 300, 600, 1200, 3600, 7200], labels=['0-5 min', '5-10 min', '10-20 min', '20-60 min', '60-120 min'])
```
<img width="732" alt="Screenshot 2024-09-23 at 20 46 29" src="https://github.com/user-attachments/assets/bc485019-cf18-4566-9ccf-c8da491e2a42">

```python
#analyze the content and the duration of videos:
# scatter plot for video length vs view count
plt.figure(figsize=(10, 6))
se.scatterplot(x='duration_seconds', y='view_count', data=trending_youtube, alpha=0.6, color='purple')
plt.title('Video Length vs View Count')
plt.xlabel('Video Length (seconds)')
plt.ylabel('View Count')
plt.show()

# bar chart for engagement metrics by duration range
length_engagement = trending_youtube.groupby('duration_range')[['view_count', 'like_count', 'comment_count']].mean()

fig, axes = plt.subplots(1, 3, figsize=(18, 8))

# view count by duration range
se.barplot(y=length_engagement.index, x=length_engagement['view_count'], ax=axes[0], palette='magma')
axes[0].set_title('Average View Count by Duration Range')
axes[0].set_xlabel('Average View Count')
axes[0].set_ylabel('Duration Range')

# like count by duration range
se.barplot(y=length_engagement.index, x=length_engagement['like_count'], ax=axes[1], palette='magma')
axes[1].set_title('Average Like Count by Duration Range')
axes[1].set_xlabel('Average Like Count')
axes[1].set_ylabel('')

# comment count by duration range
se.barplot(y=length_engagement.index, x=length_engagement['comment_count'], ax=axes[2], palette='magma')
axes[2].set_title('Average Comment Count by Duration Range')
axes[2].set_xlabel('Average Comment Count')
axes[2].set_ylabel('')

plt.tight_layout()
plt.show()
```
<img width="726" alt="Screenshot 2024-09-23 at 20 46 57" src="https://github.com/user-attachments/assets/b5e27fd9-ab7c-41cd-8c08-2eb2caae1848">
<img width="736" alt="Screenshot 2024-09-23 at 20 47 14" src="https://github.com/user-attachments/assets/d2e522eb-15ae-46af-9cd9-315bf2a7f22f">

```python
#analyze the relationship between views and number of tags used in the video:
# calculate the number of tags for each video
trending_youtube['tag_count'] = trending_youtube['tags'].apply(len)

# scatter plot for number of tags vs view count
plt.figure(figsize=(10, 6))
se.scatterplot(x='tag_count', y='view_count', data=trending_youtube, alpha=0.6, color='orange')
plt.title('Number of Tags vs View Count')
plt.xlabel('Number of Tags')
plt.ylabel('View Count')
plt.show()
```
<img width="726" alt="Screenshot 2024-09-23 at 20 47 40" src="https://github.com/user-attachments/assets/b58818f7-f804-4e4d-937c-9fd9cac40d3f">

```python
#if there’s an impact of the time a video is posted on its views:
# extract hour of publication
trending_youtube['publish_hour'] = trending_youtube['published_at'].dt.hour

# bar chart for publish hour distribution
plt.figure(figsize=(12, 6))
se.countplot(x='publish_hour', data=trending_youtube, palette='coolwarm')
plt.title('Distribution of Videos by Publish Hour')
plt.xlabel('Publish Hour')
plt.ylabel('Number of Videos')
plt.show()

# scatter plot for publish hour vs view count
plt.figure(figsize=(10, 6))
se.scatterplot(x='publish_hour', y='view_count', data=trending_youtube, alpha=0.6, color='teal')
plt.title('Publish Hour vs View Count')
plt.xlabel('Publish Hour')
plt.ylabel('View Count')
plt.show()
```
<img width="734" alt="Screenshot 2024-09-23 at 20 48 03" src="https://github.com/user-attachments/assets/dc19471c-18b8-454c-a4bd-bf047064e09f">
<img width="735" alt="Screenshot 2024-09-23 at 20 48 19" src="https://github.com/user-attachments/assets/50ca3648-63e3-4fbd-aa76-b4a61ec52885">

# Summary

# Future development
