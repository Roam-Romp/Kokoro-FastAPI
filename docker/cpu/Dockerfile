FROM python:3.10-slim

# 필수 패키지 설치
RUN apt-get update && apt-get install -y \
    espeak-ng \
    espeak-ng-data \
    git \
    libsndfile1 \
    ffmpeg \
    g++ \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && mkdir -p /usr/share/espeak-ng-data \
    && ln -s /usr/lib/*/espeak-ng-data/* /usr/share/espeak-ng-data/

WORKDIR /app

# 필요한 파일 복사
COPY api ./api
COPY web ./web
COPY docker/scripts/ ./
RUN chmod +x ./entrypoint.sh

# Python 패키지 직접 설치 (pip install . 대신)
RUN pip install --no-cache-dir fastapi==0.115.6 uvicorn==0.34.0 pydantic==2.10.4 \
    numpy scipy==1.14.1 soundfile==0.13.0 aiofiles==23.2.1 tqdm==4.67.1 \
    requests==2.32.3 pydub>=0.25.1 \
    git+https://github.com/hexgrad/kokoro.git@31a2b6337b8c1b1418ef68c48142328f640da938

# 환경 변수 설정
ENV PYTHONUNBUFFERED=1 \
    PYTHONPATH=/app:/app/api \
    PHONEMIZER_ESPEAK_PATH=/usr/bin \
    PHONEMIZER_ESPEAK_DATA=/usr/share/espeak-ng-data \
    ESPEAK_DATA_PATH=/usr/share/espeak-ng-data \
    DEVICE="cpu"

# entrypoint.sh 실행
CMD ["./entrypoint.sh"]
