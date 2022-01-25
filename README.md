# changwonaccident
창원시 3년 교통사고분석
창원시 교통 지역과 시간별 분석
현재 살고 있는 창원시 안에서 일어나는 교통사고 데이터를 분석
가설 1. 교통사고가 가장 많이 발생 하는 시간은 출퇴근 시간인 6시 ~ 7시 사이에 많을 것이다.

가설 2. 사고가 가장 많이 발생하는 요일은 회식과 약속이 많은 금요일에 많을 것이고 그에 따라 팔용동과 중앙동 상남동이 사고가 많이 날 것이다.

자료 출저: 교통사고분석시스템 GIS('http://taas.koroad.or.kr/gis/mcm/mcl/initMap.do?menuId=GIS_GMP_STS_RSN')

```python
# 최근 3년 이내 창원에서 일어난 교통사고 데이터
import pandas as pd
df = pd.read_excel('accidentInfoList.xlsx')
# 데이터 요약
df.head()
```
![image](https://user-images.githubusercontent.com/75477273/150948455-3a09e1b0-e1f9-4d30-baf1-b7f8a0a02fce.png)


```python
# 사고일시에서 시간남 남기기
time = df['사고일시']
a = []
for i in time:
    b = i.split()[3]
    a.append(b)
df['사고일시'] = a
df.head()
```
![image](https://user-images.githubusercontent.com/75477273/150948717-9b18d28b-921a-4fce-b55a-88a344f4e832.png)

```python
count={}
for i in a:
    try: count[i] += 1
    except: count[i]=1

dict = sorted(count.items(), key=lambda x:x[1])

dict.reverse()
dict
```
![image](https://user-images.githubusercontent.com/75477273/150948825-9a03d5d0-341e-472f-b5fd-326f67d6e5cb.png)

```python
# 시간대 그래프 표현
import matplotlib.pyplot as plt
plt.rcParams['font.family'] = 'Malgun Gothic'
timelist = count.items()
timelist = sorted(timelist)
x,y = zip(*timelist)

plt.figure(figsize=(15,10))
plt.bar(x,y)
plt.xlabel('시간')
plt.ylabel('사고 횟수')
plt.show()
```
![image](https://user-images.githubusercontent.com/75477273/150948989-e2229cb5-4343-4264-bc9d-391f829a79d8.png)
예상대로 교통량이 가장 많은 퇴근시간대인 17시에서 19시 사이에 가장 많은 교통사고 발생
``` python
# 요일별 분석
day = df['요일']
daylist = []
for i in day:
    daylist.append(i)
day_count={}
for i in daylist:
    try: day_count[i] += 1
    except: day_count[i]=1
dict = sorted(day_count.items(), key=lambda x:x[1])

dict.reverse()
dict

[('금요일', 542),
 ('수요일', 534),
 ('월요일', 530),
 ('목요일', 519),
 ('화요일', 493),
 ('토요일', 491),
 ('일요일', 354)]
 ```
 
 ```python
 day_list = day_count.items()
x,y = zip(*day_list)
plt.figure(figsize=(15,10))
plt.bar(x,y)
plt.xlabel('요일')
plt.ylabel('사고 횟수')
plt.show()
```
![image](https://user-images.githubusercontent.com/75477273/150949182-1369a10b-840c-4bb3-ab4b-a77b53a1b275.png)
예상대로 금요일이 가장 많이 발생하지만 주말에 사고가 많이 발생 한다는 가정은 틀림

예상: 출근의 피로함과 금요일 회식등으로 인해 주말에는 교통량이 감소한다고 생각

```python
si = df['시군구']
a = 0
d = []
for i in si:
    c = i.split()[3]
    d.append(c)
count={}
for i in d:
    try: count[i] += 1
    except: count[i]=1
dict = sorted(count.items(), key=lambda x:x[1])

dict.reverse()
df1 = pd.DataFrame(dict,columns=['동','횟수'])
df1

	동	횟수
0	상남동	180
1	용원동	135
2	팔용동	122
3	내서읍	100
4	양덕동	96
...	...	...
155	속천동	1
156	대장동	1
157	대외동	1
158	대천동	1
159	삼동동	1
```
```python
timelist = count.items()
timelist = sorted(timelist)
x,y = zip(*timelist)

plt.figure(figsize=(400,25))
plt.bar(x,y)
plt.xlabel('동이름')
plt.ylabel('사고 횟수')
plt.show()
```
![image](https://user-images.githubusercontent.com/75477273/150949504-8a753821-304d-4740-9977-d727c84803c9.png)


```python
## 지도로 보시 쉽게 표현
## 시군구 json파일 형식으로 변경

df['도'] = [eachAddress.split()[0] for eachAddress in df['시군구']]
df['시'] = [eachAddress.split()[1] for eachAddress in df['시군구']]
df['구'] = [eachAddress.split()[2] for eachAddress in df['시군구']]
df['동'] = [eachAddress.split()[3] for eachAddress in df['시군구']]
df['연도별'] = [eachAddress.split()[0] for eachAddress in df['사고일시']]
df["시군구"] = df["도"]+ " " + df["시"] + df["구"] + " " + df["동"] 

import json
import folium
import googlemaps

import requests, json
url = requests.get("https://raw.githubusercontent.com/vuski/admdongkor/master/ver2021xxxx_for%20update/HangJeongDong_ver20210701.geojson")
text = url.text
print(type(text))
geo_str = json.loads(text)
geo_str
gmap_key = '000000000000000'
gmaps = googlemaps.Client(key=gmap_key)
# 시군구에 있는 주소로 위도, 경도 불러옴
# try-except구문을 사용하여 try구문 실행하다가 에러가 나면 except구문에서 지정된 코드를 실행하게 되는데 이 경우는 NaN을 저장
from tqdm import tqdm_notebook

lat = []
lng = []

for n in tqdm_notebook(df.index):
    try:
        tmp_add = str(df['시군구'][n]).split('(')[0]
        tmp_map = gmaps.geocode(tmp_add)
        
        tmp_loc = tmp_map[0].get('geometry')
        lat.append(tmp_loc['location']['lat'])
        lng.append(tmp_loc['location']['lng'])
        
    except:
        lat.append(np.nan)
        lng.append(np.nan)
        print('Here is nan !')
        
df['lat']=lat
df['lng']=lng
df
```
![image](https://user-images.githubusercontent.com/75477273/150949727-191759e7-8047-46f7-a8cb-5acf6bae422c.png)

```python
# 지도 중앙점 표시
print(df['lat'].mean())
print(df['lng'].mean())
count={}
for i in df['시군구']:
    try: count[i] += 1
    except: count[i]=1
dict = sorted(count.items(), key=lambda x:x[1])

dict.reverse()
df_1 = pd.DataFrame(dict,columns=['시군구','횟수'])
df_1.head()
# sigungu_data = pd.pivot_table(df, index=['시군구'], values=['사고일시'],aggfunc='count')
# sigungu_data
map = folium.Map(location=[df['lat'].mean(),df['lng'].mean()], zoom_start=10.5)

map.choropleth(geo_data = geo_str, data= df_1, columns= ['시군구','횟수'],fill_color='YlGn', 
               key_on='feature.properties.adm_nm', fill_opacity=0.6, line_opacity=0.2)
map
```
![image](https://user-images.githubusercontent.com/75477273/150949895-e75994cb-09d1-47ab-96c5-691da6b70d31.png)
```python
map = folium.Map(location=[df['lat'].mean(),df['lng'].mean()], zoom_start=11, tiles='openstreetmap')

from folium import Marker
from folium.plugins import MarkerCluster

mc= MarkerCluster()
for _, row in df.iterrows():
    mc.add_child(
        Marker(location = [row['lat'], row['lng']],
          popup=row['사고일시'],icon=folium.Icon(color='red',icon='car',prefix='fa')))
    
map.add_child(mc)
map
```
![image](https://user-images.githubusercontent.com/75477273/150949983-d414d80f-6c8a-4732-b3cb-fa182fc0b2b2.png)
결과로 상남동과 팔용동은 예상과 같이 사고가 많이 나는것을 확인 할 수 있으나,다른 곳은 예상과 달리 용원동,내서읍이 그 다음으로 가장 많이 일어남

그 이유로 출퇴근 시간에 부산신항 근처로 교통량이 많은 용원동과 고속도로 IC가 있는 내서에서 많은 사고가 발생

###시간과 요일별로 사고가 가장 많이 일어나는 곳을 알려주자는 취지에서 만든 프로그램

```python
df = pd.read_excel('accidentInfoList.xlsx')
## 월요일 사고 정리
time = df['사고일시']
a = []
for i in time:
    b = i.split()[3]
    a.append(b)
df['사고일시'] = a
# 상남동 
df_sangnam = df[df['시군구'] == '경상남도 창원시 성산구 상남동']
df_sangnam = df_sangnam[['시군구','사고일시','요일']]
df_sangnam_monday = df_sangnam[df_sangnam['요일'] == '월요일']
df_sangnam_monday_count = df_sangnam_monday[['사고일시']].value_counts()
df_sangnam_monday_count = pd.DataFrame(df_sangnam_monday_count,columns=['사건횟수'])
df_sangnam_monday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_sangnam_monday_count)):
    list_sn.append('경상남도 창원시 성산구 상남동')
df_sangnam_monday_count['시군구'] = list_sn  
# print(df_sangnam_monday_count)

# 용원동
df_wongone = df[df['시군구'] == '경상남도 창원시 진해구 용원동']
df_wongone = df_wongone[['시군구','사고일시','요일']]
df_wongone_monday = df_wongone[df_wongone['요일'] == '월요일']
df_wongone_monday_count = df_wongone_monday[['사고일시']].value_counts()
df_wongone_monday_count = pd.DataFrame(df_wongone_monday_count,columns=['사건횟수'])
df_wongone_monday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_wongone_monday_count)):
    list_sn.append('경상남도 창원시 진해구 용원동')
df_wongone_monday_count['시군구'] = list_sn  
# print(df_wongone_monday_count)

# 팔용동
df_palong = df[df['시군구'] == '경상남도 창원시 의창구 팔용동']
df_palong = df_palong[['시군구','사고일시','요일']]
df_palong_monday = df_palong[df_palong['요일'] == '월요일']
df_palong_monday_count = df_palong_monday[['사고일시']].value_counts()
df_palong_monday_count = pd.DataFrame(df_palong_monday_count,columns=['사건횟수'])
df_palong_monday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_palong_monday_count)):
    list_sn.append('경상남도 창원시 의창구 팔용동')
df_palong_monday_count['시군구'] = list_sn  
# print(df_palong_monday_count)

# 내서읍
df_neseo = df[df['시군구'] == '경상남도 창원시 마산회원구 내서읍']
df_neseo = df_neseo[['시군구','사고일시','요일']]
df_neseo_monday = df_neseo[df_neseo['요일'] == '월요일']
df_neseo_monday_count = df_neseo_monday[['사고일시']].value_counts()
df_neseo_monday_count = pd.DataFrame(df_neseo_monday_count,columns=['사건횟수'])
df_neseo_monday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_neseo_monday_count)):
    list_sn.append('경상남도 창원시 마산회원구 내서읍')
df_neseo_monday_count['시군구'] = list_sn  
# print(df_neseo_monday_count)

# 양덕동
df_yangdec = df[df['시군구'] == '경상남도 창원시 마산회원구 양덕동']
df_yangdec = df_yangdec[['시군구','사고일시','요일']]
df_yangdec_monday = df_yangdec[df_yangdec['요일'] == '월요일']
df_yangdec_monday_count = df_yangdec_monday[['사고일시']].value_counts()
df_yangdec_monday_count = pd.DataFrame(df_yangdec_monday_count,columns=['사건횟수'])
df_yangdec_monday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_yangdec_monday_count)):
    list_sn.append('경상남도 창원시 마산회원구 양덕동')
df_yangdec_monday_count['시군구'] = list_sn  
# print(df_yangdec_monday_count)

df_m = [df_neseo_monday_count,df_palong_monday_count,df_sangnam_monday_count,df_wongone_monday_count,df_yangdec_monday_count]
df_monday = pd.concat(df_m,ignore_index=True)
df_monday.sort_values(by=['사건횟수'],axis=0,ascending=False,inplace=True)

df_monday['사고일시']=df['사고일시'].str.replace(pat=r'[ㄱ-ㅣ가-힣]+', repl= r'', regex=True)
df_monday['사고일시'] = pd.to_numeric(df_monday['사고일시'])
df_monday.sort_values(by=['사고일시','사건횟수'],axis=0,ascending=True,inplace=True)
df_monday.drop_duplicates(['사고일시'],keep='last', inplace=True)
df_monday

사고일시	사건횟수	시군구
23	0	1	경상남도 창원시 의창구 팔용동
15	1	1	경상남도 창원시 의창구 팔용동
40	2	2	경상남도 창원시 진해구 용원동
5	3	1	경상남도 창원시 마산회원구 내서읍
27	5	2	경상남도 창원시 성산구 상남동
19	6	1	경상남도 창원시 의창구 팔용동
37	7	4	경상남도 창원시 진해구 용원동
35	8	1	경상남도 창원시 성산구 상남동
59	9	1	경상남도 창원시 마산회원구 양덕동
11	10	3	경상남도 창원시 의창구 팔용동
13	11	2	경상남도 창원시 의창구 팔용동
53	12	2	경상남도 창원시 마산회원구 양덕동
```
```python
#top 5 화요일 사고 일시

# 상남동 
df_sangnam = df[df['시군구'] == '경상남도 창원시 성산구 상남동']
df_sangnam = df_sangnam[['시군구','사고일시','요일']]
df_sangnam_tuesday = df_sangnam[df_sangnam['요일'] == '화요일']
df_sangnam_tuesday_count = df_sangnam_tuesday[['사고일시']].value_counts()
df_sangnam_tuesday_count = pd.DataFrame(df_sangnam_tuesday_count,columns=['사건횟수'])
df_sangnam_tuesday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_sangnam_tuesday_count)):
    list_sn.append('경상남도 창원시 성산구 상남동')
df_sangnam_tuesday_count['시군구'] = list_sn  
# print(df_sangnam_tuesday_count)

# 용원동
df_wongone = df[df['시군구'] == '경상남도 창원시 진해구 용원동']
df_wongone = df_wongone[['시군구','사고일시','요일']]
df_wongone_tuesday = df_wongone[df_wongone['요일'] == '화요일']
df_wongone_tuesday_count = df_wongone_tuesday[['사고일시']].value_counts()
df_wongone_tuesday_count = pd.DataFrame(df_wongone_tuesday_count,columns=['사건횟수'])
df_wongone_tuesday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_wongone_tuesday_count)):
    list_sn.append('경상남도 창원시 진해구 용원동')
df_wongone_tuesday_count['시군구'] = list_sn  
# print(df_wongone_tuesday_count)

# 팔용동
df_palong = df[df['시군구'] == '경상남도 창원시 의창구 팔용동']
df_palong = df_palong[['시군구','사고일시','요일']]
df_palong_tuesday = df_palong[df_palong['요일'] == '화요일']
df_palong_tuesday_count = df_palong_tuesday[['사고일시']].value_counts()
df_palong_tuesday_count = pd.DataFrame(df_palong_tuesday_count,columns=['사건횟수'])
df_palong_tuesday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_palong_tuesday_count)):
    list_sn.append('경상남도 창원시 의창구 팔용동')
df_palong_tuesday_count['시군구'] = list_sn  
# print(df_palong_tuesday_count)

# 내서읍
df_neseo = df[df['시군구'] == '경상남도 창원시 마산회원구 내서읍']
df_neseo = df_neseo[['시군구','사고일시','요일']]
df_neseo_tuesday = df_neseo[df_neseo['요일'] == '화요일']
df_neseo_tuesday_count = df_neseo_tuesday[['사고일시']].value_counts()
df_neseo_tuesday_count = pd.DataFrame(df_neseo_tuesday_count,columns=['사건횟수'])
df_neseo_tuesday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_neseo_tuesday_count)):
    list_sn.append('경상남도 창원시 마산회원구 내서읍')
df_neseo_tuesday_count['시군구'] = list_sn  
# print(df_neseo_tuesday_count)

# 양덕동
df_yangdec = df[df['시군구'] == '경상남도 창원시 마산회원구 양덕동']
df_yangdec = df_yangdec[['시군구','사고일시','요일']]
df_yangdec_tuesday = df_yangdec[df_yangdec['요일'] == '화요일']
df_yangdec_tuesday_count = df_yangdec_tuesday[['사고일시']].value_counts()
df_yangdec_tuesday_count = pd.DataFrame(df_yangdec_tuesday_count,columns=['사건횟수'])
df_yangdec_tuesday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_yangdec_tuesday_count)):
    list_sn.append('경상남도 창원시 마산회원구 양덕동')
df_yangdec_tuesday_count['시군구'] = list_sn  
# print(df_yangdec_tuesday_count)

df_m = [df_neseo_tuesday_count,df_palong_tuesday_count,df_sangnam_tuesday_count,df_wongone_tuesday_count,df_yangdec_tuesday_count]
df_tuesday = pd.concat(df_m,ignore_index=True)
df_tuesday.sort_values(by=['사건횟수'],axis=0,ascending=False,inplace=True)

df_tuesday['사고일시']=df['사고일시'].str.replace(pat=r'[ㄱ-ㅣ가-힣]+', repl= r'', regex=True)
df_tuesday['사고일시'] = pd.to_numeric(df_tuesday['사고일시'])
df_tuesday.sort_values(by=['사고일시','사건횟수'],axis=0,ascending=True,inplace=True)
df_tuesday.drop_duplicates(['사고일시'],keep='last', inplace=True)
df_tuesday

	사고일시	사건횟수	시군구
23	0	1	경상남도 창원시 의창구 팔용동
15	1	2	경상남도 창원시 의창구 팔용동
40	2	3	경상남도 창원시 진해구 용원동
5	3	1	경상남도 창원시 마산회원구 내서읍
27	5	2	경상남도 창원시 성산구 상남동
10	6	1	경상남도 창원시 마산회원구 내서읍
41	7	2	경상남도 창원시 진해구 용원동
```

```python
#top 5 수요일 사고 일시

# 상남동 
df_sangnam = df[df['시군구'] == '경상남도 창원시 성산구 상남동']
df_sangnam = df_sangnam[['시군구','사고일시','요일']]
df_sangnam_wednesday = df_sangnam[df_sangnam['요일'] == '수요일']
df_sangnam_wednesday_count = df_sangnam_wednesday[['사고일시']].value_counts()
df_sangnam_wednesday_count = pd.DataFrame(df_sangnam_wednesday_count,columns=['사건횟수'])
df_sangnam_wednesday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_sangnam_wednesday_count)):
    list_sn.append('경상남도 창원시 성산구 상남동')
df_sangnam_wednesday_count['시군구'] = list_sn  
# print(df_sangnam_wednesday_count)

# 용원동
df_wongone = df[df['시군구'] == '경상남도 창원시 진해구 용원동']
df_wongone = df_wongone[['시군구','사고일시','요일']]
df_wongone_wednesday = df_wongone[df_wongone['요일'] == '수요일']
df_wongone_wednesday_count = df_wongone_wednesday[['사고일시']].value_counts()
df_wongone_wednesday_count = pd.DataFrame(df_wongone_wednesday_count,columns=['사건횟수'])
df_wongone_wednesday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_wongone_wednesday_count)):
    list_sn.append('경상남도 창원시 진해구 용원동')
df_wongone_wednesday_count['시군구'] = list_sn  
# print(df_wongone_wednesday_count)

# 팔용동
df_palong = df[df['시군구'] == '경상남도 창원시 의창구 팔용동']
df_palong = df_palong[['시군구','사고일시','요일']]
df_palong_wednesday = df_palong[df_palong['요일'] == '수요일']
df_palong_wednesday_count = df_palong_wednesday[['사고일시']].value_counts()
df_palong_wednesday_count = pd.DataFrame(df_palong_wednesday_count,columns=['사건횟수'])
df_palong_wednesday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_palong_wednesday_count)):
    list_sn.append('경상남도 창원시 의창구 팔용동')
df_palong_wednesday_count['시군구'] = list_sn  
# print(df_palong_wednesday_count)

# 내서읍
df_neseo = df[df['시군구'] == '경상남도 창원시 마산회원구 내서읍']
df_neseo = df_neseo[['시군구','사고일시','요일']]
df_neseo_wednesday = df_neseo[df_neseo['요일'] == '수요일']
df_neseo_wednesday_count = df_neseo_wednesday[['사고일시']].value_counts()
df_neseo_wednesday_count = pd.DataFrame(df_neseo_wednesday_count,columns=['사건횟수'])
df_neseo_wednesday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_neseo_wednesday_count)):
    list_sn.append('경상남도 창원시 마산회원구 내서읍')
df_neseo_wednesday_count['시군구'] = list_sn  
# print(df_neseo_wednesday_count)

# 양덕동
df_yangdec = df[df['시군구'] == '경상남도 창원시 마산회원구 양덕동']
df_yangdec = df_yangdec[['시군구','사고일시','요일']]
df_yangdec_wednesday = df_yangdec[df_yangdec['요일'] == '수요일']
df_yangdec_wednesday_count = df_yangdec_wednesday[['사고일시']].value_counts()
df_yangdec_wednesday_count = pd.DataFrame(df_yangdec_wednesday_count,columns=['사건횟수'])
df_yangdec_wednesday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_yangdec_wednesday_count)):
    list_sn.append('경상남도 창원시 마산회원구 양덕동')
df_yangdec_wednesday_count['시군구'] = list_sn  
# print(df_yangdec_wednesday_count)

df_m = [df_neseo_wednesday_count,df_palong_wednesday_count,df_sangnam_wednesday_count,df_wongone_wednesday_count,df_yangdec_wednesday_count]
df_wednesday = pd.concat(df_m,ignore_index=True)
df_wednesday.sort_values(by=['사건횟수'],axis=0,ascending=False,inplace=True)

df_wednesday['사고일시']=df['사고일시'].str.replace(pat=r'[ㄱ-ㅣ가-힣]+', repl= r'', regex=True)
df_wednesday['사고일시'] = pd.to_numeric(df_wednesday['사고일시'])
df_wednesday.sort_values(by=['사고일시','사건횟수'],axis=0,ascending=True,inplace=True)
df_wednesday.drop_duplicates(['사고일시'],keep='last', inplace=True)
df_wednesday

	사고일시	사건횟수	시군구
23	0	1	경상남도 창원시 의창구 팔용동
15	1	1	경상남도 창원시 의창구 팔용동
40	2	2	경상남도 창원시 진해구 용원동
5	3	1	경상남도 창원시 마산회원구 내서읍
27	5	3	경상남도 창원시 성산구 상남동
10	6	3	경상남도 창원시 의창구 팔용동
38	7	3	경상남도 창원시 진해구 용원동
35	8	1	경상남도 창원시 성산구 상남동
59	9	1	경상남도 창원시 마산회원구 양덕동
```

```python
#top 5 목요일 사고 일시

# 상남동 
df_sangnam = df[df['시군구'] == '경상남도 창원시 성산구 상남동']
df_sangnam = df_sangnam[['시군구','사고일시','요일']]
df_sangnam_thursday = df_sangnam[df_sangnam['요일'] == '목요일']
df_sangnam_thursday_count = df_sangnam_thursday[['사고일시']].value_counts()
df_sangnam_thursday_count = pd.DataFrame(df_sangnam_thursday_count,columns=['사건횟수'])
df_sangnam_thursday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_sangnam_thursday_count)):
    list_sn.append('경상남도 창원시 성산구 상남동')
df_sangnam_thursday_count['시군구'] = list_sn  
# print(df_sangnam_thursday_count)

# 용원동
df_wongone = df[df['시군구'] == '경상남도 창원시 진해구 용원동']
df_wongone = df_wongone[['시군구','사고일시','요일']]
df_wongone_thursday = df_wongone[df_wongone['요일'] == '목요일']
df_wongone_thursday_count = df_wongone_thursday[['사고일시']].value_counts()
df_wongone_thursday_count = pd.DataFrame(df_wongone_thursday_count,columns=['사건횟수'])
df_wongone_thursday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_wongone_thursday_count)):
    list_sn.append('경상남도 창원시 진해구 용원동')
df_wongone_thursday_count['시군구'] = list_sn  
# print(df_wongone_thursday_count)

# 팔용동
df_palong = df[df['시군구'] == '경상남도 창원시 의창구 팔용동']
df_palong = df_palong[['시군구','사고일시','요일']]
df_palong_thursday = df_palong[df_palong['요일'] == '목요일']
df_palong_thursday_count = df_palong_thursday[['사고일시']].value_counts()
df_palong_thursday_count = pd.DataFrame(df_palong_thursday_count,columns=['사건횟수'])
df_palong_thursday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_palong_thursday_count)):
    list_sn.append('경상남도 창원시 의창구 팔용동')
df_palong_thursday_count['시군구'] = list_sn  
# print(df_palong_thursday_count)

# 내서읍
df_neseo = df[df['시군구'] == '경상남도 창원시 마산회원구 내서읍']
df_neseo = df_neseo[['시군구','사고일시','요일']]
df_neseo_thursday = df_neseo[df_neseo['요일'] == '목요일']
df_neseo_thursday_count = df_neseo_thursday[['사고일시']].value_counts()
df_neseo_thursday_count = pd.DataFrame(df_neseo_thursday_count,columns=['사건횟수'])
df_neseo_thursday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_neseo_thursday_count)):
    list_sn.append('경상남도 창원시 마산회원구 내서읍')
df_neseo_thursday_count['시군구'] = list_sn  
# print(df_neseo_thursday_count)

# 양덕동
df_yangdec = df[df['시군구'] == '경상남도 창원시 마산회원구 양덕동']
df_yangdec = df_yangdec[['시군구','사고일시','요일']]
df_yangdec_thursday = df_yangdec[df_yangdec['요일'] == '목요일']
df_yangdec_thursday_count = df_yangdec_thursday[['사고일시']].value_counts()
df_yangdec_thursday_count = pd.DataFrame(df_yangdec_thursday_count,columns=['사건횟수'])
df_yangdec_thursday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_yangdec_thursday_count)):
    list_sn.append('경상남도 창원시 마산회원구 양덕동')
df_yangdec_thursday_count['시군구'] = list_sn  
# print(df_yangdec_thursday_count)

df_m = [df_neseo_thursday_count,df_palong_thursday_count,df_sangnam_thursday_count,df_wongone_thursday_count,df_yangdec_thursday_count]
df_thursday = pd.concat(df_m,ignore_index=True)
df_thursday.sort_values(by=['사건횟수'],axis=0,ascending=False,inplace=True)

df_thursday['사고일시']=df['사고일시'].str.replace(pat=r'[ㄱ-ㅣ가-힣]+', repl= r'', regex=True)
df_thursday['사고일시'] = pd.to_numeric(df_thursday['사고일시'])
df_thursday.sort_values(by=['사고일시','사건횟수'],axis=0,ascending=True,inplace=True)
df_thursday.drop_duplicates(['사고일시'],keep='last', inplace=True)
df_thursday

	사고일시	사건횟수	시군구
23	0	4	경상남도 창원시 성산구 상남동
15	1	1	경상남도 창원시 의창구 팔용동
40	2	2	경상남도 창원시 진해구 용원동
5	3	1	경상남도 창원시 마산회원구 내서읍
27	5	2	경상남도 창원시 성산구 상남동
10	6	2	경상남도 창원시 의창구 팔용동
38	7	3	경상남도 창원시 진해구 용원동
35	8	1	경상남도 창원시 성산구 상남동
59	9	1	경상남도 창원시 마산회원구 양덕동
0	10	3	경상남도 창원시 마산회원구 내서읍
```

```python
#top 5 금요일 사고 일시

# 상남동 
df_sangnam = df[df['시군구'] == '경상남도 창원시 성산구 상남동']
df_sangnam = df_sangnam[['시군구','사고일시','요일']]
df_sangnam_friday = df_sangnam[df_sangnam['요일'] == '금요일']
df_sangnam_friday_count = df_sangnam_friday[['사고일시']].value_counts()
df_sangnam_friday_count = pd.DataFrame(df_sangnam_friday_count,columns=['사건횟수'])
df_sangnam_friday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_sangnam_friday_count)):
    list_sn.append('경상남도 창원시 성산구 상남동')
df_sangnam_friday_count['시군구'] = list_sn  
# print(df_sangnam_friday_count)

# 용원동
df_wongone = df[df['시군구'] == '경상남도 창원시 진해구 용원동']
df_wongone = df_wongone[['시군구','사고일시','요일']]
df_wongone_friday = df_wongone[df_wongone['요일'] == '금요일']
df_wongone_friday_count = df_wongone_friday[['사고일시']].value_counts()
df_wongone_friday_count = pd.DataFrame(df_wongone_friday_count,columns=['사건횟수'])
df_wongone_friday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_wongone_friday_count)):
    list_sn.append('경상남도 창원시 진해구 용원동')
df_wongone_friday_count['시군구'] = list_sn  
# print(df_wongone_friday_count)

# 팔용동
df_palong = df[df['시군구'] == '경상남도 창원시 의창구 팔용동']
df_palong = df_palong[['시군구','사고일시','요일']]
df_palong_friday = df_palong[df_palong['요일'] == '금요일']
df_palong_friday_count = df_palong_friday[['사고일시']].value_counts()
df_palong_friday_count = pd.DataFrame(df_palong_friday_count,columns=['사건횟수'])
df_palong_friday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_palong_friday_count)):
    list_sn.append('경상남도 창원시 의창구 팔용동')
df_palong_friday_count['시군구'] = list_sn  
# print(df_palong_friday_count)

# 내서읍
df_neseo = df[df['시군구'] == '경상남도 창원시 마산회원구 내서읍']
df_neseo = df_neseo[['시군구','사고일시','요일']]
df_neseo_friday = df_neseo[df_neseo['요일'] == '금요일']
df_neseo_friday_count = df_neseo_friday[['사고일시']].value_counts()
df_neseo_friday_count = pd.DataFrame(df_neseo_friday_count,columns=['사건횟수'])
df_neseo_friday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_neseo_friday_count)):
    list_sn.append('경상남도 창원시 마산회원구 내서읍')
df_neseo_friday_count['시군구'] = list_sn  
# print(df_neseo_friday_count)

# 양덕동
df_yangdec = df[df['시군구'] == '경상남도 창원시 마산회원구 양덕동']
df_yangdec = df_yangdec[['시군구','사고일시','요일']]
df_yangdec_friday = df_yangdec[df_yangdec['요일'] == '금요일']
df_yangdec_friday_count = df_yangdec_friday[['사고일시']].value_counts()
df_yangdec_friday_count = pd.DataFrame(df_yangdec_friday_count,columns=['사건횟수'])
df_yangdec_friday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_yangdec_friday_count)):
    list_sn.append('경상남도 창원시 마산회원구 양덕동')
df_yangdec_friday_count['시군구'] = list_sn  
# print(df_yangdec_friday_count)

df_m = [df_neseo_friday_count,df_palong_friday_count,df_sangnam_friday_count,df_wongone_friday_count,df_yangdec_friday_count]
df_friday = pd.concat(df_m,ignore_index=True)
df_friday.sort_values(by=['사건횟수'],axis=0,ascending=False,inplace=True)

df_friday['사고일시']=df['사고일시'].str.replace(pat=r'[ㄱ-ㅣ가-힣]+', repl= r'', regex=True)
df_friday['사고일시'] = pd.to_numeric(df_friday['사고일시'])
df_friday.sort_values(by=['사고일시','사건횟수'],axis=0,ascending=True,inplace=True)
df_friday.drop_duplicates(['사고일시'],keep='last', inplace=True)
df_friday


사고일시	사건횟수	시군구
23	0	1	경상남도 창원시 의창구 팔용동
15	1	2	경상남도 창원시 의창구 팔용동
40	2	3	경상남도 창원시 진해구 용원동
5	3	1	경상남도 창원시 마산회원구 내서읍
27	5	1	경상남도 창원시 성산구 상남동
19	6	1	경상남도 창원시 의창구 팔용동
41	7	2	경상남도 창원시 진해구 용원동
35	8	1	경상남도 창원시 성산구 상남동
```
```python
#top 5 토요일 사고 일시

# 상남동 
df_sangnam = df[df['시군구'] == '경상남도 창원시 성산구 상남동']
df_sangnam = df_sangnam[['시군구','사고일시','요일']]
df_sangnam_saturday = df_sangnam[df_sangnam['요일'] == '토요일']
df_sangnam_saturday_count = df_sangnam_saturday[['사고일시']].value_counts()
df_sangnam_saturday_count = pd.DataFrame(df_sangnam_saturday_count,columns=['사건횟수'])
df_sangnam_saturday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_sangnam_saturday_count)):
    list_sn.append('경상남도 창원시 성산구 상남동')
df_sangnam_saturday_count['시군구'] = list_sn  
# print(df_sangnam_saturday_count)

# 용원동
df_wongone = df[df['시군구'] == '경상남도 창원시 진해구 용원동']
df_wongone = df_wongone[['시군구','사고일시','요일']]
df_wongone_saturday = df_wongone[df_wongone['요일'] == '토요일']
df_wongone_saturday_count = df_wongone_saturday[['사고일시']].value_counts()
df_wongone_saturday_count = pd.DataFrame(df_wongone_saturday_count,columns=['사건횟수'])
df_wongone_saturday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_wongone_saturday_count)):
    list_sn.append('경상남도 창원시 진해구 용원동')
df_wongone_saturday_count['시군구'] = list_sn  
# print(df_wongone_saturday_count)

# 팔용동
df_palong = df[df['시군구'] == '경상남도 창원시 의창구 팔용동']
df_palong = df_palong[['시군구','사고일시','요일']]
df_palong_saturday = df_palong[df_palong['요일'] == '토요일']
df_palong_saturday_count = df_palong_saturday[['사고일시']].value_counts()
df_palong_saturday_count = pd.DataFrame(df_palong_saturday_count,columns=['사건횟수'])
df_palong_saturday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_palong_saturday_count)):
    list_sn.append('경상남도 창원시 의창구 팔용동')
df_palong_saturday_count['시군구'] = list_sn  
# print(df_palong_saturday_count)

# 내서읍
df_neseo = df[df['시군구'] == '경상남도 창원시 마산회원구 내서읍']
df_neseo = df_neseo[['시군구','사고일시','요일']]
df_neseo_saturday = df_neseo[df_neseo['요일'] == '토요일']
df_neseo_saturday_count = df_neseo_saturday[['사고일시']].value_counts()
df_neseo_saturday_count = pd.DataFrame(df_neseo_saturday_count,columns=['사건횟수'])
df_neseo_saturday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_neseo_saturday_count)):
    list_sn.append('경상남도 창원시 마산회원구 내서읍')
df_neseo_saturday_count['시군구'] = list_sn  
# print(df_neseo_saturday_count)

# 양덕동
df_yangdec = df[df['시군구'] == '경상남도 창원시 마산회원구 양덕동']
df_yangdec = df_yangdec[['시군구','사고일시','요일']]
df_yangdec_saturday = df_yangdec[df_yangdec['요일'] == '토요일']
df_yangdec_saturday_count = df_yangdec_saturday[['사고일시']].value_counts()
df_yangdec_saturday_count = pd.DataFrame(df_yangdec_saturday_count,columns=['사건횟수'])
df_yangdec_saturday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_yangdec_saturday_count)):
    list_sn.append('경상남도 창원시 마산회원구 양덕동')
df_yangdec_saturday_count['시군구'] = list_sn  
# print(df_yangdec_saturday_count)

df_m = [df_neseo_saturday_count,df_palong_saturday_count,df_sangnam_saturday_count,df_wongone_saturday_count,df_yangdec_saturday_count]
df_saturday = pd.concat(df_m,ignore_index=True)
df_saturday.sort_values(by=['사건횟수'],axis=0,ascending=False,inplace=True)

df_saturday['사고일시']=df['사고일시'].str.replace(pat=r'[ㄱ-ㅣ가-힣]+', repl= r'', regex=True)
df_saturday['사고일시'] = pd.to_numeric(df_saturday['사고일시'])
df_saturday.sort_values(by=['사고일시','사건횟수'],axis=0,ascending=True,inplace=True)
df_saturday.drop_duplicates(['사고일시'],keep='last', inplace=True)
df_saturday

	사고일시	사건횟수	시군구
23	0	2	경상남도 창원시 성산구 상남동
15	1	2	경상남도 창원시 의창구 팔용동
40	2	1	경상남도 창원시 진해구 용원동
5	3	1	경상남도 창원시 마산회원구 내서읍
27	5	1	경상남도 창원시 성산구 상남동
19	6	1	경상남도 창원시 의창구 팔용동
6	7	1	경상남도 창원시 마산회원구 내서읍
```
```python
#top 5 일요일 사고 일시

# 상남동 
df_sangnam = df[df['시군구'] == '경상남도 창원시 성산구 상남동']
df_sangnam = df_sangnam[['시군구','사고일시','요일']]
df_sangnam_sunday = df_sangnam[df_sangnam['요일'] == '일요일']
df_sangnam_sunday_count = df_sangnam_sunday[['사고일시']].value_counts()
df_sangnam_sunday_count = pd.DataFrame(df_sangnam_sunday_count,columns=['사건횟수'])
df_sangnam_sunday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_sangnam_sunday_count)):
    list_sn.append('경상남도 창원시 성산구 상남동')
df_sangnam_sunday_count['시군구'] = list_sn  
# print(df_sangnam_sunday_count)

# 용원동
df_wongone = df[df['시군구'] == '경상남도 창원시 진해구 용원동']
df_wongone = df_wongone[['시군구','사고일시','요일']]
df_wongone_sunday = df_wongone[df_wongone['요일'] == '일요일']
df_wongone_sunday_count = df_wongone_sunday[['사고일시']].value_counts()
df_wongone_sunday_count = pd.DataFrame(df_wongone_sunday_count,columns=['사건횟수'])
df_wongone_sunday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_wongone_sunday_count)):
    list_sn.append('경상남도 창원시 진해구 용원동')
df_wongone_sunday_count['시군구'] = list_sn  
# print(df_wongone_sunday_count)

# 팔용동
df_palong = df[df['시군구'] == '경상남도 창원시 의창구 팔용동']
df_palong = df_palong[['시군구','사고일시','요일']]
df_palong_sunday = df_palong[df_palong['요일'] == '일요일']
df_palong_sunday_count = df_palong_sunday[['사고일시']].value_counts()
df_palong_sunday_count = pd.DataFrame(df_palong_sunday_count,columns=['사건횟수'])
df_palong_sunday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_palong_sunday_count)):
    list_sn.append('경상남도 창원시 의창구 팔용동')
df_palong_sunday_count['시군구'] = list_sn  
# print(df_palong_sunday_count)

# 내서읍
df_neseo = df[df['시군구'] == '경상남도 창원시 마산회원구 내서읍']
df_neseo = df_neseo[['시군구','사고일시','요일']]
df_neseo_sunday = df_neseo[df_neseo['요일'] == '일요일']
df_neseo_sunday_count = df_neseo_sunday[['사고일시']].value_counts()
df_neseo_sunday_count = pd.DataFrame(df_neseo_sunday_count,columns=['사건횟수'])
df_neseo_sunday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_neseo_sunday_count)):
    list_sn.append('경상남도 창원시 마산회원구 내서읍')
df_neseo_sunday_count['시군구'] = list_sn  
# print(df_neseo_sunday_count)

# 양덕동
df_yangdec = df[df['시군구'] == '경상남도 창원시 마산회원구 양덕동']
df_yangdec = df_yangdec[['시군구','사고일시','요일']]
df_yangdec_sunday = df_yangdec[df_yangdec['요일'] == '일요일']
df_yangdec_sunday_count = df_yangdec_sunday[['사고일시']].value_counts()
df_yangdec_sunday_count = pd.DataFrame(df_yangdec_sunday_count,columns=['사건횟수'])
df_yangdec_sunday_count.reset_index(inplace=True)
list_sn=[]
for i in range(0,len(df_yangdec_sunday_count)):
    list_sn.append('경상남도 창원시 마산회원구 양덕동')
df_yangdec_sunday_count['시군구'] = list_sn  
# print(df_yangdec_sunday_count)

df_m = [df_neseo_sunday_count,df_palong_sunday_count,df_sangnam_sunday_count,df_wongone_sunday_count,df_yangdec_sunday_count]
df_sunday = pd.concat(df_m,ignore_index=True)
df_sunday.sort_values(by=['사건횟수'],axis=0,ascending=False,inplace=True)

df_sunday['사고일시']=df['사고일시'].str.replace(pat=r'[ㄱ-ㅣ가-힣]+', repl= r'', regex=True)
df_sunday['사고일시'] = pd.to_numeric(df_sunday['사고일시'])
df_sunday.sort_values(by=['사고일시','사건횟수'],axis=0,ascending=True,inplace=True)
df_sunday.drop_duplicates(['사고일시'],keep='last', inplace=True)
df_sunday

	사고일시	사건횟수	시군구
23	0	1	경상남도 창원시 성산구 상남동
15	1	1	경상남도 창원시 의창구 팔용동
40	2	1	경상남도 창원시 마산회원구 양덕동
5	3	1	경상남도 창원시 마산회원구 내서읍
27	5	3	경상남도 창원시 진해구 용원동
19	6	2	경상남도 창원시 성산구 상남동
38	7	2	경상남도 창원시 마산회원구 양덕동
```

```python
a = input('요일을 입력하세요:      ')
b = input('시간을 입력하세요:       ')

if a == '월요일':
    df_monday['사고일시'] = df_monday['사고일시'].astype(str)
    place = df_monday[df_monday.사고일시.str.startswith(b)].iloc[0,2]
elif a == '화요일':
    df_tuesday['사고일시'] = df_tuesday['사고일시'].astype(str)
    place = df_tuesday[df_tuesday.사고일시.str.startswith(b)].iloc[0,2]
elif a == '수요일':
    df_wednesday['사고일시'] = df_wednesday['사고일시'].astype(str)
    place = df_wednesday[df_wednesday.사고일시.str.startswith(b)].iloc[0,2]
elif a =='목요일':
    df_tuesday['사고일시'] = df_tuesday['사고일시'].astype(str)
    place = df_tuesday[df_tuesday.사고일시.str.startswith(b)].iloc[0,2]
elif a == '금요일':
    df_friday['사고일시'] = df_friday['사고일시'].astype(str)
    place = df_friday[df_friday.사고일시.str.startswith(b)].iloc[0,2]
elif a == '토요일':
    df_saturday['사고일시'] = df_df_saturdaysunday['사고일시'].astype(str)
    place = df_saturday[df_saturday.사고일시.str.startswith(b)].iloc[0,2]
elif a == '일요일':
    df_sunday['사고일시'] = df_sunday['사고일시'].astype(str)
    place = df_sunday[df_sunday.사고일시.str.startswith(b)].iloc[0,2]
else:
    print('요일 잘 못 입력')

print('주요 사고 발생 지역:   ',place)
try:
    day = (place + '사무소')
    gmap_key = 'AIzaSyD7RNFGi4878hN8gN5RVBpwgSfZ_TBVKko'
    gmaps = googlemaps.Client(gmap_key)
    geocode_result = gmaps.geocode((day), language='ko') 
except:
    print('사건미발생시간')
latitude  = geocode_result[0]["geometry"]["location"]["lat"] # 리스트에서 위도 추출
longitude = geocode_result[0]["geometry"]["location"]["lng"] # 리스트에서 경도 추출

map = folium.Map(location=[latitude,longitude], zoom_start=15)
folium.Marker([latitude,longitude],icon = folium.Icon(color='blue')).add_to(map)
map

요일을 입력하세요:      금요일
시간을 입력하세요:       10
주요 사고 발생 지역:    경상남도 창원시 의창구 팔용동
```
![image](https://user-images.githubusercontent.com/75477273/150951063-a29acfb0-883a-4fed-8f30-34af8a48560b.png)
