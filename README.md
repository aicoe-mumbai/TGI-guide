# TGI-guide
To install TGI first install the docker

1. Install the docker -> go through via given command https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04
2. After that try to pull TGI image from cloud else do it manually
     - For Manual installation use tgi.tar file to load using
         ``` docker load -i tgi.tar ```
    - after manual pulling or to start cloud pulling of docker image use
        docker run --dns=8.8.8.8 --gpus all --shm-size 1g -p 8080:80 -e HUGGING_FACE_HUB_TOKEN=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 -v /absolute/path/to/volume:/data ghcr.io/huggingface/text-generation-inference:2.4.0 --model-id meta-llama/Llama-3.2-11B-Vision --sharded false --num-shard 1 --max-batch-prefill-tokens 512 --max-batch-total-tokens 1024 --max-input-length 512 --max-total-tokens 1024

3. sudo docker run --gpus all --shm-size 8g -p 8080:80 -e HUGGING_FACE_HUB_TOKEN=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 -e http_proxy=http://172.16.205.246:9090 -e https_proxy=http://172.16.205.246:9090 -v /absolute/path/to/volume:/data ghcr.io/huggingface/text-generation-inference:2.4.0 --model-id google/gemma-2-9b-it --sharded false --max-batch-prefill-tokens 2048 --max-batch-total-tokens 8192 --max-input-length 2048 --max-total-tokens 8192
