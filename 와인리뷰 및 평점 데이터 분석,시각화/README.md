# Kaggle 와인 데이터 시각화

```python
import pandas as pd
import nltk
from nltk import word_tokenize,sent_tokenize
from nltk.tokenize import RegexpTokenizer
from nltk.tag import pos_tag
import matplotlib.pyplot as plt
import seaborn as sns
from wordcloud import WordCloud
```

```python
## 데이터 읽기
df = pd.read_csv('winemag-data-130k-v2.csv')
df
```
<img width="745" alt="image" src="https://user-images.githubusercontent.com/75477273/167278468-27d64816-4479-46b4-b4fe-3eee86e65ee6.png">


```python
## 필요없는 Unnamed: 0열 삭제
df.drop(columns=['Unnamed: 0'],inplace=True)
df.head()
```
<img width="735" alt="image" src="https://user-images.githubusercontent.com/75477273/167278492-403a6ef4-fdf5-42fe-b610-78fd1fb9e8dc.png">

```python
df['country'].value_counts()
```
<img width="388" alt="image" src="https://user-images.githubusercontent.com/75477273/167278506-143eff8f-0d18-4618-9e4f-38f74ae453d4.png">

```python
## 1000개 이하 데이터 삭제 => 그 이하는 데이터가 적어 리뷰와 점수에 대한 명확성이 떨어진다고 생각
a = pd.DataFrame(df['country'].value_counts())>1000
a.reset_index(inplace=True)
b = a[a['country']==False]
b['index']
```
<img width="236" alt="image" src="https://user-images.githubusercontent.com/75477273/167278538-dd9e9ca1-624a-4c2a-826b-f985c2bdc7a3.png">

```python
for i in range(len(b)):
    idx = df[df['country']== b['index'].iloc[i]].index
    df = df.drop(idx)
df.country.value_counts()
```
<img width="199" alt="image" src="https://user-images.githubusercontent.com/75477273/167278545-a09c26a3-e22a-4ed6-bd27-5e038606cbe0.png">

## 나라별 분석
```python
df_country = df.groupby(['country']).mean().reset_index()
df_country
```
<img width="268" alt="image" src="https://user-images.githubusercontent.com/75477273/167278558-04bb16f2-7632-4061-8584-28cb8dd64b4c.png">

```python
fig = plt.figure(figsize=(25,10))
sns.set_style("darkgrid")
ax1 = fig.add_subplot(1,2,1)  ## (행갯수,열갯수,순서)
ax2 = fig.add_subplot(1,2,2)
sns.pointplot(x = df_country['country'] ,y = df_country.points,color='blue',ax=ax1)
sns.pointplot(x = df_country['country'] ,y = df_country.price,color='skyblue',ax=ax2)
```
<img width="746" alt="image" src="https://user-images.githubusercontent.com/75477273/167278569-db7b5c15-b058-466a-be4f-cbdcd40df3a9.png">
## 품종분석

```python
df_variety = df.groupby(['variety']).mean().reset_index()
df_variety
```
<img width="326" alt="image" src="https://user-images.githubusercontent.com/75477273/167278594-104002e6-0470-4035-a45e-71382993c946.png">

```python
fig = plt.figure(figsize=(25,10))
sns.set_style("darkgrid")
ax1 = fig.add_subplot(1,2,1)  ## (행갯수,열갯수,순서)
ax2 = fig.add_subplot(1,2,2)
sns.pointplot(x = df_variety['variety'] ,y = df_variety.points,color='blue',ax=ax1)
sns.pointplot(x = df_variety['variety'] ,y = df_variety.price,color='skyblue',ax=ax2)
```
<img width="748" alt="image" src="https://user-images.githubusercontent.com/75477273/167278616-d325a6a8-43fe-4806-8ac3-3473a1d97e29.png">


## 점수로 나타내어 비교하기
``` python
df_variety = df_variety.sort_values(['points','price'],ascending=False).head(20)
df_variety['quality_point'] = df_variety['points']/df_variety['price']
df_variety.sort_values(['quality_point'],ascending=False)
```
<img width="501" alt="image" src="https://user-images.githubusercontent.com/75477273/167278638-c86887ac-5818-47dc-a9ce-01dd255c378b.png">

