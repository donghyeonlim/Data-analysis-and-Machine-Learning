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
