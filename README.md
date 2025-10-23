# IEEE 802.15.4z UWB 거리측정 성능 최적화

> **UWB IC의 하드웨어 처리 시간과 물리계층(PHY) 파라미터에 따른 패킷 길이를 정밀 분석하여, SS-TWR 및 DS-TWR의 시간 파라미터를 최적화하고 정확도와 측정 속도를 동시 개선**

[![UWB](https://img.shields.io/badge/UWB-IEEE%20802.15.4z-blue)](https://www.ieee802.org/15/pub/TG4z.html)
[![Hardware](https://img.shields.io/badge/Hardware-DW3000%20+%20nRF52840-orange)]()

---

## 1. 프로젝트 개요

IEEE 802.15.4z UWB 기술은 실내 측위에서 우수한 정확도를 제공하지만, 실제 시스템 구현 시 **정확도(Accuracy)**와 **측정 시간(Latency)**은 상충 관계에 있습니다. 특히 거리 측정 방식(SS-TWR vs DS-TWR)이나 물리계층(PHY) 파라미터 설정에 따라 이 두 성능 지표가 크게 달라집니다.

본 프로젝트는 UWB TWR 방식에서 **시간 파라미터(예: `Treply`) 설정이 얼마나 중요한지**를 실험적으로 규명합니다. UWB IC(Qorvo DW3000)의 실제 하드웨어 처리 시간과 PHY 파라미터(프리앰블 길이 등)에 따른 정밀한 패킷 길이를 계산하여, **SS-TWR의 거리 오차를 방지**하고 **DS-TWR의 측정 횟수를 극대화**하는 최적의 시간 파라미터 선정 방법을 제안하고 구현합니다.

* **논문 (본 프로젝트 기반)**: "IEEE 802.15.4z UWB의 거리측정 정확도와 측정시간 최적화." *한국전자파학회논문지* 36.3 (2025): 274-282.

---

## 2. UWB 거리측정 원리 (SS-TWR & DS-TWR)

TWR(Two-Way Ranging)은 태그(Tag)와 앵커(Anchor) 간에 패킷을 교환하여 신호의 비행시간(ToF, Time of Flight)을 측정하는 방식입니다.

#### 2.1 SS-TWR (Single-Sided Two-Way Ranging)
<div align="center">
    <img src="/imgs/image-2.png" alt="SS-TWR 개념도" style="width:70%; height:auto; display:block; margin: 0 auto;">
</div>

#### 2.2 DS-TWR (Double-Sided Two-Way Ranging)
<div align="center">
    <img src="/imgs/image-3.png" alt="DS-TWR 개념도" style="width:70%; height:auto; display:block; margin: 0 auto;">
</div>

---

## 3. UWB TWR 측정 시간 (핵심 문제)

UWB 패킷의 SHR과 DP(데이터) 구간 길이는 전송 속도나 정보량에 따라 가변적입니다. 따라서 총 패킷 시간은 Preamble 심볼 시간과 Data 심볼 시간을 각각 계산한 후, 다음 식을 통해 구할 수 있습니다.

<div align="center">
    <img src="/imgs/image-4.png" alt="UWB 패킷 시간 계산식" style="width:70%; height:auto; display:block; margin: 0 auto;">
</div>

---

## 4. UWB 시간 파라미터 설정 방법 

본 프로젝트는 UWB IC의 물리적 한계와 PHY 파라미터를 모두 고려하여 최적의 시간 파라미터를 선정하는 방법을 제시합니다.

**SS-TWR MATLAB Simulation**
<div align="center">
    <img src="/imgs/image-5.png" alt="SS-TWR 시뮬레이션 결과 1" style="width:45%; height:auto; display:inline-block; margin: 0 2%;"> <img src="/imgs/image-6.png" alt="SS-TWR 시뮬레이션 결과 2" style="width:45%; height:auto; display:inline-block; margin: 0 2%;">
</div>

**DS-TWR MATLAB Simulation**
<div align="center">
    <img src="/imgs/image-7.png" alt="DS-TWR 시뮬레이션 결과" style="width:70%; height:auto; display:block; margin: 0 auto;">
</div>

---

## 5. 실험 결과

제안한 방법론에 따라 계산된 최적 시간 파라미터를 실제 하드웨어(Qorvo DW3000 + nRF52840)에 적용하여 실험한 결과, 제안한 결과와 일치함을 확인했습니다.

#### 5.1 SS-TWR: 오차 제거
`Treply`가 계산된 `Treply_min` 이상일 때 거리 오차가 안정적으로 수렴함을 검증했습니다. (논문 그림 5 참고)

> <div align="center">
>     <img src="/imgs/image-8.png" alt="SS-TWR Treply1 vs 거리 오차" style="width:40%; height:auto; display:block; margin: 0 auto;">
> </div>
>
> * **X축**: `Treply1` (시간)
> * **Y축**: 거리 오차 (분산 값)
> * **내용**: `Treply1`이 특정 시간(최소 처리 시간) 이하일 때 오차가 급증하고, 그 이상일 때 안정화되는 것을 보여주는 2D 그래프 (논문 그림 5)

#### 5.2 DS-TWR: 측정 시간 최적화
물리계층 파라미터(프리앰블 길이 등)에 따라 패킷 길이가 달라짐을 고려하여 `Treply`를 최적으로 설정하였고, 이를 통해 **가장 짧은 시간(최소 시간)**으로 거리 측정을 수행할 수 있음을 검증했습니다. (논문 표 3 참고)

> <div align="center">
>     <img width="300" height="300" alt="image" src="https://github.com/user-attachments/assets/7a5ccab9-f4af-4578-8e55-263c4622be08" /> <img width="300" height="300" alt="image" src="https://github.com/user-attachments/assets/832d7dff-91ed-49a7-924e-8e967d36c603" />


> </div>
> * **내용**: Preamble 128 설정과 Preamble 1024 설정을 비교.

---

## 6. 결론
* **(SS-TWR)** UWB IC가 처리할 수 있는 **최소 측정 시간**을 고려하지 않고 `Treply`를 설정하면 거리 오차가 증가함을 규명했으며, 이는 안정적인 시스템 설계에 필수적인 요소입니다.
* **(DS-TWR)** `Treply`를 최적으로 설정해야만 **주어진 시간 내 측정 횟수를 확보**하여 RTLS의 실시간성과 안정성을 높일 수 있음을 확인했습니다.
* 본 연구 결과는 UWB를 활용하는 모든 시스템(RTLS, 레이더 등)에서 **최소 시간으로 최대 효율**의 거리 측정을 구현하는 데 필요한 핵심 자료로 활용될 수 있습니다.
