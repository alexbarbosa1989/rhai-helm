## Deploy RHAI Inference in Openshift
**Important:** The following example is configured in an SNO OpenShift cluster. The GPU was enabled following the [How to enable NVIDIA GPU acceleration in OpenShift Local](https://developers.redhat.com/articles/2025/11/27/how-enable-nvidia-gpu-acceleration-openshift-local#) article procedure

### Curren procedure is to serve VLLM with a model downloaded from HuggingFace

### To serve a model with an OCI modelcar image located in an image repository, use [modelcar](https://github.com/alexbarbosa1989/rhai-helm/tree/modelcar) branch

### Export variables
~~~
export HF_TOKEN=hf_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
export AUTHFILE=$XDG_RUNTIME_DIR/containers/auth.json
export STORAGECLASS=my-storageclass
~~~

Clone the project 
~~~
git clone https://github.com/alexbarbosa1989/rhai-helm
~~~

Configure your own values in `rhai-helm/values.yaml`. for example, your own model, namespace, and so on.

To install the chart with the default values and provided environment variables, run: 
~~~
helm install rhai-helm ./rhai-helm \
--create-namespace --namespace rhai-helm \
--set persistence.storageClass=$STORAGECLASS \
--set secrets.hfToken=$HF_TOKEN \
--set-file secrets.docker.dockercfg=$AUTHFILE
~~~

Get the exposed route:
~~~
oc get route
~~~
Expected output:
~~~
NAME         HOST/PORT                                 PATH   SERVICES     PORT   TERMINATION   WILDCARD
qwen-coder   qwen-coder-rhai-helm.<ocp-cluster-domain>        qwen-coder   8000                 None
~~~


Test the model via `cURL`:
~~~
curl -X POST "http://qwen-coder-rhai-helm.<ocp-cluster-domain>/v1/chat/completions"      -H "Content-Type: application/json"     --data '{
                "model": "qwen-coder",
                "messages": [
                        {
                                "role": "user",
                                "content": "What is the capital of France?"
                        }
                ]
        }'
~~~
Expected output:
~~~
{"id":"chatcmpl-b0eb92f5dcb2c5b7","object":"chat.completion","created":1772501603,"model":"qwen-coder","choices":[{"index":0,"message":{"role":"assistant","content":"The capital of France is Paris.","refusal":null,"annotations":null,"audio":null,"function_call":null,"tool_calls":[],"reasoning":null,"reasoning_content":null},"logprobs":null,"finish_reason":"stop","stop_reason":null,"token_ids":null}],"service_tier":null,"system_fingerprint":null,"usage":{"prompt_tokens":36,"total_tokens":44,"completion_tokens":8,"prompt_tokens_details":null},"prompt_logprobs":null,"prompt_token_ids":null,"kv_transfer_params":null}
~~~


To uninstall the deployed helm chart
~~~
helm uninstall rhai-helm
~~~
