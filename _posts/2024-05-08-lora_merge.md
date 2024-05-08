---
layout: single
title: "lora merge"
categories: model_training
tags: [deep learning, llm, huggingface, python]
---

* LoRA Merge
    - 병합 함수
        - merge_and_unload()
            - 어댑터 가중치를 기본 모델과 병합 (To eliminate latency)
            - 모델 가중치를 메모리에 유지하지 않음
        - 모델 저장하는 두 가지 방향
            - https://github.com/huggingface/peft/issues/754
            - https://huggingface.co/docs/peft/developer_guides/model_merging
        - merge_adapter()
            - 추후 어댑터 병합 해제하거나 가중치 복사본을 보관해야 하는 경우
            - unmerge_adapter()를 통해 기본 모델을 반환하는 옵션 있음

* phi3
    - vllm에서 지원 아직 안함
        - https://github.com/vllm-project/vllm/issues/4375
* phi2 
    - 58 "["
    - 2235 "##"
    - 628, 198 
    - 220 개행
    - 공백(space)과 줄바꿈(newline)을 나타내는 특수 기호 Ċ 및 Ġ
    - [198, 2235, 58]
    - 0.28364767846028516
    
sh dev.sh phi3.txt phi3

    
```python
ase_model_path = 'princeton-nlp/Sheared-LLaMA-2.7B'
lora_model_path = '/workspace/llama/sheared_llama_ft_new_1600'
output_dir = './peftModel/test'
peft_config = PeftConfig.from_pretrained(lora_model_path)

base_model = AutoModelForCausalLM.from_pretrained(
    base_model_path,
    torch_dtype=torch.float16,
    #trust_remote_code=True,
)

tokenizer = AutoTokenizer.from_pretrained(base_model_path)

new_model = PeftModel.from_pretrained(
    base_model,
    lora_model_path,
    torch_dtype=torch.float16,
)

new_model.eval()
print(f"Merging with merge_and_unload...")
base_model = new_model.merge_and_unload()

tokenizer.save_pretrained(output_dir)
base_model.save_pretrained(output_dir)

```