```python
df[df['variety']=='Blauburgunder']
```
<img width="734" alt="image" src="https://user-images.githubusercontent.com/75477273/167278663-87be590c-36e7-485d-b57b-1bc891197e52.png">

```python
df[df['variety']=='Sirica']
```

<img width="736" alt="image" src="https://user-images.githubusercontent.com/75477273/167278670-58ab6fc8-2d27-4291-8a2f-88f4b22294d8.png">


```python
df[df['variety']=='Roviello']
```
<img width="742" alt="image" src="https://user-images.githubusercontent.com/75477273/167278673-b4cb87aa-3343-4c8b-a689-54c27b078159.png">

```python
df[df['variety']=='Gelber Traminer']
```
<img width="724" alt="image" src="https://user-images.githubusercontent.com/75477273/167278680-48086caa-4c9a-4f62-bc05-7e05f764551e.png">
이탈리아와 오스트리아에서 나오는 품종으로 만든 와인이 가성비가 좋은걸 볼수 있음

### 앞그래프와 비교했을 떄 오스트리아 와인이 가장 가성비가 높아 보인다

## 리뷰(맛 정보) 시각화

```python
county_name = set(df.country.values)
county_name
```
{'Argentina',
 'Australia',
 'Austria',
 'Chile',
 'France',
 'Germany',
 'Italy',
 'New Zealand',
 'Portugal',
 'South Africa',
 'Spain',
 'US',
 nan}
 
 ```python
 dic={}
 var_index={}

#ls는 2차원 list로 초기화 한다.
ls = [ [] for i in  range(len(county_name))] # variety 개수 만큼 리스트 추가

# var_index dic을 만든다.
idx_i = 0 
for i in county_name:
  var_index[i] = idx_i
  idx_i = idx_i +1 
var_index
 ```
 {nan: 0,
 'Portugal': 1,
 'Australia': 2,
 'New Zealand': 3,
 'Spain': 4,
 'Argentina': 5,
 'Germany': 6,
 'US': 7,
 'Chile': 8,
 'Italy': 9,
 'Austria': 10,
 'France': 11,
 'South Africa': 12}
 
 ```python
 for j in range(len(df)):
    country_name = df.iloc[j].country
    kw_idx =   var_index[country_name]
    ls[kw_idx].append(df.iloc[j].description)
 #앞에서 만들어진 list를 dic에 만들어 준다.
for i in county_name:
    ls_idx = var_index[i] # 해당 list의 위치를 얻어냄
    dic[i] = ls[ls_idx]
 ```
 
 
 ```python
des_df = pd.DataFrame(dic.items(),
                   columns=['county', 'description'])
des_df
 ```
<img width="402" alt="image" src="https://user-images.githubusercontent.com/75477273/167278871-2cafeca0-7e1a-438d-983e-6142acbf5685.png">

```python
## nan 값 삭제
des_df.drop(index=0,inplace=True)
des_df.reset_index(drop=True,inplace=True)
des_df
```
<img width="430" alt="image" src="https://user-images.githubusercontent.com/75477273/167278879-96d79fff-0b84-449e-8d90-da1d92ee2e69.png">

