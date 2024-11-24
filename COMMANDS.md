
```bash
python -m venv .venv

source $STORAGE_DIR/hello-ollama/.venv/bin/activate

curl -fsSL https://ollama.com/install.sh | sh

export INFERENCE_MODEL="meta-llama/Llama-3.2-3B-Instruct"
# ollama names this model differently, and we must use the ollama name when loading the model
export OLLAMA_INFERENCE_MODEL="llama3.2:3b-instruct-fp16"
export LLAMA_STACK_PORT=5001

ollama run $OLLAMA_INFERENCE_MODEL --keepalive 60m


# Using net=host - regular docker networking not working for me
sudo docker run \
  -it \
  -p $LLAMA_STACK_PORT:$LLAMA_STACK_PORT \
  -v ~/.llama:/root/.llama \
  --net=host \
  llamastack/distribution-ollama \
  --port $LLAMA_STACK_PORT \
  --env INFERENCE_MODEL=$INFERENCE_MODEL \
  --env OLLAMA_URL=http://localhost:11434


pip install -r requirements.txt

llama-stack-client --endpoint http://localhost:$LLAMA_STACK_PORT models list

llama-stack-client --endpoint http://localhost:$LLAMA_STACK_PORT \
  inference chat-completion \
  --message "hello, what model are you?"

# Show logs
sudo journalctl -u ollama --no-pager

# Follow logs
sudo journalctl -u ollama --no-pager -f


# Ollama server not working
export INFERENCE_PORT=8000
export INFERENCE_MODEL=meta-llama/Llama-3.2-3B-Instruct
export CUDA_VISIBLE_DEVICES=0

sudo docker run --gpus all \
    -v ~/.cache/huggingface:/root/.cache/huggingface \
    --env "HUGGING_FACE_HUB_TOKEN=$HUGGING_FACE_HUB_TOKEN" \
    -p 8000:$INFERENCE_PORT \
    --ipc=host \
    vllm/vllm-openai:latest \
    --model $INFERENCE_MODEL

export INFERENCE_PORT=8000
export INFERENCE_MODEL=meta-llama/Llama-3.2-3B-Instruct
export LLAMA_STACK_PORT=5001

sudo docker run \
  -it \
  --net=host \
  -p $LLAMA_STACK_PORT:$LLAMA_STACK_PORT \
  -v ./run.yaml:/root/my-run.yaml \
  llamastack/distribution-remote-vllm \
  --yaml-config /root/my-run.yaml \
  --port $LLAMA_STACK_PORT \
  --env INFERENCE_MODEL=$INFERENCE_MODEL \
  --env VLLM_URL=http://localhost:$INFERENCE_PORT/v1
```