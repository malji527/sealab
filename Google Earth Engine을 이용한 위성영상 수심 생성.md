Google Earth Engine을 이용한 위성영상 수심 생산
===================================

## SDB(Satellite derived bathymetry)
최근 위성영상을 이용하여 생산된 수심 정보의 생산이 증가하고 있다. 아직 정확도는 IHO S-44 기준에 미치지는 못하지만, 수로조사를 계획할 경우나 오래된 측량 자료만 있는 경우 위성영상으로 생산된 수심 정보가 이용된다. 또한 선박을 통한 측량이 어렵고, 수심의 변동이 지속적으로 이루어지는 곳에서 광범위한 영역에 대해 쉽게 수심 정보를 획득할 수도 있다.    
위성영상으로 수심 정보를 생산하는 기법은 해저표면의 형태와 해수면의 상태에 영향을 미칠 수 있으므로 탁도나 해저표면의 구성물질의 영향이 적은 곳을 대상으로 하는 것이 좋다.    
본 튜토리얼은 제주해역을 대상으로하여 위성영상 수심을 생산한다.

## Challenges
1. 영상 촬영 시간이 만조시에 가깝고 구름 오염이 적은 영상이어야 한다.
2. 해양에서는 태양 반사광에 의한 효과(sun glint)가 발생한다.
3. Sentinel-2 영상을 사용하면 제주도가 포함된 영상은 1매가 아닌 여러 영상이다.
4. 수심 생산이기 때문에 육상부는 제거해야 한다.
5. Ratio 알고리즘을 적용하여 수심을 산출한다.

## Methods
### 영상선정 조건
제주도 인근 해역을 포함한 영상을 불러오기 위해 126E, 33N ~ 127E, 33.7N의 영역을 지정하였다. 이 영역은 코드에서 ‘site’라는 변수명으로 사용된다. 
```
var site = ee.Geometry.Polygon([126, 33.7, 126, 33, 127, 33, 127, 33.7], null, false);
```
영상은 2019년 3월 24일 영상을 사용할 것이고, Sentinel-2가 갖고있는 밴드 중 ‘BLUE’, ‘GREEN’, ‘RED’, ‘NIR’, ‘SCL’ 밴드만 사용할 것이므로 해당 밴드만 조회할 수 있도록 'globOptions’ 변수를 만들어준다. 
```
var globOptions = { 
  startDate: '2019-03-24', 
  endDate: '2019-03-25',
  bandSelect: ['blue', 'green', 'red', 'nir', 'SCL'],
  bands: ['B2', 'B3', 'B4', 'B8', 'SCL']
};
```    
### 구름제거
구름제거는 SCL 밴드를 이용하였다. 'cloud_low’, ‘cloud_medium’, ‘cloud_high’, ‘shadow’ 픽셀을 모두 제거하였다.
```
var cld = require('users/fitoprincipe/geetools:cloud_masks')

var maskcloud = function(image) {
  var masked = cld.sclMask(['cloud_low', 'cloud_medium', 'cloud_high', 'shadow'])(image)
  return masked
};
```    
### 태양광 반사 보정
해수면에서 발생하는 태양광 반사 보정 식은 다음과 같다.    
            R<sup>'</sup><sub>i</sub>=R<sub>i</sub>-b<sub>i</sub>(R<sub>NIR</sub>-MIN<sub>NIR</sub>)     
b<sub>i</sub> : 샘플영역 내 NIR 밴드와의 선형 회귀분석 결과로 산출한 기울기     
R<sub>i</sub> : 구하고자 하는 밴드의 전체 픽셀값     
R<sub>NIR</sub> : NIR 밴드의 전체 픽셀값     
MIN<sub>NIR</sub> : 샘플영역 내 NIR 밴드의 최소값       

각 영상에서 샘플 영역을 선택한 후, 위의 식을 적용하면 태양광 반사가 보정된 각 밴드의 픽셀값을 획득할 수 있다. 샘플 영역은 GEE의 사각형 그리기를 이용하여 설정하였다.

### 영상 불러오기
### 육상 마스킹
### Ratio 알고리즘 적용
### 내보내기

## Code