```python
## 맛에 관련되어 있는 명사와 형용사만 남김
des_list = []
for i in range(len(des_df['description'])):
    taged_list = pos_tag(word_tokenize(str(des_df['description'][i])))
    nouns_list = [t[0] for t in taged_list if t[1] == "NN" or  t[1] == "JJ"]
    des_list.append(nouns_list)
print(des_list)
```
['[', 'ripe', 'fruity', 'wine', 'Firm', 'juicy', 'red', 'berry', 'acidity', 'drinkable', 'sandy', 'soil', 'wine', 'soft', 'open', 'accessible—with', 'black', 'light', 'grape', 'ready', 'bottling', 'rich', 'wood-aged', 'wine', 'full', 'ripe', 'yellow', 'tropical', 'rich', 'toasty', 'wine', 'warm', 'spicy', 'impressive', 'structure', 'potential', 'age', 'wine', 'second', 'generation', 'charge', 'estate', 'Drink', 'estate', 'south', 'rich', 'hot-country', 'wine', 'firm', 'solid', 'dark', 'juicy', 'black', 'fruit', 'background', 'dense', 'wine', 'lift', 'young', 'dark', 'concentrated', 'wine', 'year', 'wood', 'bottle', 'release', 'attractive', 'wine', 'solid', 'smooth', 'texture', 'concentration', 'bright', 'black', 'currant', 'fruit', 'vibrant', 'acidity', 'Ready', 'ripe', 'blend', 'richness', 'density', 'full', 'black', 'fruit', 'toast', 'touch', 'pepper', 'ripe', 'crisp', 'acidity', 'age', 'year', 'powerful', 'wine', 'full', 'ripe', 'solid', 'feel', 'dark', 'juicy', 'berry', 'blend', 'serious', 'aging', 'herb-dominant', 'wine', 'texture', 'creamy', 'character', 'green', 'acidity', 'bright', 'orange-zest', 'aftertaste', 'light', 'prickle', 'tongue', 'fresh', 'possible', 'clean', 'green', 'citrus', 'textured', 'richness', 'new', 'wine', 'master', 'winemaker', 'rich', 'black', 'dense', 'wine', 'layer', 'complexity', 'dark', 'name', 'selection', 'rich', 'smooth', 'full', 'ripe', 'fruit', 'dusty', 'suspension', 'black', 'plum', 'blueberry', 'fruit', 'juicy', 'acidity', 'wine', 'age', '.....

```python
des_df['description_word'] = des_list
des_df
```
<img width="652" alt="image" src="https://user-images.githubusercontent.com/75477273/167278886-262e2ae7-45a4-4ffa-9b96-26d7c845772c.png">
```python
## 이중 리스트 삭제
ls = []
for i in range(len(des_df)):
    a = des_df.iloc[i]['description_word']
    b = ' '.join(filter(str.isalnum, a)) 
    b = b.split(' ')
    ls.append(b)
des_df['description_word'] = ls
des_df
```
<img width="700" alt="image" src="https://user-images.githubusercontent.com/75477273/167278895-719ce5b1-1970-4490-9e84-b5226374cd27.png">

```python
## wordcloud 시각화
for i in range(len(des_df)):
    review_wordcloud = WordCloud(width=400,height=300,background_color='white').generate(str(des_df['description_word'][i]))
    plt.imshow(review_wordcloud, interpolation='bilinear')
    plt.axis('off')
    plt.title(str(des_df['county'][i]))
    plt.show()
```
![download](https://user-images.githubusercontent.com/75477273/167278827-556ee951-5a55-4d91-b2b8-6f7039c27f3f.png)
![download](https://user-images.githubusercontent.com/75477273/167278828-86ae19bc-51c4-413b-b756-9db9f33e4849.png)
![download](https://user-images.githubusercontent.com/75477273/167278829-a5812400-e1c4-4e83-b887-2814f3b5807d.png)
![download](https://user-images.githubusercontent.com/75477273/167278834-8336bf0b-7a89-4cce-8700-a09e76c62343.png)
![download](https://user-images.githubusercontent.com/75477273/167278836-59d51d33-4c2b-45e4-840c-33cacea92ce3.png)
![download](https://user-images.githubusercontent.com/75477273/167278837-58dc1a79-9cbc-4ed0-a75c-b10f5e8565d3.png)
![download](https://user-images.githubusercontent.com/75477273/167278838-f3728cd1-eb0b-4baf-8949-8f58f3da483f.png)
![download](https://user-images.githubusercontent.com/75477273/167278839-8706617f-29ea-41be-aef3-735efb10010a.png)
![download](https://user-images.githubusercontent.com/75477273/167278842-4238a310-27b9-4bd0-a49f-7d1be115760d.png)
![download](https://user-images.githubusercontent.com/75477273/167278844-01e782f6-a4b6-421c-9855-ff763bcc496e.png)
![download](https://user-images.githubusercontent.com/75477273/167278845-1eeefc37-d2f4-40a4-8876-9c156d87a3bb.png)
![download](https://user-images.githubusercontent.com/75477273/167278846-834ac8e6-3d3a-484f-9d0e-a10f5fdc8f89.png)
