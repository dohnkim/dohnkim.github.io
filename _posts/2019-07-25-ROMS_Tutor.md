---
layout: posts
title: ROMS 로 태풍 솔릭(Soulik) 시기의 해양순환 모의해 보기
author: 김동훈, 문일주
categories: []
tags: [ROMS, Soulik]

---



<p style="text-align:right">작성 : 김동훈 (<a href="http://www.dhkim.info">http://www.dhkim.info</a>, 인하대학교)<br/> & 문일주 (제주대학교)<br/>2019년 7월 25일</p>

# 소 개

ROMS (Regional Ocean Modeling System) 모델(모형)의 설치 및 실험에 관한 상세한 내용은
"[Installing and Running ROMS for First Time Users](https://www.myroms.org/wiki/ROMS_UNSW2008)"에 잘 설명되어 있으므로 영어에 친숙하신 분이라면 [here](https://www.myroms.org/wiki/ROMS_UNSW2008)를 참고하세요.

여기에서는 **태풍 솔릭(Soulik)** 시기의 해양순환 모의를 설명하며, 최근 버전인 **ROMS-3.6 r964** (2019년 5월 1일 배포)을 사용합니다.
***절의 제목 앞에 숫자****가 붙어 있는 부분은 사용자가 직접 실행해야 하는 부분을 의미합니다.*

# 소프트웨어 요구사항

- Intel Fortran Compiler
- MPI library (openmpi 또는 mpich, mvapich 등)
- python 관련 : pyroms 모듈, jupyter notebook 또는 lab

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



# 2. 프로젝트 디렉토리 만들고 컴파일 하기

```bash
mkdir -p $PROJ; cd $PROJ

# build 스크립트는 bash와 sh의 두가지를 지원하고 있으며 여기에서는 bash 기준으로 설명합니다.
cp ${ROMSsrc}/trunk/ROMS/Bin/build_roms.bash .

# build_roms.bash의 사용자 정의
vi ./build_roms.bash
> export   ROMS_APPLICATION=SOULIK # 모두 대문자로 쓰며, 컴파일 시 소문자인 soulik.h 를 참조하게 됨
> export        MY_ROOT_DIR=${ROMSsrc}
> export        USE_NETCDF4=on     # netcdf의 버전에 따라 선택하세요.
```

> Note) **USE_NETCDF4** 를 on 하였을 경우는 컴파일 시에 NETCDF 라이브러리의 위치를 묻는 "**nf-config**" 명령을 사용하게 되는데, 이 명령이 없거나 실행 시에 문제가 있다면 **${ROMSsrc}/trunk/Compilers/Linux-ifort.mk** (*시스템에 따라 파일이 다를 수 있음*) 을 편집하여 "nf-config" 를 "nc-config" 로 바꾸어 줍니다. 또한, nc-config 의 명령이 제대로 실행되는지도 확인해야 합니다.<br/>참고로, nf-config 는 Fortran 용 실행화일이며, nc-config 는 C 용 실행화일입니다. 보통은 Fortran과 C 용 라이브러리가 동일 디렉토리에 설치되므로 아무거나 써도 상관이 없습니다.

## 2-1. Header 파일 만들기

ROMS의 컴파일 시 사용할 header 파일을 다음과 같이 작성합니다. 파일 이름은 $PROJ 디렉토리에 사용한 이름과 같은 soulik.h 로 저장합니다. 파일 이름은 모두 **소문자**이어야 합니다.

```cpp
cd $PROJ
cat > soulik.h << END
/*
** Options for NWP Test.
** Application flag: NWP
*/

/* Physics + numerics */
#define UV_ADV           /* 이류항 계산 */
#define UV_QDRAG         /* Quadratic bottom friction */
#define UV_COR           /* 코리올리 힘 계산 */
#define UV_VIS2          /* Harmonic horizontal mixing */

#define TS_U3HADVECTION  /* 3rd-order upstream horiz. advection */
#define TS_DIF2          /* Harmonic horizontal mixing for tracers */
#define SALINITY         /* 염분 계산 */

#define DJ_GRADPS        /* Splines density Jacobian (Shchepetkin, 2000) */
#define MIX_S_UV         /* mixing along constant S-surfaces */
#define MIX_GEO_TS       /* mixing along geopotential (constant Z) surfaces */

#define SOLVE3D          /* 3D 원시방정식 계산 */
#define CURVGRID         /* Curvilinear coordinates grid */

#define SPLINES_VDIFF
#define SPLINES_VVISC
#define NONLIN_EOS       /* 비선형 상태방정식 사용 */
/*#define WET_DRY*/      /* Wetting and Drying 옵션 */

/* Grid and Initial */
#define MASKING          /* land/sea masking */

/* Forcing */
/*#define BULK_FLUXES*/  /* Bulk fluxes 계산 */
#define ATM_PRESS        /* 해수면 위에 대기압 적용 */
#define ANA_BTFLUX       /* analytical bottom temperature flux 사용 */
#define ANA_BSFLUX       /* analytical bottom salinity flux 사용 */
#define ANA_BPFLUX       /* analytical bottom passive tracers fluxes 사용 */
#define LONGWAVE_OUT     /* outgoing longwave radiation */
#define EMINUSP          /* E-P 계산 */
#define SOLAR_SOURCE     /*solar radiation source term*/
#undef QCORRECTION       /*net heat flux correction */
#undef SCORRECTION       /*freshwater flux correction*/
#undef SRELAXATION       /*salinity relaxation as a freshwater flux*/
#undef DIURNAL_SRFLUX    /*mdulate input shortwave with diurnal cycle*/

/* Turbulence closure */
#define GLS_MIXING       /* Generic Length-Scale mixing */
#undef  MY25_MIXING      /* Mellor/Yamada Level-2.5 closure */
#undef  LMD_MIXING       /* Large et al. (1994) K-profile vertical parameterization */
/*#define LIMIT_VDIFF*/
/*#define LIMIT_VVISC*/

#if defined GLS_MIXING || defined MY25_MIXING
# define KANTHA_CLAYSON  /* Kantha and Clayson stability function */
# define N2S2_HORAVG     /* Horizontal smoothing of buoyancy/shear */
# define RI_SPLINES      
# define CRAIG_BANNER    /* Craig and Banner wave breaking surface flux */
# define CHARNOK         /* Charnok surface roughness from wind stress */
#endif
#ifdef LMD_MIXING
# define LMD_RIMIX       /* Add diffusivity due to shear instability */
# define LMD_CONVEC      /* Add convective mixing due to shear instability */
# undef LMD_DDMIX        /* Add double-diffusive mixing */
# define LMD_SKPP        /* Surface boundary layer KPP mixing */
# undef LMD_BKPP         /* Bottom boundary layer KPP mixing */
# define LMD_NONLOCAL    /* Nonlocal transport */
#endif

#undef SSH_TIDES         /* Imposing tidal elevation */
#undef UV_TIDES          /* Imposing tidal currents */
#undef RAMP_TIDES        /* Ramping (over one day) tidal forcing */

/* Output */
#define AVERAGES         /* 시간 평균된 결과로 저장 */
#define AVERAGES_FLUXES  /* 시간 평균된 flux로 저장 */
#undef DIAGNOSTICS_UV    /* Writing out momentum diagnostics */
#undef DIAGNOSTICS_TS    /* Writing out tracer diagnostics */

END
```



## 2-2. 컴파일 하기

프로젝트의 header 파일을 만들었으면, ROMS 모델을 컴파일 합니다. 다음과 같이 컴파일하고 표출되는 메세지에 error 가 없는지 유심히 살펴 봅니다. 최종적으로 "**romsM**" 이 제대로 생성되었는지 확인합니다.

```bash
cd $PROJ
\rm -f Build_roms romsM
./build_roms.bash      # 1개의 processor를 사용하여 컴파일
./build_roms.bash -j 8 # 시스템이 멀티코아를 가지고 있을 경우 최대 8개의 코아를 사용하여 컴파일
ls -l romsM
```



# 3. 격자 및 입력자료 만들기

사용자의 연구 상황에 따라서 격자 및 입력자료를 만드는 방법이 다를 수 있으며, 여기에서는 **pyroms를 이용하는 방법**을 설명하도록 하겠습니다.

여기에서는 제시된 스크립트들은 pyroms의 예제들을 참조하여 저자가 직접 만든 것입니다.

## 3-1. pyroms 및 jupyter 설치하기

pyroms는 설치하기가 쉽지않은 모듈이지만, 성공적으로 설치할 경우 편리성이 매우 높은 도구입니다.<br/>설치과정 대한 자세한 설명은 지면이 길어지므로 **[pyroms 설치하기](http://blog.dhkim.info/pyroms/)** 를 참고하여 설치하시기 바랍니다.

pyroms을 사용하여 격자 및 입력자료를 만드는 과정은 복잡하고 이해하기 힘들기 때문에 설명을 쉽게 하기 위하여 jupyter notebook 또는 jupyter lab을 사용하도록 하였습니다. Jupyter와 관련된 설치 및 사용법은 여기의 설명 범위에서 벗어나므로 인터넷 검색을 통하면 설치 및 사용법 익히시기 바랍니다. 또한, anaconda를 설치하였다면 jupyter가 이미 설치되어 있어서 바로 사용할 수 있습니다.

jupyter 가 아닌 python 으로 직접 실행하실 경우는 다음의 명령으로 *.ipynb 화일을 *.py 로 변환하여 사용하시기 바랍니다.

```bash
conda install jupyter # jupyter가 설치되어 있지 않은 경우
jupyter jupyter nbconvert --to script myFile.ipynb
# myFile.py 가 생성됨.
```



## 3-2. Grid 만들기

ROMS의 영역 격자를 만드는 방법은 모델의 역사 만큼이나 다양하게 존재합니다. 대표적으로 다음과 같은 것들이 있으니 참고하세요.

- Gridgen: http:code.google.com/p/gridgen-c, or https://github.com/sakov/gridgen-c
  - pyroms, pygridgen, octant
- Easygrid: https://www.myroms.org/wiki/easygrid 
- Gridbuilder: http://austides.com/downloads
- COAWST Tools: wrf2roms_mw.m, create_roms_xygrid.m
- … ...



1. Jupyter notebook의 형식으로 작성된 다음의 링크를 따라서 격자를 만듭니다.
   [makeGrid4Soulik](https://github.com/dohnkim/ROMS/blob/master/Grid/1_makeGrid4Soulik.ipynb)
2. 생성된 "Soulik_grd_v2.nc" 를 $PROJ 디렉토리에 복사합니다.



## 3-3. 초기장(I.C.) 만들기

1. **HYCOM** 자료 중에서 격자 크기에 맞는 영역의 해당 시작 날짜의 자료를 가져 옵니다. 
   [getHYCOM4SoulikIC](https://github.com/dohnkim/ROMS/blob/master/IC/1_getHYCOM4SoulikIC-v2.ipynb)
2. HYCOM 자료를 ROMS의 초기장으로 만듭니다.
   [makeIC4Soulik](https://github.com/dohnkim/ROMS/blob/master/IC/2_makeIC4Soulik-v2.ipynb)
3. 생성된 "ROMS4Soulik_ic_2018-08-21T00:00:00Z.nc4"를 $PROJ 디렉토리에 복사합니다.



## 3-4. 측면경계장(L.B.C.) 만들기

1. **HYCOM** 자료 중에서 격자의 측면에 맞는 영역에 대해 모델의 전체 계산 시간에 해당하는 시계열 자료를 가져 옵니다.
   [getHYCOM4SoulikLBCs](https://github.com/dohnkim/ROMS/blob/master/LBC/1_getHYCOM4SoulikLBCs-v2.ipynb)
2. HYCOM 격자와 ROMS 격자의 내삽 가중치 비율을 계산합니다.
   [makeRemapWeights](https://github.com/dohnkim/ROMS/blob/master/LBC/2_makeRemapWeights-v2.ipynb)
3. HYCOM 자료를 ROMS의 측면경계장으로 만듭니다.
   [makeLBC4Soulik](https://github.com/dohnkim/ROMS/blob/master/LBC/3_makeLBC4Soulik-v2.ipynb)
4. 생성된 "ROMS4Soulik_bc.nc"를 $PROJ 디렉토리에 복사합니다.



## 3-5. 대기경계장(S.B.C, 표층외력) 만들기

1. **GFS** 로 부터 모델의 전체 계산 시간에 해당하는 전지구 시계열 자료를 가져 옵니다. 
   [getGFS4SoulikSBC](https://github.com/dohnkim/ROMS/blob/master/SBC/1_getGFS4SoulikSBC-v1.ipynb)

2. GFS 자료를 ROMS의 대기경계장으로 만듭니다.
   [makeSBC4Soulik](https://github.com/dohnkim/ROMS/blob/master/SBC/2_makeSBC4Soulik-v2.ipynb)

3. 생성된 "roms_gfs_0p25_sbc_Soulik.nc"를 $PROJ 디렉토리에 복사합니다.

   

# 4. ROMS 모의하기



## 4-1. input file 만들기

ocean.in 파일은 ${ROMSsrc}/test 디렉토리에 있는 각 종 예제 자료들을 참조하여 만든 후 다음 내용을 수정합니다.<br/>예제 자료들에 있는 ocean.in 파일의 후반 부에는 각 변수들에 대한 자세한 사항이 있으니 참고하세요. 

```bash
cd $PROJ
cat ocean.in << END
... ...
    TITLE = Typhoon Soulik
 MyAppCPP = SOULIK
  VARNAME = ${ROMSsrc}/trunk/ROMS/External/varinfo.dat
       Lm == 338
       Mm == 338
        N == 30
   NtileI == 6
   NtileJ == 6
   NTIMES == 1440
       DT == 240.0d0
  NDTFAST == 20
  GRDNAME == ./Soulik_grd_v2.nc
  ININAME == ./ROMS4Soulik_ic_2018-08-21T00:00:00Z.nc4
  BRYNAME == ./ROMS4Soulik_bc.nc
  NFFILES == 1
  FRCNAME == ./roms_gfs_0p25_sbc_Soulik.nc
... ...
END
```



## 4-2. 모의하기

모델의 실행은 사용자의 시스템에 따라 다를 수 있으므로, 여기에서는 1개 노드에 36개 이상의 코어를 가지고 있는 시스템을 가정하여 다음과 같이 실행합니다.

```bash
mpirun -np 36 ./romsM ocean.in >& out.log
```

결과 파일로 ocean_avg.nc, ocean_his_????.nc, ocean_rst.nc  등이 제대로 생성되었는지 확인합니다.

# 5. ROMS 결과 분석하기

ROMS의 결과 분석은 다양한 방법이 있으며 과학적 지식이 필요한 부분이므로 추후에 다른 지면을 사용하여 설명하도록 하겠습니다.

 

 

 

---

=== END ===

---

