# TGI-guide
To install TGI first install the docker

1. Install the docker -> go through via given command https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04
2. After that try to pull TGI image from cloud else do it manually
     - For Manual installation use tgi.tar file to load using
         ``` docker load -i tgi.tar ```
    - after manual pulling or to start cloud pulling of docker image use
        docker run --dns=8.8.8.8 --gpus all --shm-size 1g -p 8080:80 -e HUGGING_FACE_HUB_TOKEN=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 -v /absolute/path/to/volume:/data ghcr.io/huggingface/text-generation-inference:2.4.0 --model-id meta-llama/Llama-3.2-11B-Vision --sharded false --num-shard 1 --max-batch-prefill-tokens 512 --max-batch-total-tokens 1024 --max-input-length 512 --max-total-tokens 1024

3. sudo docker run --gpus all --shm-size 8g -p 8080:80 -e HUGGING_FACE_HUB_TOKEN=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 -e http_proxy=http://172.16.205.246:9090 -e https_proxy=http://172.16.205.246:9090 -v /absolute/path/to/volume:/data ghcr.io/huggingface/text-generation-inference:2.4.0 --model-id google/gemma-2-9b-it --sharded false --max-batch-prefill-tokens 2048 --max-batch-total-tokens 8192 --max-input-length 2048 --max-total-tokens 8192


curl command to use in servers
http_proxy=http://172.16.205.246:9090 \
https_proxy=http://172.16.205.246:9090 \
curl http://172.16.34.235:8080/v1/chat/completions \
    --noproxy 172.16.34.235 \
    -X POST \
    -d '{
  "model": "tgi",
  "messages": [
    {
      "role": "system",
      "content": "You are an expert assistant skilled in explaining advanced concepts across machine learning, biology, history, and ethics."
    },
    {
      "role": "user",
      "content": "Can you explain how deep learning models, such as transformers, can be applied to biological data to uncover insights about the genetic basis of diseases like cancer? Start by describing the fundamental principles of deep learning and transformers, then explain how these principles are adapted to biological sequence data such as DNA or RNA. Provide real-world examples of models or studies that have successfully used transformers in this domain. Additionally, discuss the ethical implications of using AI in biological research, particularly in terms of data privacy and the potential for AI to reinforce biases in genetic studies. Finally, suggest a strategy for integrating domain expertise (e.g., from biologists and ethicists) into AI workflows to ensure responsible and effective use of these models in healthcare and research."
    }
  ],
  "stream": false,
  "max_tokens": 1500
}' \
    -H 'Content-Type: application/json'
