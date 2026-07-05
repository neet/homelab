# Ollama -> vLLM移行

なんかOllamaよりもvLLMのほうがスケールするらしい。

## やったこと

### 素組み

とりあえず素組みしてみる。

```
mkdir ~/vllm && cd ~/vllm
uv venv --python 3.12 --seed
source .venv/bin/activate
uv pip install vllm --torch-backend=auto
```

これで配信できる？？？

```
source .venv/bin/activate
vllm serve Qwen/Qwen2.5-1.5B-Instruct
```

なんかうまくいかなかった。CUDA Toolkitが必要なのかしら？PyTorchってそのへんをいい感じにやってくれるんじゃなかったんだっけ…

![](https://i.imgur.com/xIjPhLZ.png)


[CUDA Toolkit 13.3 Update 1 Downloads | NVIDIA Developer](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=26.04&target_type=deb_network)を参考にCUDA Toolkitを入れてみる

```
sudo apt-get -y install cuda-toolkit-13-3
```

ん、これをやっても `nvcc` っていうバイナリが入らない…？

```
(vllm) neet@compute-mitaka-02:~/vllm$ nvcc
Command 'nvcc' not found, but can be installed with:
sudo apt install nvidia-cuda-toolkit
```

nvccは入らなかったけど動いた。さっきのメッセージ的に `/usr/local/cuda` または `nvcc` があればいいってことだったのかな？

![](https://i.imgur.com/C3LdLRV.png)

勘でQwen3.6も動かしてみる。設定は [club-3090/models/qwen3.6-27b/vllm/compose/dual/autoround-int4/fp8-mtp.yml at master · noonghunna/club-3090](https://github.com/noonghunna/club-3090/blob/master/models/qwen3.6-27b/vllm/compose/dual/autoround-int4/fp8-mtp.yml) から拝借。

```
vllm serve Lorbus/Qwen3.6-27B-int4-AutoRound \
  --quantization auto_round \
  --dtype float16 \
  --tensor-parallel-size 2 \
  --pipeline-parallel-size 1 \
  --gpu-memory-utilization 0.92 \
  --max-num-seqs 2 \
  --max-num-batched-tokens 8192 \
  --kv-cache-dtype fp8_e5m2 \
  --enable-auto-tool-choice \
  --tool-call-parser qwen3_coder \
  --reasoning-parser qwen3 \
  --enable-prefix-caching \
  --enable-chunked-prefill
```

### Docker で動かす

uv よりかは docker のほうが取り回しが良さそう。

NVIDIA Container Toolkit というのが必要らしいので入れる？

```
sudo apt install -y \
  nvidia-container-toolkit \
  nvidia-container-toolkit-base \
  libnvidia-container-tools \
  libnvidia-container1
```

nvidia-ctkでデーモンの設定をいじる

```
neet@compute-mitaka-02:~$ cat /etc/docker/daemon.json
cat: /etc/docker/daemon.json: No such file or directory
neet@compute-mitaka-02:~$ sudo nvidia-ctk runtime configure --runtime=docker
INFO[0000] Config file does not exist; using empty config
INFO[0000] Wrote updated config to /etc/docker/daemon.json
INFO[0000] It is recommended that docker daemon be restarted.
neet@compute-mitaka-02:~$ cat /etc/docker/daemon.json
{
    "runtimes": {
        "nvidia": {
            "args": [],
            "path": "nvidia-container-runtime"
        }
    }
```

Dockerを再起動

```
sudo systemctl restart docker
```

これで起動してみる

```
docker run \
  --runtime nvidia \
  --gpus all \
  -v ~/.cache/huggingface:/root/.cache/huggingface \
  --env "HF_TOKEN=$HF_TOKEN" \
  -p 8000:8000 \
  --ipc=host \
  vllm/vllm-openai:latest \
    Lorbus/Qwen3.6-27B-int4-AutoRound \
    --quantization auto_round \
    --dtype float16 \
    --tensor-parallel-size 2 \
    --pipeline-parallel-size 1 \
    --gpu-memory-utilization 0.92 \
    --max-num-seqs 2 \
    --max-num-batched-tokens 8192 \
    --kv-cache-dtype fp8_e5m2 \
    --enable-auto-tool-choice \
    --tool-call-parser qwen3_coder \
    --reasoning-parser qwen3 \
    --enable-prefix-caching \
    --enable-chunked-prefill
```

## 参考

- [Ollama vs. vLLM: A deep dive into performance benchmarking | Red Hat Developer](https://developers.redhat.com/articles/2025/08/08/ollama-vs-vllm-deep-dive-performance-benchmarking#)
