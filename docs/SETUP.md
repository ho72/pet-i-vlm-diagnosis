# 코드 진입점과 실행 준비

[프로젝트 소개](../README.md) · [평가 결과](EVALUATION.md)

이 저장소는 학습·평가·서빙·앱 코드의 통합 스냅샷입니다. 원본 데이터 전체와 학습된 LoRA·YOLO 가중치가 모두 포함된 즉시 실행 패키지는 아닙니다. GPU 학습·배포를 새로 검증한 설치 안내가 아니라, 현재 소스와 필요한 외부 자원을 구분한 진입 안내입니다.

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

AI 의존성 목록은 버전이 고정되어 있지 않으며, 스크립트에서 사용하는 `jsonlines`도 목록에 없습니다. 위 명령을 clone 직후 실행하거나 현재 임의의 최신 패키지 조합으로 원래 실험을 재현할 수 있다고 보장하지 않습니다. 이번 정리에서는 학습 소스·의존성 조합을 변경하지 않았습니다.

## VLM 평가

진입점은 [ai/evaluation/validate.py](../ai/evaluation/validate.py)입니다. 코드 헤더의 옛 파일명과 달리 실행 파일은 `validate.py`입니다.

```bash
python ai/evaluation/validate.py --help

# 평가 이미지와 자신의 학습 LoRA 디렉터리를 준비한 경우
python ai/evaluation/validate.py \
  --lora-path /path/to/checkpoint-15020 \
  --instruction-type json \
  --image-dir /path/to/evaluation-images
```

`/path/to/...`는 사용자가 준비하는 경로의 자리표시자입니다. 기본 이미지 경로는 `/workspace/eval_700/crop_padding_image`이며, 출력·라벨 판정 등도 기존 평가 코드를 확인해야 합니다. `--help`의 CLI 확인과 모델을 실행한 평가 검증은 다릅니다.

## 설명형 데이터 생성

현재 개선·검증한 실행 안내는 별도 [pet-i-explanation-data](https://github.com/ho72/pet-i-explanation-data)의 README를 따릅니다. 원본 AI Hub 이미지가 없어도 공개 샘플과 `--dry-run`으로 입력·컨텍스트·프롬프트를 확인할 수 있습니다. 통합 레포의 [ai/notebooks/auto_ctx.py](../ai/notebooks/auto_ctx.py)는 당시 코드 보관본입니다.

## RunPod 서빙

- [backend/src/handler.py](../backend/src/handler.py): 서버리스 작업 진입점.
- [backend/src/analysis.py](../backend/src/analysis.py): 이미지 분석·보고서 생성 모듈.
- [backend/src/rag_chatbot.py](../backend/src/rag_chatbot.py): 검색·대화 컨텍스트 모듈.
- [backend/requirements.txt](../backend/requirements.txt): 당시 GPU 서빙 의존성 조합.

기본 모델·학습 LoRA·YOLO 가중치 경로, GPU 메모리, RunPod의 템플릿·환경변수·요청 형식을 자신의 배포에 맞춰 준비해야 합니다. 실제 인증정보를 Git에 넣지 않습니다.

현재 [Dockerfile](../backend/Dockerfile)의 `COPY`는 루트의 `handler.py`, `eye_analysis_module.py`, `eye_rag_chatbot2.py`를 참조하지만 저장소는 `src/handler.py`, `src/analysis.py`, `src/rag_chatbot.py` 구조입니다. 따라서 현재 스냅샷을 그대로 `docker build`하면 완성된 서빙 이미지가 만들어진다고 안내하지 않습니다. 배포용 파일 배치와 모듈 import를 맞추는 후속 작업이 필요합니다. 이번 정리에서는 서빙 코드를 변경하거나 GPU 배포하지 않았습니다.

## Flutter와 FastAPI 중계

```bash
cd mobile
flutter pub get
flutter run -d chrome
```

Flutter 코드·자산이 들어 있으며 Dart SDK 조건은 [pubspec.yaml](../mobile/pubspec.yaml)을 따릅니다. 위 명령은 화면 개발 진입점입니다. 실제 분석·대화 사용에는 중계 API와 모델 서버 연결 설정이 추가로 필요합니다.

중계 앱은 [mobile/app_backend/main.py](../mobile/app_backend/main.py)이며 시작 명령은 해당 디렉터리에서 `uvicorn main:app`입니다. 중계 [requirements.txt](../mobile/app_backend/requirements.txt)는 UTF-16 인코딩의 기존 환경 목록입니다. 실제 배포 설정·키 파일은 이번 문서 검토 대상에서 제외했습니다. 현재 Flutter·중계·RunPod의 통합 연결을 재검증한 것은 아닙니다.

## 이번 확인 범위

최종 보고서의 본문·표와 공개 파일 경로·문서 상대 링크를 대조했습니다. GPU 모델·RunPod·Flutter를 설치·실행하거나 원래 결과를 재측정하지 않았습니다. 별도 데이터 생성 레포의 테스트와 이 레포의 전체 서비스 실행 성공을 혼동하지 않습니다.
