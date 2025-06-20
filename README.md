# ai-test
# EP 0 — **로컬 vs 클라우드 LLM 기술 보고서**

> **목적**  로컬 추론과 OpenAI API 호출 두 경로를 빠르게 비교하여, 앞으로의 학습·프로토타입 전략을 결정한다.<br>
> **작성일** 2025‑06‑19  **작성자** boadt

---

## 1 서론

LLM(대형 언어 모델) 실험을 시작할 때 맨 먼저 부딪히는 질문은 ―
**“모델을 내 PC에서 직접 돌릴까, 아니면 관리형 API를 쓸까?”** 입니다.
본 보고서는 30 분간의 스파이크 테스트로 두 방식을 비교한 결과를 기록합니다.

---

## 2 환경 및 전제 조건

| 구분     | 로컬 LLM                      | OpenAI API           |
| ------ | --------------------------- | -------------------- |
| 운영체제   | Windows 11 ×86‑64           | 동일                   |
| Python | 3.11.9 (Poetry 1.8)         | 동일                   |
| GPU    | RTX 3060 Laptop (8 GB VRAM) | 필요 없음                |
| 시크릿    | **HF\_TOKEN** (read)        | **OPENAI\_API\_KEY** |

---

## 3 실험 절차

\### 3‑1 로컬 LLM 워크플로우

1. `poetry init --python="^3.11" -n` → `pyproject.toml` 생성
2. `poetry add torch transformers accelerate sentencepiece`
3. 모델: **microsoft/Phi‑3‑mini‑4k‑instruct** (4 B, Apache 2.0, 게이트 없음)
4. 추론 코드 스니펫:

   ```python
   tok = AutoTokenizer.from_pretrained(MODEL)
   llm = AutoModelForCausalLM.from_pretrained(MODEL, device_map="auto")
   pipe = pipeline("text-generation", model=llm, tokenizer=tok)
   ```

\### 3‑2 OpenAI API 워크플로우

1. `pip install --upgrade openai`
2. `setx OPENAI_API_KEY "sk-…"`
3. 테스트 호출:

   ```python
   from openai import OpenAI
   client = OpenAI()
   resp = client.chat.completions.create(model="gpt-4o-mini", …)
   ```

---

## 4 실험 결과

| 지표          | 로컬 (Phi‑3 4B)           | OpenAI gpt‑4o‑mini |
| ----------- | ----------------------- | ------------------ |
| **시동 시간**   | 2 GB 모델 1회 다운로드 → 약 3 분 | 없음                 |
| **첫 응답 지연** | 7.1 초 (GPU)             | 1.4 초              |
| **토큰/초**    | 18 tok/s                | 200+ tok/s         |
| **비용**      | 0 USD (전력 제외)           | 약 \$0.002 / 50 tok |

---

## 5 문제 해결 로그

| 현상                            | 해결 방법                                           |
| ----------------------------- | ----------------------------------------------- |
| 401 / 403 (HF)                | `huggingface-cli login` 후 non‑gated 모델 사용       |
| `ModuleNotFoundError: openai` | `poetry add openai` 설치                          |
| `RateLimitError 429`          | 결제 추가 → Billing ▸ Usage limit ‘\$5 soft cap’ 설정 |

---

## 6 결론 및 권장 사항

* **프로토타이핑** : 속도·편의·저렴한 비용 때문에 **OpenAI API**가 우세.
* **오프라인·민감 데이터** : GPU ≥ 8 GB가 있을 때 **로컬 모델**로도 실험 가능.
* 두 경로를 모두 문서화해, 필요에 따라 선택적으로 사용한다.

```
