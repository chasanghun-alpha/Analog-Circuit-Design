# Multistage Amplifier Design

**과목/분야**: 전자회로 (2025년 2학기 설계 과제, 2인 팀) — 보고서 표지 과목명: 전자회로2
**기간**: ~2025.11 (제출일 2025.11.26)
**사용 도구**: LTspice

## 개요
Differential input / single-ended output 구조의 2-stage MOSFET 증폭기 설계
Gain, 대역폭, 입력 저항, 전력 사양을 LTspice 시뮬레이션으로 만족함

## 문제 정의
| 요구 조건 | 목표 | 달성 |
|---|---|---|
| 최종단 형태 | Differential input / single-ended output | ✅ |
| 3-dB Bandwidth | 100 Hz ~ 100 kHz | ✅ |
| 입력 저항 R_in | ≥ 100 kΩ | ✅ (100 kΩ @ 100 kHz) |
| 부하 저항 R_L | ≤ 100 Ω | ✅ |
| 전체 전압 이득 A_VM | ≥ 50 dB | ✅ (51.195 dB) |
| 총 소비 전력 | < 500 mW | ✅ (72 mW) |

## 설계 및 구현
**Stage 1 — Fully-NMOS differential amplifier (resistive load + tail current mirror)**
- Iref 1.2 mA(M3-M4 1:1 mirror) → M1, M2 각 0.6 mA
- AC coupling 입력(C1, C2)과 R_G gate bias
- R_D 전압강하 3 V로 drain 7 V, 동작점을 (V_DD+V_ov)/2 ≈ 5 V로 잡아 bias 저항비 7:3에서 시작. 이후 R_in 조건 충족을 위해 하단 저항을 1.2 MΩ으로 조정
- gm = √(2k_nI_D) = 13.86 mS → A_vm = gm·R_D ≈ 69 (≈ 36 dB), 시뮬레이션 36 dB와 일치

**Stage 2 — Current-mirror active load differential amplifier (PMOS M5/M6 load)**
- 폭 비 4배의 MOS로 current load 전류를 4배(I_D ≈ 2.4 mA)로 증가. Stage 1 전류원 변경 없이 gain만 높이기 위함
- M5/M6, M7/M8 saturation 동작 확인, Wilson MOS mirror current sink
- gm ≈ 55 mS, Rout‖R_L ≈ 100 Ω → A_vm ≈ 5.5 (≈ 14.8 dB), 시뮬레이션 약 15 dB와 일치

**대역폭 · 입력 저항 — Miller compensation**
- 저역: 결합 커패시터(C1~C4)와 C5-R_L(fc ≈ 100 Hz)이 high-pass 역할
- 고역: 기생 pole이 수 MHz에 있어 대역폭이 과도하게 넓음. 또한 C_gd의 Miller 효과(`C_in = C_gs + C_gd(1+|A_v|)`)로 고주파 R_in 감소
- Stage 1 출력 사이에 **C_m = 150 pF**를 추가해 100 kHz dominant pole 형성. Mid-band gain을 유지하면서 R_in을 약 20 kΩ 향상

## 결과
- **AC**: Midband 100 Hz ~ 100 kHz, **A_vm = 51.195 dB**
- **R_in**: 100 kΩ @ 100 kHz
- **Transient**: Vin_max = 200 µV → Vout_max = 72.15 mV
- **Total power**: 72 mW (조건 500 mW 대비 현저히 낮음)

## 배운 점 / 의의
- 고주파 R_in 저하는 I_D, R_D, 결합 커패시터 조정만으로는 gain 손실 없이 해결 불가. Miller compensation 커패시터로 gain을 유지하면서 대역폭과 R_in을 함께 맞춤
- 한계와 고찰:
  - R_in 충족을 위해 동작점을 옮기면서 signal swing 감소(trade-off)
  - Stage 2 bias를 Stage 1과 같은 방식으로 구성해 최적화 부족
  - Rout이 큰 2단에 작은 R_L을 직접 연결해 실용성 낮음. 일반적으로는 source follower 출력단이 적합
  - 낮은 전력이 swing 범위와 noise 측면에서 항상 장점은 아님
