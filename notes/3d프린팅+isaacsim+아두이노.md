# 로봇공학 및 임베디드 시스템 개발 인수인계 문서 (Handover Document)
 
본 문서는 사용자와 AI 간의 고성능 워크스테이션(NVIDIA RTX 5070 탑재) 환경을 기반으로 진행된 **NVIDIA Isaac Sim, 3D 프린팅, 아두이노(Arduino) 연동 로봇암 및 로봇 시뮬레이션 개발**에 관한 핵심 기술 내용과 워크플로우를 정리한 통합 인수인계 파일입니다.
 
---
 
## 1. 프로젝트 개요 및 아키텍처
본 프로젝트는 가상 환경의 디지털 트윈(Digital Twin)과 실물 로봇을 연동하는 **Sim-to-Real(시뮬레이션-투-리얼)** 구현을 목표로 합니다.
 
### 🏗️ 시스템 아키텍처 구조
1. **NVIDIA Isaac Sim (상위 제어 및 가상 환경)**: 역기구학(IK) 계산, 강화학습(RL) 보행 및 행동 정책 수립, 가상 로봇 관절 상태 트래킹.
2. **Python API Bridge (데이터 중계)**: Isaac Sim의 실시간 관절 각도(Radian)를 추출하여 Degree 단위로 변환 후, 시리얼 통신을 통해 하드웨어로 송신.
3. **Arduino 제어기 (임베디드 제어)**: 수신된 각도 패킷을 파싱하여 각 관절의 서보 모터(PWM) 유기적 구동.
4. **3D 프린팅 기반 로봇암 (물리 하드웨어)**: 실제 다관절 매커니즘 작동 및 그리퍼 동작.
---
 
## 2. 하드웨어 및 기구 설계 (Mechanical & Electronics)
 
### 3D 프린팅 가이드
* **기구학적 분리**: Fusion 360 등 CAD 툴에서 설계 시 각 파트는 독립된 바디(Body)로 분리되어야 하며, 회전축(Joint)의 중심점이 명확해야 Isaac Sim 임포트 시 오류가 없습니다.
* **출력 가설 설정**: 서보 모터의 토크와 가속 구동 시 발생하는 관성력을 견디기 위해 내부 채움(**Infill 30~40% 이상**), 벽 두께(**Wall Line Count 3~4줄**)를 필수로 확보해야 합니다.
* **파일 변환**: 시뮬레이션 가상 환경 이식을 위해 최종 조립체를 **URDF(Unified Robot Description Format)** 파일 형태로 익스포트합니다.
### 🔌 전원 및 회로 구성 (Critical)
* **공통 접지(Common GND)**: 아두이노 내부 전원으로는 복수의 서보 모터(MG996R, SG90 등)를 구동할 수 없으므로, 외부 전원(5V 2A 이상)을 분리하여 공급해야 합니다. 이때 **외부 전원의 GND와 아두이노의 GND를 반드시 공통으로 묶어주어야** 신호 노이즈와 모터 떨림(Jittering) 현상을 방지할 수 있습니다.
---
 
## 3. 임베디드 소스코드 (Arduino C++)
 
아두이노는 Python 브릿지로부터 `각도1,각도2,각도3,그리퍼\n` 형태의 직렬 문자열 패킷을 수신받아 실시간 구동합니다.
 
```cpp
#include <Servo.h>
 
Servo joint1, joint2, joint3, gripper;
 
void setup() {
  Serial.begin(115200); // 실시간 고속 통신을 위해 115200 Baudrate 설정
  joint1.attach(3);
  joint2.attach(5);
  joint3.attach(6);
  gripper.attach(9);
}
 
void loop() {
  if (Serial.available() > 0) {
    // 줄바꿈 문자를 기준으로 패킷 수신
    String data = Serial.readStringUntil('\n');
    
    // 콤마(,) 기준 데이터 파싱 및 분할
    int comma1 = data.indexOf(',');
    int comma2 = data.indexOf(',', comma1 + 1);
    int comma3 = data.indexOf(',', comma2 + 1);
    
    if (comma1 > 0 && comma2 > 0 && comma3 > 0) {
      int angle1 = data.substring(0, comma1).toInt();
      int angle2 = data.substring(comma1 + 1, comma2).toInt();
      int angle3 = data.substring(comma2 + 1, comma3).toInt();
      int gripValue = data.substring(comma3 + 1).toInt();
      
      // 하드웨어 보호를 위한 제약 조건 적용 후 서보 모터 제어
      joint1.write(constrain(angle1, 0, 180));
      joint2.write(constrain(angle2, 0, 180));
      joint3.write(constrain(angle3, 0, 180));
      gripper.write(constrain(gripValue, 0, 180));
    }
  }
}