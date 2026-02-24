## yolo 설치 및 실행 가이드
> [!NOTE]
> 실시간 영상 분석이 목적이므로, huggingface의 SEU-WYL/HccePose에서 test_imgs, test_videos 가져올 필요 없음.   
> iphone에 IruinWebcam, imac에도 IruinWebcam을 설치함 ~ iphone을 외부 웹캠으로 사용.   
> `yolo_live.py` 코드 내, `cap = cv2.VideoCapture(1)` 으로 작성해 1 = Iruin Webcam 사용 알리기.
#### docker에서 실행하는 경우

1. 내 컴퓨터(로컬) 세팅
```
# 1. 폴더 이동
cd ~/HCCEPose/yolo_project

# 비어있는 맥 폴더가 도커의 입력 데이터들을 지워버릴 수 있으므로, 맥 폴더에 미리 입력 데이터를 다운받아 놓기
# demo-bin-picking은 모델이 입력 데이터를 해석하기 위한 설정이 들어있는 폴더이기 때문에 중요함. (객체 ID, 이름, 원본 설계도 )
huggingface-cli download SEU-WYL/HccePose --include "demo-bin-picking/*" --local-dir . --repo-type dataset
```
2. 도커 사용 후 실행
```
docker-compose up --build
```

3. 결과 이미지
<p align="center">
  <img src="https://github.com/user-attachments/assets/1af5e2a0-7506-4eb9-bf7f-925a719e0a51" width="45%">
  <img src="https://github.com/user-attachments/assets/8377fc22-0c6b-4fd0-af73-96771e3fe054" width="45%">
</p>

#### 내 로컬의 가상환경에서 실행하는 경우  
1. 내 컴퓨터(로컬) 가상환경 세팅
```
# 1. 프로젝트 폴더 이동
cd ~/HCCEPose/yolo_project

# 2. 가상환경 생성 (이걸 해야 venv 폴더가 생깁니다!)
python3 -m venv venv

# 3. 가상환경 활성화
source venv/bin/activate
```
2. 필수 라이브러리 설치
```
# 1. pip 자체를 최신 버전으로 업데이트
pip install --upgrade pip

# 2. 핵심 AI 및 영상처리 라이브러리 설치
pip install "numpy<2.0" torch torchvision torchaudio ultralytics opencv-python

# 3. 데이터 및 유틸리티 라이브러리 설치
pip install huggingface_hub bop-toolkit-lib pycocotools ruamel.yaml scipy matplotlib tqdm
```
3. Git 및 HuggingFace 데이터 다운로드
```
# 1. GitHub 코드 클론 (이미 폴더가 있다면 생략 가능)
git clone https://github.com/WangYuLin-SEU/HCCEPose.git

# 2. 핵심 데이터셋 다운로드 (현재 폴더 . 에 저장)
huggingface-cli download SEU-WYL/HccePose --include "demo-bin-picking/*" --local-dir . --repo-type dataset
```

4. 파일 정리 및 압축 해제
```
# 1. bop_toolkit 압축 해제
unzip -o HCCEPose/bop_toolkit.zip -d .

# 2. 필요한 부가 폴더 복사
cp -r HCCEPose/kasal ./kasal

# 3. 사용이 끝난 임시 클론 폴더 삭제 (선택 사항)
# rm -rf HCCEPose
```

5. 실행
```
python yolo_live.py
```

6. 결과 이미지
<p align="center">
  <img src="https://github.com/user-attachments/assets/1af5e2a0-7506-4eb9-bf7f-925a719e0a51" width="45%">
  <img src="https://github.com/user-attachments/assets/8377fc22-0c6b-4fd0-af73-96771e3fe054" width="45%">
</p>

