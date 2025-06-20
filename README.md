# 🛫 활주로 결함 객체 탐지 프로젝트 (Air Force Hackathon 예선)

본 리포지토리는 2025년 **공군 해커톤 예선**에서 진행된 고해상도 항공 이미지 기반 **활주로 결함 객체 탐지 프로젝트**의 구조와 흐름을 복원한 것입니다.  
해당 프로젝트는 인트라넷 환경에서 수행되었으며, 외부 반출이 불가능한 상황으로 인해 코드 전체를 직접적으로 옮기지는 못하였고, 주요 로직 흐름과 구조를 문서화하는 방식으로 정리하였습니다.

---

## 📌 프로젝트 개요

- **문제 유형:** 객체 탐지 (Object Detection)
- **목표:** 고해상도 활주로 이미지 내 8종류의 결함 및 Normal 이미지 탐지
- **입력 데이터:** 7600×7600 이상 크기의 항공 이미지
- **모델:** YOLOv11s (두 가지 학습 전략 + 앙상블 적용)
- **참여 인원:** 2명 (역할 분담은 유동적으로, 유사한 작업 병행)

---

### 🧰 Tech Stack

#### Language & Framework
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![YOLOv5/7/11](https://img.shields.io/badge/YOLOv11s-FFCC00?style=for-the-badge&logo=github&logoColor=black)

#### Data Processing & Visualization
![OpenCV](https://img.shields.io/badge/OpenCV-5C3EE8?style=for-the-badge&logo=opencv&logoColor=white)
![NumPy](https://img.shields.io/badge/NumPy-013243?style=for-the-badge&logo=numpy&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-150458?style=for-the-badge&logo=pandas&logoColor=white)

#### Environment
![Jupyter](https://img.shields.io/badge/Jupyter-F37626?style=for-the-badge&logo=jupyter&logoColor=white)
![CUDA](https://img.shields.io/badge/CUDA-76B900?style=for-the-badge&logo=nvidia&logoColor=white)

---

## ⚙️ 전체 파이프라인 흐름

### 1. 데이터 준비 및 분석

- `divide_to_train_val.ipynb`: 주어진 Train Set을 임의로 **Train/Validation으로 분할**
- `yolo_to_format.ipynb`: JSON 라벨 데이터를 YOLO format(txt)으로 변환
- OpenCV 기반 시각화를 통해 결함 bbox 빈도 및 크기 분석

### 2. 클래스 기반 데이터 가공

- 클래스별 bbox 크기에 따라 두 가지 전략 수립:
    - **소형 결함 (Class 0,1,2,3,4,6,7,8)** → `crop.ipynb`로 1024x1024 crop + bbox 미포함 이미지 제거
    - **대형 결함 (Class 4,5)** → `only2class.ipynb`로 1280x1280 resize 수행

### 3. YOLOv11s 이중 학습

- `cropped(1024x1024).ipynb`: crop된 데이터로 YOLOv11s 학습
- `only2class(1280x1280).ipynb`: 리사이즈된 대형 bbox 데이터로 학습
- 두 모델의 가중치 분리 저장 후, 앙상블 준비

### 4. 결과 추론 및 앙상블 후처리

- `generate_to_csv.ipynb`: 테스트 이미지 crop(stride=512) 및 resize 적용 후, 각각 추론
- Confidence threshold를 0.1로 설정하여 mAP@50 기준 최적화
- IOU Merge (threshold=0.01) 기반 결과 병합 → 겹치는 bbox 통합 처리

---

## 🔍 성능 개선 포인트

- 리사이즈 방식은 소형 결함 탐지 성능 저하 → 크롭 방식으로 개선
- GPU 메모리 이슈(OOM) 발생 → 크롭 및 선택적 학습 이미지 구성으로 해결
- IOU Merge를 통해 중복 bbox 통합 → 정확도 개선

---

## 🧠 주요 시행착오

- 리사이즈 vs 크롭 성능 비교 실험
- Confidence threshold 튜닝 (0.1이 최적)
- 다양한 결함 크기에 따른 처리 전략 분리
- 테스트용 데이터 추론 로직 구현 시 stride 조정 및 merge 기준 설정

---

## 🗂️ 주요 파일 구성 (복원 기준)

| 파일명 | 설명 |
|--------|------|
| `divide_to_train_val.ipynb` | Train/Validation 데이터 분할 |
| `yolo_to_format.ipynb` | JSON → YOLO 라벨 포맷 변환 |
| `crop.ipynb` | 이미지 1024x1024 크롭 및 bbox 필터링 |
| `only2class.ipynb` | Class 4/5 데이터 1280x1280 리사이즈 처리 |
| `cropped(1024x1024).ipynb` | YOLOv11s 학습 (소형 결함용) |
| `only2class(1280x1280).ipynb` | YOLOv11s 학습 (대형 결함용) |
| `generate_to_csv.ipynb` | 최종 테스트 이미지 추론 및 결과 병합 |

---

## 🛡️ 왜 전체 코드를 업로드하지 않았나요?

이 프로젝트는 **공군 인트라넷 내부에서 수행되었으며**, 외부 인터넷 접근이 차단된 환경이었습니다. 따라서 개발된 코드와 데이터는 복사나 이동이 불가능하였으며, 외부 반출 역시 보안상 허용되지 않았습니다.

그럼에도 불구하고, 프로젝트의 핵심 아이디어, 모델링 흐름, 데이터 가공 방식, 실험 결과를 재구성하여 공개하는 것이 이 리포지토리의 목적입니다.

---

## 📌 회고

- 실제 대규모 이미지 데이터를 다루며 실질적인 성능 최적화 흐름을 경험
- 제한된 컴퓨팅 환경에서의 효율적인 학습 구조 설계
- 데이터 기반 분석을 통해 모델 구조보다 전처리의 중요성을 체감
