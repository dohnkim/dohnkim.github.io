---
layout: posts
title: ROMS 로 태풍 솔릭(Soulik) 모의해 보기
author: 김동훈, 문일주
categories: []
tags: [WRF, Soulik]

---



<p style="text-align:right">작성 : 김동훈 (<a href="http://blog.dhkim.info">http://blog.dhkim.info</a>, 인하대학교)<br/> & 문일주 (제주대학교)<br/>2019년 5월 1일</p>

ROMS (Regional Ocean Modeling System) 모델(모형)의 설치 및 실험에 관한 상세한 내용은
"[Installing and Running ROMS for First Time Users](https://www.myroms.org/wiki/ROMS_UNSW2008)"에 잘 설명되어 있으므로 영어에 친숙하신 분이라면 [here](https://www.myroms.org/wiki/ROMS_UNSW2008)를 참고하세요.

여기에서는 **태풍 솔릭(Soulik)을 대상** (2 grids nested)으로 설명하며, 최근 버전인 **ROMS-3.6 r964** (2019년 5월 1일 배포)을 사용합니다.
***절의 제목 앞에 숫자****가 붙어 있는 부분은 사용자가 직접 실행해야 하는 부분을 의미합니다.*

# 소 개



# 소프트웨어 요구사항



# 0. 환경 설정

```bash
/usr/bin/env bash   # Bash Shell 사용
export ROMS=${HOME}/models/ROMS
export ROMSsrc=${ROMS}/Sources/ROMS
export PROJ=${ROMS}/Projects/Soulik
```



# 1. 모델 및 자료 가져오기

ROMS를 사용하려면 ROMS 포털에 등록되어 있어야 합니다. 등록은 [여기](https://www.myroms.org/wiki/ROMS_Cygwin#Register) 또는 [여기](http://www.myroms.org/index.php?page=RomsCode)에서 하면 됩니다.
등록한 아이디와 암호는 모델을 svn으로 가져올때 사용됩니다.

```bash
mkdir -p $ROMSsrc; cd $ROMSsrc
# 모델 소스코드만 받는 경우
svn checkout --username ??? https://www.myroms.org/svn/src/trunk trunk
# Test cases와 plot 관련 자료
svn checkout --username ??? https://www.myroms.org/svn/src/test test
svn checkout --username ??? https://www.myroms.org/svn/src/matlab matlab
svn checkout --username ??? https://www.myroms.org/svn/src/plot plot

# ROMS 관련 모든 자료를 한번에 받을 경우 (약 2.3G): 
#    참고) ROMS/tags 디렉토리에 이전 버전(v2.0 ~ v3.6)들이 포함되어 있음
svn checkout --username ??? https://www.myroms.org/svn/src .

# revision number 확인
svn info
```



# 2. 프로젝트 디렉토리 만들기

```bash
mkdir -p $PROJ; cd $PROJ
cp ${ROMSsrc}/trunk/ROMS/Bin/build_roms.bash .
```

~~ROMS 홈페이지에서는 build.bash 만을 프로젝트 디렉토리에 복사하고 소스코드를 참조하도록 설명하고 있지만, 경험 상 프로젝트 마다 모델 소스코드를 모두 복사하는 것이 편리합니다.~~ 

build 스크립트는 bash와 sh의 두가지를 지원하고 있으며 여기에서는 bash 기준으로 설명합니다.



# 3. build.bash 사용자 정의


$PROJ/build_roms.bash 를 다음과 같이 편집합니다.

```bash
vi $PROJ/build_roms.bash
> export   ROMS_APPLICATION=SOULIK #모두 대문자로 쓰며, 컴파일 시 소문자인 soulik.h 를 참조하게 됨
> export        MY_ROOT_DIR=${ROMSsrc}
```



# 4. Grid 만들기

ROMS의 영역 격자를 만드는 방법은 모델의 역사 만큼이나 다양하게 존재합니다. 대표적으로 다음과 같은 것들이 있으니 참고하세요.

- Gridgen: http:code.google.com/p/gridgen-c, or https://github.com/sakov/gridgen-c
  - pyroms, pygridgen, octant
- Easygrid: https://www.myroms.org/wiki/easygrid 
- Gridbuilder: http://austides.com/downloads
- COAWST Tools: wrf2roms_mw.m, create_roms_xygrid.m
- … ...

사용자의 연구상황에 따라서 격자를 만드는 방법이 다를 수 있으므로 여기에서는 "matlab을 사용하여 WRF의 격자 정보를 이용하여 ROMS grid를 만드는 방법"과 "pyroms를 이용하여 격자를 직접 만드는 방법"을 설명하도록 하겠습니다. 

## 4-1. wrf2roms_mw.m

COAWST에서 제공하는 도구 중에 포함되어 있는 matlab 스크립트를 이용하는 방법 입니다. 이 방법은 주로 ROMS를 WRF와 결합하여 모델을 수행할 경우에 유용합니다.



## 4-2. pyroms 를 사용하여 격자 만들기

Python의 모듈로 제공되고 있는 pyroms를 이용하는 방법으로써, 스크립트는 예제들을 참조하여 저자가 직접 만든 것입니다.

pyroms는 설치하기가 쉽지않은 모듈이지만, 성공적으로 설치할 경우 편리성이 매우 높은 도구이므로 설치과정을 자세히 설명하고자 합니다.ㄴ

### 4-2-1. pyroms 설치하기

**pyroms**는 Python의 설치 프로그램(pip) 또는 Anaconda의 conda에서 지원하지 않는 모듈이므로 직접 설치해야 합니다. 사용자 환경에 따라서 설치 시에 여러가지 문제점이 발생할 수 있으므로 여기에서는 Anaconda의 축소버전인 Miniconda를 사용하여 설치하도록 하겠습니다. 

#### 4-2-1-1. Miniconda 설치 및 ROMS 환경 구축

```bash
# https://docs.conda.io/en/latest/miniconda.html 에서 사용자 환경에 맞는 최신 버전을 가져 오세요.
# 저자는 "64-bit(bash installer) for Linux" 버전을 사용하였습니다.
# 다음의 명령으로 설치를 시작합니다.
bash ./Miniconda3-latest-Linux-x86_64.sh

# 설치과정 중 물어보는 "설치할 디렉토리"는 사용자의 적절한 디렉토리를 지정합니다. 
# 여기에서는 ${HOME}/local/miniconda3 에 설치하였습니다.
# 설치의 마지막 단계에서 묻는 "conda 환경을 구성하겠냐"는 질문에는 "yes"를 선택하여 
# ~/.bashrc 에 환경설정이 자동으로 추가되도록 하세요.
# 사용자가 사용하는 Shell이 bash가 아닌 경우에는 ~/.bashrc의 내용을 참조하여 적절하게 반영하여야 합니다.
# 제대로 설치되었다면 "conda" 명령이 실행 가능해야 합니다.

# conda에 가장 많이 사용되는 다운로드 채널을 추가합니다.
conda config --add channels conda-forge

# ROMS를 위한 독립 환경을 생성합니다. "-n" 옵션 다음에 붙는 이름은 사용자가 임의로 정하세요.
conda create -n ROMS python=3 ipython numpy scipy netCDF4
conda activate ROMS
conda install matplotlib basemap basemap-data-hires esmf nco pygrib
# "conda activate ROMS"를 수행하여 사용자 프롬프트 맨 앞에 "(ROMS)"가 추가되었다면 성공적으로 설치한 것입니다.
```



#### 4-2-1-2. 삭제: natgrid 설치 (*필요하지 않음*)

**pyroms** 는 NCAR의 natgrid (a natural neighbor gridding package) 라이브러리를 사용한다고 합니다. 
conda의 ROMS 환경 내에서 다음과 같이 설치합니다.

```bash
(ROMS) cd ~/local
# git으로 최신 버전의 소스코드를 받아 옵니다.
(ROMS) git clone https://github.com/matplotlib/natgrid.git
(ROMS) cd natgrid
(ROMS) CC=gcc python3 setup.py build
# warning 메세지가 많이 나올 수 있으나 error 메세지가 나오지 않으면 됨
(ROMS) python3 setup.py install  # 라이브러리를 python3의 site-packages에 설치
(ROMS) python3 setup.py test.py  # 라이브러리 테스트
```



#### 4-2-1-3. pyroms 설치

```bash
(ROMS) export DESTDIR=${HOME}/local/miniconda3/envs/ROMS
(ROMS) cd ~/local
# git으로 최신 버전의 소스코드를 받아 옵니다. 
# 상세한 설명은 https://github.com/ESMG/pyroms 를 방문하세요.
(ROMS) git clone https://github.com/ESMG/pyroms.git

# pyroms의 설치를 시작합니다. ----------------------------------------------------
# pyroms는 여러가지 패키지를 함께 사용하기 때문에 설치를 성공하는데 많은 어려움이 있을 수 있습니다. 
# 저자도 수십번의 과정을 거쳐서 성공한 결과를 요약하여 공유하는 것이니 하나 하나 잘 따라하셔야 합니다. 
(ROMS) cd pyroms/pyroms_toolbox
(ROMS) python ../pyinstall.py ${DESTDIR}
(ROMS) cd ../bathy_smoother
(ROMS) python ../pyinstall.py ${DESTDIR}
(ROMS) cd ../pyroms
(ROMS) vi install_pyroms.sh
          > DESTDIR=${DESTDIR}
          > PYROMS_PATH=$DESTDIR/lib/python3.7/site-packages/pyroms #Note) python 버전 확인
          > #python setup.py build --fcompiler=gnu95;  #주석 처리
          > #python setup.py install --prefix=$DESTDIR #주석 처리
          > python3 ../pyinstall.py $DESTDIR           #위에 주석 처리한 곳 바로 아래에 추가
(ROMS) vi external/scrip/source/makefile
          > NC_CONFIG = nc-config #nf-config 를 nc-config 로 수정
          > LIB = $(shell $(NC_CONFIG) --libs) #--flibs 를 --libs 로 수정
(ROMS) ./install_pyroms.sh
(ROMS) export PYROMS_GRIDID_FILE=${HOME}/local/pyroms/pyroms/pyroms/gridid.txt

(ROMS) cd $DESTDIR/lib/python3.7/site-packages #Note) python 버전에 따라 path가 다름
(ROMS) cp pyroms_toolbox/*.so . ; cp bathy_smoother/*.so . ; cp pyroms/*.so .
```



```python
import pyroms
import pyroms_toolbox
from bathy_smoother import *

```

 