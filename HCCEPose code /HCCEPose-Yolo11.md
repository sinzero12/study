
- 파일 경로 찾기 + 열기
```
open /users/shinyoung/HCCEPose/yolo_project
```
1. 터미널 다시 켜고 환경 준비
```
# 1. 프로젝트 폴더로 이동
cd ~/yolo_project

# 2. 가상환경 활성화 (이걸 해야 'pip'으로 설치했던 것들이 작동해요)
source venv/bin/activate

# 3. Iriun 화면이 맥북에 잘 나오고 있는지 확인 (안 나오고 있다면 Iriun 앱들 재실행)
```

2. 실행 및 사각형 확인
```
python yolo_live.py
```

***

- 터미널에서 직접 실행한 경우
```
git clone https://github.com/WangYuLin-SEU/HCCEPose.git
cd ~/HCCEPose
mkdir yolo_project
python3 -m venv venv
source venv/bin/activate

pip install --upgrade pip && \
pip install setuptools wheel "numpy<2.0" "opencv-python<4.10" imgaug scipy pyyaml \ matplotlib tqdm trimesh plyfile bop-toolkit-lib pycocotools pypng ruamel.yaml pytz \ 
ultralytics

cp -r HccePose/kasal ./kasal

unzip HccePose/bop_toolkit.zip -d .

open yolo_result_2d.jpg
```
