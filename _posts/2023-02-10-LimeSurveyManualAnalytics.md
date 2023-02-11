---
layout: post
title:  "Automating Lime Survey Analytics with Python"
date:   2023-02-10 12:07:25 +0000
categories:
  - data
---


# Automating Analytics from a Lime Survey Manually Featuring Python 

Survey data is important to organizations because it provides valuable insights into the attitudes, opinions, behaviors, and needs of their target audience or customers. This information can be used to make informed decisions, improve products and services, identify areas for improvement, and develop effective marketing strategies. Surveys can also help organizations measure customer satisfaction, track trends over time, and benchmark their performance against competitors. Overall, survey data can provide a deeper understanding of the market and help organizations make data-driven decisions to drive business success.

There are a variety of analytics tools and approaches that organizations can use to exploit survey data. Here are a few examples:

<ol>
    <li>Descriptive analytics: This approach involves summarizing the survey data to provide insights into the distribution and characteristics of the responses. Descriptive analytics can be used to identify patterns, trends, and key metrics such as response rates, satisfaction scores, or net promoter scores.</li>

    <li>Inferential analytics: This approach involves using statistical techniques to infer relationships or differences between variables in the survey data. Inferential analytics can be used to identify significant differences between groups, perform hypothesis testing, or build predictive models.</li>

    <li>Text analytics: This approach involves analyzing the open-ended responses in the survey data to extract themes, sentiments, and opinions. Text analytics can be used to identify customer needs, pain points, or preferences that are not captured by closed-ended questions.</li>

    <li>Data visualization tools: This approach involves using charts, graphs, or dashboards to visualize the survey data and communicate the insights effectively. Data visualization can help identify trends, outliers, or patterns that are not apparent in the raw data.</li>
</ol>

Overall, the choice of analytics tools and approaches depends on the objectives of the survey, the type of data collected, and the expertise of the analysts.

In this post I will go over some basic automation of survey data extraction and visualization using Python & jupyter. The key libraries needed to acheive the same results are pandas, numpy, seaborn, the standard re library and everyone's favourite... you guessed it, matplotlib! 

Let's get started :)

## Start by importing necessary libraries


```python
import pandas as pd
import seaborn as sns
import numpy as np
import re
import matplotlib.pyplot as plt
```

## Load Rough Data into DataFrame


```python
dataframe = pd.read_excel('example_survey.xlsx',sheet_name="Rough data (online)")
```


```python
region_country = pd.read_excel('example_survey.xlsx',sheet_name='Follow-up',header=2).iloc[:,0:2].to_numpy()
```

### Create a Country -> Region Mapping to be used for Region Pie Chart


```python
country_region_lookup = {}
for i in range(len(region_country[:,0])):
    country_region_lookup[str(region_country[i,1]).lower().strip()] = str(region_country[i,0]).upper().strip()

```

### Get Participating Countries 
<ol>
    <li> Get Country Column from dataframe</li>
    <li> Augment participant dataframe with Region of Each Country from lookup table (previous step)</li>
    <li> Get Value Count of each country  </li>
    <li> Get sum of all participants by region</li>
    <li> Convert value counts to percentages using sum  </li>
    <li> Plot pie chart </li>
</ol>



```python
participating_countries = dataframe.filter(like='Country')
participating_countries["Region"] = participating_countries.iloc[:,0].apply(lambda x: country_region_lookup[x.lower()])
value_counts = participating_countries['Region'].value_counts()
value_sum = value_counts.sum()
percentages = (value_counts / value_sum) * 100
plt.pie(percentages,labels=[f'{num:.2g}%' for num in percentages])
plt.legend(participating_countries['Region'].value_counts().index)
plt.title('Respondents by Region')
```

    
![png](/assets/images/LimeSurveyAnalytics_files/LimeSurveyAnalytics_9_2.png)
    


## Perform Question Analytics
<ol>
    <li> Save Question number as text </li>
    <li> Use regular expressions to match the column names with question number</li>
    <li> Get rid of columns corresponding to question that contain comment answers</li>
</ol>


```python
query = 'G00Qxx'
columns = [re.fullmatch(f'{query}.*',d) for d in dataframe.columns]
columns = [x for x in columns if x is not None]
columns = [x.group() for x in columns]
columns = [x for x in columns if 'comment]' not in x]
```

### Capture Counts corresponding to each answer and store in lookup table for plotting


```python
question_yes_table = {}
for column in columns:
    yeses = len(dataframe[dataframe[column]=='Yes'].filter(like='Country'))
    i = column.rfind('[')
    answer = column[i:]
    question_yes_table[answer] = yeses
```

### Load lookup table in dataframe so we can use seaborn's fancy plotting


```python
df = pd.DataFrame(question_yes_table,index=[0])
ax = sns.barplot(data=df,orient='h')
plt.xticks(rotation="vertical")
sns.despine(left=True, bottom=True)
plt.title('Bar Plot for Question Analytics')
ax.set_yticklabels([f'Answer {i}' for i in range(len(df.columns))])
plt.show()
```


    
![png](/assets/images/LimeSurveyAnalytics_files/LimeSurveyAnalytics_15_0.png)
    
