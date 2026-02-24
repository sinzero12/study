## Environment setting

> [!NOTE]
>- iphone에 IruinWebcam, imac에도 IruinWebcam을 설치함 ~ iphone을 외부 웹캠으로 사용.
>    - wifi/bluetooth로도 가능하지만 연결이 잘 안 돼서 유선케이블 사용
>- 기존 yolo_test.py는 저장된 이미지/영상 분석용이므로 제외. 
> - 실시간 스트리밍 처리를 위해 yolo_live.py를 신규 제작하여 적용.   

#### yolo_live.py 구현
- 설명: Iriun Webcam 연동 및 실시간 탐지 로직   
- yolo_live.py 코드
```
import cv2
from ultralytics import YOLO

# 1. 모델 로드
# 기존: model = YOLO('yolo11n.pt')
# 변경: demo 폴더 안에 있는 전용 모델 경로를 넣습니다. (파일명은 다를 수 있으니 확인 필수!)
model_path = '/Users/shinyoung/HCCEPose/yolo_project/demo-bin-picking/demo-bin-picking/yolo11/train_obj_s/detection/obj_s/yolo11-detection-obj_s.pt'
# 위에서 찾은 모델 경로를 인자로 YOLO() 괄호 안에 넣어주기
model = YOLO(model_path)
# 2. 카메라 설정 (Iriun이 연결되었다면 0 또는 1 또는 2)
# 0이 맥북 카메라면 1이 Iriun일 확률이 높습니다.
cap = cv2.VideoCapture(1)

print("실시간 YOLO 창을 엽니다... (종료하려면 창을 클릭하고 'q' 키를 누르세요)")

while cap.isOpened():
    success, frame = cap.read()
    if not success:
        print("카메라를 찾을 수 없습니다. 인덱스(0, 1, 2)를 확인하세요.")
        break

    # YOLO 실시간 추론 (추론 결과 시각화)
    results = model.predict(frame, conf=0.5, show=True)

    # 'q' 키를 누르면 종료
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()
```
#### docker 환경 구축
> [!NOTE]
> - 실시간 영상 분석이 목적이므로, huggingface의 SEU-WYL/HccePose에서 test_imgs, test_videos 가져올 필요 없음.   
> - `yolo_live.py` 코드 내, `cap = cv2.VideoCapture(1)` 으로 작성해 1 = Iruin Webcam 사용 알리기.   
>   - Mac 내장 카메라, Iriun Webcam 번호 확인 이후 VideoCapture() 숫자 각자 수정.

- docker file 
```
# 1. 베이스 환경
FROM python:3.12-slim

# 2. 시스템 도구 설치
RUN apt-get update && apt-get install -y \
    git curl unzip libgl1 libglib2.0-0 \
    && rm -rf /var/lib/apt/lists/*

# 폴더를 만들고(mkdir) 그 안으로 이동해라(cd)
WORKDIR /app

# 3. 파이썬 라이브러리 (성공했던 버전 고정)
RUN pip install --upgrade pip && \
    pip install "numpy<2.0" torch torchvision ultralytics opencv-python \
    huggingface_hub bop-toolkit-lib pycocotools ruamel.yaml 
# ultralytics(YOLO)를 설치하면 matplotlib, scipy, tqdm, pyyaml 같은 기본 라이브러리들은 의존성에 의해 자동으로 함께 설치됨

RUN git clone https://github.com/WangYuLin-SEU/HCCEPose.git

# HuggingFace에서 핵심 데이터셋 다운로드 (자동화)
# --local-dir을 사용하여 경로를 명시적으로 지정합니다.
RUN huggingface-cli download SEU-WYL/HccePose --include "demo-bin-picking/*" --local-dir . --repo-type dataset

# 5. 파일 정리 자동화
# bop_toolkit 압축 해제 및 필요한 폴더 복사
RUN unzip -o HCCEPose/bop_toolkit.zip -d . || true && \
    cp -r HCCEPose/kasal ./kasal || true && \
    rm -rf HCCEPose  # 사용이 끝난 클론 폴더는 용량 절약을 위해 삭제

# 6. 현재 폴더의 내 코드(yolo_test.py 등) 복사
COPY . .

CMD ["python", "yolo_live.py"]
```

- docker compose file

```
version: '3.8'

services:
  yolo-app:
    build: 
      context: .
      dockerfile: Dockerfile_yolo  # 아까 만든 도커파일 이름
    container_name: yolo_worker
    volumes:
      - .:/app  # 컨테이너 안에서 만든 결과물이 내 맥 폴더에도 바로 생김!
      # 내 맥북에 이미 데이터(demo-bin-picking 등)가 있다면 -> 도커는 그걸 그대로 씁
      # 새로 세팅하는 경우에는, volumes 코드는 주석처리하기
    stdin_open: true
    tty: true
    command: python yolo_live.py
```

## 실행
- 로컬 환경에 demo-bin-picking 데이터 준비 (docker-compose 파일에 있는 :./app 코드 때문에 작성)
```
cd ~/HCCEPose/yolo_project
huggingface-cli download SEU-WYL/HccePose --include "demo-bin-picking/*" --local-dir . --repo-type dataset
```
- 도커 실행
```
docker-compose up --build
```

- 결과 이미지
<p align="center">
  <img src="https://github.com/user-attachments/assets/1af5e2a0-7506-4eb9-bf7f-925a719e0a51" width="45%">
  <img src="https://github.com/user-attachments/assets/8377fc22-0c6b-4fd0-af73-96771e3fe054" width="45%">
</p>
