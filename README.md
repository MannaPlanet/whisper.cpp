# 커스텀 whisper.cpp

참고 [오리지널 README 파일](/README_whisper.cpp.md)
  이를 포크하여 개발 추진

# 히스토리

## 버그 픽스 (2023-05-18)

개요
- [git-repo: 한국어 음성 인식 추가한 미세 튜닝 학습](https://github.com/MannaPlanet/ASR-for-noisy-edge-devices.git)
- 이를 whisper.cpp로 C++ 포팅하였더니 음성인식이 제대로 되지 않음
  - 생성 전후로 더미 값들이 많이 생성됨
  - 종료를 찾지 못해, whisper 모델의 컨텍스트 길이 한계인 30초까지 디코딩 해서야 마무리가 됨 (즉 추론 시간도 오래 걸림)
- 이에 대한 버그를 수정하여 whisper.cpp를 푸시함



# whisper.cpp의 음성 인식 확인

절차
1. 소스 다운로드: 
   ```bash
   git clone https://github.com/MannaPlanet/whisper.cpp.git
   ```
2. 임시폴더 생성: `whisper.cpp.models`
3. HuggingFace 허브로부터 학습 모델 다운 로드: `jangmin/whisper-medium-ko-1195h`
   ```bash
   git clone https://huggingface.co/jangmin/whisper-medium-ko-1195h
   ```
4. 소스 폴더로 이동하여 실행파일 `.main` 빌드
   ```bash
   cd whisper.cpp
   make
   ```
5. HuggingFace transformers 학습 모델을 whisper.cpp 포맷 (`ggml`)으로 변경
   ```bash
   python ./models/convert-h5-to-ggml.py  ../whisper-medium-ko-1195h/ ../whisper ../whisper.cpp.models
   mv ../whisper.cpp.models/ggml-model.bin ../whisper.cpp.models/whisper-medium-ko-1195h.bin
   ```
6. 4비트 양자화 (quantize)
   ```bash
   ./quantize ../whisper.cpp.models/whisper-medium-ko-1195h.bin ../whisper.cpp.models/whisper-medium-ko-1195h-q4_0.bin q4_0
   ```
7. 음성 인식 테스트
   ```bash
   ./main -m ../whisper.cpp.models/whisper-medium-ko-1195h-q4_0.bin -nf -bo 1 -bs 2 -ps -l ko -f ./manna_samples/sample.wav
   ``` 