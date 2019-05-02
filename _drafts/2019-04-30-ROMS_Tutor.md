# ROMS 로 태풍 솔릭(Soulik) 모의해 보기

작성 : 김동훈 (http://blog.dhkim.info, 인하대학교), 2019년 5월 1일



ROMS (Regional Ocean Modeling System) 모델(모형)의 설치 및 실험에 관한 상세한 내용은
"[Installing and Running ROMS for First Time Users](https://www.myroms.org/wiki/ROMS_UNSW2008)"에 잘 설명되어 있으므로 영어에 친숙하신 분이라면 [here](https://www.myroms.org/wiki/ROMS_UNSW2008)를 참고하세요.

여기에서는 **태풍 솔릭(Soulik)을 대상**으로 설명하며, 최신 버전인 **ROMS-3.6 r964** (2019년 5월 1일 배포)을 사용합니다.
***절의 제목 앞에 숫자****가 붙어 있는 부분은 사용자가 직접 실행해야 하는 부분을 의미합니다.*

## 소 개



## 소프트웨어 요구사항



## 0. 환경 설정

```bash
export ROMS=${HOME}/models/ROMS
export ROMSsrc=${ROMS}/Sources/ROMS
export PROJ=${ROMS}/Projects/Soulik
```



## 1. 모델 및 자료 가져오기

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



## 2. 프로젝트 디렉토리 만들기

```bash
mkdir -p $PROJ; cd $PROJ
cp ${ROMSsrc}/trunk/ROMS/Bin/build_roms.bash .
```

~~ROMS 홈페이지에서는 build.bash 만을 프로젝트 디렉토리에 복사하고 소스코드를 참조하도록 설명하고 있지만, 경험 상 프로젝트 마다 모델 소스코드를 모두 복사하는 것이 편리합니다.~~ 

build 스크립트는 bash와 sh의 두가지를 지원하고 있으며 여기에서는 bash 기준으로 설명합니다.



## 3. build.bash 사용자 정의


$PROJ/build_roms.bash 를 다음과 같이 편집합니다.

```bash
vi $PROJ/build_roms.bash
> export   ROMS_APPLICATION=SOULIK #모두 대문자로 쓰며, 컴파일 시 소문자인 soulik.h 를 참조하게 됨
> export        MY_ROOT_DIR=${ROMSsrc}
```

