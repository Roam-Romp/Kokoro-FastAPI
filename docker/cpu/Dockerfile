FROM python:3.10-slim 

# Install dependencies and check espeak location
RUN apt-get update && apt-get install -y \
    espeak-ng \
    espeak-ng-data \
    git \
    libsndfile1 \
    curl \
    ffmpeg \
    g++ \
&& apt-get clean \
&& rm -rf /var/lib/apt/lists/* \
&& mkdir -p /usr/share/espeak-ng-data \
&& ln -s /usr/lib/*/espeak-ng-data/* /usr/share/espeak-ng-data/

# Create non-root user and set up directories and permissions
RUN useradd -m -u 1000 appuser && \
    mkdir -p /app/api/src/models/v1_0 && \
    touch /app/api/src/__init__.py && \
    touch /app/README.md && \
    chown -R appuser:appuser /app

USER appuser
WORKDIR /app

# Copy project files
COPY --chown=appuser:appuser api ./api
COPY --chown=appuser:appuser web ./web
COPY --chown=appuser:appuser docker/scripts/ ./
COPY --chown=appuser:appuser pyproject.toml ./pyproject.toml
RUN chmod +x ./entrypoint.sh

# Install Python dependencies directly
RUN pip install --user --no-cache-dir fastapi==0.115.6 uvicorn==0.34.0 pydantic==2.10.4 \
    numpy scipy==1.14.1 soundfile==0.13.0 aiofiles==23.2.1 tqdm==4.67.1 \
    requests==2.32.3 pydub>=0.25.1 \
    git+https://github.com/hexgrad/kokoro.git@31a2b6337b8c1b1418ef68c48142328f640da938

# Set environment variables
ENV PYTHONUNBUFFERED=1 \
    PYTHONPATH=/app:/app/api \
    PATH="/home/appuser/.local/bin:$PATH" \
    USE_GPU=false \
    PHONEMIZER_ESPEAK_PATH=/usr/bin \
    PHONEMIZER_ESPEAK_DATA=/usr/share/espeak-ng-data \
    ESPEAK_DATA_PATH=/usr/share/espeak-ng-data

ENV DOWNLOAD_MODEL=true
# Download model if enabled
RUN if [ "$DOWNLOAD_MODEL" = "true" ] && [ -f download_model.py ]; then \
    python download_model.py --output api/src/models/v1_0; \
    fi

ENV DEVICE="cpu"
# Run FastAPI server through entrypoint.sh
CMD ["./entrypoint.sh"]
