# 코드 진입점과 실행 준비

[프로젝트 소개](../README.md) · [평가 결과](EVALUATION.md)

학습·평가·서빙·앱의 코드 진입점과 실행 준비 사항입니다. 원본 이미지·학습 JSONL·LoRA·YOLO 가중치 및 GPU 환경을 별도로 준비해야 합니다.

```bash
git clone https://github.com/ho72/pet-i-vlm-diagnosis.git
cd pet-i-vlm-diagnosis
```

## VLM 학습

진입점은 [ai/main_model_train/train.py](../ai/main_model_train/train.py), 의존성 목록은 [ai/requirements.txt](../ai/requirements.txt)입니다. 모델은 `unsloth/Qwen3-VL-8B-Instruct`를 사용하며 현재 스크립트는 `/workspace/train_all.jsonl` 등 원래 학습 환경의 경로를 전제로 합니다. 데이터 내용과 이미지 경로, GPU·CUDA·Unsloth 호환 환경을 준비한 뒤 경로를 자신의 환경에 맞춰야 합니다.

```bash
# 데이터·가중치·경로·GPU 환경을 준비한 후 학습 실행
python ai/main_model_train/train.py
```

AI 의존성 목록은 버전이 고정되어 있지 않으며 `jsonlines`가 누락되어 있습니다. 실행 전에 `jsonlines`를 추가하고 GPU·CUDA·Unsloth에 맞는 의존성 버전을 구성해야 합니다.

## VLM 평가

진입점은 [ai/evaluation/validate.py](../ai/evaluation/validate.py)입니다.

```bash
python ai/evaluation/validate.py --help

# 평가 이미지와 자신의 학습 LoRA 디렉터리를 준비한 경우
python ai/evaluation/validate.py \
  --lora-path /path/to/checkpoint-15020 \
  --instruction-type json \
  --image-dir /path/to/evaluation-images
```

`/path/to/...`는 사용자가 준비하는 경로의 자리표시자입니다. 기본 이미지 경로는 `/workspace/eval_700/crop_padding_image`이며, 출력·라벨 판정 등도 기존 평가 코드를 확인해야 합니다.

## 설명형 데이터 생성

설명형 데이터 생성은 별도 [pet-i-explanation-data](https://github.com/ho72/pet-i-explanation-data)의 README를 따릅니다. 원본 AI Hub 이미지가 없어도 공개 샘플과 `--dry-run`으로 입력·컨텍스트·프롬프트를 확인할 수 있습니다. 통합 레포의 [ai/notebooks/auto_ctx.py](../ai/notebooks/auto_ctx.py)는 당시 코드 보관본입니다.

## RunPod 서빙

- [backend/src/handler.py](../backend/src/handler.py): 서버리스 작업 진입점.
- [backend/src/analysis.py](../backend/src/analysis.py): 이미지 분석·보고서 생성 모듈.
- [backend/src/rag_chatbot.py](../backend/src/rag_chatbot.py): 검색·대화 컨텍스트 모듈.
- [backend/requirements.txt](../backend/requirements.txt): 당시 GPU 서빙 의존성 조합.

기본 모델·학습 LoRA·YOLO 가중치 경로, GPU 메모리, RunPod의 템플릿·환경변수·요청 형식을 자신의 배포에 맞춰 준비해야 합니다. 실제 인증정보를 Git에 넣지 않습니다.

현재 [Dockerfile](../backend/Dockerfile)의 `COPY`는 루트의 `handler.py`, `eye_analysis_module.py`, `eye_rag_chatbot2.py`를 참조하지만 저장소는 `src/handler.py`, `src/analysis.py`, `src/rag_chatbot.py` 구조입니다. Docker 이미지를 빌드하려면 `COPY` 경로와 모듈 import를 실제 파일 배치에 맞춰 수정해야 합니다.

## Flutter와 FastAPI 중계

```bash
cd mobile
flutter pub get
flutter run -d chrome
```

Flutter 코드·자산이 들어 있으며 Dart SDK 조건은 [pubspec.yaml](../mobile/pubspec.yaml)을 따릅니다. 위 명령은 화면 개발 진입점입니다. 실제 분석·대화 사용에는 중계 API와 모델 서버 연결 설정이 추가로 필요합니다.

중계 앱은 [mobile/app_backend/main.py](../mobile/app_backend/main.py)이며 시작 명령은 해당 디렉터리에서 `uvicorn main:app`입니다. 중계 [requirements.txt](../mobile/app_backend/requirements.txt)는 UTF-16 인코딩의 기존 환경 목록입니다.
