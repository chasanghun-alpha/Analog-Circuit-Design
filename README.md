# Multistage Amplifier Design

**과목/분야**: 전자회로 (2025년 2학기 설계 과제, 2인 팀) — 보고서 표지 과목명: 전자회로2
**기간**: ~2025.11 (제출일 2025.11.26)
**사용 도구**: LTspice

## 개요
Differential input / single-ended output 구조의 2-stage MOSFET 증폭기를 설계했습니다.
Gain, 대역폭, 입력 저항, 전력 사양을 LTspice 시뮬레이션으로 만족시켰습니다.

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
- R_D 전압강하 3 V로 drain 7 V, 동작점을 (V_DD+V_ov)/2 ≈ 5 V로 잡아 bias 저항비 7:3에서 시작했습니다. 이후 R_in 조건을 맞추려고 하단 저항을 1.2 MΩ으로 조정했습니다.
- gm = √(2k_nI_D) = 13.86 mS → A_vm = gm·R_D ≈ 69 (≈ 36 dB), 시뮬레이션 36 dB와 일치

**Stage 2 — Current-mirror active load differential amplifier (PMOS M5/M6 load)**
- 폭 비 4배의 MOS로 current load 전류를 4배(I_D ≈ 2.4 mA)로 키웠습니다. Stage 1 전류원을 건드리지 않고 gain만 높이기 위해서입니다.
- M5/M6, M7/M8 saturation 동작 확인, Wilson MOS mirror current sink
- gm ≈ 55 mS, Rout‖R_L ≈ 100 Ω → A_vm ≈ 5.5 (≈ 14.8 dB), 시뮬레이션 약 15 dB와 일치

**대역폭 · 입력 저항 — Miller compensation**
- 저역: 결합 커패시터(C1~C4)와 C5-R_L(fc ≈ 100 Hz)이 high-pass 역할
- 고역: 기생 pole이 수 MHz에 있어 대역폭이 너무 넓었습니다. 게다가 C_gd의 Miller 효과(`C_in = C_gs + C_gd(1+|A_v|)`) 때문에 고주파 R_in이 감소했습니다.
- Stage 1 출력 사이에 **C_m = 150 pF**를 추가해 100 kHz dominant pole을 만들었습니다. Mid-band gain을 유지하면서 R_in을 약 20 kΩ 끌어올렸습니다.

## 결과
- **AC**: Midband 100 Hz ~ 100 kHz, **A_vm = 51.195 dB**
- **R_in**: 100 kΩ @ 100 kHz
- **Transient**: Vin_max = 200 µV → Vout_max = 72.15 mV
- **Total power**: 72 mW (조건 500 mW 대비 현저히 낮음)

## 배운 점 / 의의
- 고주파 R_in 저하는 I_D, R_D, 결합 커패시터 조정만으로는 gain 손실 없이 해결되지 않았습니다. Miller compensation 커패시터로 gain을 유지하면서 대역폭과 R_in을 함께 맞췄습니다.
- 한계와 고찰:
  - R_in을 맞추려고 동작점을 옮기면서 signal swing이 줄었습니다(trade-off).
  - Stage 2 bias를 Stage 1과 같은 방식으로 구성해 최적화가 부족했습니다.
  - Rout이 큰 2단에 작은 R_L을 직접 달아 실용성이 떨어집니다. 일반적으로는 source follower 출력단이 적합합니다.
  - 낮은 전력이 swing 범위와 noise 측면에서 항상 장점은 아닙니다.
