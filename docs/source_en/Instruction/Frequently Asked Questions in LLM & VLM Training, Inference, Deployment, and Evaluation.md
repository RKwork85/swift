# Frequently Asked Questions in LLM & VLM Training, Inference, Deployment, and Evaluation

Here are some common issues encountered when using Swift.

## Training

### Q1: Which models and datasets are supported for fine-tuning with Swift?
For details, please refer to the documentation [Supported-models-datasets](https://swift.readthedocs.io/en/latest/Instruction/Supported-models-datasets.html).

### Q2: What data formats are supported when using custom datasets for training?
For the LLM custom dataset format, please refer to the documentation [Customization and Extension](https://swift.readthedocs.io/en/latest/Instruction/Customization.html).
VLM custom dataset format, different models support different numbers of images. For specific details, please refer to the best practice documentation corresponding to the model [Multi-Modal Documentation](https://swift.readthedocs.io/en/latest/Multi-Modal/index.html).

### Q3: How to use a custom dataset through this method, using the custom dataset format dataset_info.json?
For the format of dataset_info.json, refer to the documentation [Customization and Extension](https://swift.readthedocs.io/en/latest/Instruction/Customization.html). CLI，`--custom_dataset_info xxx.json`, `--dataset dataset_name`.

### Q4: How to train using a custom dataset in the UI interface?
Interface training using custom datasets is consistent with the command line. Refer to the documentation [Customization and Extension](https://swift.readthedocs.io/en/latest/Instruction/Customization.html).

### Q5: Can a line in the dataset's jsonl file be written like this? {"index": "00000", "query": "11111", "response": "22222", 'source':'qqq'}
Additional fields are allowed, but they will not be used.

### Q6: Where can I find documentation on command-line arguments?
See the document for details, [Command Line Arguments](https://swift.readthedocs.io/en/latest/Instruction/Command-line-parameters.html).

### Q7: What parameters need to be configured for offline environment training?
`--model_id_or_path 本地路径`, `--check_model_is_latest false`, for details, refer to the documentation [Command Line Arguments](https://swift.readthedocs.io/en/latest/Instruction/Command-line-parameters.html).

### Q8: Where can I check the model_type?
Check the document [Supported-models-datasets](https://swift.readthedocs.io/en/latest/Instruction/Supported-models-datasets.html).

### Q9: Can the model be directly converted to gguf format after training?
Currently only supports exporting ModelFile,  for details, see documentation [OLLaMA Export Documentation](https://swift.readthedocs.io/en/latest/LLM/OLLaMA-Export.html).

### Q10: Does swift support pre-training? I only see SFT (Supervised Fine-Tuning).
Supported, command line `swift pt`. For dataset format, see [Custom and Extension](https://swift.readthedocs.io/en/latest/Instruction/Customization.html).

### Q11: For a model fine-tuned using LoRA, if I want to resume training from a checkpoint, should I merge it into a complete model first, or can I directly specify the paths to the original model and LoRA blocks without merging?
No merging, `--resume_from_checkpoint output/xxx/vx-xxx/checkpoint-xxx`, for details, see [Command Line Arguments](https://swift.readthedocs.io/en/latest/Instruction/Command-line-parameters.html).

### Q12: I want to control the location of the original model weights downloaded from the internet. How can I place the original model in a specified folder?
You can configure the environment variable `MODELSCOPE_CACHE=your_path` to store the original model in a specified path; if using sdk to download, use `cache_dir="local_address"`; you can also use the modelscope download command-line tool or git to download, see modelscope documentation [Model Download](https://modelscope.cn/docs/Download%20Model) for details. During training, configure `--model_id_or_path` with the local path. If you need to train in an offline environment, configure `--check_model_is_latest false`, see [Command Line Arguments](https://swift.readthedocs.io/en/latest/Instruction/Command-line-parameters.html) for details.

### Q13: Has anyone encountered this issue when using ms-swift?
```text
[rank6]: pydantic_core._pydantic_core.ValidationError: 1 validation error for DeepSpeedZeroConfig
[rank6]: stage3_prefetch_bucket_size
[rank6]: Input should be a valid integer, got a number with a fractional part [type=int_from_float,input_value=11560550.4，in put_type=float]
[rank6]: For further information visit https://errors.pydantic.dev/2.8/v/int_fro_float
```
Downgrade deepspeed version to `0.14.*.`。

### Q14: Is there a complete tutorial and command line for fine-tuning Qwen-2-VL?
[Qwen2-VL Best Practice](https://swift.readthedocs.io/en/latest/Multi-Modal/qwen2-vl-best-practice.html).

### Q15: Are there any supported tricks for fine-tuning multimodal large models, similar to NEFTune for LLMs?
You can try `piassa/olora/dora` these `lora` variants or `fourierft`. Refer to various tricks in the `sft` parameters, some may not be applicable to multimodal tasks.

### Q16: The accuracy obtained during evaluation in the training process is not consistent with the accuracy calculated by re-inferencing using the corresponding saved checkpoint.
The eval_acc during training and the acc during inference are calculated differently. `acc_strategy`: default is `'token'`, available options include: `'token'`, `'sentence'`.

### Q17: Official ModelScope docker image and Swift environment
Start the container with the `docker run` command, e.g.: `docker run --gpus all -p 8000:8000 -it -d --name ms registry.cn-beijing.aliyuncs.com/modelscope-repo/modelscope:ubuntu22.04-cuda12.1.0-py310-torch2.3.0-tf2.16.1-1.16.0 /bin/bash`, after starting the container, pull the latest code and install swift.

### Q18: Command line for multi-node, multi-GPU training
```shell
# multi-node, multi-GPU
# If not using a shared disk, please additionally specify --save_on_each_node true in each machine's sh.
# node0
CUDA_VISIBLE_DEVICES=0,1,2,3 \
NNODES=2 \
NODE_RANK=0 \
MASTER_ADDR=127.0.0.1 \
NPROC_PER_NODE=4 \
swift sft \
    --model_id_or_path qwen/Qwen-7B-Chat \
    --dataset AI-ModelScope/blossom-math-v2 \
    --output_dir output \
# node1
CUDA_VISIBLE_DEVICES=0,1,2,3 \
NNODES=2 \
NODE_RANK=1 \
MASTER_ADDR=xxx.xxx.xxx.xxx \
NPROC_PER_NODE=4 \
swift sft \
    --model_id_or_path qwen/Qwen-7B-Chat \
    --dataset AI-ModelScope/blossom-math-v2 \
    --output_dir output \
```
For details, see [LLM Fine-tuning Documentation](https://swift.readthedocs.io/en/latest/Instruction/LLM-fine-tuning.html).

### Q19: How to choose a template?
See [issue](https://github.com/modelscope/ms-swift/issues/1813).

### Q20: How to use torchrun and Swift SFT for multi-GPU training?
`swift sft` uses `torchrun`.

### Q21: I have a question: my SFT dataset is too large, and tokenizing takes a long time each time. Is there a solution?
Use lazy_tokenize, see [Command Line Arguments](https://swift.readthedocs.io/en/latest/Instruction/Command-line-parameters.html) for details.

### Q22: During training, if two datasets are directly appended together in the training set, does the model have an internal shuffling process during training? Or does it take data in order for training?
Randomization occurs in the trainer.

## Inference

### Q1:Is there documentation for Swift inference?
Swift supports Python script, command line, and UI interface inference, see [LLM Inference Documentation](https://swift.readthedocs.io/en/latest/Instruction/LLM-inference.html) for details.

### Q2: How to use the trained model for inference on a dataset?
Parameter `--load_dataset_config true` or `--val_dataset <your-val-dataset>`, see documentation [LLM Fine-tuning Documentation](https://swift.readthedocs.io/en/latest/Instruction/LLM-fine-tuning.html#).

### Q3: Can we specify a pre-downloaded model when using Swift for inference?
Configure `--model_id_or_path` with the local path, see [Command Line Arguments](https://swift.readthedocs.io/en/latest/Instruction/Command-line-parameters.html) for details.

### Q4: I want to perform inference on a dataset without labels. How can I do this? I see that the dataset formats in the documentation are all for training sets.
Configure parameter `--val_dataset <your-val-dataset>`.

### Q5: I encountered an error: ValueError: Input length of input_ids is 35, but max_length is set to 20. How to solve this?
```text
raise ValueError(
ValueError: Input length of input_ids is 35, but `max_length` is set to 20. This can lead to unexpected behavior. You should consider increasing `max_length` or, better yet, setting `max_new_tokens`.
```
Set `model.generation_config.max_new_tokens`.

### Q6: Qwen2-VL inference causes out of memory error
Set environment variables, `SIZE_FACTOR=8 MAX_PIXELS=602112`, see documentation [Qwen2-VL Best Practice](https://swift.readthedocs.io/en/latest/Multi-Modal/qwen2-vl-best-practice.html).

## Deployment

### Q1: How to deploy the trained model?
`swift deploy --ckpt_dir xxx`, see documentation [VLLM Inference Acceleration and Deployment](https://swift.readthedocs.io/en/latest/LLM/VLLM-inference-acceleration-and-deployment.html).

### Q2: How to use vLLM for multi-GPU deployment?
`RAY_memory_monitor_refresh_ms=0 CUDA_VISIBLE_DEVICES=0,1,2,3 swift deploy --model_type qwen-7b --tensor_parallel_size 4`, see documentation [VLLM Inference Acceleration and Deployment](https://swift.readthedocs.io/en/latest/LLM/VLLM-inference-acceleration-and-deployment.html).

### Q3: When deploying with vLLM, how can the client pass in images?
See multimodal documentation, [vLLM Inference Acceleration Documentation](https://swift.readthedocs.io/en/latest/Multi-Modal/vllm-inference-acceleration.html) for details.

### Q4: I have a question: when deploying qwen2-7b and using the client, we need to use client.completions.create with the OpenAI API, not client.chat.completions.create. However, when using qwen2-7b-instruct-q5_k_m.gguf, we can use client.chat.completions.create. Why is this?
Base models can use client.chat.completions.create, but this is a compatibility behavior.

### Q5: After starting the server with Swift deploy using two GPUs, when exiting with Ctrl+C, there's always a Python process that keeps occupying the memory of one GPU. Is this normal?
Need to kill it, this is a vllm issue.

## Evaluation

### Q1: What evaluation datasets does Swift support?
NLP eval datasets：
```text
'obqa', 'cmb', 'AX_b', 'siqa', 'nq', 'mbpp', 'winogrande', 'mmlu', 'BoolQ', 'cluewsc', 'ocnli', 'lambada',
'CMRC', 'ceval', 'csl', 'cmnli', 'bbh', 'ReCoRD', 'math', 'humaneval', 'eprstmt', 'WSC', 'storycloze',
'MultiRC', 'RTE', 'chid', 'gsm8k', 'AX_g', 'bustm', 'afqmc', 'piqa', 'lcsts', 'strategyqa', 'Xsum', 'agieval',
'ocnli_fc', 'C3', 'tnews', 'race', 'triviaqa', 'CB', 'WiC', 'hellaswag', 'summedits', 'GaokaoBench',
'ARC_e', 'COPA', 'ARC_c', 'DRCD'
```

Multi Modal eval datasets：
```text
'COCO_VAL', 'MME', 'HallusionBench', 'POPE', 'MMBench_DEV_EN', 'MMBench_TEST_EN', 'MMBench_DEV_CN', 'MMBench_TEST_CN',
'MMBench', 'MMBench_CN', 'MMBench_DEV_EN_V11', 'MMBench_TEST_EN_V11', 'MMBench_DEV_CN_V11',
'MMBench_TEST_CN_V11', 'MMBench_V11', 'MMBench_CN_V11', 'SEEDBench_IMG', 'SEEDBench2',
'SEEDBench2_Plus', 'ScienceQA_VAL', 'ScienceQA_TEST', 'MMT-Bench_ALL_MI', 'MMT-Bench_ALL',
'MMT-Bench_VAL_MI', 'MMT-Bench_VAL', 'AesBench_VAL', 'AesBench_TEST', 'CCBench', 'AI2D_TEST', 'MMStar',
'RealWorldQA', 'MLLMGuard_DS', 'BLINK', 'OCRVQA_TEST', 'OCRVQA_TESTCORE', 'TextVQA_VAL', 'DocVQA_VAL',
'DocVQA_TEST', 'InfoVQA_VAL', 'InfoVQA_TEST', 'ChartQA_TEST', 'MathVision', 'MathVision_MINI',
'MMMU_DEV_VAL', 'MMMU_TEST', 'OCRBench', 'MathVista_MINI', 'LLaVABench', 'MMVet', 'MTVQA_TEST',
'MMLongBench_DOC', 'VCR_EN_EASY_500', 'VCR_EN_EASY_100', 'VCR_EN_EASY_ALL', 'VCR_EN_HARD_500',
'VCR_EN_HARD_100', 'VCR_EN_HARD_ALL', 'VCR_ZH_EASY_500', 'VCR_ZH_EASY_100', 'VCR_ZH_EASY_ALL',
'VCR_ZH_HARD_500', 'VCR_ZH_HARD_100', 'VCR_ZH_HARD_ALL', 'MMDU', 'MMBench-Video', 'Video-MME',
'MMBench_DEV_EN', 'MMBench_TEST_EN', 'MMBench_DEV_CN', 'MMBench_TEST_CN', 'MMBench', 'MMBench_CN',
'MMBench_DEV_EN_V11', 'MMBench_TEST_EN_V11', 'MMBench_DEV_CN_V11', 'MMBench_TEST_CN_V11', 'MMBench_V11',
'MMBench_CN_V11', 'SEEDBench_IMG', 'SEEDBench2', 'SEEDBench2_Plus', 'ScienceQA_VAL', 'ScienceQA_TEST',
'MMT-Bench_ALL_MI', 'MMT-Bench_ALL', 'MMT-Bench_VAL_MI', 'MMT-Bench_VAL', 'AesBench_VAL',
'AesBench_TEST', 'CCBench', 'AI2D_TEST', 'MMStar', 'RealWorldQA', 'MLLMGuard_DS', 'BLINK'
```

See documentation [LLM Evaluation Documentation](https://swift.readthedocs.io/en/latest/Instruction/LLM-eval.html) for details.

### Q2: How to use custom evaluation datasets?
Custom evaluation datasets for NLP and multimodal must follow the data format (pattern) of an official evaluation dataset, see documentation [LLM Evaluation Documentation](https://swift.readthedocs.io/en/latest/Instruction/LLM-eval.html).
