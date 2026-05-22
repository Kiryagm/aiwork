
https://chat.deepseek.com/share/6i8eo3tonp80j516cq


## 1. Введение в локальные LLM и Qwen2.5-3B

Локальные LLM — это модели, которые вы запускаете на своём оборудовании, а не через API OpenAI или Anthropic. Главные преимущества: конфиденциальность (данные не уходят на сервер), отсутствие оплаты за токены, полный контроль над версией модели и возможность работы без интернета.

**Qwen2.5-3B** — модель от Alibaba Cloud из семейства Qwen2.5. Цифра 3B означает 3 миллиарда параметров. Это компромиссный размер: достаточно мала для запуска даже на CPU с 8-16 ГБ RAM, но при этом показывает результаты, близкие к более старым моделям на 7B (например, Llama 2).

**Ключевые характеристики Qwen2.5-3B для локальной работы:**

- **Контекстное окно**: 32 768 токенов (достаточно для обработки целых глав кода или документов)
- **Поддерживаемые форматы квантизации**: GPTQ, AWQ, GGUF (через llama.cpp)
- **Родные языки**: отличное знание английского, китайского, коде — хороший Python, неплохой JavaScript и SQL
- **Требования к RAM**: в полной точности (FP16) — ~6 ГБ; в 4-битном виде — ~2 ГБ

**Почему именно эта модель для старта?** Она загружается за 10-30 секунд на современном ноутбуке, даёт связные ответы по программированию и позволяет изучать пайплайны инференса без необходимости покупать дорогую видеокарту.

**Типичные боевые сценарии на Python:**
- Замена платного код-ассистента внутри изолированной среды
- Пакетная обработка конфиденциальных текстов (медицинские, финансовые данные)
- Локальный RAG (Retrieval-Augmented Generation) с векторизованными документами

**Главное ограничение:** 3B параметров — это не GPT-4. Модель будет галлюцинировать на сложных рассуждениях в 5+ шагов и плохо справляется с редкими языками программирования (например, Rust с нестандартными крейтами).

В следующей главе разберём, как поднять Qwen2.5-3B через `transformers` + `bitsandbytes` минимальным кодом. Скидывайте главу 2.



## 2. Настройка окружения и требования к оборудованию

### Минимальные и рекомендуемые конфигурации

Qwen2.5-3B — одна из самых доступных моделей для локального запуска. Вот точные цифры для разных сценариев:

**Формат FP16 (полная точность):**
- VRAM: ~6 ГБ
- RAM: дополнительно 2-3 ГБ на контекст
- Требования: GPU с 6+ ГБ (GTX 1060 6GB, RTX 2060, M1/M2 Mac с 8 ГБ)

**Формат INT4 (квантизация через bitsandbytes):**
- VRAM: ~2 ГБ
- RAM: ~3 ГБ
- Подходит для: любых GPU, включая старые и встроенные

**Формат GGUF (запуск на CPU):**
- Q4_K_M: ~1.8 ГБ на диске, ~3 ГБ RAM
- Q3_K_M: ~1.5 ГБ на диске, ~2.5 ГБ RAM
- Скорость на CPU: ~4-6 токенов/сек

**Проверка совместимости перед установкой:**
```python
# Запустите в Python, чтобы оценить возможности вашей системы
import torch, psutil, platform

print(f"CPU: {platform.processor()}")
print(f"RAM: {psutil.virtual_memory().total / 1e9:.1f} GB")
if torch.cuda.is_available():
    print(f"GPU: {torch.cuda.get_device_name(0)}")
    print(f"VRAM: {torch.cuda.get_device_properties(0).total_memory / 1e9:.1f} GB")
else:
    print("GPU не обнаружен — будет использоваться CPU (медленно для FP16)")
```

### Установка Python и менеджера окружений

**Для Windows:**
- Скачайте Python 3.10–3.12 с python.org
- При установке **обязательно** отметьте "Add Python to PATH"
- Альтернатива: установите Anaconda через conda create -n qwen python=3.10

**Для macOS/Linux:**
```bash
# macOS через brew
brew install python@3.11

# Linux (Ubuntu/Debian)
sudo apt update && sudo apt install python3.11 python3.11-venv
```

**Создание виртуального окружения (обязательно — изолирует зависимости):**
```bash
python -m venv qwen_env
# Windows: qwen_env\Scripts\activate
# macOS/Linux: source qwen_env/bin/activate
```

### Установка ключевых библиотек

**Базовый набор для всех сценариев:**
```bash
pip install --upgrade pip
pip install torch --index-url https://download.pytorch.org/whl/cu118  # Для CUDA 11.8
# или pip install torch  # Версия только для CPU
pip install transformers accelerate huggingface_hub
```

**Для запуска на GPU с 4-битной квантизацией:**
```bash
pip install bitsandbytes accelerate
```

**Для запуска на CPU через GGUF:**
```bash
pip install llama-cpp-python
# Для ускорения с BLAS:
CMAKE_ARGS="-DLLAMA_BLAS=ON -DLLAMA_BLAS_VENDOR=OpenBLAS" pip install llama-cpp-python
```

**Для упрощённого запуска (Ollama):**
```bash
# Скачайте с ollama.com, затем в терминале:
ollama run qwen2.5:3b
```

### Выбор правильного формата под ваше железо

| Ваше оборудование | Рекомендуемый формат | Команда загрузки |
|------------------|---------------------|------------------|
| GPU 6+ ГБ VRAM | FP16 | `model = AutoModelForCausalLM.from_pretrained("Qwen/Qwen2.5-3B-Instruct", torch_dtype=torch.float16)` |
| GPU 2-5 ГБ VRAM | 4-bit (bitsandbytes) | `from transformers import BitsAndBytesConfig; bnb_config = BitsAndBytesConfig(load_in_4bit=True)` |
| Только CPU, 8+ ГБ RAM | GGUF Q4_K_M | Скачайте .gguf из HuggingFace |
| CPU, 4-6 ГБ RAM | GGUF Q3_K_M | Тот же процесс, файл Q3_K_M |
| Очень мало памяти (2-3 ГБ) | GGUF Q2_K | Только для тестирования — потеря качества значительна |

### Проблемы и их решения

**Ошибка "CUDA out of memory"** → переключитесь на 4-битную версию или GGUF. FP16 требует 6 ГБ VRAM строго.

**Медленный инференс на CPU** → используйте GGUF с оптимизациями BLAS или уменьшите контекстное окно (по умолчанию 32K токенов).

**Ошибка импорта bitsandbytes на Windows** → требуется Visual Studio Build Tools. Альтернатива: используйте GGUF через llama-cpp-python.

**Проверка успешности установки:**
```python
from transformers import AutoTokenizer, AutoModelForCausalLM
import torch

model_name = "Qwen/Qwen2.5-3B-Instruct"
tokenizer = AutoTokenizer.from_pretrained(model_name)

# Для 4-битного варианта (экономит память)
from transformers import BitsAndBytesConfig
quant_config = BitsAndBytesConfig(load_in_4bit=True)
model = AutoModelForCausalLM.from_pretrained(
    model_name, 
    quantization_config=quant_config,
    device_map="auto"
)

print(f"Модель загружена на: {model.device}")
print(f"Параметры: {model.num_parameters() / 1e9:.1f}B")
```

Если код выполнился без ошибок — окружение готово. Если нет — начните с CPU-версии (удалите quantization_config) или используйте Ollama как самый простой путь.

Скидывайте главу 3.



## 3. Установка Transformers и зависимостей PyTorch

### Критическое требование к версиям

Для Qwen2.5-3B **обязательны** актуальные версии библиотек. Устаревшие пакеты вызывают конкретные ошибки:

| Ошибка | Причина | Решение |
|--------|---------|---------|
| `KeyError: 'qwen2'` | transformers < 4.37.0 | Обновить до ≥ 4.37.0 |
| `AttributeError: module 'torch.library' has no attribute 'register_fake'` | PyTorch < 2.4.0 | Обновить до ≥ 2.4.0  |

### Базовая установка (минимальный рабочий вариант)

```bash
pip install --upgrade torch transformers accelerate sentencepiece
```

Эта команда устанавливает:
- **torch** — движок тензорных вычислений (выбор CPU/GPU ниже)
- **transformers** — загрузка и запуск моделей от Hugging Face
- **accelerate** — оптимизация памяти и распределение по устройствам
- **sentencepiece** — токенизатор для Qwen (без него модель не загрузится) 

### Выбор версии PyTorch под ваше железо

**Для NVIDIA GPU (CUDA 11.8 или 12.1):**
```bash
# CUDA 11.8 (стабильнее для старых карт)
pip install torch --index-url https://download.pytorch.org/whl/cu118

# CUDA 12.1 (новые карты, RTX 30/40 серии)
pip install torch --index-url https://download.pytorch.org/whl/cu121
```

**Только CPU (нет GPU или мало VRAM):**
```bash
pip install torch --index-url https://download.pytorch.org/whl/cpu
```

**Проверка успешной установки:**
```python
import torch
print(f"PyTorch {torch.__version__}")
print(f"CUDA доступна: {torch.cuda.is_available()}")
if torch.cuda.is_available():
    print(f"Версия CUDA: {torch.version.cuda}")
```

### Полный набор для работы с Qwen2.5-3B

```bash
# Базовые (обязательные)
pip install torch transformers accelerate sentencepiece

# Для квантизации 4-bit (экономит VRAM)
pip install bitsandbytes

# Для LoRA и PEFT (тонкая настройка)
pip install peft

# Для GGUF-версий (запуск на CPU)
pip install llama-cpp-python

# Для GPTQ/AWQ (продвинутая квантизация)
pip install auto-gptq autoawq optimum
```

### Установка через conda (альтернатива)

```bash
conda create -n qwen python=3.10 -y
conda activate qwen
conda install pytorch torchvision torchaudio pytorch-cuda=11.8 -c pytorch -c nvidia
pip install transformers accelerate bitsandbytes sentencepiece
```

### Полный рабочий код с проверкой

```python
# test_install.py — запустите для проверки окружения
import sys
import torch
import transformers
import accelerate

print(f"Python {sys.version}")
print(f"PyTorch {torch.__version__}")
print(f"Transformers {transformers.__version__}")
print(f"Accelerate {accelerate.__version__}")

# Проверка CUDA
if torch.cuda.is_available():
    print(f"GPU: {torch.cuda.get_device_name(0)}")
    print(f"VRAM: {torch.cuda.get_device_properties(0).total_memory / 1e9:.1f} GB")
else:
    print("GPU не обнаружен — будет использоваться CPU")

# Проверка импорта Qwen
try:
    from transformers import AutoModelForCausalLM, AutoTokenizer
    print("✓ transformers готов")
except ImportError as e:
    print(f"✗ Ошибка: {e}")

# Проверка bitsandbytes (опционально)
try:
    import bitsandbytes as bnb
    print(f"✓ bitsandbytes {bnb.__version__} готов")
except ImportError:
    print("⚠ bitsandbytes не установлен (нужен только для 4-bit квантизации)")
```

### Типичные проблемы и решения

**Проблема:** `No module named 'torch'`  
**Решение:** PyTorch не установился. Переустановите с правильным `--index-url`.

**Проблема:** `CUDA out of memory` при первой загрузке  
**Решение:** Версия FP16 требует ~6 ГБ VRAM. Используйте 4-bit квантизацию (следующая глава).

**Проблема:** Ошибка `bitsandbytes` на Windows  
**Решение:** Требуется Visual Studio Build Tools. Как временное решение — используйте GGUF-версию через `llama-cpp-python`.

**Проблема:** Медленная загрузка из Hugging Face  
**Решение:** Используйте зеркало или загрузите модель заранее:  
```bash
huggingface-cli download Qwen/Qwen2.5-3B-Instruct --local-dir ./qwen_local
```

Если все проверки прошли — окружение готово. Следующая глава покажет, как загрузить модель с 4-битной квантизацией прямо на GPU.

Скидывайте главу 4.



## 4. Загрузка Qwen2.5-3B через AutoModelForCausalLM

### Базовая загрузка (FP16, полная точность)

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
import torch

model_name = "Qwen/Qwen2.5-3B-Instruct"

tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    torch_dtype=torch.float16,          # FP16 вместо FP32 (экономит память)
    device_map="auto",                   # Автоматическое распределение на GPU/CPU
    trust_remote_code=False              # Qwen2.5 не требует remote code
)

print(f"Модель загружена на: {model.device}")
print(f"Тип данных: {model.dtype}")
```

**Важное примечание:** `trust_remote_code=False` работает только для Qwen2.5. Для старых Qwen (1.0, 1.5) требовалось `True`.

### Загрузка с 4-битной квантизацией (рекомендуемый вариант)

```python
from transformers import AutoModelForCausalLM, AutoTokenizer, BitsAndBytesConfig
import torch

model_name = "Qwen/Qwen2.5-3B-Instruct"

# Конфигурация квантизации
bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,                   # Включить 4-bit
    bnb_4bit_quant_type="nf4",           # Тип квантизации: nf4 (лучше) или fp4
    bnb_4bit_compute_dtype=torch.float16, # Вычисления в FP16
    bnb_4bit_use_double_quant=True,      # Двойная квантизация (экономит ещё ~0.5 ГБ)
)

tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    quantization_config=bnb_config,
    device_map="auto",
    trust_remote_code=False
)

print(f"VRAM занято: {model.get_memory_footprint() / 1e9:.2f} GB")
```

**Результат:** ~2 ГБ VRAM вместо 6 ГБ. Качество почти не теряется для большинства задач.

### Загрузка с 8-битной квантизацией (баланс память/качество)

```python
bnb_config_8bit = BitsAndBytesConfig(
    load_in_8bit=True,
    llm_int8_threshold=6.0               # Порог для смещения в FP16
)

model = AutoModelForCausalLM.from_pretrained(
    model_name,
    quantization_config=bnb_config_8bit,
    device_map="auto"
)
# Занято: ~3 ГБ VRAM
```

### Загрузка для CPU (через трансформацию вручную)

```python
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    torch_dtype=torch.float32,           # CPU лучше работает с FP32
    device_map="cpu",
    low_cpu_mem_usage=True               # Экономит RAM при загрузке
)
```

**Внимание:** FP32 на CPU потребляет ~8 ГБ RAM и даёт 1-2 токена/сек. Для CPU используйте лучше GGUF (глава 6).

### Параметры для ограниченной памяти

```python
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    torch_dtype=torch.float16,
    device_map="auto",
    max_memory={0: "4GiB", "cpu": "8GiB"},  # GPU 4 ГБ, остальное на CPU
    offload_folder="./offload"               # Сброс на диск при нехватке
)
```

### Полный пример с обработкой ошибок

```python
from transformers import AutoModelForCausalLM, AutoTokenizer, BitsAndBytesConfig
import torch

def load_qwen(mode="4bit"):
    model_name = "Qwen/Qwen2.5-3B-Instruct"
    
    try:
        if mode == "4bit":
            bnb_config = BitsAndBytesConfig(load_in_4bit=True, bnb_4bit_quant_type="nf4")
            model = AutoModelForCausalLM.from_pretrained(
                model_name, quantization_config=bnb_config, device_map="auto"
            )
        elif mode == "8bit":
            bnb_config = BitsAndBytesConfig(load_in_8bit=True)
            model = AutoModelForCausalLM.from_pretrained(
                model_name, quantization_config=bnb_config, device_map="auto"
            )
        else:  # fp16
            model = AutoModelForCausalLM.from_pretrained(
                model_name, torch_dtype=torch.float16, device_map="auto"
            )
        
        tokenizer = AutoTokenizer.from_pretrained(model_name)
        return model, tokenizer
    
    except RuntimeError as e:
        if "out of memory" in str(e):
            print("Не хватает VRAM. Попробуйте режим '4bit'")
        raise e

# Использование
model, tokenizer = load_qwen("4bit")
```

### Кэширование модели (избегаем повторной загрузки)

```python
from huggingface_hub import snapshot_download

# Скачать один раз в локальную папку
local_path = "./models/qwen-2.5-3b"
snapshot_download(repo_id="Qwen/Qwen2.5-3B-Instruct", local_dir=local_path)

# Загружать оттуда
model = AutoModelForCausalLM.from_pretrained(local_path, ...)
```

### Сравнение режимов загрузки

| Режим | VRAM | RAM | Скорость (токен/сек) | Качество |
|-------|------|-----|---------------------|----------|
| FP16 (GPU) | 6 ГБ | 3 ГБ | 40-60 | 100% |
| 8-bit | 3 ГБ | 2 ГБ | 35-50 | 98-99% |
| 4-bit (nf4) | 2 ГБ | 1.5 ГБ | 30-45 | 96-97% |
| FP32 (CPU) | 0 | 8 ГБ | 1-3 | 100% |

**Рекомендация:** Начинайте с 4-bit. Только если видите серьёзное падение качества (например, в редких языках или точных расчётах) — пробуйте 8-bit или FP16.

Если загрузка прошла успешно — проверяем токенизатор и отправляем первый промпт. Скидывайте главу 5.



## 5. Загрузка Qwen2.5-3B через Pipeline API

### Почему Pipeline проще, чем AutoModelForCausalLM

Pipeline скрывает детали токенизации, генерации и декодирования. Вместо 15 строк кода — 3 строки. Минус — меньше контроля над внутренними параметрами.

### Базовая загрузка через pipeline

```python
from transformers import pipeline
import torch

pipe = pipeline(
    "text-generation",
    model="Qwen/Qwen2.5-3B-Instruct",
    torch_dtype=torch.float16,
    device_map="auto"
)

response = pipe("Hello, how are you?")
print(response[0]['generated_text'])
```

### Pipeline с 4-битной квантизацией

```python
from transformers import pipeline, BitsAndBytesConfig
import torch

bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.float16
)

pipe = pipeline(
    "text-generation",
    model="Qwen/Qwen2.5-3B-Instruct",
    model_kwargs={"quantization_config": bnb_config},
    device_map="auto"
)
```

### Настройка параметров генерации внутри pipeline

```python
pipe = pipeline(
    "text-generation",
    model="Qwen/Qwen2.5-3B-Instruct",
    torch_dtype=torch.float16,
    device_map="auto",
    max_new_tokens=512,          # Максимум новых токенов
    do_sample=True,              # Включить семплирование
    temperature=0.7,             # Случайность (0 = детерминированно)
    top_p=0.9,                   # Ядерное семплирование
    repetition_penalty=1.1,      # Штраф за повторения
    pad_token_id=151643          # ID паддинга Qwen2.5
)

response = pipe("Write a Python function to reverse a string")
```

### Pipeline с четкой инструкцией (промпт-формат Qwen)

```python
def qwen_chat(prompt, system_message="You are a helpful assistant."):
    # Формат для instruct-версии
    formatted = f"<|im_start|>system\n{system_message}<|im_end|>\n<|im_start|>user\n{prompt}<|im_end|>\n<|im_start|>assistant\n"
    
    result = pipe(formatted, max_new_tokens=256, do_sample=False)
    response = result[0]['generated_text'][len(formatted):]  # Обрезаем промпт
    
    return response

print(qwen_chat("Explain what is a dictionary in Python"))
```

### Пакетная обработка (batch inference)

```python
prompts = [
    "What is 2+2?",
    "Translate 'hello' to French",
    "Write a list of 3 colors"
]

# Pipeline автоматически батчит токенизацию
results = pipe(prompts, max_new_tokens=50, batch_size=3)

for i, res in enumerate(results):
    print(f"Prompt {i+1}: {res[0]['generated_text']}")
```

### Pipeline для CPU (медленно, но без GPU)

```python
pipe = pipeline(
    "text-generation",
    model="Qwen/Qwen2.5-3B-Instruct",
    device=-1,                   # Принудительно CPU
    torch_dtype=torch.float32    # CPU лучше с FP32
)
```

### Полный пример с обработкой ошибок и освобождением памяти

```python
from transformers import pipeline, BitsAndBytesConfig
import torch
import gc

def create_qwen_pipeline(use_4bit=True):
    try:
        if use_4bit:
            bnb_config = BitsAndBytesConfig(load_in_4bit=True)
            pipe = pipeline(
                "text-generation",
                model="Qwen/Qwen2.5-3B-Instruct",
                model_kwargs={"quantization_config": bnb_config},
                device_map="auto"
            )
        else:
            pipe = pipeline(
                "text-generation",
                model="Qwen/Qwen2.5-3B-Instruct",
                torch_dtype=torch.float16,
                device_map="auto"
            )
        return pipe
    
    except RuntimeError as e:
        print(f"Ошибка загрузки: {e}")
        return None

def cleanup_pipeline(pipe):
    del pipe
    gc.collect()
    torch.cuda.empty_cache() if torch.cuda.is_available() else None

# Использование
pipe = create_qwen_pipeline(use_4bit=True)
if pipe:
    result = pipe("Hello", max_new_tokens=20)
    print(result[0]['generated_text'])
    cleanup_pipeline(pipe)
```

### Ограничения Pipeline по сравнению с AutoModelForCausalLM

| Функция | Pipeline | AutoModelForCausalLM |
|---------|----------|----------------------|
| Контроль logits | ❌ (скрыто) | ✅ (через output_attentions) |
| Доступ к скрытым состояниям | ❌ | ✅ |
| Кастомная логика декодирования | ❌ | ✅ |
| Батчинг с разными длинами | ✅ | ❌ (нужен вручную) |
| Простота использования | ✅ | ❌ |
| Логирование токенов | ❌ | ✅ (через LogitsProcessor) |

### Практический выбор

**Используйте Pipeline если:**
- Пишете прототип или скрипт для себя
- Не нужен доступ к внутренним весам
- Хотите минимум кода

**Используйте AutoModelForCausalLM если:**
- Нужна точная настройка семплирования
- Требуется логирование вероятностей токенов
- Строите прод систему с кастомной постобработкой

### Проверка успешной загрузки

```python
pipe = pipeline("text-generation", model="Qwen/Qwen2.5-3B-Instruct", device_map="auto")
test_output = pipe("Test", max_new_tokens=5, return_full_text=False)

print("Статус: OK" if test_output else "Ошибка генерации")
print(f"Пример: {test_output[0]['generated_text'] if test_output else 'None'}")
```

Если видите осмысленный текст из 5 токенов — pipeline работает. Следующая глава покажет, как использовать GGUF-версию через llama.cpp (единственный разумный способ на CPU).

Скидывайте главу 6.



## 6. Понимание GGUF-квантизации и llama-cpp-python

### Что такое GGUF и почему это важно

GGUF (GGML Universal Format) — формат, разработанный для llama.cpp, библиотеки инференса на C++ с биндингами для Python. В отличие от квантизации через `bitsandbytes`, GGUF оптимизирован для **CPU-инференса** и смешанных сценариев CPU+GPU.

**Ключевое отличие от GPTQ/AWQ:** GGUF хранит веса уже квантованными на диске. Не нужно каждый раз загружать полную модель и квантизировать в памяти — вы скачиваете готовый `.gguf` файл и работаете с ним напрямую.

### Архитектура llama-cpp-python

```python
# Установка
pip install llama-cpp-python

# Базовая загрузка .gguf файла
from llama_cpp import Llama

llm = Llama(
    model_path="./qwen-3b-q4_k_m.gguf",  # Прямой путь к файлу
    n_ctx=4096,                           # Размер контекстного окна
    n_threads=4,                          # Количество CPU потоков
    n_gpu_layers=0                        # Слоёв на GPU (0 = только CPU)
)

response = llm("Explain Python decorators", max_tokens=256)
print(response["choices"][0]["text"])
```

### Загрузка напрямую из Hugging Face

```python
# Автоматическая загрузка из репозитория
from llama_cpp import Llama

llm = Llama.from_pretrained(
    repo_id="lmstudio-community/Qwen2.5-3B-Instruct-GGUF",  # 
    filename="Qwen2.5-3B-Instruct-Q4_K_M.gguf",             # 
    verbose=False
)
```

### Чат-формат Qwen2.5 через create_chat_completion

```python
response = llm.create_chat_completion(
    messages=[
        {"role": "system", "content": "You are a Python expert."},
        {"role": "user", "content": "Write a function to check if a number is prime."}
    ],
    max_tokens=512,
    temperature=0.7
)

print(response["choices"][0]["message"]["content"])
```

### Выбор правильного квантизационного файла для Qwen2.5-3B

Измерения perplexity для Qwen2.5-3B :

| Формат | Размер (MB) | PPL | Accuracy от F16 | Рекомендация |
|--------|-------------|-----|-----------------|--------------|
| F16 (baseline) | 5893 | 8.9888 | 100% | Только для GPU |
| Q8_0 | 3134 | 9.0114 | 99.75% | Макс. качество |
| Q6_K | 2421 | 9.0690 | 99.12% | Отличный выбор |
| Q5_K_M | 2122 | 9.1339 | 98.41% | Рекомендуется |
| Q5_K_S | 2070 | 9.1338 | 98.41% | + быстрее |
| **Q4_K_M** | **1841** | **9.2305** | **97.38%** | **Лучший баланс** |
| Q4_K_S | 1750 | 9.2573 | 97.10% | Компактнее |
| IQ4_XS | 1659 | 9.2935 | 96.72% | Новый формат |
| Q3_K_M | 1517 | 14.0989 | 63.76% | ❌ Плохое качество |
| IQ3_M | 1420 | 9.9957 | 89.93% | 3-битный компромисс |
| IQ2_M | 1088 | 12.8779 | 69.80% | Только для крайней экономии |
| IQ1_M | 811 | 42.7456 | 21.03% | Не использовать |

**Вывод по таблице:** Форматы Q3_K_M и Q3_K_L дают аномально высокий PPL (14+). Для Qwen2.5-3B **избегайте Q3_K_***. Используйте Q4_K_M как стандарт, IQ4_XS или IQ3_M если нужно меньше памяти .

### Конфигурация GPU-оффлоуда через n_gpu_layers

```python
# Частичная выгрузка на GPU (ускоряет инференс)
llm = Llama(
    model_path="./qwen-3b-q4_k_m.gguf",
    n_gpu_layers=30,        # Количество слоёв на GPU (всего в Qwen2.5-3B: 36 слоёв) 
    n_ctx=4096,
    n_threads=4
)
# С 30 слоями на GPU: ~2.5 ГБ VRAM, скорость ~40 токен/сек на RTX 3060
```

**Правило для n_gpu_layers:**
- `n_gpu_layers = -1` или `999` → все слои на GPU (макс. скорость)
- `n_gpu_layers = 0` → только CPU (медленно)
- Для Qwen2.5-3B с 4-битной квантизацией: 36 слоёв помещаются в 3-4 ГБ VRAM 

### Управление KV-кэшем и контекстом

```python
# Настройки для ограниченной памяти
llm = Llama(
    model_path="./qwen-3b-q4_k_m.gguf",
    n_ctx=8192,                  # Контекст 8K токенов
    n_batch=512,                 # Размер батча для промпта
    flash_attn=True,             # Flash Attention (экономия памяти) 
    cache_type_k="q8_0",         # KV-кэш в 8-бит (экономит ~50% памяти) 
    cache_type_v="q8_0",
    n_gpu_layers=36              # Все слои на GPU
)
```

**Формула KV-кэша для Qwen2.5-3B:**
- 32 слоя (фактически 36, упрощённо), 2 KV-головы, размерность 128 
- FP16: `2 * 32768 * 36 * 2 * 128 * 2 байта ≈ 1.2 ГБ` для 32K контекста
- В 8-битном KV-кэше: `~0.6 ГБ`

### Полный рабочий пример

```python
from llama_cpp import Llama
import json

class QwenGGUF:
    def __init__(self, model_path, n_gpu_layers=0, n_ctx=4096):
        self.llm = Llama(
            model_path=model_path,
            n_ctx=n_ctx,
            n_gpu_layers=n_gpu_layers,
            n_threads=4,
            flash_attn=True,
            verbose=False
        )
    
    def chat(self, user_message, system="You are a helpful assistant"):
        response = self.llm.create_chat_completion(
            messages=[
                {"role": "system", "content": system},
                {"role": "user", "content": user_message}
            ],
            max_tokens=512,
            temperature=0.7,
            top_p=0.9
        )
        return response["choices"][0]["message"]["content"]
    
    def stream_chat(self, user_message):
        # Потоковая генерация (токен за токеном)
        stream = self.llm.create_chat_completion(
            messages=[{"role": "user", "content": user_message}],
            max_tokens=256,
            stream=True
        )
        for chunk in stream:
            token = chunk["choices"][0]["delta"].get("content", "")
            print(token, end="", flush=True)

# Использование
qwen = QwenGGUF(
    model_path="./Qwen2.5-3B-Instruct-Q4_K_M.gguf",
    n_gpu_layers=36,      # Все 36 слоёв на GPU (если хватает VRAM)
    n_ctx=8192
)

response = qwen.chat("What is a closure in Python?")
print(response)
```

### Где скачать GGUF-файлы для Qwen2.5-3B

| Репозиторий | Форматы | Ссылка |
|-------------|---------|--------|
| lmstudio-community | Q4_K_M, Q5_K_M, Q8_0 | [Hugging Face](https://huggingface.co/lmstudio-community/Qwen2.5-3B-Instruct-GGUF)  |
| bartowski | IQ2_M, IQ3_M, Q4_K_M | [Hugging Face](https://huggingface.co/bartowski/Qwen2.5-3B-Instruct-GGUF)  |
| ThomasBaruzier | Полная таблица с PPL | [Hugging Face](https://huggingface.co/ThomasBaruzier/Qwen2.5-3B-Instruct-GGUF)  |

### Частая проблема: RAM не освобождается

При использовании `n_gpu_layers > 0` модель сначала загружается в RAM, затем копируется на GPU, но RAM остаётся занятой . Это ожидаемое поведение llama.cpp на Windows. Решения:
1. Игнорировать — Python процесс всё равно использует память
2. Использовать `mmap=False` (замедляет загрузку)
3. Перезапускать процесс после завершения работы

### GGUF vs BitsAndBytes: что выбрать

| Критерий | GGUF (llama-cpp-python) | BitsAndBytes |
|----------|------------------------|--------------|
| Целевое железо | CPU, CPU+GPU | GPU |
| Скорость на CPU | ✅ 5-15 токен/сек | ❌ 1-2 токен/сек |
| Скорость на GPU | ✅ 40+ токен/сек | ✅ 50+ токен/сек |
| Простота установки | ⚠️ Требует компиляцию на Windows | ✅ `pip install` |
| Совместимость с PEFT/LoRA | ❌ | ✅ |
| Качество 4-бита | 97-98% (Q4_K_M) | 96-97% (NF4)  |

**Вывод:** Для CPU используйте GGUF. Для GPU с 4+ ГБ VRAM можно использовать оба подхода. Исследования показывают, что GGUF стабильнее на низких битрейтах .

Следующая глава покажет, как отправлять запросы с правильным шаблоном чата Qwen2.5. Скидывайте главу 7.



## 7. Загрузка GGUF-моделей через класс Llama

### Основные параметры класса Llama

Класс `Llama` из библиотеки `llama-cpp-python` — это высокоуровневый API для работы с GGUF-моделями. Он управляет загрузкой весов, KV-кэшем, семплированием и генерацией .

**Критическая особенность:** В отличие от `transformers`, GGUF-модель хранится в одном `.gguf` файле. Не нужно загружать отдельно веса, конфиг и токенизатор — всё упаковано в единый файл .

### Способ 1: Загрузка из локального файла (самый надёжный)

```python
from llama_cpp import Llama

# Базовый вариант (CPU)
llm = Llama(
    model_path="./Qwen2.5-3B-Instruct-Q4_K_M.gguf",
    n_ctx=4096,        # Размер контекстного окна (токенов)
    n_threads=4,       # Количество CPU-потоков
    verbose=False
)

# Ускорение с GPU (если установлена GPU-версия)
llm = Llama(
    model_path="./Qwen2.5-3B-Instruct-Q4_K_M.gguf",
    n_gpu_layers=-1,   # -1 = все слои на GPU, иначе число слоёв
    n_ctx=8192,
    n_threads=4
)
```

**Параметр `n_gpu_layers`** :
| Значение | Результат |
|----------|-----------|
| `0` | Только CPU (2-5 токен/сек на хорошем CPU) |
| `20` | Первые 20 слоёв на GPU (остальные на CPU) |
| `-1` или `999` | Все слои на GPU (максимальная скорость) |

**Для Qwen2.5-3B (36 слоёв):**
- `n_gpu_layers=36` или `-1` → требуется ~2.5-3 ГБ VRAM в 4-бите

### Способ 2: Автоматическая загрузка из Hugging Face Hub

```python
from llama_cpp import Llama

# Самый простой метод — Llama.from_pretrained
llm = Llama.from_pretrained(
    repo_id="lmstudio-community/Qwen2.5-3B-Instruct-GGUF",
    filename="Qwen2.5-3B-Instruct-Q4_K_M.gguf",
    local_dir="./models/",      # Опционально: сохранить локально
    n_gpu_layers=-1,
    n_ctx=4096
)
```

Параметр `filename` поддерживает glob-паттерны :
```python
# Скачает первый файл, подходящий под паттерн
llm = Llama.from_pretrained(
    repo_id="lmstudio-community/Qwen2.5-3B-Instruct-GGUF",
    filename="*Q4_K_M.gguf",   # Любой Q4_K_M файл
)
```

### Доступные репозитории с Qwen2.5-3B GGUF

| Репозиторий | Форматы | Ссылка |
|-------------|---------|--------|
| `lmstudio-community/Qwen2.5-3B-Instruct-GGUF` | Q4_K_M, Q5_K_M, Q8_0 | [HF] |
| `bartowski/Qwen2.5-3B-Instruct-GGUF` | IQ4_XS, IQ3_M, Q4_K_M | [HF] |
| `Lucy-in-the-Sky/Qwen2.5-3B-Instruct-Q8_0-GGUF` | Q8_0 | [HF] |
| `mradermacher/Qwen2.5-3B-instruct-medical-finetuned-GGUF` | IQ4_XS (fine-tuned) | [HF] |

### Полный пример с чат-форматом Qwen

```python
from llama_cpp import Llama

llm = Llama.from_pretrained(
    repo_id="lmstudio-community/Qwen2.5-3B-Instruct-GGUF",
    filename="Qwen2.5-3B-Instruct-Q4_K_M.gguf",
    n_gpu_layers=-1,          # Все слои на GPU
    n_ctx=8192,               # Контекст 8K токенов
    n_threads=6,              # CPU-потоков для预热ки
    flash_attn=True,          # Flash Attention (экономия памяти)
    verbose=False
)

# Правильный формат чата для Qwen2.5 Instruct
response = llm.create_chat_completion(
    messages=[
        {"role": "system", "content": "You are a Python expert."},
        {"role": "user", "content": "Write a function to reverse a linked list."}
    ],
    max_tokens=512,
    temperature=0.7,
    top_p=0.9,
    stop=["<|im_end|>"]       # Стоп-токен Qwen
)

print(response["choices"][0]["message"]["content"])
```

### Ключевые параметры инициализации

**Управление памятью и скоростью** :

```python
llm = Llama(
    model_path="./model.gguf",
    
    # Контекст
    n_ctx=4096,              # Размер контекста (по умолчанию 512!) 
    n_batch=512,             # Токенов за один forward pass
    
    # GPU
    n_gpu_layers=-1,         # Число слоёв на GPU (-1 = все)
    main_gpu=0,              # Индекс основного GPU (при нескольких)
    tensor_split=[0.5, 0.5], # Распределение между GPU
    
    # Производительность
    n_threads=4,             # CPU-потоков
    flash_attn=True,         # Flash Attention для длинных контекстов
    
    # KV-кэш (экономия памяти)
    cache_type_k="q8_0",     # Тип KV-кэша: f16, q8_0, q4_0
    cache_type_v="q8_0",
    
    # Прочее
    seed=42,                 # Фиксация random seed
    verbose=False,           # Логирование
    use_mlock=False,         # Блокировка страниц в RAM
    use_mmap=True            # Memory-mapped файлы
)
```

**Внимание:** По умолчанию `n_ctx=512` — для Qwen с 32K контекстом это критично! Всегда указывайте `n_ctx` явно .

### Потоковая генерация (token by token)

```python
# Метод create_chat_completion с stream=True
stream = llm.create_chat_completion(
    messages=[{"role": "user", "content": "Count from 1 to 10"}],
    max_tokens=100,
    stream=True
)

for chunk in stream:
    token = chunk["choices"][0]["delta"].get("content", "")
    print(token, end="", flush=True)
```

### Практический пример: класс-обёртка с кэшированием

```python
from llama_cpp import Llama
from functools import lru_cache

class QwenGGUF:
    _instances = {}
    
    def __new__(cls, quant="Q4_K_M", n_gpu_layers=-1):
        key = f"{quant}_{n_gpu_layers}"
        if key not in cls._instances:
            cls._instances[key] = super().__new__(cls)
        return cls._instances[key]
    
    def __init__(self, quant="Q4_K_M", n_gpu_layers=-1):
        if hasattr(self, '_initialized'):
            return
        self._initialized = True
        
        quant_map = {
            "Q4_K_M": "Qwen2.5-3B-Instruct-Q4_K_M.gguf",
            "Q5_K_M": "Qwen2.5-3B-Instruct-Q5_K_M.gguf",
            "Q8_0": "Qwen2.5-3B-Instruct-Q8_0.gguf"
        }
        
        self.llm = Llama.from_pretrained(
            repo_id="lmstudio-community/Qwen2.5-3B-Instruct-GGUF",
            filename=quant_map[quant],
            n_gpu_layers=n_gpu_layers,
            n_ctx=8192,
            verbose=False
        )
    
    def generate(self, prompt, max_tokens=256, temperature=0.7):
        response = self.llm(
            prompt,
            max_tokens=max_tokens,
            temperature=temperature,
            echo=False
        )
        return response["choices"][0]["text"]

# Использование
model = QwenGGUF(quant="Q4_K_M", n_gpu_layers=-1)
print(model.generate("Explain Python decorators in one sentence:"))
```

### Типичные проблемы

| Проблема | Решение |
|----------|---------|
| `FileNotFoundError` | Убедитесь, что путь к `.gguf` правильный |
| Медленный инференс на GPU | Проверьте `n_gpu_layers=-1` и установку `llama-cpp-python` с GPU-поддержкой: `CMAKE_ARGS="-DGGML_CUDA=on" pip install llama-cpp-python` |
| Ошибка `failed to load model` | Файл повреждён — перекачайте |
| RAM не освобождается после `del llm` | Ожидаемое поведение на Windows. Перезапустите процесс или используйте `mmap=False` |

Следующая глава покажет, как правильно форматировать промпты для Qwen2.5 Instruct и работать с системными сообщениями.

Скидывайте главу 8.



## 8. Конфигурация токенизатора и применение шаблона чата

### Устройство токенизатора Qwen2.5

Qwen2.5 использует токенизатор на базе **tiktoken** (как GPT-4) с размером словаря 151 936 токенов. В отличие от BPE-токенизаторов Llama, он более эффективен для китайского и кода, но требует особого подхода к специальным токенам.

**Ключевые токены Qwen2.5:**
```
<|im_start|>     - 151644 (начало сообщения в чате)
<|im_end|>       - 151645 (конец сообщения)
<|fim_prefix|>   - 151659 (для задач заполнения кода)
<|fim_suffix|>   - 151660
<|fim_middle|>   - 151661
<|endoftext|>    - 151643 (аналог EOS)
```

### Базовое использование токенизатора

```python
from transformers import AutoTokenizer

model_name = "Qwen/Qwen2.5-3B-Instruct"

tokenizer = AutoTokenizer.from_pretrained(model_name)

# Обычная токенизация
tokens = tokenizer("Hello, Qwen!")
print(f"Текст: {tokens}")
print(f"IDs: {tokens['input_ids']}")
print(f"Декодировано: {tokenizer.decode(tokens['input_ids'])}")

# Проверка спецтокенов
print(f"pad_token: {tokenizer.pad_token} (id: {tokenizer.pad_token_id})")
print(f"eos_token: {tokenizer.eos_token} (id: {tokenizer.eos_token_id})")
print(f"bos_token: {tokenizer.bos_token} (id: {tokenizer.bos_token_id})")
# Вывод: pad_token: <|endoftext|> (id: 151643)
#        bos_token: None (Qwen не использует BOS)
```

**Важно:** У Qwen2.5 нет отдельного BOS-токена. Первый токен в последовательности — всегда первый токен контента.

### Шаблон чата (chat template)

Шаблон хранится в файле `tokenizer_config.json` и применяется через `apply_chat_template()`:

```python
messages = [
    {"role": "system", "content": "You are a Python coding assistant."},
    {"role": "user", "content": "Write a function to reverse a string"},
    {"role": "assistant", "content": "Here's a simple solution:"},
    {"role": "user", "content": "Now make it recursive"}
]

# Применение шаблона
formatted = tokenizer.apply_chat_template(
    messages,
    tokenize=False,           # Возвращает строку, не токены
    add_generation_prompt=True  # Добавить <|im_start|>assistant\n в конце
)

print(formatted)
```

**Результат (с сокращениями):**
```
<|im_start|>system
You are a Python coding assistant.<|im_end|>
<|im_start|>user
Write a function to reverse a string<|im_end|>
<|im_start|>assistant
Here's a simple solution:<|im_end|>
<|im_start|>user
Now make it recursive<|im_end|>
<|im_start|>assistant
```

### Ручное построение промпта (для полного контроля)

```python
def build_qwen_prompt(messages, add_assistant_prefix=True):
    """
    messages: список dict с ключами 'role' и 'content'
    roles: system, user, assistant
    """
    formatted = ""
    for msg in messages:
        formatted += f"<|im_start|>{msg['role']}\n{msg['content']}<|im_end|>\n"
    
    if add_assistant_prefix:
        formatted += "<|im_start|>assistant\n"
    
    return formatted

# Использование
msgs = [
    {"role": "system", "content": "You are helpful."},
    {"role": "user", "content": "What is 2+2?"}
]
prompt = build_qwen_prompt(msgs, add_assistant_prefix=True)
```

### Токенизация с паддингом (batch inference)

```python
from transformers import AutoTokenizer
import torch

tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen2.5-3B-Instruct")

# Несколько промптов разной длины
prompts = [
    "Hello, how are you?",
    "What is the capital of France?",
    "Explain quantum computing in simple terms"  # Длинный
]

# Токенизация с паддингом
encoded = tokenizer(
    prompts,
    padding=True,               # Добавить паддинг до макс. длины
    truncation=True,            # Обрезать слишком длинные
    max_length=128,             # Максимальная длина
    return_tensors="pt"         # Вернуть PyTorch тензоры
)

print(f"Input IDs shape: {encoded['input_ids'].shape}")  # (3, 128)
print(f"Attention mask shape: {encoded['attention_mask'].shape}")

# Проверка паддинг-токена
print(f"Pad token ID: {tokenizer.pad_token_id}")  # 151643
```

### Важные настройки токенизатора

```python
# Для работы с поколением текста
tokenizer = AutoTokenizer.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    
    # Настройки по умолчанию
    padding_side="left",        # Для генерации pad слева!
    truncation_side="right",    # Обрезать справа
    model_max_length=32768,     # Максимальный контекст модели
    use_fast=True               # Использовать быстрый токенизатор
)

# Причина padding_side="left":
# При генерации токенов модель видит pad-токены слева от реального промпта
# и корректно их игнорирует. Если pad справа — генерация ломается.
```

### Полный пример: от промпта до генерации

```python
from transformers import AutoTokenizer, AutoModelForCausalLM, BitsAndBytesConfig
import torch

# 1. Загрузка
tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen2.5-3B-Instruct")
tokenizer.padding_side = "left"

model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    quantization_config=BitsAndBytesConfig(load_in_4bit=True),
    device_map="auto"
)

# 2. Построение чата
messages = [
    {"role": "system", "content": "You are a helpful math tutor."},
    {"role": "user", "content": "Solve for x: 2x + 5 = 13"}
]

# 3. Применение шаблона и токенизация
prompt = tokenizer.apply_chat_template(
    messages, 
    tokenize=False, 
    add_generation_prompt=True
)

inputs = tokenizer(prompt, return_tensors="pt").to(model.device)

# 4. Генерация
with torch.no_grad():
    outputs = model.generate(
        **inputs,
        max_new_tokens=256,
        temperature=0.7,
        do_sample=True,
        pad_token_id=tokenizer.pad_token_id,
        eos_token_id=tokenizer.eos_token_id
    )

# 5. Декодирование (обрезаем входную часть)
response = tokenizer.decode(outputs[0][inputs.input_ids.shape[1]:], skip_special_tokens=True)
print(response)
```

### Проблемы и решения

**Проблема:** `apply_chat_template()` возвращает строку с лишними символами  
**Решение:** Проверьте актуальность transformers: `pip install --upgrade transformers`

**Проблема:** `pad_token_id` не установлен  
**Решение:** 
```python
if tokenizer.pad_token is None:
    tokenizer.pad_token = tokenizer.eos_token  # <|endoftext|>
    # или создайте новый:
    # tokenizer.add_special_tokens({'pad_token': '[PAD]'})
```

**Проблема:** Китайские символы декодируются в `[UNK]`  
**Решение:** Убедитесь, что используется `use_fast=False` или обновите transformers до ≥4.37.0

**Проблема:** Шаблон чата не соответствует документации Qwen  
**Решение:** Шаблон хранится в модели. Проверьте его:
```python
print(tokenizer.chat_template)
# Вывод: {% for message in messages %}{{'<|im_start|>' + message['role'] + '\n' + message['content'] + '<|im_end|>' + '\n'}}{% endfor %}{% if add_generation_prompt %}{{ '<|im_start|>assistant\n' }}{% endif %}
```

### Сравнение: ручной шаблон vs apply_chat_template

| Подход | Преимущества | Недостатки |
|--------|-------------|-------------|
| Ручная сборка | Полный контроль, нет зависимости от версии transformers | Можно ошибиться в спецтокенах |
| `apply_chat_template()` | Всегда актуальный формат, поддержка разных моделей | Требует transformers ≥4.36 |

**Рекомендация:** Используйте `apply_chat_template()`. Именно его применяют авторы Qwen в официальных примерах.

Следующая глава покажет, как управлять генерацией: температура, top_p, repetition_penalty и другие параметры.

Скидывайте главу 9.



## 9. Применение шаблонов чата через `apply_chat_template`

### Почему `apply_chat_template` — стандарт де-факто

Метод `apply_chat_template()` — единственный официальный способ форматирования диалогов, поддерживаемый авторами модели. Он гарантирует, что промпт будет соответствовать формату, на котором модель обучалась. Ручная сборка строк приводит к незаметным ошибкам: лишний пробел, отсутствие `\n` или неправильный спецтокен снижают качество генерации на 10-30%.

### Базовое использование

```python
from transformers import AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen2.5-3B-Instruct")

messages = [
    {"role": "system", "content": "You are a coding assistant"},
    {"role": "user", "content": "Write a Python function to check if a number is prime"}
]

# Простейший вызов
prompt = tokenizer.apply_chat_template(
    messages,
    tokenize=False,              # Вернуть строку, а не токены
    add_generation_prompt=True   # Добавить "assistant\n" в конец
)

print(prompt)
```

**Вывод:**
```
<|im_start|>system
You are a coding assistant<|im_end|>
<|im_start|>user
Write a Python function to check if a number is prime<|im_end|>
<|im_start|>assistant
```

### Токенизация сразу после шаблона

```python
# Получение тензоров за один проход
inputs = tokenizer.apply_chat_template(
    messages,
    add_generation_prompt=True,
    return_tensors="pt"          # Вернуть PyTorch тензор
)

# inputs — это словарь с ключами 'input_ids' и 'attention_mask'
print(inputs['input_ids'].shape)  # (1, количество_токенов)

# Отправка в модель
outputs = model.generate(**inputs, max_new_tokens=256)
```

### Работа с многооборотными диалогами

```python
# Полная история разговора
conversation = [
    {"role": "system", "content": "You are a Python tutor. Keep answers short."},
    {"role": "user", "content": "What is a list comprehension?"},
    {"role": "assistant", "content": "It's a concise way to create lists: [expr for item in iterable]"},
    {"role": "user", "content": "Give me an example with numbers 1 to 10"}
]

# Применяем шаблон ко всей истории
full_prompt = tokenizer.apply_chat_template(
    conversation,
    tokenize=False,
    add_generation_prompt=True
)

# Модель видит полный контекст
```

### Управление системным сообщением

```python
# Без системного сообщения (только user/assistant)
simple_messages = [
    {"role": "user", "content": "What is 2+2?"}
]

prompt = tokenizer.apply_chat_template(
    simple_messages,
    tokenize=False,
    add_generation_prompt=True
)
# Вывод:
# <|im_start|>user
# What is 2+2?<|im_end|>
# <|im_start|>assistant

# С пустым системным сообщением (не рекомендуется)
empty_system = [
    {"role": "system", "content": ""},
    {"role": "user", "content": "Hello"}
]
```

### Параметры метода

Полная сигнатура для Qwen2.5:

```python
tokenizer.apply_chat_template(
    conversation,                # список сообщений
    tokenize=True,               # False → строка, True → list/tensor
    return_tensors=None,         # "pt", "tf" или None
    return_dict=False,           # Вернуть dict с input_ids, attention_mask
    padding=False,               # Паддинг внутри батча
    truncation=False,            # Обрезать до max_length
    max_length=None,             # Максимальная длина
    add_generation_prompt=True,  # Добавить суффикс "assistant\n"
    **kwargs                     # Доп. параметры токенизатора
)
```

### Полный пример с генерацией (4-битная версия)

```python
from transformers import AutoTokenizer, AutoModelForCausalLM, BitsAndBytesConfig
import torch

# 1. Загрузка
tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen2.5-3B-Instruct")
tokenizer.padding_side = "left"

quant_config = BitsAndBytesConfig(load_in_4bit=True)
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    quantization_config=quant_config,
    device_map="auto"
)

# 2. Диалог
messages = [
    {"role": "system", "content": "You are a helpful assistant that speaks like a pirate."},
    {"role": "user", "content": "What time is it?"}
]

# 3. Шаблон → токены → генерация
inputs = tokenizer.apply_chat_template(
    messages,
    add_generation_prompt=True,
    return_tensors="pt"
).to(model.device)

outputs = model.generate(
    **inputs,
    max_new_tokens=128,
    temperature=0.7,
    do_sample=True
)

# 4. Декодирование ответа (обрезаем вход)
response = tokenizer.decode(
    outputs[0][inputs.shape[1]:], 
    skip_special_tokens=True
)
print(response)  # "Arr, me matey, the time be..."
```

### Батчинг нескольких диалогов

```python
# Несколько независимых разговоров
conversations = [
    [{"role": "user", "content": "Hello"}],
    [{"role": "user", "content": "What is AI?"}],
    [{"role": "system", "content": "Be brief."}, {"role": "user", "content": "Explain gravity"}]
]

# Применяем шаблон к каждому
prompts = [
    tokenizer.apply_chat_template(conv, tokenize=False, add_generation_prompt=True)
    for conv in conversations
]

# Токенизация с паддингом (разные длины)
inputs = tokenizer(
    prompts,
    padding=True,
    truncation=True,
    max_length=512,
    return_tensors="pt"
).to(model.device)

# Генерация для всего батча
outputs = model.generate(
    **inputs,
    max_new_tokens=128,
    pad_token_id=tokenizer.pad_token_id
)
```

### Типичные ошибки и их исправление

| Ошибка | Причина | Решение |
|--------|---------|---------|
| `KeyError: 'role'` | В сообщении нет поля `role` | Добавьте `{"role": "user", "content": ...}` |
| `add_generation_prompt` не работает | transformers < 4.36 | `pip install --upgrade transformers` |
| Лишние `<|im_end|>` после ответа | Вы добавили `add_generation_prompt=False` | Должно быть `True` для генерации |
| Модель повторяет промпт в ответе | Не обрезаете вход | `outputs[0][inputs.shape[1]:]` |
| Медленная работа | `tokenize=False` + отдельная токенизация | Используйте `return_tensors="pt"` |

### Просмотр реального шаблона

```python
# Шаблон хранится в токенизаторе
print(tokenizer.chat_template)

# Вывод (для Qwen2.5):
# {% for message in messages %}{{'<|im_start|>' + message['role'] + '\n' + 
# message['content'] + '<|im_end|>' + '\n'}}{% endfor %}
# {% if add_generation_prompt %}{{ '<|im_start|>assistant\n' }}{% endif %}
```

Это Jinja2-шаблон. Вы можете изменить его, но **не делайте этого** — модель обучена именно на таком формате.

### Когда нельзя использовать `apply_chat_template`

Единственный случай — если вы работаете с GGUF через `llama-cpp-python`. Там метод `create_chat_completion()` принимает список сообщений напрямую:

```python
# Для llama-cpp-python не нужен apply_chat_template
from llama_cpp import Llama

llm = Llama(model_path="./model.gguf")
response = llm.create_chat_completion(
    messages=[
        {"role": "user", "content": "Hello"}
    ]
)  # Библиотека сама применяет шаблон
```

**Для всех остальных случаев (transformers, vLLM, TGI) используйте `apply_chat_template`.**

### Чек-лист правильного использования

- [ ] Установлен transformers ≥ 4.37.0
- [ ] Токенизатор загружен для инструкции: `AutoTokenizer.from_pretrained("Qwen/Qwen2.5-3B-Instruct")`
- [ ] Ключи в сообщениях: `role` и `content`
- [ ] `add_generation_prompt=True` перед генерацией
- [ ] Для батчей: `padding=True` и `return_tensors="pt"`
- [ ] `pad_token_id` установлен (обычно равен eos_token_id)

Следующая глава покажет, как работать с параметрами генерации: temperature, top_p, repetition_penalty и другими.

Скидывайте главу 10.



## 10. Базовая генерация текста через `model.generate()`

### Основы метода `generate()`

Метод `generate()` — основной интерфейс для инференса в transformers. Он принимает токенизированный вход и возвращает последовательность выходных токенов.

```python
from transformers import AutoTokenizer, AutoModelForCausalLM
import torch

tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen2.5-3B-Instruct")
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    torch_dtype=torch.float16,
    device_map="auto"
)

# Подготовка промпта
messages = [{"role": "user", "content": "Explain what is a Python generator"}]
prompt = tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
inputs = tokenizer(prompt, return_tensors="pt").to(model.device)

# Базовая генерация
outputs = model.generate(**inputs, max_new_tokens=100)
response = tokenizer.decode(outputs[0][inputs.input_ids.shape[1]:], skip_special_tokens=True)
print(response)
```

### Ключевые параметры генерации

```python
outputs = model.generate(
    **inputs,
    
    # Длина
    max_new_tokens=256,           # Максимум новых токенов (рекомендуется)
    # max_length=512,             # Максимум включая вход (альтернатива)
    min_new_tokens=10,            # Минимум новых токенов
    
    # Семплирование (включено через do_sample=True)
    do_sample=True,               # Включить случайность
    temperature=0.7,              # Чем выше, тем разнообразнее (0.5-1.5)
    top_p=0.9,                    # Nucleus sampling: топ токенов с суммой вероятностей p
    top_k=50,                     # Только топ-K токенов
    
    # Повторы
    repetition_penalty=1.1,       # Штраф за повторения (>1.0)
    no_repeat_ngram_size=3,       # Запретить повтор n-грамм
    
    # Остановка
    eos_token_id=tokenizer.eos_token_id,    # 151643
    pad_token_id=tokenizer.pad_token_id,    # 151643
    stop_strings=["<|im_end|>"],            # Стоп-строки (требует transformers≥4.44)
    
    # Прочее
    num_beams=1,                  # Beam search (>1 отключает do_sample)
    early_stopping=True,          # Остановиться при достижении eos
)
```

### Режимы генерации: семплирование vs жадный поиск

**Жадный поиск (детерминированный):**
```python
outputs = model.generate(**inputs, max_new_tokens=100, do_sample=False)
# Всегда выбирает токен с максимальной вероятностью
```

**Семплирование (стохастический):**
```python
outputs = model.generate(**inputs, max_new_tokens=100, do_sample=True, temperature=0.8)
# Выбирает токены случайно, но с весами из вероятностей
```

**Beam search (альтернатива семплированию):**
```python
outputs = model.generate(**inputs, max_new_tokens=100, num_beams=3, do_sample=False)
# Держит 3 лучшие последовательности, выбирает лучшую в конце
```

### Настройка температуры

```python
for temp in [0.3, 0.7, 1.2]:
    outputs = model.generate(
        **inputs, 
        max_new_tokens=50, 
        do_sample=True, 
        temperature=temp
    )
    response = tokenizer.decode(outputs[0][inputs.input_ids.shape[1]:], skip_special_tokens=True)
    print(f"Temperature {temp}: {response[:100]}...\n")
```

| Температура | Эффект |
|-------------|--------|
| 0.1 - 0.4 | Почти детерминированно, для фактов и кода |
| 0.5 - 0.8 | Баланс, для диалогов |
| 0.9 - 1.5 | Творческие ответы, может галлюцинировать |
| > 1.5 | Слишком случайно, часто бессвязно |

### Управление повторами через repetition_penalty

```python
# Без штрафа (повторы возможны)
outputs = model.generate(**inputs, max_new_tokens=100, repetition_penalty=1.0)

# Со штрафом (рекомендуется для Qwen2.5)
outputs = model.generate(**inputs, max_new_tokens=100, repetition_penalty=1.1)

# Сильный штраф (для коротких ответов, может нарушить грамматику)
outputs = model.generate(**inputs, max_new_tokens=100, repetition_penalty=1.5)
```

### Top-p (nucleus sampling)

```python
# top_p=1.0 — все токены (эквивалентно temperature только)
# top_p=0.9 — берутся токены, чья кумулятивная вероятность ≤ 0.9

outputs = model.generate(
    **inputs,
    do_sample=True,
    temperature=0.8,
    top_p=0.95,   # Мягкое ограничение
)
```

**Рекомендация для Qwen2.5-3B:** `top_p=0.9` и `temperature=0.7` как отправная точка.

### Полный пример с обработкой ошибок

```python
def generate_response(messages, max_new_tokens=256, temperature=0.7):
    try:
        # Применяем шаблон чата
        prompt = tokenizer.apply_chat_template(
            messages, 
            tokenize=False, 
            add_generation_prompt=True
        )
        
        # Токенизация
        inputs = tokenizer(prompt, return_tensors="pt").to(model.device)
        
        # Генерация
        with torch.no_grad():  # Отключаем градиенты (экономит память)
            outputs = model.generate(
                **inputs,
                max_new_tokens=max_new_tokens,
                temperature=temperature,
                do_sample=True,
                top_p=0.9,
                repetition_penalty=1.1,
                pad_token_id=tokenizer.pad_token_id,
                eos_token_id=tokenizer.eos_token_id
            )
        
        # Декодирование
        response = tokenizer.decode(
            outputs[0][inputs.input_ids.shape[1]:], 
            skip_special_tokens=True
        )
        
        return response
    
    except Exception as e:
        return f"Ошибка генерации: {e}"

# Использование
response = generate_response([
    {"role": "system", "content": "You are a Python expert"},
    {"role": "user", "content": "Write a function to calculate factorial recursively"}
])
print(response)
```

### Сравнение параметров для разных сценариев

| Сценарий | do_sample | temperature | top_p | repetition_penalty |
|----------|-----------|-------------|-------|---------------------|
| **Фактический вопрос** | False | N/A | N/A | 1.05 |
| **Кодогенерация** | False | N/A | N/A | 1.05 |
| **Креативный диалог** | True | 0.8-1.0 | 0.9 | 1.1 |
| **Резюмирование** | False | N/A | N/A | 1.05 |
| **Ролевые игры** | True | 0.7 | 0.95 | 1.1 |
| **Трансляция языков** | True | 0.3-0.5 | 0.9 | 1.1 |

### Важные тонкости

**1. Установка pad_token_id**
```python
# Для Qwen2.5 используйте eos_token как pad
if tokenizer.pad_token is None:
    tokenizer.pad_token = tokenizer.eos_token  # <|endoftext|>
```

**2. Отключение градиентов**
```python
# Всегда используйте для инференса
with torch.no_grad():
    outputs = model.generate(**inputs)
```

**3. Очистка кэша (при многократной генерации)**
```python
torch.cuda.empty_cache() if torch.cuda.is_available() else None
```

**4. Подавление нежелательных токенов**
```python
from transformers import LogitsProcessorList, NoBadWordsLogitsProcessor

# Запретить определённые слова
bad_words = ["плохое_слово", "оскорбление"]
bad_ids = [tokenizer.encode(w, add_special_tokens=False) for w in bad_words]
processor = NoBadWordsLogitsProcessor(bad_ids, eos_token_id=tokenizer.eos_token_id)

outputs = model.generate(**inputs, logits_processor=LogitsProcessorList([processor]))
```

### Производительность: что влияет на скорость

| Параметр | Влияние на скорость |
|----------|---------------------|
| `do_sample=False` | На 20-30% быстрее |
| `num_beams > 1` | Линейное замедление (2 beams = 2x) |
| `max_new_tokens` | Прямо пропорционально |
| `temperature` | Не влияет |
| `top_p` | Не влияет |

**Оптимальная конфигурация для скорости:**
```python
outputs = model.generate(
    **inputs,
    max_new_tokens=128,
    do_sample=False,     # Жадный поиск
    repetition_penalty=1.05
)
```

Следующая глава покажет, как управлять потоковой генерацией (токен за токеном) для создания отзывчивых интерфейсов.

Скидывайте главу 11.



## 11. Управление параметрами генерации (max_new_tokens, temperature, top_p)

### max_new_tokens: ограничение длины ответа

`max_new_tokens` — самый важный параметр. Он определяет, сколько **новых** токенов сгенерирует модель после промпта.

```python
# max_new_tokens vs max_length
outputs = model.generate(
    **inputs,
    max_new_tokens=200,    # 200 новых токенов (рекомендуется)
    # max_length=250,      # 250 токенов включая вход (НЕ рекомендуется)
)
```

**Почему max_new_tokens лучше max_length:**

| Ситуация | max_new_tokens=100 | max_length=200 |
|----------|-------------------|----------------|
| Промпт 50 токенов | ✅ 100 новых | ✅ 150 новых |
| Промпт 180 токенов | ✅ 100 новых | ❌ только 20 новых |
| Промпт 300 токенов | ✅ 100 новых | ❌ ошибка (меньше входа) |

**Расчёт потребления памяти под KV-кэш:**
```
KV-кэш ~ (max_new_tokens + промпт) * скрытая_размерность * слои * 2
Для Qwen2.5-3B с 32K контекста:
- 1024 новых токена: ~600 MB
- 8192 новых токена: ~4.8 GB
```

### temperature: управление случайностью

Temperature масштабирует логиты (предсказания) перед softmax. Более высокая температура = более равномерное распределение.

```python
def softmax_scale(logits, temperature):
    return np.exp(logits / temperature) / np.sum(np.exp(logits / temperature))
```

**Практические значения для Qwen2.5-3B:**

```python
# Температура 0 = детерминированно (жадный поиск)
outputs = model.generate(**inputs, do_sample=False)  # не использует temperature

# Низкая температура (0.1-0.4) — фактические ответы
outputs = model.generate(**inputs, do_sample=True, temperature=0.2)

# Средняя температура (0.5-0.8) — диалоги
outputs = model.generate(**inputs, do_sample=True, temperature=0.7)

# Высокая температура (0.9-1.5) — творчество
outputs = model.generate(**inputs, do_sample=True, temperature=1.2)
```

**Визуализация влияния температуры:**

```python
import torch.nn.functional as F
import torch

logits = torch.tensor([2.0, 1.0, 0.1, -1.0])  # Пример логитов

for temp in [0.5, 1.0, 2.0]:
    probs = F.softmax(logits / temp, dim=-1)
    print(f"temp={temp}: {probs.numpy().round(3)}")

# Вывод:
# temp=0.5: [0.788, 0.290, 0.096, 0.013]  # очень неравномерно
# temp=1.0: [0.659, 0.242, 0.089, 0.010]  # нормально
# temp=2.0: [0.514, 0.259, 0.152, 0.075]  # почти равномерно
```

### top_p: nucleus sampling

Top_p выбирает наименьший набор токенов, чья кумулятивная вероятность превышает p. Динамически адаптируется к распределению.

```python
outputs = model.generate(
    **inputs,
    do_sample=True,
    temperature=0.7,
    top_p=0.9,    # Классическое значение
)

# Эквивалентные настройки:
# top_p=1.0   → все токены
# top_p=0.95  → мягкое ограничение
# top_p=0.5   → жёсткое ограничение
```

**Когда top_p выше 0.9:** Модель не может выбрать, сомневается между несколькими вариантами.

**Когда top_p ниже 0.8:** Модель уверена в выборе, но может повторяться.

### Комбинации temperature и top_p для конкретных задач

```python
# 1. Кодогенерация (детерминированно)
config = {"do_sample": False, "max_new_tokens": 512}

# 2. Математические задачи (низкая температура)
config = {"do_sample": True, "temperature": 0.2, "top_p": 0.9}

# 3. Диалоговый ассистент (стандарт)
config = {"do_sample": True, "temperature": 0.7, "top_p": 0.9}

# 4. Креативное письмо (высокая температура)
config = {"do_sample": True, "temperature": 1.1, "top_p": 0.95}

# 5. Заполнение пропусков (широкий поиск)
config = {"do_sample": True, "temperature": 0.9, "top_p": 1.0}
```

### Полная функция с валидацией

```python
def generate_with_params(messages, max_new_tokens=256, temperature=0.7, top_p=0.9, do_sample=True):
    """
    Универсальная обёртка для генерации с валидацией параметров
    """
    # Валидация max_new_tokens
    if max_new_tokens < 1:
        max_new_tokens = 1
    if max_new_tokens > 8192:
        print(f"Предупреждение: {max_new_tokens} > 8192, ограничено до 8192")
        max_new_tokens = 8192
    
    # Валидация temperature
    if temperature < 0.0:
        temperature = 0.0
    if temperature > 2.0:
        print(f"Предупреждение: temperature={temperature} может дать плохие результаты")
    
    # Валидация top_p
    if top_p < 0.01:
        top_p = 0.01
    if top_p > 1.0:
        top_p = 1.0
    
    # Принудительный жадный поиск при low temperature с do_sample
    if temperature < 0.1 and do_sample:
        print("Предупреждение: temperature < 0.1 с do_sample=True неэффективно, используйте do_sample=False")
        do_sample = False
    
    # Применение шаблона и генерация
    prompt = tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
    inputs = tokenizer(prompt, return_tensors="pt").to(model.device)
    
    with torch.no_grad():
        outputs = model.generate(
            **inputs,
            max_new_tokens=max_new_tokens,
            temperature=temperature,
            top_p=top_p,
            do_sample=do_sample,
            pad_token_id=tokenizer.pad_token_id,
            eos_token_id=tokenizer.eos_token_id
        )
    
    response = tokenizer.decode(outputs[0][inputs.input_ids.shape[1]:], skip_special_tokens=True)
    return response

# Тестирование
result = generate_with_params(
    [{"role": "user", "content": "Tell me a joke"}],
    max_new_tokens=50,
    temperature=0.9,
    top_p=0.95
)
```

### Экспериментальный поиск оптимальных параметров

```python
# Тестовый промпт
test_prompt = [{"role": "user", "content": "Explain quantum computing in one sentence"}]

# Сетка параметров
param_grid = [
    {"temp": 0.3, "top_p": 0.9, "name": "Факты"},
    {"temp": 0.7, "top_p": 0.9, "name": "Стандарт"},
    {"temp": 1.0, "top_p": 0.95, "name": "Творчество"},
]

for params in param_grid:
    response = generate_with_params(
        test_prompt, 
        max_new_tokens=100,
        temperature=params["temp"],
        top_p=params["top_p"]
    )
    print(f"\n--- {params['name']} (temp={params['temp']}) ---")
    print(response)
```

### top_k: дополнительный фильтр

Top_k оставляет только K токенов с наибольшими вероятностями, остальные обнуляет.

```python
outputs = model.generate(
    **inputs,
    do_sample=True,
    temperature=0.8,
    top_k=50,      # Только 50 лучших токенов
    top_p=0.95,    # Применяется после top_k
)

# Комбинация top_k + top_p
# Сначала сокращаем до top_k (быстро), затем применяем top_p (точно)
```

**Рекомендация для Qwen2.5-3B:** Используйте только top_p. top_k даёт минимальный прирост за счёт усложнения.

### Сравнение параметров на одном примере

```python
prompt = [{"role": "user", "content": "Write a haiku about Python"}]

configs = [
    {"name": "Facts", "do_sample": False},
    {"name": "Creative 1", "do_sample": True, "temp": 0.8, "top_p": 0.9},
    {"name": "Creative 2", "do_sample": True, "temp": 1.2, "top_p": 0.95},
]

for config in configs:
    print(f"\n{config['name']}:")
    response = generate_with_params(prompt, **{k:v for k,v in config.items() if k != 'name'})
    print(response)
```

### Практические рекомендации

| Задача | do_sample | temperature | top_p | max_new_tokens |
|--------|-----------|-------------|-------|----------------|
| Классификация | False | N/A | N/A | 10-50 |
| Извлечение фактов | False | N/A | N/A | 50-150 |
| Code completion | False | N/A | N/A | 200-500 |
| Вопрос-ответ | True | 0.3-0.5 | 0.9 | 100-200 |
| Диалог | True | 0.7 | 0.9 | 150-300 |
| Резюмирование | True | 0.4-0.6 | 0.9 | 200-400 |
| Перефразирование | True | 0.8-1.0 | 0.95 | 100-200 |
| Мозговой штурм | True | 1.1-1.3 | 1.0 | 300-500 |

### Частые ошибки

**Ошибка 1:** `do_sample=False` + `temperature=0.7` — temperature игнорируется.

**Ошибка 2:** `max_new_tokens=4096` с 4-bit на GPU 4GB — out of memory.

**Ошибка 3:** `top_p=0.9` без `do_sample=True` — top_p игнорируется.

**Ошибка 4:** Слишком высокая температура (1.5+) — бессвязный текст.

### Динамическое изменение температуры во время генерации

```python
from transformers import LogitsProcessor

class DynamicTemperatureLogitsProcessor(LogitsProcessor):
    def __init__(self, start_temp=0.7, end_temp=1.0, decay_steps=100):
        self.start_temp = start_temp
        self.end_temp = end_temp
        self.decay_steps = decay_steps
    
    def __call__(self, input_ids, scores):
        step = min(input_ids.shape[-1], self.decay_steps)
        temp = self.start_temp + (self.end_temp - self.start_temp) * (step / self.decay_steps)
        return scores / temp

processor = DynamicTemperatureLogitsProcessor(start_temp=0.5, end_temp=1.0)
outputs = model.generate(**inputs, logits_processor=[processor], do_sample=True)
```

Следующая глава покажет, как настраивать паддинг и маски внимания для батчевой обработки нескольких промптов.

Скидывайте главу 12.



## 12. Потоковый вывод для интерактивных приложений

### Зачем нужен потоковый вывод

Потоковая генерация (token-by-token) превращает ожидание в 10-20 секунд в плавный вывод текста по мере генерации. Для интерактивных приложений (чаты, IDE-плагины, голосовые ассистенты) это обязательное требование.

### Способ 1: TextStreamer (самый простой)

```python
from transformers import AutoTokenizer, AutoModelForCausalLM, TextStreamer
import torch

tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen2.5-3B-Instruct")
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    torch_dtype=torch.float16,
    device_map="auto"
)

messages = [{"role": "user", "content": "Write a poem about programming"}]
prompt = tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
inputs = tokenizer(prompt, return_tensors="pt").to(model.device)

# Создаём стример с выводом в консоль
streamer = TextStreamer(tokenizer, skip_prompt=True, skip_special_tokens=True)

# Генерация с потоковым выводом
_ = model.generate(
    **inputs,
    max_new_tokens=256,
    temperature=0.7,
    do_sample=True,
    streamer=streamer
)
# Текст появляется токен за токеном прямо в консоли
```

### Способ 2: TextIteratorStreamer (для веб-приложений)

```python
from transformers import TextIteratorStreamer
from threading import Thread

# Создаём стример, который возвращает итератор
streamer = TextIteratorStreamer(tokenizer, skip_prompt=True, skip_special_tokens=True)

# Запускаем генерацию в отдельном потоке
generation_kwargs = dict(
    **inputs,
    max_new_tokens=256,
    temperature=0.7,
    do_sample=True,
    streamer=streamer
)
thread = Thread(target=model.generate, kwargs=generation_kwargs)
thread.start()

# Итерируем по токенам
for token_text in streamer:
    print(token_text, end="", flush=True)  # или отправляем в веб-сокет
```

### Способ 3: Кастомный стример для API

```python
from transformers import streamers
from typing import Optional

class CustomStreamer(streamers.BaseStreamer):
    def __init__(self, tokenizer, callback=None):
        self.tokenizer = tokenizer
        self.callback = callback  # функция, вызываемая при каждом токене
        self.generated_tokens = []
    
    def put(self, value):
        # value — тензор с новыми токенами
        token_id = value[0][-1].item()
        token_text = self.tokenizer.decode(token_id, skip_special_tokens=True)
        
        if token_text.strip():  # Игнорируем пробелы
            self.generated_tokens.append(token_text)
            if self.callback:
                self.callback(token_text)
    
    def end(self):
        # Генерация завершена
        if self.callback:
            self.callback(None)  # Сигнал окончания

# Использование
def on_token(token):
    if token is None:
        print("\n[Генерация завершена]")
    else:
        print(f"Received: {token}")

streamer = CustomStreamer(tokenizer, callback=on_token)
```

### Способ 4: Потоковая генерация с сохранением в буфер

```python
from queue import Queue
from threading import Thread

class BufferedStreamer:
    def __init__(self, tokenizer, buffer_size=10):
        self.tokenizer = tokenizer
        self.buffer = Queue()
        self.buffer_size = buffer_size
        self.generated = []
    
    def __call__(self, token_id):
        self.generated.append(token_id)
        
        # Отдаём токен только если накопилось buffer_size или это последний
        if len(self.generated) >= self.buffer_size:
            text = self.tokenizer.decode(self.generated, skip_special_tokens=True)
            self.buffer.put(text)
            self.generated = []
    
    def finalize(self):
        if self.generated:
            text = self.tokenizer.decode(self.generated, skip_special_tokens=True)
            self.buffer.put(text)
        self.buffer.put(None)  # Сигнал окончания
    
    def stream(self):
        while True:
            chunk = self.buffer.get()
            if chunk is None:
                break
            yield chunk

# Использование
buffered = BufferedStreamer(tokenizer, buffer_size=5)
streamer = TextIteratorStreamer(tokenizer, skip_prompt=True)

thread = Thread(target=model.generate, kwargs={**generation_kwargs, "streamer": streamer})
thread.start()

full_response = ""
for token in streamer:
    full_response += token
    # Обновляем UI каждые 50 мс
```

### Полный пример: чат-бот с потоковым выводом

```python
import time
from transformers import AutoTokenizer, AutoModelForCausalLM, BitsAndBytesConfig, TextIteratorStreamer
from threading import Thread

class StreamingQwenChatbot:
    def __init__(self):
        self.tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen2.5-3B-Instruct")
        quant_config = BitsAndBytesConfig(load_in_4bit=True)
        self.model = AutoModelForCausalLM.from_pretrained(
            "Qwen/Qwen2.5-3B-Instruct",
            quantization_config=quant_config,
            device_map="auto"
        )
        self.history = []
    
    def chat(self, user_input, on_token_callback=None):
        # Добавляем сообщение пользователя
        self.history.append({"role": "user", "content": user_input})
        
        # Применяем шаблон
        prompt = self.tokenizer.apply_chat_template(
            self.history, 
            tokenize=False, 
            add_generation_prompt=True
        )
        inputs = self.tokenizer(prompt, return_tensors="pt").to(self.model.device)
        
        # Создаём стример
        streamer = TextIteratorStreamer(
            self.tokenizer, 
            skip_prompt=True, 
            skip_special_tokens=True
        )
        
        # Запускаем генерацию
        generation_kwargs = dict(
            **inputs,
            max_new_tokens=512,
            temperature=0.7,
            do_sample=True,
            top_p=0.9,
            repetition_penalty=1.1,
            streamer=streamer
        )
        thread = Thread(target=self.model.generate, kwargs=generation_kwargs)
        thread.start()
        
        # Собираем ответ и вызываем колбэк
        response = ""
        for token in streamer:
            response += token
            if on_token_callback:
                on_token_callback(token)
        
        # Сохраняем ответ в историю
        self.history.append({"role": "assistant", "content": response})
        return response

# Использование
bot = StreamingQwenChatbot()

def print_token(token):
    print(token, end="", flush=True)

print("Chatbot ready! Type 'quit' to exit")
while True:
    user = input("\n\nYou: ")
    if user.lower() == 'quit':
        break
    print("Bot: ", end="")
    bot.chat(user, on_token_callback=print_token)
```

### Интеграция с Gradio (веб-интерфейс)

```python
import gradio as gr
from threading import Thread

def stream_response(message, history):
    history.append({"role": "user", "content": message})
    
    prompt = tokenizer.apply_chat_template(history, tokenize=False, add_generation_prompt=True)
    inputs = tokenizer(prompt, return_tensors="pt").to(model.device)
    
    streamer = TextIteratorStreamer(tokenizer, skip_prompt=True, skip_special_tokens=True)
    
    thread = Thread(
        target=model.generate,
        kwargs={
            **inputs,
            "max_new_tokens": 256,
            "temperature": 0.7,
            "do_sample": True,
            "streamer": streamer
        }
    )
    thread.start()
    
    response = ""
    for token in streamer:
        response += token
        yield response

# Gradio интерфейс
gr.ChatInterface(
    fn=stream_response,
    title="Qwen2.5-3B Chatbot",
    description="Потоковая генерация с локальной моделью"
).launch()
```

### Интеграция с FastAPI (WebSocket)

```python
from fastapi import FastAPI, WebSocket
import asyncio
from threading import Thread

app = FastAPI()

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    
    while True:
        # Получаем сообщение от клиента
        data = await websocket.receive_text()
        
        # Подготавливаем промпт
        messages = [{"role": "user", "content": data}]
        prompt = tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
        inputs = tokenizer(prompt, return_tensors="pt").to(model.device)
        
        # Создаём стример
        streamer = TextIteratorStreamer(tokenizer, skip_prompt=True, skip_special_tokens=True)
        
        # Запускаем генерацию
        thread = Thread(target=model.generate, kwargs={
            **inputs, "max_new_tokens": 256, "temperature": 0.7, "streamer": streamer
        })
        thread.start()
        
        # Отправляем токены клиенту
        for token in streamer:
            await websocket.send_text(token)
        
        await websocket.send_text("[END]")
```

### Оптимизация потоковой генерации

```python
# 1. Уменьшите задержку между токенами (по умолчанию стример не добавляет задержек)
# 2. Используйте меньший batch size для префиллов
inputs = tokenizer(prompt, return_tensors="pt", padding=False)  # без паддинга

# 3. Для long-running генераций добавьте timeout
import signal

class TimeoutError(Exception):
    pass

def timeout_handler(signum, frame):
    raise TimeoutError("Генерация превысила лимит времени")

signal.signal(signal.SIGALRM, timeout_handler)
signal.alarm(30)  # 30 секунд

try:
    for token in streamer:
        process_token(token)
except TimeoutError:
    print("Генерация прервана по таймауту")
finally:
    signal.alarm(0)
```

### Сравнение методов стриминга

| Метод | Простота | Контроль | Для веба | Скорость |
|-------|----------|----------|----------|----------|
| TextStreamer | ⭐⭐⭐⭐⭐ | ⭐⭐ | ❌ | ⭐⭐⭐⭐ |
| TextIteratorStreamer | ⭐⭐⭐⭐ | ⭐⭐⭐ | ✅ | ⭐⭐⭐⭐⭐ |
| Кастомный стример | ⭐⭐ | ⭐⭐⭐⭐⭐ | ✅ | ⭐⭐⭐⭐ |
| Websocket + Thread | ⭐⭐ | ⭐⭐⭐⭐ | ✅ | ⭐⭐⭐⭐ |

### Частые проблемы

**Проблема:** Токены приходят кусками, а не по одному  
**Решение:** TextIteratorStreamer буферизирует для скорости. Используйте кастомный стример для побайтового вывода.

**Проблема:** Зависание при `streamer.put()`  
**Решение:** Всегда запускайте генерацию в отдельном потоке. Основной поток должен только читать из стримера.

**Проблема:** Ошибка `RuntimeError: Cannot call streamer in non-streaming mode`  
**Решение:** Убедитесь, что передали `streamer` в `model.generate()`.

**Проблема:** Специальные токены появляются в выводе  
**Решение:** Установите `skip_special_tokens=True` в стримере.

### Производительность потоковой генерации

```python
# Замер задержки между токенами
import time

class TimingStreamer(TextIteratorStreamer):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.last_time = None
    
    def put(self, value):
        now = time.time()
        if self.last_time:
            print(f"Интервал: {(now - self.last_time)*1000:.1f} мс")
        self.last_time = now
        super().put(value)

# Типичные интервалы на GPU: 20-50 мс
# На CPU: 100-300 мс
```

Следующая глава покажет, как настраивать паддинг и attention mask для эффективной батчевой обработки нескольких запросов.

Скидывайте главу 13.



## 13. Управление распределением на устройства: CPU vs GPU инференс

### Понимание device_map

`device_map` — механизм Hugging Face Accelerate, который автоматически или вручную распределяет слои модели между доступными устройствами (GPU, CPU, диск). Для Qwen2.5-3B с 36 слоями правильное распределение критически влияет на скорость и память.

### Автоматическое распределение (рекомендуется для начала)

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
import torch

# Автоматическое распределение
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    torch_dtype=torch.float16,
    device_map="auto"  # Hugging Face сам решает
)

print(model.hf_device_map)
# Вывод: {'': 0}  — всё на GPU 0
# Или: {'model.embed_tokens': 0, 'model.layers.0-20': 0, 'model.layers.21-35': 'cpu', ...}
```

**Что делает "auto":**
1. Проверяет доступную VRAM на всех GPU
2. Если VRAM хватает — всё на GPU
3. Если не хватает — распределяет слои между GPU и CPU
4. При сильной нехватке — использует диск (очень медленно)

### Явное распределение: когда "auto" недостаточно

```python
# 1. Фиксированное устройство
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    device_map="cuda:0"  # Только GPU 0
)

# 2. Только CPU
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    device_map="cpu"
)

# 3. Ручное распределение по слоям
device_map = {
    'model.embed_tokens': 0,           # Эмбеддинги на GPU 0
    'model.layers.0': 0,
    'model.layers.1': 0,
    # ... слои 2-30 пропущены
    'model.layers.31': 'cpu',          # Последние слои на CPU
    'model.norm': 'cpu',
    'lm_head': 'cpu'
}
```

### Распределение между несколькими GPU

```python
# Два GPU: первые 20 слоёв на GPU 0, остальные на GPU 1
device_map = {
    'model.embed_tokens': 0,
    'model.layers.0': 0,
    # ... 
    'model.layers.19': 0,
    'model.layers.20': 1,
    'model.layers.21': 1,
    # ...
    'model.layers.35': 1,
    'model.norm': 1,
    'lm_head': 1
}

model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    device_map=device_map,
    torch_dtype=torch.float16
)
```

### Утилиты для анализа device_map

```python
def inspect_device_map(model):
    """Показать распределение слоёв по устройствам"""
    if hasattr(model, 'hf_device_map'):
        print("=== Device Map ===")
        for module, device in model.hf_device_map.items():
            print(f"{module:30} -> {device}")
    else:
        print("Модель не использует device_map")
    
    # Показать текущее устройство каждого параметра
    print("\n=== Первые 5 параметров ===")
    for name, param in list(model.named_parameters())[:5]:
        print(f"{name:40} {param.device}")

def estimate_memory_per_device(model):
    """Оценить VRAM/RAM на устройство"""
    from collections import defaultdict
    mem_by_device = defaultdict(int)
    
    for name, param in model.named_parameters():
        device = str(param.device)
        mem_by_device[device] += param.numel() * param.element_size()
    
    for device, mem in mem_by_device.items():
        print(f"{device}: {mem / 1e9:.2f} GB")
```

### Сравнение стратегий распределения

| Стратегия | VRAM GPU | RAM CPU | Скорость | Когда использовать |
|-----------|----------|---------|----------|-------------------|
| `device_map="cuda:0"` (FP16) | 6 GB | 2 GB | 50 т/с | RTX 3060+ |
| `device_map="auto"` (4-bit) | 2 GB | 1.5 GB | 45 т/с | Любая GPU |
| `device_map="cpu"` | 0 | 8 GB | 2 т/с | Без GPU |
| Слои 0-20 на GPU, остальное на CPU | 3 GB | 4 GB | 15 т/с | GPU 4GB |
| `device_map="balanced"` | Переменная | Переменная | Средняя | Несколько GPU |

### Оффлоадинг на диск (крайний случай)

```python
# Для систем с <4 GB RAM
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    device_map="auto",
    offload_folder="./offload",  # Папка для сброса на диск
    offload_state_dict=True       # Сбрасывать state_dict'и
)
# Медленно: 0.1-0.5 токенов/сек
```

### Практические конфигурации

**Сценарий A: Игровой ПК с RTX 3060 (12 GB VRAM)**
```python
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    torch_dtype=torch.float16,
    device_map="cuda:0"  # Всё на GPU
)
# 6 GB VRAM, 50+ т/с
```

**Сценарий B: Ноутбук с GTX 1650 (4 GB VRAM)**
```python
from transformers import BitsAndBytesConfig

quant_config = BitsAndBytesConfig(load_in_4bit=True)
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    quantization_config=quant_config,
    device_map="auto"  # Распределит автоматически
)
# ~2 GB VRAM, остальное на CPU, 20-30 т/с
```

**Сценарий C: MacBook с M1/M2 (общая память)**
```python
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    torch_dtype=torch.float16,
    device_map="mps"  # Metal Performance Shaders
)
# Использует общую память, 10-20 т/с
```

**Сценарий D: Сервер без GPU**
```python
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    torch_dtype=torch.float32,
    device_map="cpu",
    low_cpu_mem_usage=True
)
# 8 GB RAM, 1-3 т/с — используйте GGUF вместо этого
```

### Динамическое переключение между устройствами

```python
class DynamicDeviceModel:
    def __init__(self, model, tokenizer):
        self.model = model
        self.tokenizer = tokenizer
        self.current_device = next(model.parameters()).device
    
    def to_device(self, device):
        """Переместить модель на другое устройство"""
        print(f"Перемещение с {self.current_device} на {device}")
        self.model = self.model.to(device)
        self.current_device = device
    
    def generate_with_fallback(self, prompt, fallback_to_cpu=True):
        try:
            inputs = self.tokenizer(prompt, return_tensors="pt").to(self.current_device)
            return self.model.generate(**inputs, max_new_tokens=100)
        except RuntimeError as e:
            if "out of memory" in str(e) and fallback_to_cpu:
                print("OOM на GPU, переключаюсь на CPU")
                self.to_device("cpu")
                return self.generate_with_fallback(prompt, fallback_to_cpu=False)
            raise e
```

### Мониторинг использования памяти

```python
import psutil
import torch

def monitor_memory():
    """Отслеживание памяти во время инференса"""
    if torch.cuda.is_available():
        print(f"GPU VRAM: {torch.cuda.memory_allocated() / 1e9:.2f} GB")
        print(f"GPU Reserved: {torch.cuda.memory_reserved() / 1e9:.2f} GB")
    
    process = psutil.Process()
    print(f"RAM: {process.memory_info().rss / 1e9:.2f} GB")

# Инференс с мониторингом
monitor_memory()
outputs = model.generate(**inputs, max_new_tokens=256)
monitor_memory()
```

### Очистка памяти после инференса

```python
import gc

def cleanup_memory(model):
    """Освободить память после использования"""
    del model
    gc.collect()
    
    if torch.cuda.is_available():
        torch.cuda.empty_cache()
        torch.cuda.synchronize()
    
    print("Память очищена")

# Использование
model = load_model(...)
outputs = model.generate(...)
cleanup_memory(model)
```

### Частые ошибки и решения

| Ошибка | Причина | Решение |
|--------|---------|---------|
| `CUDA out of memory` | FP16 не влезает | Используйте 4-bit квантизацию |
| `RuntimeError: Expected all tensors to be on the same device` | Входные тензоры на CPU, модель на GPU | `inputs.to(model.device)` |
| `device_map="auto"` игнорируется | `quantization_config` уже задал устройство | Удалите quantization_config или задайте device_map вручную |
| Медленный инференс после `to("cpu")` | Модель в FP32 на CPU | Конвертируйте в GGUF |

### Лучшие практики

1. **Всегда указывайте `device_map="auto"`** для квантизированных моделей
2. **Для чистого GPU** используйте `device_map="cuda:0"` (быстрее, чем "auto")
3. **Перед генерацией** убедитесь, что входные тензоры на том же устройстве: `inputs.to(model.device)`
4. **При ошибках памяти** сначала уменьшите `max_new_tokens`, затем переходите к оффлоадингу
5. **Для CPU** не используйте transformers — берите GGUF через llama-cpp-python

Следующая глава покажет, как использовать квантизацию на лету с BitsAndBytes для максимальной экономии памяти.

Скидывайте главу 14.



## 14. Снижение потребления памяти через torch_dtype и квантизацию

### Понимание типов данных в PyTorch

Для Qwen2.5-3B критически важно выбирать правильный тип данных. Базовые варианты:

| Тип данных | Бит на параметр | Память (3B параметров) | Точность |
|-----------|----------------|----------------------|----------|
| float32 (FP32) | 32 | ~12 GB | Полная |
| float16 (FP16) | 16 | ~6 GB | Полусловная |
| bfloat16 (BF16) | 16 | ~6 GB | Сбалансированная |
| int8 | 8 | ~3 GB | Сниженная |
| int4 | 4 | ~1.5 GB | Сильно сниженная |

**Рекомендация для Qwen2.5-3B:** Используйте `torch_dtype=torch.float16` или `torch.bfloat16` как базовый уровень .

### torch_dtype: что это и как работает

`torch_dtype` определяет тип данных для весов и активаций модели при загрузке:

```python
import torch
from transformers import AutoModelForCausalLM

# FP16 (рекомендуется для GPU)
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    torch_dtype=torch.float16,  # 6 GB VRAM
    device_map="auto"
)

# BF16 (лучше для стабильности, требуется Ampere GPU или новее)
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    torch_dtype=torch.bfloat16,  # 6 GB VRAM
    device_map="auto"
)

# FP32 (избыточно для инференса)
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    torch_dtype=torch.float32,  # 12 GB VRAM
    device_map="auto"
)
```

**Важный нюанс:** При загрузке с `device_map="auto"` или квантизацией параметр `torch_dtype` применяется к **неквантованным слоям** и активациям. Он не переопределяет квантизацию весов .

### Сравнение torch_dtype вариантов

```python
from transformers import AutoModelForCausalLM
import torch

def measure_memory(dtype):
    model = AutoModelForCausalLM.from_pretrained(
        "Qwen/Qwen2.5-3B-Instruct",
        torch_dtype=dtype,
        device_map="auto",
        low_cpu_mem_usage=True
    )
    
    # Расчёт памяти
    mem = model.get_memory_footprint()
    print(f"{dtype}: {mem / 1e9:.1f} GB")
    return model

# Тестирование
for dtype in [torch.float32, torch.float16, torch.bfloat16]:
    measure_memory(dtype)
    torch.cuda.empty_cache()

# Результат:
# float32: 11.8 GB
# float16: 5.9 GB
# bfloat16: 5.9 GB
```

### Квантизация через BitsAndBytes (4-bit и 8-bit)

**8-bit квантизация (INT8) — снижение памяти на ~50%:**

```python
from transformers import AutoModelForCausalLM, BitsAndBytesConfig
import torch

bnb_config = BitsAndBytesConfig(
    load_in_8bit=True,           # 8-bit веса
    llm_int8_threshold=6.0,      # Порог для FP16 (6.0 по умолчанию)
)

model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    quantization_config=bnb_config,
    device_map="auto"
)
# Память: ~3 GB VRAM
```

**4-bit квантизация (NF4) — максимальная экономия:**

```python
bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",           # Normal Float 4 (лучше чем fp4)
    bnb_4bit_compute_dtype=torch.float16, # Вычисления в FP16
    bnb_4bit_use_double_quant=True,      # Двойная квантизация (экономия ещё 0.5 GB)
)

model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    quantization_config=bnb_config,
    device_map="auto"
)
# Память: ~2 GB VRAM
```

**Важно:** При использовании `quantization_config` параметр `torch_dtype` не переопределяет веса, но влияет на внутренние вычисления. Обычно устанавливают `bnb_4bit_compute_dtype=torch.float16` для баланса скорости и точности .

### GPTQ-квантизация (альтернативный подход)

GPTQ (Gradient Post-Training Quantization) — метод, который минимизирует потерю качества при квантизации:

```python
from transformers import AutoModelForCausalLM, GPTQConfig

quantization_config = GPTQConfig(
    bits=4,
    group_size=128,
    desc_act=True,              # Act-order для точности
    dataset="c4"                # Калибровочный датасет
)

model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    quantization_config=quantization_config,
    device_map="auto"
)
```

**Для уже квантизованных GPTQ-моделей:**

```python
# Загрузка готовой 4-bit GPTQ модели
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct-GPTQ-Int4",
    device_map="auto"
)
```

### Сравнение методов квантизации для Qwen2.5-3B

| Метод | Память | Качество (от FP16) | Скорость | Сложность |
|-------|--------|-------------------|----------|-----------|
| FP16 (baseline) | 6 GB | 100% | 50 т/с | Низкая |
| 8-bit (BitsAndBytes) | 3 GB | 98-99% | 45 т/с | Низкая |
| 4-bit NF4 | 2 GB | 96-97% | 40 т/с | Низкая |
| GPTQ 4-bit | 2 GB | 96-97% | 35-40 т/с | Средняя |
| INT8 (веса) | 3 GB | 98% | 45 т/с | Низкая |

**Вывод:** Для Qwen2.5-3B 4-bit квантизация через BitsAndBytes (NF4) даёт лучший баланс память/качество .

### Комбинирование torch_dtype и квантизации

```python
from transformers import AutoModelForCausalLM, BitsAndBytesConfig
import torch

# Правильная комбинация
bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_compute_dtype=torch.float16,  # Вычисления в FP16
    bnb_4bit_quant_type="nf4"
)

model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    quantization_config=bnb_config,
    torch_dtype=torch.float16,   # Для неквантованных компонентов
    device_map="auto"
)
```

**Как это работает:**
1. Веса загружаются в 4-bit
2. При вычислениях веса деквантуются во FP16
3. Активации и неквантованные слои работают в FP16

### Интеграция с Unsloth (оптимизированная 4-bit)

Unsloth предлагает оптимизированную 4-bit квантизацию с лучшей точностью:

```python
from unsloth import FastLanguageModel

model, tokenizer = FastLanguageModel.from_pretrained(
    model_name="unsloth/Qwen2.5-3B-Instruct-bnb-4bit",
    max_seq_length=4096,
    dtype=None,  # Автоопределение
    load_in_4bit=True,
)
# Память: ~2 GB, но на 10-30% быстрее стандартной 4-bit
```

### Полная конфигурация для разных сценариев

**Сценарий 1: Максимальная точность (GPU 8+ GB VRAM)**
```python
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    torch_dtype=torch.float16,
    device_map="cuda:0"
)
```

**Сценарий 2: Баланс (GPU 4-6 GB VRAM)**
```python
bnb_config = BitsAndBytesConfig(load_in_8bit=True)
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    quantization_config=bnb_config,
    device_map="auto"
)
```

**Сценарий 3: Минимум памяти (GPU 2-4 GB VRAM)**
```python
bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_use_double_quant=True
)
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    quantization_config=bnb_config,
    device_map="auto"
)
```

**Сценарий 4: CPU с ограниченной памятью (8-12 GB RAM)**
```python
# Не через transformers — используйте GGUF (глава 6)
```

### Проверка реального использования памяти

```python
def print_memory_usage():
    """Отслеживание памяти во время работы"""
    if torch.cuda.is_available():
        allocated = torch.cuda.memory_allocated() / 1e9
        reserved = torch.cuda.memory_reserved() / 1e9
        print(f"VRAM: {allocated:.2f} GB allocated, {reserved:.2f} GB reserved")

# После загрузки модели
print_memory_usage()

# Во время генерации
outputs = model.generate(**inputs, max_new_tokens=256)
print_memory_usage()

# После очистки
del model
torch.cuda.empty_cache()
print_memory_usage()
```

### Типичные проблемы и решения

| Проблема | Причина | Решение |
|----------|---------|---------|
| `torch_dtype` игнорируется | Квантизация переопределяет веса | Это нормально, dtype применяется к вычислениям |
| Ошибка с bfloat16 на старых GPU | Требуется Ampere (A100, RTX 30+) | Используйте `torch.float16` |
| Flash Attention не работает с float32 | Поддерживает только FP16/BF16 | Смените dtype на FP16  |
| 4-bit модель даёт бессвязный текст | Слишком агрессивная квантизация | Используйте NF4 с double_quant или 8-bit |

### Сравнение качества: FP16 vs 4-bit на реальных задачах

```python
# Тестовый промпт для оценки потери качества
test_prompts = [
    "Explain the difference between a list and a tuple in Python",
    "Calculate 15% of 240", 
    "Write a function to check if a string is a palindrome",
    "What is the capital of Burkina Faso?"
]

def evaluate_quality(model, prompts):
    """Оценка качества ответов (субъективно)"""
    results = []
    for prompt in prompts:
        messages = [{"role": "user", "content": prompt}]
        formatted = tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
        inputs = tokenizer(formatted, return_tensors="pt").to(model.device)
        
        with torch.no_grad():
            outputs = model.generate(**inputs, max_new_tokens=100, do_sample=False)
        
        response = tokenizer.decode(outputs[0][inputs.input_ids.shape[1]:], skip_special_tokens=True)
        results.append(response)
    return results

# Типичное наблюдение:
# FP16 и 4-bit NF4 дают почти идентичные ответы
# Разница заметна только на сложных рассуждениях (>3 шагов)
```

### Рекомендации по выбору

1. **Для большинства пользователей:** 4-bit NF4 через `BitsAndBytesConfig` — оптимальный выбор
2. **Для production с GPU 8GB+:** 8-bit или FP16 для максимальной стабильности
3. **Для edge-девайсов:** Используйте предварительно квантизованные GGUF-версии (глава 6)
4. **Для тонкой настройки:** Начинайте с FP16 или 8-bit, 4-bit может снизить качество дообучения

Следующая глава покажет, как реализовать эффективную батчевую обработку нескольких промптов одновременно.

Скидывайте главу 15.



## 15. Создание простого командного чат-интерфейса

### Минимальный рабочий чат-бот

```python
#!/usr/bin/env python3
from transformers import AutoTokenizer, AutoModelForCausalLM, BitsAndBytesConfig
import torch

# Загрузка модели (4-bit для экономии памяти)
quant_config = BitsAndBytesConfig(load_in_4bit=True)
tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen2.5-3B-Instruct")
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    quantization_config=quant_config,
    device_map="auto"
)

print("Chatbot ready. Type 'quit' to exit.\n")

while True:
    user_input = input("You: ")
    if user_input.lower() in ["quit", "exit", "q"]:
        break
    
    # Форматирование промпта
    messages = [{"role": "user", "content": user_input}]
    prompt = tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
    inputs = tokenizer(prompt, return_tensors="pt").to(model.device)
    
    # Генерация
    with torch.no_grad():
        outputs = model.generate(
            **inputs,
            max_new_tokens=256,
            temperature=0.7,
            do_sample=True,
            pad_token_id=tokenizer.eos_token_id
        )
    
    response = tokenizer.decode(outputs[0][inputs.input_ids.shape[1]:], skip_special_tokens=True)
    print(f"Bot: {response}\n")
```

### Чат с историей диалога (многооборотный)

```python
#!/usr/bin/env python3
from transformers import AutoTokenizer, AutoModelForCausalLM, BitsAndBytesConfig
import torch

class QwenChat:
    def __init__(self, system_prompt="You are a helpful assistant."):
        quant_config = BitsAndBytesConfig(load_in_4bit=True)
        self.tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen2.5-3B-Instruct")
        self.model = AutoModelForCausalLM.from_pretrained(
            "Qwen/Qwen2.5-3B-Instruct",
            quantization_config=quant_config,
            device_map="auto"
        )
        
        # История диалога
        self.history = [{"role": "system", "content": system_prompt}]
        
    def generate(self, user_input, max_new_tokens=256, temperature=0.7):
        # Добавляем сообщение пользователя
        self.history.append({"role": "user", "content": user_input})
        
        # Применяем шаблон ко всей истории
        prompt = self.tokenizer.apply_chat_template(
            self.history, 
            tokenize=False, 
            add_generation_prompt=True
        )
        inputs = self.tokenizer(prompt, return_tensors="pt").to(self.model.device)
        
        # Генерация
        with torch.no_grad():
            outputs = self.model.generate(
                **inputs,
                max_new_tokens=max_new_tokens,
                temperature=temperature,
                do_sample=True,
                pad_token_id=self.tokenizer.eos_token_id
            )
        
        response = self.tokenizer.decode(
            outputs[0][inputs.input_ids.shape[1]:], 
            skip_special_tokens=True
        )
        
        # Сохраняем ответ в историю
        self.history.append({"role": "assistant", "content": response})
        return response
    
    def clear_history(self, keep_system=True):
        if keep_system:
            system = self.history[0] if self.history and self.history[0]["role"] == "system" else None
            self.history = [system] if system else []
        else:
            self.history = []
    
    def print_history(self):
        for msg in self.history:
            if msg["role"] != "system":
                print(f"{msg['role'].capitalize()}: {msg['content'][:100]}...")

# Использование
chat = QwenChat(system_prompt="You are a Python coding expert. Keep answers concise.")

print("Python Coding Assistant. Commands: /clear, /history, /quit\n")

while True:
    user_input = input(">>> ")
    
    if user_input.lower() in ["/quit", "/exit", "q"]:
        break
    elif user_input == "/clear":
        chat.clear_history()
        print("History cleared.\n")
        continue
    elif user_input == "/history":
        chat.print_history()
        print()
        continue
    
    response = chat.generate(user_input)
    print(f"Assistant: {response}\n")
```

### Чат с поддержкой команд и параметров

```python
#!/usr/bin/env python3
import cmd
import torch
from transformers import AutoTokenizer, AutoModelForCausalLM, BitsAndBytesConfig

class QwenShell(cmd.Cmd):
    intro = """
    ╔══════════════════════════════════════╗
    ║   Qwen2.5-3B Command-Line Chat      ║
    ╚══════════════════════════════════════╝
    
    Commands:
      /temp [value]  - Set temperature (0.1-2.0)
      /max [tokens]  - Set max new tokens
      /clear         - Clear conversation history
      /system [text] - Change system prompt
      /save [file]   - Save conversation to file
      /load [file]   - Load conversation from file
      /stats         - Show model and memory stats
      /help          - Show this help
      /quit          - Exit
    
    Type your message and press Enter.
    """
    
    def __init__(self):
        super().__init__()
        self.load_model()
        self.history = [{"role": "system", "content": "You are a helpful assistant."}]
        self.temperature = 0.7
        self.max_new_tokens = 256
        self.prompt = ">>> "
    
    def load_model(self):
        print("Loading model... (this may take a minute)")
        quant_config = BitsAndBytesConfig(load_in_4bit=True)
        self.tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen2.5-3B-Instruct")
        self.model = AutoModelForCausalLM.from_pretrained(
            "Qwen/Qwen2.5-3B-Instruct",
            quantization_config=quant_config,
            device_map="auto"
        )
        print("Model ready!\n")
    
    def default(self, line):
        """Обработка обычного сообщения (не команды)"""
        if line.startswith('/'):
            print(f"Unknown command: {line}. Type /help for commands.")
            return
        
        # Добавляем сообщение пользователя
        self.history.append({"role": "user", "content": line})
        
        # Генерация
        prompt = self.tokenizer.apply_chat_template(
            self.history, tokenize=False, add_generation_prompt=True
        )
        inputs = self.tokenizer(prompt, return_tensors="pt").to(self.model.device)
        
        print("Bot: ", end="", flush=True)
        
        with torch.no_grad():
            outputs = self.model.generate(
                **inputs,
                max_new_tokens=self.max_new_tokens,
                temperature=self.temperature,
                do_sample=True,
                pad_token_id=self.tokenizer.eos_token_id
            )
        
        response = self.tokenizer.decode(
            outputs[0][inputs.input_ids.shape[1]:], 
            skip_special_tokens=True
        )
        print(response)
        
        # Сохраняем ответ
        self.history.append({"role": "assistant", "content": response})
    
    def do_temp(self, arg):
        """Set temperature: /temp 0.7"""
        try:
            temp = float(arg)
            if 0.0 <= temp <= 2.0:
                self.temperature = temp
                print(f"Temperature set to {temp}")
            else:
                print("Temperature must be between 0.0 and 2.0")
        except ValueError:
            print(f"Current temperature: {self.temperature}")
    
    def do_max(self, arg):
        """Set max tokens: /max 512"""
        try:
            max_tok = int(arg)
            if 1 <= max_tok <= 4096:
                self.max_new_tokens = max_tok
                print(f"Max new tokens set to {max_tok}")
            else:
                print("Max tokens must be between 1 and 4096")
        except ValueError:
            print(f"Current max tokens: {self.max_new_tokens}")
    
    def do_clear(self, arg):
        """Clear conversation history: /clear"""
        system = self.history[0] if self.history and self.history[0]["role"] == "system" else None
        self.history = [system] if system else []
        print("Conversation history cleared.")
    
    def do_system(self, arg):
        """Change system prompt: /system You are an expert in..."""
        if not arg:
            current = self.history[0]["content"] if self.history else "None"
            print(f"Current system prompt: {current}")
            return
        
        if self.history and self.history[0]["role"] == "system":
            self.history[0]["content"] = arg
        else:
            self.history.insert(0, {"role": "system", "content": arg})
        print(f"System prompt updated: {arg[:50]}...")
    
    def do_save(self, arg):
        """Save conversation: /save chat_log.txt"""
        filename = arg.strip() or "conversation.txt"
        with open(filename, 'w', encoding='utf-8') as f:
            for msg in self.history:
                if msg["role"] != "system":
                    f.write(f"{msg['role'].upper()}: {msg['content']}\n\n")
        print(f"Conversation saved to {filename}")
    
    def do_load(self, arg):
        """Load conversation: /load chat_log.txt"""
        filename = arg.strip()
        if not filename:
            print("Please specify filename: /load chat_log.txt")
            return
        
        try:
            with open(filename, 'r', encoding='utf-8') as f:
                content = f.read()
            print(f"Loaded {len(content)} characters from {filename}")
            # Простая загрузка — добавляем как контекст
            self.history.append({"role": "user", "content": f"Previous conversation context:\n{content}"})
        except FileNotFoundError:
            print(f"File {filename} not found")
    
    def do_stats(self, arg):
        """Show model statistics: /stats"""
        print(f"\n=== Model Statistics ===")
        print(f"Model: Qwen2.5-3B-Instruct")
        print(f"Device: {self.model.device}")
        print(f"Temperature: {self.temperature}")
        print(f"Max new tokens: {self.max_new_tokens}")
        print(f"History length: {len(self.history)} messages")
        
        if torch.cuda.is_available():
            print(f"VRAM allocated: {torch.cuda.memory_allocated() / 1e9:.2f} GB")
            print(f"VRAM reserved: {torch.cuda.memory_reserved() / 1e9:.2f} GB")
        
        print(f"Memory footprint: {self.model.get_memory_footprint() / 1e9:.2f} GB\n")
    
    def do_quit(self, arg):
        """Exit the chat: /quit"""
        print("Goodbye!")
        return True
    
    def do_help(self, arg):
        """Show help: /help"""
        print(self.intro)

if __name__ == "__main__":
    QwenShell().cmdloop()
```

### Чат с цветным выводом (для улучшенного UX)

```python
#!/usr/bin/env python3
# pip install colorama
from colorama import init, Fore, Style
init(autoreset=True)

class ColorfulQwenChat(QwenChat):
    def generate(self, user_input):
        print(f"{Fore.GREEN}You:{Style.RESET_ALL} {user_input}")
        
        response = super().generate(user_input)
        
        print(f"{Fore.CYAN}Bot:{Style.RESET_ALL} {response}\n")
        return response

# Использование
chat = ColorfulQwenChat()
print(f"{Fore.YELLOW}Qwen2.5-3B Chatbot (colorful){Style.RESET_ALL}\n")

while True:
    user = input(f"{Fore.GREEN}> {Style.RESET_ALL}")
    if user.lower() == "quit":
        break
    chat.generate(user)
```

### Чат с потоковым выводом (по токену)

```python
#!/usr/bin/env python3
from transformers import TextIteratorStreamer
from threading import Thread

class StreamingQwenChat(QwenChat):
    def generate_stream(self, user_input, temperature=0.7):
        self.history.append({"role": "user", "content": user_input})
        
        prompt = self.tokenizer.apply_chat_template(
            self.history, tokenize=False, add_generation_prompt=True
        )
        inputs = self.tokenizer(prompt, return_tensors="pt").to(self.model.device)
        
        streamer = TextIteratorStreamer(
            self.tokenizer, 
            skip_prompt=True, 
            skip_special_tokens=True
        )
        
        generation_kwargs = dict(
            **inputs,
            max_new_tokens=self.max_new_tokens,
            temperature=temperature,
            do_sample=True,
            pad_token_id=self.tokenizer.eos_token_id,
            streamer=streamer
        )
        
        thread = Thread(target=self.model.generate, kwargs=generation_kwargs)
        thread.start()
        
        print("Bot: ", end="", flush=True)
        response = ""
        for token in streamer:
            print(token, end="", flush=True)
            response += token
        print()
        
        self.history.append({"role": "assistant", "content": response})
        return response

# Использование
chat = StreamingQwenChat()
print("Streaming Chatbot (type 'quit' to exit)\n")

while True:
    user = input("You: ")
    if user.lower() == "quit":
        break
    chat.generate_stream(user)
```

### Минимальный чат для слабых систем (GGUF через llama-cpp-python)

```python
#!/usr/bin/env python3
from llama_cpp import Llama

# Загрузка GGUF модели
llm = Llama.from_pretrained(
    repo_id="lmstudio-community/Qwen2.5-3B-Instruct-GGUF",
    filename="Qwen2.5-3B-Instruct-Q4_K_M.gguf",
    n_ctx=2048,
    n_threads=4,
    verbose=False
)

print("Qwen Chat (GGUF version)\n")

while True:
    user = input(">>> ")
    if user.lower() in ["quit", "exit"]:
        break
    
    response = llm.create_chat_completion(
        messages=[{"role": "user", "content": user}],
        max_tokens=256,
        temperature=0.7
    )
    
    print(f"Bot: {response['choices'][0]['message']['content']}\n")
```

### Чек-лист для стабильной работы чата

- [ ] Установлен `transformers>=4.37.0`
- [ ] Модель загружается с `device_map="auto"`
- [ ] `pad_token_id` установлен (обычно = eos_token_id)
- [ ] Используется `apply_chat_template()` для форматирования
- [ ] При длительных диалогах история ограничена (иначе контекст переполнится)
- [ ] Добавлен `torch.no_grad()` для экономии памяти

### Ограничение длины истории (защита от переполнения контекста)

```python
class SafeQwenChat(QwenChat):
    def __init__(self, max_context_tokens=8000, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.max_context_tokens = max_context_tokens
    
    def estimate_tokens(self, messages):
        """Приблизительная оценка токенов в истории"""
        text = self.tokenizer.apply_chat_template(messages, tokenize=False)
        return len(self.tokenizer.encode(text))
    
    def add_message(self, role, content):
        self.history.append({"role": role, "content": content})
        
        # Проверяем размер контекста
        while len(self.history) > 1:  # Оставляем system prompt
            token_count = self.estimate_tokens(self.history)
            if token_count < self.max_context_tokens - 1000:
                break
            # Удаляем второе сообщение (после system)
            if len(self.history) > 2:
                removed = self.history.pop(1)
                print(f"[Context limit reached, removed: {removed['content'][:50]}...]")
            else:
                break
```

Следующая глава покажет, как создавать системные промпты и управлять ролями для специализированных задач.

Скидывайте главу 16.



## 16. Создание OpenAI-совместимого локального API сервера

### Почему OpenAI-совместимый API

Единый стандарт позволяет использовать любые инструменты экосистемы OpenAI: официальные SDK, LangChain, LlamaIndex, Continue, Cursor, и тысячи других приложений — без изменения кода. Достаточно сменить `base_url` на локальный эндпоинт .

### Способ 1: vLLM (максимальная производительность на GPU)

vLLM — самый производительный инференс-сервер для GPU с поддержкой PagedAttention и непрерывной батчингом.

```bash
# Установка
pip install vllm

# Запуск сервера с Qwen2.5-3B
vllm serve Qwen/Qwen2.5-3B-Instruct \
    --host 0.0.0.0 \
    --port 8000 \
    --api-key any-secure-key \
    --max-model-len 8192 \
    --gpu-memory-utilization 0.9
```

**Параметры:**
| Флаг | Значение |
|------|----------|
| `--host 0.0.0.0` | Доступ с других машин |
| `--port 8000` | Порт сервера |
| `--api-key` | Ключ аутентификации (опционально) |
| `--max-model-len` | Максимальный контекст |
| `--gpu-memory-utilization` | Доля VRAM для использования |

**Использование с OpenAI SDK:**
```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:8000/v1",
    api_key="any-secure-key"  # Если указали при запуске
)

response = client.chat.completions.create(
    model="Qwen/Qwen2.5-3B-Instruct",
    messages=[{"role": "user", "content": "Hello!"}],
    temperature=0.7,
    max_tokens=256
)

print(response.choices[0].message.content)
```

**Потоковая генерация через vLLM:**
```python
stream = client.chat.completions.create(
    model="Qwen/Qwen2.5-3B-Instruct",
    messages=[{"role": "user", "content": "Write a poem"}],
    stream=True
)

for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

### Способ 2: llama.cpp (лучший для CPU и смешанных сценариев)

llama.cpp имеет встроенный OpenAI-совместимый сервер .

```bash
# Установка (macOS)
brew install llama.cpp

# Запуск сервера с GGUF моделью
llama-server \
    -hf Lucy-in-the-Sky/Qwen2.5-3B-Instruct-Q8_0-GGUF:Q8_0 \
    --host 0.0.0.0 \
    --port 8080 \
    -ngl 32 \
    -c 4096
```

**С предварительно скачанной моделью:**
```bash
llama-server \
    -m ./Qwen2.5-3B-Instruct-Q4_K_M.gguf \
    --host 0.0.0.0 \
    --port 8080 \
    -ngl 32 \
    -c 8192 \
    --threads 8
```

**Использование через OpenAI SDK:**
```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:8080/v1",
    api_key="not-needed"  # llama.cpp по умолчанию без ключа
)

response = client.chat.completions.create(
    model="qwen2.5-3b",
    messages=[{"role": "user", "content": "Explain quantum computing"}],
    temperature=0.7
)
```

### Способ 3: FastAPI + Transformers (полный контроль)

Собственная реализация на FastAPI даёт максимальную гибкость .

```python
#!/usr/bin/env python3
# qwen_api_server.py

from fastapi import FastAPI, HTTPException
from fastapi.responses import StreamingResponse
from pydantic import BaseModel
from typing import List, Optional, AsyncGenerator
import torch
from transformers import AutoTokenizer, AutoModelForCausalLM, BitsAndBytesConfig
import asyncio
from threading import Thread
from transformers import TextIteratorStreamer
import uvicorn

# ============ Конфигурация ============
MODEL_NAME = "Qwen/Qwen2.5-3B-Instruct"
API_KEY = "sk-local-qwen-key"  # Опционально
DEVICE = "cuda" if torch.cuda.is_available() else "cpu"

# Загрузка модели (4-bit для экономии памяти)
bnb_config = BitsAndBytesConfig(load_in_4bit=True) if DEVICE == "cuda" else None
tokenizer = AutoTokenizer.from_pretrained(MODEL_NAME)
model = AutoModelForCausalLM.from_pretrained(
    MODEL_NAME,
    quantization_config=bnb_config,
    device_map="auto" if DEVICE == "cuda" else "cpu",
    torch_dtype=torch.float16 if DEVICE == "cuda" else torch.float32
)

# ============ Pydantic модели ============
class Message(BaseModel):
    role: str
    content: str

class ChatRequest(BaseModel):
    model: str = "qwen2.5-3b"
    messages: List[Message]
    temperature: Optional[float] = 0.7
    max_tokens: Optional[int] = 256
    top_p: Optional[float] = 0.9
    stream: Optional[bool] = False

class ChatResponse(BaseModel):
    id: str
    object: str = "chat.completion"
    created: int
    model: str
    choices: List[dict]
    usage: dict

# ============ FastAPI приложение ============
app = FastAPI(title="Qwen2.5-3B API", description="OpenAI-compatible local API")

def verify_api_key(authorization: Optional[str] = None):
    if API_KEY == "disabled":
        return True
    if not authorization:
        return False
    scheme, _, key = authorization.partition(" ")
    return scheme.lower() == "bearer" and key == API_KEY

@app.get("/v1/models")
async def list_models(authorization: Optional[str] = None):
    if not verify_api_key(authorization):
        raise HTTPException(status_code=401, detail="Invalid API key")
    return {
        "object": "list",
        "data": [{
            "id": "qwen2.5-3b-instruct",
            "object": "model",
            "created": 1700000000,
            "owned_by": "local"
        }]
    }

@app.post("/v1/chat/completions")
async def chat_completion(request: ChatRequest, authorization: Optional[str] = None):
    if not verify_api_key(authorization):
        raise HTTPException(status_code=401, detail="Invalid API key")
    
    # Применяем шаблон чата Qwen
    messages = [msg.dict() for msg in request.messages]
    prompt = tokenizer.apply_chat_template(
        messages,
        tokenize=False,
        add_generation_prompt=True
    )
    
    inputs = tokenizer(prompt, return_tensors="pt").to(model.device)
    
    if request.stream:
        return StreamingResponse(
            stream_generator(inputs, request),
            media_type="text/event-stream"
        )
    else:
        with torch.no_grad():
            outputs = model.generate(
                **inputs,
                max_new_tokens=request.max_tokens,
                temperature=request.temperature,
                top_p=request.top_p,
                do_sample=request.temperature > 0,
                pad_token_id=tokenizer.eos_token_id
            )
        
        response = tokenizer.decode(
            outputs[0][inputs.input_ids.shape[1]:],
            skip_special_tokens=True
        )
        
        import time
        return ChatResponse(
            id=f"chatcmpl-{int(time.time())}",
            created=int(time.time()),
            model=request.model,
            choices=[{
                "index": 0,
                "message": {"role": "assistant", "content": response},
                "finish_reason": "stop"
            }],
            usage={
                "prompt_tokens": inputs.input_ids.shape[1],
                "completion_tokens": outputs.shape[1] - inputs.input_ids.shape[1],
                "total_tokens": outputs.shape[1]
            }
        ).dict()

async def stream_generator(inputs, request):
    """Потоковая генерация (token by token)"""
    streamer = TextIteratorStreamer(
        tokenizer,
        skip_prompt=True,
        skip_special_tokens=True
    )
    
    generation_kwargs = dict(
        **inputs,
        max_new_tokens=request.max_tokens,
        temperature=request.temperature,
        top_p=request.top_p,
        do_sample=request.temperature > 0,
        pad_token_id=tokenizer.eos_token_id,
        streamer=streamer
    )
    
    thread = Thread(target=model.generate, kwargs=generation_kwargs)
    thread.start()
    
    import time
    created = int(time.time())
    
    # Отправляем первый chunk
    yield f"data: {ChatStreamChunk(id=f'chatcmpl-{created}', created=created, model=request.model, delta={'role': 'assistant'}, finish_reason=None).json()}\n\n"
    
    for token in streamer:
        yield f"data: {ChatStreamChunk(id=f'chatcmpl-{created}', created=created, model=request.model, delta={'content': token}, finish_reason=None).json()}\n\n"
    
    # Отправляем финальный chunk
    yield f"data: {ChatStreamChunk(id=f'chatcmpl-{created}', created=created, model=request.model, delta={}, finish_reason='stop').json()}\n\n"
    yield "data: [DONE]\n\n"

class ChatStreamChunk(BaseModel):
    id: str
    object: str = "chat.completion.chunk"
    created: int
    model: str
    choices: List[dict]

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

**Запуск сервера:**
```bash
python qwen_api_server.py
# Сервер доступен на http://localhost:8000
# Swagger документация: http://localhost:8000/docs
```

### Способ 4: OpenLLM (простота использования)

OpenLLM — готовое решение с одним Docker-командами или Python-пакетом .

```bash
# Установка
pip install openllm

# Запуск сервера с Qwen2.5-3B
openllm serve qwen2.5:3b

# С авто-квантизацией для слабых GPU
openllm serve qwen2.5:3b --quantize awq

# С кастомными параметрами
openllm serve qwen2.5:3b \
    --host 0.0.0.0 \
    --port 3000 \
    --workers 2 \
    --max-model-len 8192
```

**Использование через OpenAI SDK:**
```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:3000/v1",
    api_key="na"  # OpenLLM по умолчанию без ключа
)

response = client.chat.completions.create(
    model="qwen2.5:3b",
    messages=[{"role": "user", "content": "Hello!"}]
)
```

### Способ 5: FastChat (модель как сервис)

FastChat предоставляет полноценную платформу для деплоя LLM с контроллером и воркерами .

```bash
# Установка
pip install fschat

# Запуск контроллера
python -m fastchat.serve.controller --host 0.0.0.0 --port 21001

# Запуск воркера с моделью
python -m fastchat.serve.vllm_worker \
    --model-path Qwen/Qwen2.5-3B-Instruct \
    --controller http://localhost:21001 \
    --host 0.0.0.0 \
    --port 21002

# Запуск OpenAI-совместимого API сервера
python -m fastchat.serve.openai_api_server \
    --controller http://localhost:21001 \
    --host 0.0.0.0 \
    --port 8000
```

### Матрица выбора решения

| Решение | Скорость | Память | Установка | Контроль | Когда использовать |
|---------|----------|--------|-----------|----------|-------------------|
| **vLLM** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ | Высокая нагрузка, GPU |
| **llama.cpp** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | CPU, бюджетные системы |
| **FastAPI + Transformers** | ⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | Кастомная логика |
| **OpenLLM** | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ | Быстрый старт |
| **FastChat** | ⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ | Многопользовательский режим |

### Проверка работоспособности API

```bash
# curl запрос к серверу
curl -X POST "http://localhost:8000/v1/chat/completions" \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer sk-local-qwen-key" \
     -d '{
         "model": "qwen2.5-3b",
         "messages": [{"role": "user", "content": "Hello!"}],
         "max_tokens": 50
     }'
```

### Интеграция с популярными инструментами

**LangChain:**
```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    base_url="http://localhost:8000/v1",
    api_key="sk-local-qwen-key",
    model="qwen2.5-3b"
)

response = llm.invoke("Hello!")
```

**Continue (VS Code плагин):**
```json
// ~/.continue/config.json
{
  "models": [{
    "title": "Qwen Local",
    "provider": "openai",
    "model": "qwen2.5-3b",
    "apiBase": "http://localhost:8000/v1",
    "apiKey": "sk-local-qwen-key"
  }]
}
```

### Безопасность и production-рекомендации

1. **API ключи**: Всегда используйте аутентификацию в production 
2. **Rate limiting**: Добавьте `slowapi` для ограничения запросов
3. **Логирование**: Включите access logs для аудита
4. **Мониторинг**: Отслеживайте использование VRAM и latency
5. **Docker**: Упакуйте сервер в контейнер для воспроизводимости

```dockerfile
# Dockerfile для API сервера
FROM pytorch/pytorch:2.0.1-cuda11.7-cudnn8-runtime
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "qwen_api_server.py"]
```

Следующая глава покажет, как реализовать продвинутую систему промптов с контекстным окном и управлением памятью.

Скидывайте главу 17.



## 17. Работа с длинными контекстами (128k токенов)

### Официальная позиция: 32K vs 128K

Согласно документации Qwen2.5, модель **официально поддерживает контекст до 128K токенов** при инференсе. Однако архитектурное ограничение для Qwen2.5-3B составляет **32K токенов** в полной точности.

**Важное различие:**
| Параметр | Значение |
|----------|----------|
| Максимальный контекст (RoPE расширение) | 128K токенов |
| Базовый контекст (без расширения) | 32K токенов |
| Генерация (max_new_tokens) | 8K токенов |

**Что это значит:** Qwen2.5-3B может обрабатывать до 128K токенов с правильной настройкой, но нативные веса оптимизированы под 32K. Расширение достигается через методы позиционного кодирования (RoPE scaling).

### Почему контекст 128K требует особого подхода

При обработке 128K токенов возникает проблема **KV-кэша** — памяти, хранящей ключи и значения для attention механизма. Размер KV-кэша растёт линейно с длиной последовательности.

**Расчёт памяти для Qwen2.5-3B:**

```python
# Параметры модели (из документации)
num_layers = 36
num_kv_heads = 2        # GQA: 2 KV heads
head_dim = 128          # hidden_dim / num_heads = 896 / 16 = 56? Уточнение: официально 128
hidden_size = 896       # Для 3B модели

# KV-кэш для 1 токена (BF16)
kv_cache_per_token = num_layers * num_kv_heads * head_dim * 2 (K and V) * 2 bytes
kv_cache_per_token = 36 * 2 * 128 * 2 * 2 = 36,864 байт ≈ 36 KB

# Для 128K токенов
kv_cache_128k = 36 KB * 131,072 = 4.7 GB
```

**Итог:** KV-кэш для 128K токенов занимает **~4.7 GB** в BF16. Добавьте 2 GB весов (4-bit) = **~6.7 GB** минимум.

### Решение 1: Установка максимального контекста при загрузке

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
import torch

# Ключевой параметр: model_max_length
tokenizer = AutoTokenizer.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    model_max_length=131072  # 128K токенов
)

model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    torch_dtype=torch.float16,
    device_map="auto",
    max_position_embeddings=131072  # Расширение позиций
)

# Проверка
print(f"Max context: {tokenizer.model_max_length:,} токенов")
print(f"Max position embeddings: {model.config.max_position_embeddings:,}")
```

### Решение 2: RoPE Scaling для контекста > 32K

Для корректной работы на 128K необходимо настроить RoPE (Rotary Position Embedding) scaling:

```python
from transformers import AutoConfig

# Загрузка конфигурации
config = AutoConfig.from_pretrained("Qwen/Qwen2.5-3B-Instruct")

# Настройка RoPE scaling для длинных контекстов
config.rope_scaling = {
    "type": "linear",           # или "dynamic"
    "factor": 4.0,              # 128K / 32K = 4
}
config.max_position_embeddings = 131072

# Загрузка модели с новой конфигурацией
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    config=config,
    torch_dtype=torch.float16,
    device_map="auto"
)
```

### Решение 3: Эффективное управление KV-кэшем

**Проблема:** Хранение полного KV-кэша для 128K требует >4 GB памяти. **Решения:**

#### 3.1 Sliding Window Attention (SWA)

Qwen2.5 поддерживает скользящее окно внимания, где каждый токен видит только последние N токенов:

```python
# В transformers это настраивается через конфиг
config.use_sliding_window = True
config.sliding_window = 32768  # Окно 32K токенов

model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    config=config,
    torch_dtype=torch.float16
)
```

**Как работает SWA в Qwen:**
- Кэш хранит только последние `sliding_window` токенов
- При добавлении нового токена старые вытесняются
- KV-кэш фиксированного размера ~1.2 GB (для 32K окна)

#### 3.2 KV Cache Quantization (FP8/INT8)

Сжатие KV-кэша в 8-бит:

```python
from transformers import BitsAndBytesConfig

# Только для KV-кэша, не для весов
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    torch_dtype=torch.float16,
    device_map="auto",
    cache_implementation="quantized",
    cache_config={
        "backend": "bitsandbytes",
        "quant_type": "fp8"  # или "int8"
    }
)
```

**Экономия:** FP8 сокращает KV-кэш в 2 раза (~2.35 GB для 128K).

### Решение 4: Продвинутые методы (L2A, AHN)

Исследования показывают, что большинству токенов не нужен глобальный контекст. Метод **L2A (Learning To Attend)** достигает 128K контекста с пропуском 80% токенов.

Байтданс предлагает **AHN (Artificial Hippocampus Networks)** для Qwen2.5-3B:

```python
# Загрузка модели с AHN модулем
from transformers import AutoModelForCausalLM

model = AutoModelForCausalLM.from_pretrained(
    "ByteDance-Seed/AHN-DN-for-Qwen-2.5-Instruct-3B",
    torch_dtype=torch.float16,
    device_map="auto"
)
# Модель поддерживает 128K контекст сжатием старого кэша в RNN-состояние
```

**Результаты AHN для 3B модели:**
| Метрика | Sinks+SWA | AHN-DN |
|---------|-----------|--------|
| LongBench Avg. | 34.31 | **35.89** |
| NarrativeQA | 16.55 | **19.78** |
| QMSum | 21.54 | **22.35** |

### Практическая конфигурация для 128K

```python
from transformers import AutoModelForCausalLM, AutoTokenizer, BitsAndBytesConfig
import torch

# Конфигурация для максимальной экономии памяти
bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.float16
)

tokenizer = AutoTokenizer.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    model_max_length=131072,
    use_fast=True
)

model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    quantization_config=bnb_config,
    device_map="auto",
    max_position_embeddings=131072,
    # Включение Flash Attention 2 для длинных контекстов
    attn_implementation="flash_attention_2"
)

# Проверка памяти
def estimate_kv_cache_memory(seq_len, dtype_size=2):  # BF16 = 2 bytes
    num_layers = 36
    num_kv_heads = 2
    head_dim = 128
    bytes_per_token = num_layers * num_kv_heads * head_dim * 2 * dtype_size
    return bytes_per_token * seq_len / 1e9

print(f"KV-кэш для 128K: {estimate_kv_cache_memory(131072):.1f} GB")
```

### Ограничение генерации в длинных контекстах

Модель может генерировать **до 8K токенов** на выходе, даже при контексте в 128K:

```python
# Правильная настройка: max_new_tokens не может превышать 8192
outputs = model.generate(
    **inputs,
    max_new_tokens=8192,  # Максимум для Qwen2.5
    min_new_tokens=1,
    do_sample=True,
    temperature=0.7
)
```

### Мониторинг памяти при длинных контекстах

```python
import torch
import psutil

def monitor_long_context(model, tokenizer, text):
    """Отслеживание памяти при обработке длинного текста"""
    
    # Токенизация
    inputs = tokenizer(text, return_tensors="pt", truncation=False).to(model.device)
    seq_len = inputs.input_ids.shape[1]
    
    print(f"Длина последовательности: {seq_len:,} токенов")
    
    # Память до генерации
    if torch.cuda.is_available():
        torch.cuda.reset_peak_memory_stats()
        mem_before = torch.cuda.memory_allocated() / 1e9
    
    # Генерация
    with torch.no_grad():
        outputs = model.generate(
            **inputs,
            max_new_tokens=100,
            pad_token_id=tokenizer.eos_token_id
        )
    
    # Память после
    if torch.cuda.is_available():
        mem_peak = torch.cuda.max_memory_allocated() / 1e9
        print(f"Пиковая VRAM: {mem_peak:.1f} GB")
    
    return outputs

# Пример с синтетическим длинным текстом
long_text = "Это тестовый текст. " * 20000  # ~40K токенов
monitor_long_context(model, tokenizer, long_text)
```

### Частые проблемы при 128K контексте

| Проблема | Причина | Решение |
|----------|---------|---------|
| OOM при 64K+ токенов | KV-кэш > VRAM | Используйте SWA или 8-bit KV |
| Ошибка `max_position_embeddings` | Конфиг не расширен | Установите `rope_scaling` |
| Медленный инференс (1-2 т/с) | CPU offloading | Включите Flash Attention 2 |
| Модель "забывает" начало | Позиционное смещение | Используйте dynamic RoPE scaling |

### Рекомендации по выбору стратегии

| Сценарий | Рекомендация | Ожидаемая память |
|----------|--------------|------------------|
| 32K контекст, базовая работа | Стандартная загрузка | 4-5 GB VRAM |
| 64K контекст, 4-bit веса | SWA (окно 32K) | 4 GB VRAM |
| 128K контекст, максимальная точность | 8-bit KV + 4-bit веса | 6-7 GB VRAM |
| 128K контекст, ограниченная память | AHN или L2A модель | 4-5 GB VRAM |
| Документы >128K | RAG + чанкинг | N/A (другой подход) |

### Альтернатива: RAG вместо расширенного контекста

Если 128K контекста не влезает в память, используйте Retrieval-Augmented Generation:

```python
from langchain_community.document_loaders import TextLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_community.vectorstores import FAISS
from langchain_huggingface import HuggingFaceEmbeddings

# Разбивка документа на чанки
splitter = RecursiveCharacterTextSplitter(chunk_size=8000, chunk_overlap=200)
chunks = splitter.split_text(long_document)

# Эмбеддинги и поиск
embeddings = HuggingFaceEmbeddings(model_name="BAAI/bge-small-en")
vectorstore = FAISS.from_texts(chunks, embeddings)

# Поиск релевантных чанков перед запросом
relevant_chunks = vectorstore.similarity_search(query, k=3)
context = "\n\n".join([chunk.page_content for chunk in relevant_chunks])
```

Следующая глава покажет, как реализовать продвинутые техники для длинных контекстов, включая StreamingLLM и позиционное интерполирование.

Скидывайте главу 18.



## 18. Кодогенерация и специализированные задачи программирования

### Выбор правильной модели: Qwen2.5-Coder vs Qwen2.5-Instruct

Для задач программирования Qwen2.5 существует две ветки. Если ваша цель — именно код, **используйте Coder-версию**, а не Instruct.

| Характеристика | Qwen2.5-3B-Instruct | Qwen2.5-Coder-3B |
|----------------|---------------------|------------------|
| Основное назначение | Общие диалоги, ассистент | Кодогенерация, рефакторинг |
| Объём обучающих токенов | ~18 трлн | **5.5 трлн** (код + текстово-кодовые связки)  |
| Заполнение середины (FIM) | ❌ | ✅ |
| Репозиторный уровень | ❌ | ✅ |
| LiveCodeBench Pass@1 | 15.85% | **18.85%** (+3%) на дообученной версии  |

**Ключевое открытие:** Coder-модель обучалась на синтетических данных и code-text grounding, что даёт лучшее понимание структуры программ . Если у вас GPU с 4+ ГБ VRAM — ставьте `Qwen/Qwen2.5-Coder-3B-Instruct`. Если 2-3 ГБ — используйте 4-битную квантизацию этой же модели.

### Загрузка Coder-модели

```python
from transformers import AutoModelForCausalLM, AutoTokenizer, BitsAndBytesConfig
import torch

model_name = "Qwen/Qwen2.5-Coder-3B-Instruct"

# 4-битная квантизация для экономии памяти
bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.float16
)

tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    quantization_config=bnb_config,
    device_map="auto"
)

print(f"Модель загружена. Параметров: {model.num_parameters() / 1e9:.1f}B")
```

### Fill-In-The-Middle (FIM): код, который вы ждали

FIM — ключевая особенность Coder-моделей. Вы передаёте **префикс** (код до курсора) и **суффикс** (код после курсора), модель генерирует то, что должно быть между ними .

**Формат промпта:**
```
<|fim_prefix|>{prefix}<|fim_suffix|>{suffix}<|fim_middle|>
```

**Практический пример — завершение функции:**

```python
# Префикс: код до того места, где нужно вставить логику
prefix = """def reverse_words_with_special_chars(s):
    '''
    Переворачивает каждое слово в строке, сохраняя позиции не-букв.
    Пример: reverse_words_with_special_chars("Hello, world!") -> "olleH, dlrow!"
    
    Параметры:
        s (str): Входная строка
    
    Возвращает:
        str: Обработанная строка
    '''
    # Разбиваем строку на части, сохраняя разделители
    import re
    words = re.findall(r'\\b\\w+\\b|[^\\w]+', s)
    result_parts = []
    
    for part in words:
        if part.isalpha():"""

# Суффикс: код, который идёт ПОСЛЕ вставляемого блока
suffix = """
        else:
            result_parts.append(part)
    
    return ''.join(result_parts)"""

# Формируем FIM-промпт
fim_prompt = f"<|fim_prefix|>{prefix}<|fim_suffix|>{suffix}<|fim_middle|>"

inputs = tokenizer(fim_prompt, return_tensors="pt").to(model.device)

with torch.no_grad():
    outputs = model.generate(
        **inputs,
        max_new_tokens=128,
        temperature=0.2,  # Низкая температура для кода
        do_sample=False    # Детерминированность
    )

completion = tokenizer.decode(outputs[0][inputs.input_ids.shape[1]:], skip_special_tokens=True)
print(f"Сгенерированная логика:\n{completion}")

# Ожидаемый вывод:
# for word in parts:
#     if word.isalpha():
#         result_parts.append(word[::-1])
```

### Продвинутый FIM: многофайловый контекст

Для работы с целыми репозиториями Qwen поддерживает **репозиторный FIM** с указанием структуры файлов :

```
<|repo_name|>my_project
<|file_sep|>src/utils.py
def helper():
    return True

<|file_sep|>src/main.py
from utils import helper

<|fim_prefix|>def process_data(data):
    <|fim_suffix|>return result
<|fim_middle|>
```

**Пример кода:**

```python
def generate_with_repo_context(repo_name, files, prefix, suffix):
    """
    Генерация кода с учётом контекста всего репозитория
    """
    context = f"<|repo_name|>{repo_name}\n"
    
    for file_path, content in files.items():
        context += f"<|file_sep|>{file_path}\n{content}\n"
    
    fim_prompt = (
        f"{context}<|fim_prefix|>{prefix}<|fim_suffix|>{suffix}<|fim_middle|>"
    )
    
    inputs = tokenizer(fim_prompt, return_tensors="pt").to(model.device)
    outputs = model.generate(**inputs, max_new_tokens=256, temperature=0.2)
    
    return tokenizer.decode(outputs[0][inputs.input_ids.shape[1]:], skip_special_tokens=True)

# Использование
files = {
    "src/database.py": "import sqlite3\n\ndef get_connection():\n    return sqlite3.connect('app.db')",
    "src/models.py": "class User:\n    def __init__(self, name):\n        self.name = name"
}

prefix = "def save_user(user):\n    \"\"\"Сохраняет пользователя в БД\"\"\"\n    conn = get_connection()\n    cursor = conn.cursor()\n    "
suffix = "\n    conn.commit()\n    conn.close()\n    return True"

result = generate_with_repo_context("my_app", files, prefix, suffix)
print(result)
# Сгенерирует: cursor.execute('INSERT INTO users (name) VALUES (?)', (user.name,))
```

### Code Review и рефакторинг

Coder-модель хорошо справляется с анализом и улучшением существующего кода:

```python
def code_review(code_snippet):
    """Запрашивает ревью кода с конкретными критериями"""
    prompt = f"""Review the following Python code for:
1. Potential bugs
2. Performance issues
3. Style violations (PEP 8)
4. Security concerns

Code:
```python
{code_snippet}
```

Provide specific suggestions for improvement."""

    messages = [
        {"role": "system", "content": "You are a senior Python code reviewer. Be specific and actionable."},
        {"role": "user", "content": prompt}
    ]
    
    formatted = tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
    inputs = tokenizer(formatted, return_tensors="pt").to(model.device)
    
    outputs = model.generate(**inputs, max_new_tokens=512, temperature=0.3)
    return tokenizer.decode(outputs[0][inputs.input_ids.shape[1]:], skip_special_tokens=True)

# Пример
buggy_code = """
def process_items(items):
    result = []
    for i in range(len(items)):
        if items[i] > 0:
            result.append(items[i] * 2)
    return result
"""

print(code_review(buggy_code))
```

### Сравнение моделей для кода

Исследования показывают, что **fine-tuned версия** Qwen2.5-3B на наборе OpenCodeReasoning даёт улучшения :

| Метрика | Базовая Qwen2.5-3B | Fine-tuned версия | Изменение |
|---------|--------------------|--------------------|-----------|
| LiveCodeBench Pass@1 | 15.85% | **18.85%** | +3.0% |
| Easy задачи | 31.27% | **42.39%** | +11.12% |
| Medium задачи | 11.31% | **13.57%** | +2.26% |

**Формат ответа fine-tuned модели:**
```
<think>
[Пошаговое рассуждение о решении]
</think>
```python
[Код решения]
```

**Загрузка fine-tuned версии:**
```python
from transformers import AutoModelForCausalLM, AutoTokenizer

model = AutoModelForCausalLM.from_pretrained(
    "MarioCap/Qwen2.5-3B-OCR-100S-GGUF",
    device_map="auto"
)
tokenizer = AutoTokenizer.from_pretrained("MarioCap/Qwen2.5-3B-OCR-100S-GGUF")
```

### Интеграция с редакторами кода

Qwen2.5-Coder интегрируется с редакторами через протокол, совместимый с OpenAI API .

**Для Emacs (Minuet):**
```elisp
(use-package minuet
  :config
  (setq minuet-provider 'openai-fim-compatible)
  (setq minuet-n-completions 1)
  (setq minuet-context-window 512)
  (plist-put minuet-openai-fim-compatible-options :end-point "http://localhost:11434/v1/completions")
  (plist-put minuet-openai-fim-compatible-options :model "qwen2.5-coder:3b")
  (plist-put minuet-openai-fim-compatible-options :max_tokens 56))
```

**Для Ollama:**
```bash
ollama pull qwen2.5-coder:3b
ollama run qwen2.5-coder:3b
```

### Оптимальные параметры для кода

| Параметр | Кодогенерация | Рефакторинг | Code Review | Объяснение |
|----------|---------------|-------------|-------------|------------|
| `temperature` | 0.0-0.2 | 0.3 | 0.3-0.5 | 0.2-0.4 |
| `do_sample` | False | True | True | True |
| `top_p` | 0.9 | 0.9 | 0.95 | 0.9 |
| `max_new_tokens` | 512-1024 | 512 | 1024 | 256 |
| `repetition_penalty` | 1.05 | 1.05 | 1.1 | 1.05 |

**Почему низкая температура для кодогенерации:** Код должен быть детерминированным. При temperature=0.7 модель может начать генерировать комментарии или объяснения вместо кода .

### Полный пример: агент анализа кодовой базы

```python
import os
from pathlib import Path

class CodeAnalyzer:
    def __init__(self, model, tokenizer):
        self.model = model
        self.tokenizer = tokenizer
    
    def chunk_code(self, file_path, max_chunk_size=1500):
        """Разбивает код на функции/классы"""
        with open(file_path, 'r') as f:
            content = f.read()
        
        # Простая эвристика: разбиваем по def/class
        chunks = []
        current_chunk = []
        lines = content.split('\n')
        
        for line in lines:
            current_chunk.append(line)
            if (line.strip().startswith(('def ', 'class ')) or 
                len('\n'.join(current_chunk)) > max_chunk_size):
                if current_chunk:
                    chunks.append('\n'.join(current_chunk))
                    current_chunk = []
        
        if current_chunk:
            chunks.append('\n'.join(current_chunk))
        
        return chunks
    
    def query_codebase(self, question, context_chunks):
        """Задаёт вопрос о кодовой базе на основе контекста"""
        context = "\n\n".join(context_chunks[:5])  # Ограничиваем контекст
        
        prompt = f"""You are analyzing a codebase. Use the following code context to answer the question.

Code context:
```python
{context}
```

Question: {question}

Provide a clear, specific answer based ONLY on the code provided."""

        messages = [
            {"role": "system", "content": "You are a code analysis expert. Answer based strictly on the provided code."},
            {"role": "user", "content": prompt}
        ]
        
        formatted = self.tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
        inputs = self.tokenizer(formatted, return_tensors="pt").to(self.model.device)
        
        outputs = self.model.generate(
            **inputs,
            max_new_tokens=512,
            temperature=0.3,
            do_sample=True,
            top_p=0.9
        )
        
        return self.tokenizer.decode(outputs[0][inputs.input_ids.shape[1]:], skip_special_tokens=True)
    
    def find_similar_functions(self, function_signature, codebase_path):
        """Находит функции, похожие на заданную сигнатуру"""
        all_functions = []
        
        for py_file in Path(codebase_path).rglob("*.py"):
            chunks = self.chunk_code(py_file)
            for chunk in chunks:
                if 'def ' in chunk:
                    all_functions.append(chunk)
        
        prompt = f"""Find functions similar to this signature:
{function_signature}

Among these functions from the codebase:
{chr(10).join(all_functions[:10])}

List the names of similar functions and explain why they are similar."""

        messages = [{"role": "user", "content": prompt}]
        formatted = self.tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
        inputs = self.tokenizer(formatted, return_tensors="pt").to(self.model.device)
        
        outputs = self.model.generate(**inputs, max_new_tokens=256, temperature=0.2)
        
        return self.tokenizer.decode(outputs[0][inputs.input_ids.shape[1]:], skip_special_tokens=True)

# Использование
analyzer = CodeAnalyzer(model, tokenizer)
result = analyzer.query_codebase(
    "What database operations are performed in this code?",
    ["import sqlite3", "conn.execute('SELECT * FROM users')"]
)
print(result)
```

### Ограничения Coder-модели

При всех преимуществах, 3B-модель имеет объективные ограничения :

| Задача | Ожидаемое качество |
|--------|---------------------|
| Простые алгоритмы (FizzBuzz, факториал) | Отлично |
| Средние алгоритмы (бинарный поиск, сортировка) | Хорошо |
| Сложные алгоритмы (графы, динамическое программирование) | Средне |
| Комплексная архитектура ПО | Плохо |
| Генерация production-кода без ревью | Не рекомендуется |

**Что модель не умеет:**
- Генерировать безопасный код для критических систем
- Принимать сложные архитектурные решения
- Работать с редкими языками программирования (кроме Python/JS/TS/Go/Rust/C++ в базовом виде) 

### Рекомендации

1. **Для повседневной кодогенерации** — используйте Qwen2.5-Coder-3B с temperature=0.0
2. **Для объяснения кода** — повысьте temperature до 0.3-0.5
3. **Для автодополнения в редакторе** — настройте FIM-формат через Ollama или локальный API
4. **Для production-проектов** — рассмотрите Coder-7B или 14B, если позволяет VRAM

Следующая глава покажет, как создавать встраивания (embeddings) для RAG-систем на основе Qwen2.5.

Скидывайте главу 19.



## 19. Основы тонкой настройки локальных моделей

### Что такое тонкая настройка и зачем она нужна

Базовая модель Qwen2.5-3B — это универсальный инструмент. Тонкая настройка (fine-tuning) превращает его в специалиста в конкретной области: отвечать на технические вопросы, генерировать код в определённом стиле, понимать отраслевую терминологию.

**Когда нужна тонкая настройка:**
- Модель не знает вашу внутреннюю терминологию (названия продуктов, API вашей компании)
- Нужен определённый формат ответов (JSON, XML, строгие шаблоны)
- Базовая модель путается в специфических сценариях (например, медицинские протоколы)

**Когда тонкая настройка не нужна:**
- Достаточно добавить примеры в промпт (few-shot)
- Задача общего характера (перевод, суммаризация)
- Нет данных для обучения (минимум 200-500 качественных примеров)

### LoRA: ресурсо-эффективный подход

Полная тонкая настройка 3B модели требует 12+ ГБ VRAM даже в 4-bit. LoRA (Low-Rank Adaptation) решает эту проблему — вместо изменения всех весов, она добавляет маленькие адаптеры (ранга 8-128) только к матрицам внимания.

**Что даёт LoRA для Qwen2.5-3B:**
| Параметр | Полная настройка | LoRA |
|----------|-----------------|------|
| VRAM (4-bit) | 10-12 GB | **4-6 GB** |
| VRAM (8-bit) | 16+ GB | **6-8 GB** |
| Время (3 эпохи, 5k примеров) | ~4 часа на A100 | **~2 часа на T4** |
| Размер сохранённых весов | 6 GB (FP16) | **~50 MB** |

**Цитата из документации Unsloth:** "Fine-tuning Qwen 2.5-3B Instruct with LoRA is optimized for low-cost GPUs like NVIDIA T4, using 4-bit quantization to reduce memory footprint" .

### Unsloth: самый быстрый путь для новичков

Unsloth — библиотека, которая ускоряет тонкую настройку в 2-5 раз по сравнению с стандартным Hugging Face и автоматически применяет оптимизации памяти.

**Установка:**
```bash
pip install unsloth
pip install xformers trl peft accelerate bitsandbytes
```

**Минимальный код для тонкой настройки Qwen2.5-3B:**

```python
from unsloth import FastLanguageModel
import torch
from trl import SFTTrainer
from transformers import TrainingArguments
from datasets import load_dataset

# 1. Загрузка модели с 4-bit квантизацией для экономии памяти
model, tokenizer = FastLanguageModel.from_pretrained(
    model_name="unsloth/Qwen2.5-3B-Instruct-unsloth-bnb-4bit",  # предварительно квантизованная версия
    max_seq_length=4096,
    dtype=None,  # автоопределение
    load_in_4bit=True,
)

# 2. Применение LoRA (дообучаем только 1-2% параметров)
model = FastLanguageModel.get_peft_model(
    model,
    r=32,                      # ранг LoRA (8-128, чем выше, тем точнее)
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj",
                    "gate_proj", "up_proj", "down_proj"],
    lora_alpha=32,             # масштабирующий коэффициент (обычно = r)
    lora_dropout=0.1,          # dropout для регуляризации
    bias="none",               # не обучаем bias
    use_gradient_checkpointing="unsloth",  # экономит память
    random_state=42,
)

# 3. Подготовка данных (формат Qwen chat)
dataset = load_dataset("json", data_files="my_data.json")

def format_chat(example):
    """Преобразование в формат Qwen с system/user/assistant ролями"""
    messages = [
        {"role": "system", "content": "You are a helpful coding assistant."},
        {"role": "user", "content": example["question"]},
        {"role": "assistant", "content": example["answer"]}
    ]
    return {"text": tokenizer.apply_chat_template(messages, tokenize=False)}

dataset = dataset.map(format_chat)

# 4. Настройка и запуск обучения
trainer = SFTTrainer(
    model=model,
    tokenizer=tokenizer,
    train_dataset=dataset["train"],
    dataset_text_field="text",
    max_seq_length=4096,
    args=TrainingArguments(
        per_device_train_batch_size=2,      # зависит от VRAM
        gradient_accumulation_steps=4,      # эффективный батч = 2*4=8
        warmup_steps=10,
        num_train_epochs=3,
        learning_rate=2e-4,
        fp16=not torch.cuda.is_bf16_supported(),
        bf16=torch.cuda.is_bf16_supported(),
        logging_steps=10,
        optim="adamw_8bit",
        weight_decay=0.01,
        lr_scheduler_type="linear",
        seed=42,
        output_dir="./qwen_finetuned",
        report_to="none",                   # отключаем wandb для простоты
    ),
)

trainer.train()

# 5. Сохранение
model.save_pretrained("qwen_finetuned_lora")
tokenizer.save_pretrained("qwen_finetuned_lora")
```

### Пример из практики: дообучение на структурированных данных

В проекте Trasgu-3B разработчики дообучали Qwen2.5-3B на парах "вход→выход" для диалекта испанского языка (льонский). Конфигурация:

```python
# Гиперпараметры из успешного проекта Trasgu-3B 
training_config = {
    "learning_rate": 2e-4,
    "batch_size": 4,
    "num_epochs": 3,
    "warmup_ratio": 0.1,
    "weight_decay": 0.01,
    "lr_scheduler": "linear",
    "max_seq_length": 4096,
    "max_grad_norm": 1.0,
    "optimizer": "paged_adamw_8bit",
    "precision": "fp16",  # или bf16 на новых GPU
}

# LoRA настройки 
lora_config = {
    "r": 128,                    # высокий ранг для сложной задачи
    "alpha": 256,               
    "dropout": 0.1,
    "target_modules": ["q_proj", "k_proj", "v_proj", "o_proj",
                       "gate_proj", "up_proj", "down_proj", 
                       "embed_tokens", "lm_head"],  # включая выходной слой
}
```

**Результаты после 3 эпох на 3000 примерах:**
| Метрика | Значение |
|---------|----------|
| mean_similarity (final) | 82.93% |
| mean_fuzzy (text match) | 64.04% |
| mean_semantic (embeddings) | 82.84% |

**Вывод:** Модель значительно улучшила способность генерировать правильные ответы в целевом формате .

### Продвинутые методы: DPO и GRPO

Если у вас есть предпочтения между ответами (какой лучше, какой хуже), используйте DPO (Direct Preference Optimization). Он обучает модель предпочитать хорошие ответы плохим без сложного RL-цикла .

**Пример DPO конфигурации (Qwen2.5-Taiwan):**
```python
# Из проекта Qwen2.5-Taiwan-3B-Instruct 
dpo_config = {
    "learning_rate": 1e-6,      # намного ниже чем для SFT
    "batch_size": 4,
    "num_epochs": 1,
    "warmup_ratio": 0.05,
    "weight_decay": 1e-5,
    "lr_scheduler": "cosine",
    "max_seq_length": 4096,
}
# Требования: ~3 часа на A100, результат — улучшение стиля ответов
```

**GRPO (Group Relative Policy Optimization)** — новейший метод для задач рассуждения (математика, логика). Он оптимизирует модель через reward-функции, оценивающие и формат, и правильность ответа .

### QLoRA: 4-bit базовая модель + LoRA

Стандарт де-факто для дообучения на слабых GPU — загрузить модель в 4-bit через `BitsAndBytesConfig`, а сверху добавить LoRA.

```python
from transformers import BitsAndBytesConfig

bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.bfloat16,
    bnb_4bit_use_double_quant=True,
)

model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    quantization_config=bnb_config,
    device_map="auto",
    trust_remote_code=False,
)

# Затем добавляем LoRA как обычно
```

**VRAM при QLoRA:** ~4-6 GB. Этого достаточно для T4 (Colab) и даже некоторых ноутбуков с дискретной графикой .

### Подготовка данных: критический этап

**Формат данных для Qwen2.5:**
```json
{
  "conversations": [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "What is Python?"},
    {"role": "assistant", "content": "Python is a programming language..."}
  ]
}
```

**Преобразование через apply_chat_template:**
```python
def format_dataset(example):
    text = tokenizer.apply_chat_template(
        example["conversations"],
        tokenize=False,
        add_generation_prompt=False  # для обучения не нужно
    )
    return {"text": text}
```

**Минимальные требования к датасету:**
| Размер датасета | Ожидаемый эффект |
|-----------------|------------------|
| 100-500 примеров | Слабый, лучше использовать few-shot |
| 500-2000 примеров | Модель начинает осваивать формат |
| 2000-10000 примеров | Хорошее усвоение стиля и фактов |
| 10000+ примеров | Риск переобучения, нужна регуляризация |

### Запуск в Google Colab (T4 GPU)

```python
# Установка в Colab
!pip install unsloth
!pip install xformers trl peft accelerate bitsandbytes

# Проверка GPU
!nvidia-smi
# T4: ~15 GB VRAM — достаточно для QLoRA + LoRA

# Весь код обучения из предыдущего раздела работает
# Для T4 рекомендуется batch_size=2, gradient_accumulation=4
```

### Nemo Framework: для production-scale

Для серьёзных проектов NVIDIA NeMo предоставляет готовые рецепты для Qwen2.5 :

```python
from nemo.collections import llm
import nemo_run as run

recipe = llm.qwen2_500m.finetune_recipe(
    name="qwen_finetuning",
    dir="/path/to/checkpoints",
    num_nodes=1,
    num_gpus_per_node=8,
    peft_scheme='lora',        # 'lora' или 'none' для полной настройки
    packed_sequence=False,     # True для упаковки последовательностей
)

# Запуск
run.run(recipe, executor=run.LocalExecutor())
```

### После тонкой настройки

Сохранённые LoRA веса занимают ~50 MB. Загрузка обученной модели:

```python
from peft import PeftModel

# Загружаем базовую модель
base_model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    device_map="auto"
)

# Загружаем LoRA адаптер
model = PeftModel.from_pretrained(base_model, "./qwen_finetuned_lora")
```

### Типичные проблемы и решения

| Проблема | Решение |
|----------|---------|
| OOM при обучении | Уменьшить `batch_size`, увеличить `gradient_accumulation_steps`  |
| Модель не обучается | Проверить формат данных и `apply_chat_template` |
| Переобучение (loss падает, но качество на тесте не растёт) | Увеличить `lora_dropout`, уменьшить `num_epochs`, добавить `weight_decay` |
| Медленное обучение (1-2 steps/sec) | Включить `flash_attention_2`, использовать Unsloth вместо стандартного тренажёра |

**Ключевой вывод для Qwen2.5-3B:** Для большинства задач оптимальный выбор — QLoRA (4-bit модель + LoRA) с Unsloth. Это позволяет дообучать модель на обычном игровом GPU или в бесплатном Colab с сохранением 95%+ качества полной тонкой настройки.

Следующая глава покажет, как оценивать качество дообученной модели и проводить A/B тестирование.

Скидывайте главу 20.



## 20. Работа с Vision-Language моделями (Qwen2.5-VL)

### Что такое Qwen2.5-VL и чем отличается от обычного Qwen

Qwen2.5-VL — мультимодальная версия модели, способная одновременно обрабатывать текст и изображения (а также видео). Выпущена в версиях 3B, 7B и 72B параметров, обучена на **4.1 трлн токенов**.

**Ключевые отличия от текстовой версии:**
- Нативная поддержка изображений и видео
- **MRoPE** (Multi-resolutional Rotary Positional Embedding) — улучшенный механизм позиционного кодирования для работы с визуальными данными
- Поддержка динамического FPS для видео
- Возможность работы как визуальный агент (работа с интерфейсами, использование инструментов)

**Зачем нужна Vision-модель, если можно конвертировать изображение в текст?** OCR теряет контекст, пространственные отношения, цвета, эмоции. Qwen2.5-VL понимает изображение как целостную картину.

### Загрузка модели (упрощённый способ)

Для Qwen2.5-VL требуется **transformers ≥ 4.45.0**. Обновите перед началом:

```bash
pip install --upgrade transformers accelerate
pip install qwen-vl-utils[decord]==0.0.8  # для поддержки видео
```

**Минимальный код для работы с изображением:**

```python
from transformers import Qwen2_5_VLForConditionalGeneration, AutoProcessor
from qwen_vl_utils import process_vision_info
import torch

# Загрузка модели (3B версия)
model = Qwen2_5_VLForConditionalGeneration.from_pretrained(
    "Qwen/Qwen2.5-VL-3B-Instruct",
    torch_dtype=torch.bfloat16,  # или torch.float16
    device_map="auto",
    attn_implementation="sdpa"   # или "flash_attention_2" для ускорения
)

processor = AutoProcessor.from_pretrained("Qwen/Qwen2.5-VL-3B-Instruct")

# Подготовка сообщения с изображением
messages = [
    {
        "role": "user",
        "content": [
            {"type": "image", "image": "https://example.com/image.jpg"},
            {"type": "text", "text": "Что изображено на этой картинке?"}
        ]
    }
]

# Применение шаблона и подготовка входных данных
text = processor.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
image_inputs, video_inputs = process_vision_info(messages)
inputs = processor(
    text=[text],
    images=image_inputs,
    videos=video_inputs,
    padding=True,
    return_tensors="pt"
).to(model.device)

# Генерация
with torch.no_grad():
    generated_ids = model.generate(**inputs, max_new_tokens=256)

# Декодирование (обрезаем входную часть)
generated_ids_trimmed = [
    out_ids[len(in_ids):] for in_ids, out_ids in zip(inputs.input_ids, generated_ids)
]
output_text = processor.batch_decode(
    generated_ids_trimmed, 
    skip_special_tokens=True, 
    clean_up_tokenization_spaces=False
)
print(output_text[0])
```

### Использование pipeline (ещё проще)

```python
from transformers import pipeline

pipe = pipeline(
    task="image-text-to-text",
    model="Qwen/Qwen2.5-VL-3B-Instruct",
    device=0  # или "cpu"
)

messages = [
    {
        "role": "user",
        "content": [
            {"type": "image", "url": "https://example.com/cat.jpg"},
            {"type": "text", "text": "Describe this image"}
        ]
    }
]

result = pipe(messages, max_new_tokens=128, return_full_text=False)
print(result[0]['generated_text'])
```

### Настройка разрешения изображений

Важный параметр для баланса качества и производительности — контроль количества визуальных токенов:

```python
# Формула: пиксели = число_токенов * 28 * 28
# Каждый токен "видит" область 28x28 пикселей

# Минимум — слишком низкое качество
min_pixels = 256 * 28 * 28    # ~200K пикселей, 256 токенов

# Максимум — много памяти, но высокое качество  
max_pixels = 1280 * 28 * 28   # ~1M пикселей, 1280 токенов

processor = AutoProcessor.from_pretrained(
    "Qwen/Qwen2.5-VL-3B-Instruct",
    min_pixels=min_pixels,
    max_pixels=max_pixels
)
```

**Рекомендации по разрешению:**

| Сценарий | min_pixels | max_pixels | Токенов | VRAM |
|----------|------------|------------|---------|------|
| Быстрый тест | 128*28*28 | 256*28*28 | 128-256 | ~2 GB |
| Стандартный | 256*28*28 | 640*28*28 | 256-640 | ~3 GB |
| Высокое качество | 256*28*28 | 1280*28*28 | 256-1280 | ~4-5 GB |

### Работа с видео (потоковая обработка кадров)

Qwen2.5-VL поддерживает видео через параметр `fps` (кадров в секунду):

```python
messages = [
    {
        "role": "user",
        "content": [
            {"type": "video", "path": "/path/to/video.mp4"},
            {"type": "text", "text": "What happens in this video?"}
        ]
    }
]

# Ключевой параметр — fps
inputs = processor.apply_chat_template(
    messages,
    fps=1.0,                     # сколько кадров в секунду извлекать
    add_generation_prompt=True,
    tokenize=True,
    return_dict=True,
    return_tensors="pt"
).to(model.device)

output_ids = model.generate(**inputs, max_new_tokens=256)
```

**Совет по видео:** Для 3B модели на одном GPU с 8-12 GB VRAM используйте `fps=0.5-1`. Более высокий FPS даёт больше деталей, но быстро заполняет память.

### Квантизация для экономии памяти

**Способ 1: bitsandbytes 4-bit (как и для текстовых моделей)**

```python
from transformers import BitsAndBytesConfig

bnb_config = BitsAndBytesConfig(load_in_4bit=True)

model = Qwen2_5_VLForConditionalGeneration.from_pretrained(
    "Qwen/Qwen2.5-VL-3B-Instruct",
    quantization_config=bnb_config,
    device_map="auto"
)
```

**Способ 2: GPTQ (готовая квантизованная версия)**

```python
# Существуют готовые GPTQ-квантизации
model = Qwen2_5_VLForConditionalGeneration.from_pretrained(
    "hfl/Qwen2.5-VL-3B-Instruct-GPTQ-Int4",  # или Int3
    attn_implementation="flash_attention_2",
    device_map="auto"
)
```

**Способ 3: AWQ (для vLLM сервера)**

```bash
vllm serve Qwen/Qwen2.5-VL-3B-Instruct-AWQ \
    --port=8000 \
    --quantization=awq
```

**Сравнение методов для 3B VL модели:**

| Метод | VRAM | Качество (от FP16) | Совместимость |
|-------|------|-------------------|---------------|
| FP16 (baseline) | ~6-7 GB | 100% | transformers |
| 4-bit (bitsandbytes) | ~2.5-3 GB | 96-97% | transformers |
| GPTQ-Int4 | ~2.5 GB | 96-97% | transformers + optimum |
| GPTQ-Int3 | ~2 GB | ~93-94% | gptqmodel |
| AWQ | ~2.5 GB | 96-97% | vLLM |

### API сервер с vLLM (OpenAI-совместимый)

Самый производительный способ деплоя для продуктивных сред:

```bash
# Запуск сервера
vllm serve Qwen/Qwen2.5-VL-3B-Instruct \
    --host 0.0.0.0 \
    --port 8000 \
    --dtype bfloat16 \
    --max-model-len 20000 \
    --limit-mm-per-prompt '{"image": 3, "video": 1}' \
    --mm-processor-kwargs '{"max_pixels": 65536, "fps": 1}' \
    --gpu-memory-utilization 0.8
```

**Использование через OpenAI SDK:**

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:8000/v1",
    api_key="dummy"
)

response = client.chat.completions.create(
    model="Qwen/Qwen2.5-VL-3B-Instruct",
    messages=[{
        "role": "user",
        "content": [
            {"type": "image_url", "image_url": {"url": "https://example.com/photo.jpg"}},
            {"type": "text", "text": "Describe this image"}
        ]
    }],
    max_tokens=256
)
```

### Специализированные версии модели

**Qwen2.5-VL-3B-Geo** — дообученная версия для географических/картографических задач

**remote-sensing-Qwen2.5-VL-3B-Instruct** — для анализа спутниковых снимков и дистанционного зондирования Земли

Пример загрузки дообученной модели:
```python
from transformers import AutoModelForImageTextToText
from peft import PeftModel

base_model = AutoModelForImageTextToText.from_pretrained(
    "Qwen/Qwen2.5-VL-3B-Instruct",
    device_map='auto',
    torch_dtype=torch.bfloat16
)

model = PeftModel.from_pretrained(base_model, "kxxinDave/Qwen2.5-VL-instruct-3B-Geo")
```

### Альтернативный бэкенд: OpenVINO (для CPU)

Если GPU нет, используйте OpenVINO для ускорения на CPU:

```bash
# Конвертация в OpenVINO IR с INT4 квантизацией
optimum-cli export openvino --model Qwen/Qwen2.5-VL-3B-Instruct --weight-format int4 ov_model
```

```python
from optimum.intel.openvino import OVModelForVisualCausalLM

model = OVModelForVisualCausalLM.from_pretrained("./ov_model", device="CPU")
# Дальше API тот же, что и у transformers
```

### Типичные проблемы и решения

**Проблема:** `KeyError: 'qwen2_5_vl'`  
**Решение:** Обновите transformers до последней версии:
```bash
pip install git+https://github.com/huggingface/transformers accelerate
```

**Проблема:** Модель даёт бессвязный ответ, не связанный с изображением  
**Причина:** Неправильная настройка `process_vision_info()` или отсутствие передачи `image_inputs`  
**Решение:** Всегда используйте `image_inputs, video_inputs = process_vision_info(messages)` и передавайте оба в processor.

**Проблема:** OOM (Out of Memory) при работе с несколькими изображениями  
**Решение:** Уменьшите `max_pixels` или используйте квантизацию:
```python
processor = AutoProcessor.from_pretrained(
    "Qwen/Qwen2.5-VL-3B-Instruct",
    min_pixels=128*28*28,
    max_pixels=512*28*28
)
```

**Проблема:** Медленный инференс с видео  
**Решение:** Уменьшите FPS или используйте vLLM сервер с оптимизациями.

### Выбор между 3B и 7B версией

| Критерий | 3B | 7B |
|----------|----|----|
| VRAM (FP16) | ~6 GB | ~14 GB |
| VRAM (4-bit) | ~2.5 GB | ~5 GB |
| Скорость | Быстрее (~1.5-2x) | Медленнее |
| Качество распознавания | Хорошее | Отличное |
| Поддержка видео | Ограничена (2-3 кадра/сек) | Хорошая |
| Рекомендация | Edge-устройства, бюджетные GPU | Серверы, высокие требования |

**Для большинства локальных задач 3B достаточно.** По тестам Qwen2.5-VL-3B даже превосходит предыдущую 7B версию Qwen2-VL-7B.

### Полный пример: анализ изображения с потоковым выводом

```python
from transformers import Qwen2_5_VLForConditionalGeneration, AutoProcessor, TextStreamer
from qwen_vl_utils import process_vision_info
import torch

model = Qwen2_5_VLForConditionalGeneration.from_pretrained(
    "Qwen/Qwen2.5-VL-3B-Instruct",
    torch_dtype=torch.bfloat16,
    device_map="auto"
)
processor = AutoProcessor.from_pretrained("Qwen/Qwen2.5-VL-3B-Instruct")

messages = [{
    "role": "user",
    "content": [
        {"type": "image", "image": "https://example.com/dog.jpg"},
        {"type": "text", "text": "What breed of dog is this? Describe its features."}
    ]
}]

text = processor.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
image_inputs, video_inputs = process_vision_info(messages)
inputs = processor(
    text=[text],
    images=image_inputs,
    videos=video_inputs,
    padding=True,
    return_tensors="pt"
).to(model.device)

# Потоковый вывод
streamer = TextStreamer(processor.tokenizer, skip_prompt=True, skip_special_tokens=True)

with torch.no_grad():
    _ = model.generate(
        **inputs,
        max_new_tokens=256,
        temperature=0.7,
        streamer=streamer
    )
```

Следующая глава покажет, как деплоить Qwen2.5 в production с мониторингом, масштабированием и отказоустойчивостью.

Скидывайте главу 21.



## 21. Обработка изображений и видео входов

### Принцип работы: динамическое масштабирование

Qwen2.5-VL использует динамическую обработку визуальных данных, где изображения и видео кадры преобразуются в последовательность визуальных токенов. Ключевой параметр — **размер патча 28x28 пикселей**, где каждый патч соответствует одному визуальному токену после обработки.

**Математика преобразования:**
```
Размер_изображения = число_токенов * 28 * 28
```

Например, `min_pixels = 256 * 28 * 28` означает: минимальное количество визуальных токенов — 256 (изображение не менее ~7168x7168 пикселей после масштабирования).

### Константы обработки

Система использует строгие константы для контроля качества и памяти :

| Константа | Значение | Назначение |
|-----------|----------|-------------|
| `IMAGE_FACTOR` | 28 | Размеры изображений должны быть кратны 28 |
| `MIN_PIXELS` | 4 × 28 × 28 | Минимум пикселей (~3136) |
| `MAX_PIXELS` | 16,384 × 28 × 28 | Максимум пикселей (~12.8 млн) |
| `MAX_RATIO` | 200 | Максимальное соотношение сторон |
| `VIDEO_MIN_PIXELS` | 128 × 28 × 28 | Минимум для видео |
| `VIDEO_MAX_PIXELS` | 768 × 28 × 28 | Максимум для видео |
| `FPS` (по умолчанию) | 2.0 | Кадров в секунду для видео |
| `FRAME_FACTOR` | 2 | Округление числа кадров |

**Важное замечание:** Итоговые размеры всегда округляются до числа, кратного 28, с сохранением исходного соотношения сторон и соблюдением границ MIN_PIXELS/MAX_PIXELS .

### Настройка min_pixels и max_pixels

Это главные параметры баланса между качеством и потреблением памяти .

```python
from transformers import AutoProcessor

# Производительность (экономия памяти)
processor_fast = AutoProcessor.from_pretrained(
    "Qwen/Qwen2.5-VL-3B-Instruct",
    min_pixels=128 * 28 * 28,   # ~100K пикселей, 128 токенов
    max_pixels=256 * 28 * 28    # ~200K пикселей, 256 токенов
)

# Качество (больше деталей)
processor_quality = AutoProcessor.from_pretrained(
    "Qwen/Qwen2.5-VL-3B-Instruct",
    min_pixels=256 * 28 * 28,   # ~200K пикселей
    max_pixels=1280 * 28 * 28   # ~1M пикселей, 1280 токенов
)

# Стандартный баланс
processor = AutoProcessor.from_pretrained("Qwen/Qwen2.5-VL-3B-Instruct")
# min_pixels=4*28*28 (~3136), max_pixels=16384*28*28 (~12.8M) 
```

**Рекомендации по выбору:**

| Сценарий | min_pixels | max_pixels | Виз. токенов | VRAM (3B) |
|----------|------------|------------|--------------|-----------|
| Текст + иконки | 64×28×28 | 128×28×28 | 64-128 | ~2 GB |
| Стандартные фото | 128×28×28 | 640×28×28 | 128-640 | ~3 GB |
| Документы, текст | 256×28×28 | 1280×28×28 | 256-1280 | ~4-5 GB |
| Высокое качество | 256×28×28 | 4096×28×28 | 256-4096 | ~6-8 GB |

### Обработка видео: FPS и ограничение кадров

Для видео ключевые параметры — `fps` (кадров в секунду) и ограничение общего числа кадров через `nframes` .

**Базовый пример:**
```python
messages = [
    {
        "role": "user",
        "content": [
            {"type": "video", "video": "/path/to/video.mp4"},
            {"type": "text", "text": "Describe this video"}
        ]
    }
]

# Через qwen_vl_utils
text = processor.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
image_inputs, video_inputs = process_vision_info(messages)
inputs = processor(
    text=[text],
    images=image_inputs,
    videos=video_inputs,
    padding=True,
    return_tensors="pt"
).to(model.device)
```

**Контроль кадров через параметры:**

```python
# В messages можно передать дополнительные параметры
messages = [
    {
        "role": "user",
        "content": [
            {
                "type": "video",
                "video": "video.mp4",
                "fps": 1.0,                    # 1 кадр в секунду
                "nframes": 32,                  # максимум 32 кадра
                "total_pixels": 768 * 28 * 28,  # ограничение пикселей на кадр
                "min_pixels": 128 * 28 * 28,
                "max_pixels": 768 * 28 * 28,
            },
            {"type": "text", "text": "Describe the action"}
        ]
    }
]
```

### Практический код: полный пайплайн

```python
from transformers import Qwen2_5_VLForConditionalGeneration, AutoProcessor
from qwen_vl_utils import process_vision_info
import torch

# 1. Загрузка модели (с flash_attention_2 для ускорения)
model = Qwen2_5_VLForConditionalGeneration.from_pretrained(
    "Qwen/Qwen2.5-VL-3B-Instruct",
    torch_dtype=torch.bfloat16,          # экономия памяти
    attn_implementation="flash_attention_2",  # ускорение для видео 
    device_map="auto"
)

# 2. Настройка процессора под ваши задачи
processor = AutoProcessor.from_pretrained(
    "Qwen/Qwen2.5-VL-3B-Instruct",
    min_pixels=256 * 28 * 28,    # достаточно для большинства задач
    max_pixels=1280 * 28 * 28    # баланс качества и памяти
)

# 3. Формирование сообщения
messages = [
    {
        "role": "user",
        "content": [
            {"type": "image", "image": "https://example.com/photo.jpg"},
            {"type": "text", "text": "Что изображено на этом фото?"}
        ]
    }
]

# 4. Подготовка входных данных
text = processor.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
image_inputs, video_inputs = process_vision_info(messages)
inputs = processor(
    text=[text],
    images=image_inputs,
    videos=video_inputs,
    padding=True,
    return_tensors="pt"
).to(model.device)

# 5. Генерация
with torch.no_grad():
    generated_ids = model.generate(**inputs, max_new_tokens=256)

# 6. Декодирование ответа
generated_ids_trimmed = [
    out_ids[len(in_ids):] for in_ids, out_ids in zip(inputs.input_ids, generated_ids)
]
output_text = processor.batch_decode(
    generated_ids_trimmed,
    skip_special_tokens=True,
    clean_up_tokenization_spaces=False
)
print(output_text[0])
```

### Батчевая обработка с паддингом

**Критическое замечание:** При батчевой обработке нескольких изображений необходимо установить `padding_side="left"`, иначе генерация может быть обрезана или содержать артефакты .

```python
# Корректная настройка для батча
processor = AutoProcessor.from_pretrained("Qwen/Qwen2.5-VL-3B-Instruct")
processor.tokenizer.padding_side = "left"  # обязательно для батчевой генерации!

# Разные сообщения для батча
messages_batch = [
    [{"role": "user", "content": [{"type": "image", "image": "img1.jpg"}, {"type": "text", "text": "Describe"}]}],
    [{"role": "user", "content": [{"type": "image", "image": "img2.jpg"}, {"type": "text", "text": "What is this?"}]}]
]

texts = [processor.apply_chat_template(msg, tokenize=False, add_generation_prompt=True) for msg in messages_batch]
image_inputs, video_inputs = process_vision_info(messages_batch)

inputs = processor(
    text=texts,
    images=image_inputs,
    videos=video_inputs,
    padding=True,          # автоматический паддинг до max длины в батче
    return_tensors="pt"
).to(model.device)

# Генерация для всего батча
generated_ids = model.generate(**inputs, max_new_tokens=128)
```

**Почему `padding_side="left"`?** При генерации модель добавляет токены справа от входной последовательности. Если паддинг справа, модель видит пустые токены перед началом реального контента, что нарушает позиционное кодирование .

### vLLM для высокопроизводительного инференса

Для production-сценариев с видео используйте vLLM :

```python
from vllm import LLM, SamplingParams
from transformers import AutoProcessor
from qwen_vl_utils import process_vision_info

MODEL_PATH = "Qwen/Qwen2.5-VL-3B-Instruct"

# Оптимизация для видео
llm = LLM(
    model=MODEL_PATH,
    gpu_memory_utilization=0.8,
    enforce_eager=True,                    # решение проблем с CUDA graph
    limit_mm_per_prompt={"video": 1},      # ограничение на количество видео
    mm_processor_kwargs={
        "max_pixels": 768 * 768,           # ограничение разрешения
        "nframes": 8,                      # ограничение кадров
        "fps": 1                           # кадров в секунду
    }
)

sampling_params = SamplingParams(
    temperature=0.1,
    max_tokens=1024,
    repetition_penalty=1.05
)

# Формирование запроса с видео
messages = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": [
        {"type": "text", "text": "Describe this video"},
        {"type": "video", "video": "/path/to/video.mp4"}
    ]}
]

processor = AutoProcessor.from_pretrained(MODEL_PATH)
prompt = processor.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
image_inputs, video_inputs, video_kwargs = process_vision_info(messages, return_video_kwargs=True)

llm_inputs = {
    "prompt": prompt,
    "multi_modal_data": {"video": video_inputs},
    "mm_processor_kwargs": video_kwargs
}

outputs = llm.generate([llm_inputs], sampling_params=sampling_params)
```

### Flash Attention 2: когда включать

```python
# Включение для ускорения и экономии памяти
model = Qwen2_5_VLForConditionalGeneration.from_pretrained(
    "Qwen/Qwen2.5-VL-3B-Instruct",
    torch_dtype=torch.bfloat16,
    attn_implementation="flash_attention_2",
    device_map="auto"
)
```

**Когда использовать:**
- ✅ Мульти-изображения в одном запросе
- ✅ Видео с большим числом кадров
- ✅ Длинные контексты (>4096 токенов)
- ❌ Не работает на старых GPU без поддержки (Ampere+)

По данным тестов, Flash Attention 2 даёт ускорение до 2-3x для видео .

### Выбор бэкенда для видео

Система автоматически выбирает бэкенд в порядке приоритета :

1. **torchcodec** — самый быстрый, если доступен
2. **decord** — рекомендуется установить через `pip install qwen-vl-utils[decord]`
3. **torchvision** — резервный вариант

```bash
# Установка с поддержкой decord (рекомендуется)
pip install qwen-vl-utils[decord]==0.0.8

# Если вы не на Linux (decord может не установиться)
pip install qwen-vl-utils
# тогда будет использоваться torchvision
```

### Решение проблем

**Проблема: `KeyError: 'qwen2_5_vl'`**

Обновите transformers до последней версии:
```bash
pip install git+https://github.com/huggingface/transformers accelerate
```

**Проблема: Нехватка памяти при видео**

```python
# Решение: ограничить разрешение и кадры
processor = AutoProcessor.from_pretrained(
    "Qwen/Qwen2.5-VL-3B-Instruct",
    min_pixels=128 * 28 * 28,
    max_pixels=512 * 28 * 28   # уменьшаем до ~500K пикселей
)

# В messages для видео:
{"type": "video", "video": "video.mp4", "nframes": 16, "fps": 0.5}
```

**Проблема: Артефакты при батчевой обработке**

Убедитесь, что установлен `processor.tokenizer.padding_side = "left"` . Проблема особенно заметна при смешивании сообщений разной длины.

**Проблема: Параметры min_pixels/max_pixels не применяются**

Проверьте, что параметры передаются при инициализации процессора :

```python
# Правильно
processor = AutoProcessor.from_pretrained(
    "Qwen/Qwen2.5-VL-3B-Instruct",
    min_pixels=256 * 28 * 28,
    max_pixels=1280 * 28 * 28
)

# Неправильно (параметры будут проигнорированы)
processor = AutoProcessor.from_pretrained("Qwen/Qwen2.5-VL-3B-Instruct")
processor.min_pixels = 256 * 28 * 28  # не сработает
```

### Чек-лист для эффективной работы

- [ ] Установлен `qwen-vl-utils[decord]` для видео
- [ ] Transformers обновлён до версии ≥4.45.0
- [ ] Для батчей: `processor.tokenizer.padding_side = "left"`
- [ ] Для видео: заданы `fps` или `nframes` ограничения
- [ ] Flash Attention 2 включён при работе с мульти-изображениями
- [ ] `min_pixels` и `max_pixels` настроены под ваше железо

Следующая глава покажет, как создавать специализированные модели для задач вроде распознавания таблиц и анализа документов.

Скидывайте главу 22.



## 22. Обработка ошибок и отладка типичных проблем

### Классификация ошибок: от частых к редким

Все ошибки при работе с Qwen2.5-3B делятся на четыре категории. Ниже — полный разбор каждой с готовыми решениями.

### Категория 1: Память и устройства

**Ошибка 1: CUDA out of memory**

Самая частая проблема. Возникает, когда модель + KV-кэш не влезают в VRAM.

```python
# Диагностика
import torch
print(f"VRAM allocated: {torch.cuda.memory_allocated() / 1e9:.2f} GB")
print(f"VRAM reserved: {torch.cuda.memory_reserved() / 1e9:.2f} GB")
print(f"Max VRAM: {torch.cuda.get_device_properties(0).total_memory / 1e9:.2f} GB")
```

**Решения (в порядке эффективности):**

1. Включить 4-битную квантизацию:
```python
from transformers import BitsAndBytesConfig
bnb_config = BitsAndBytesConfig(load_in_4bit=True)
model = AutoModelForCausalLM.from_pretrained(..., quantization_config=bnb_config)
```

2. Уменьшить `max_new_tokens`:
```python
outputs = model.generate(..., max_new_tokens=256)  # вместо 1024
```

3. Использовать `device_map="auto"` с оффлоадингом:
```python
model = AutoModelForCausalLM.from_pretrained(..., device_map="auto", offload_folder="./offload")
```

4. Очистить кэш перед генерацией:
```python
torch.cuda.empty_cache()
```

**Ошибка 2: Expected all tensors to be on the same device**

Модель на GPU, а входные данные на CPU (или наоборот).

```python
# Решение: явно перемещаем тензоры
inputs = tokenizer(prompt, return_tensors="pt").to(model.device)
# Или: inputs = {k: v.to(model.device) for k, v in inputs.items()}
```

**Ошибка 3: device_map="auto" игнорируется при quantization_config**

```python
# Неправильно
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    quantization_config=bnb_config,
    device_map="auto"  # может игнорироваться
)

# Правильно: device_map передаётся после quantization_config
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    quantization_config=bnb_config,
    device_map="auto"
)
# Или используйте accelerate: model = accelerator.prepare(model)
```

### Категория 2: Токенизация и шаблоны

**Ошибка 4: `KeyError: 'qwen2'` при загрузке токенизатора**

```python
# Причина: устаревший transformers
# Решение:
pip install --upgrade transformers
```

**Ошибка 5: pad_token_id не установлен**

```python
# Симптом: предупреждение при генерации
# Решение:
if tokenizer.pad_token is None:
    tokenizer.pad_token = tokenizer.eos_token  # <|endoftext|>
```

**Ошибка 6: apply_chat_template возвращает странные символы**

```python
# Причина: несоответствие версии
# Диагностика:
print(tokenizer.chat_template)

# Решение: обновить transformers или использовать ручной шаблон
manual_template = "<|im_start|>user\n{user}<|im_end|>\n<|im_start|>assistant\n"
```

**Ошибка 7: Модель повторяет входной промпт в ответе**

```python
# Причина: неправильное декодирование
# Решение: обрезать входную часть
output_text = tokenizer.decode(outputs[0][inputs.input_ids.shape[1]:], skip_special_tokens=True)
```

### Категория 3: GGUF и llama-cpp-python

**Ошибка 8: `FileNotFoundError` при загрузке .gguf**

```python
# Причина: неверный путь или файл не скачан
# Решение 1: скачать вручную
from huggingface_hub import hf_hub_download
model_path = hf_hub_download(
    repo_id="lmstudio-community/Qwen2.5-3B-Instruct-GGUF",
    filename="Qwen2.5-3B-Instruct-Q4_K_M.gguf"
)

# Решение 2: использовать from_pretrained
from llama_cpp import Llama
llm = Llama.from_pretrained(
    repo_id="lmstudio-community/Qwen2.5-3B-Instruct-GGUF",
    filename="Qwen2.5-3B-Instruct-Q4_K_M.gguf"
)
```

**Ошибка 9: Ошибка компиляции llama-cpp-python на Windows**

```bash
# Решение: установить предварительно собранную версию
pip install llama-cpp-python --extra-index-url https://abetlen.github.io/llama-cpp-python/whl/cpu
# Для GPU:
pip install llama-cpp-python --extra-index-url https://abetlen.github.io/llama-cpp-python/whl/cu121
```

**Ошибка 10: Медленный инференс на CPU с GGUF**

```python
# Решение: включить BLAS оптимизации
llm = Llama(
    model_path="./model.gguf",
    n_threads=8,           # количество CPU потоков
    n_gpu_layers=20,       # часть слоёв на GPU (если есть)
    n_batch=512            # размер батча для обработки
)
```

### Категория 4: Генерация и качество

**Ошибка 11: Бессмысленный или повторяющийся текст**

```python
# Решение: настроить repetition_penalty и temperature
outputs = model.generate(
    **inputs,
    repetition_penalty=1.1,   # штраф за повторы
    temperature=0.7,          # не слишком высоко
    do_sample=True,           # включить семплирование
    top_p=0.9
)
```

**Ошибка 12: Модель не следует формату инструкции**

```python
# Причина: неправильный шаблон чата
# Решение: всегда использовать apply_chat_template
prompt = tokenizer.apply_chat_template(
    messages, 
    tokenize=False, 
    add_generation_prompt=True
)
```

**Ошибка 13: Генерация обрывается на полуслове**

```python
# Причина: достигнут max_new_tokens или встречен eos
# Решение: увеличить max_new_tokens или убрать early_stopping
outputs = model.generate(
    **inputs,
    max_new_tokens=512,      # больше
    early_stopping=False     # не останавливаться досрочно
)
```

### Полная функция отладки

```python
def diagnose_model(model, tokenizer):
    """Диагностика всех компонентов перед генерацией"""
    issues = []
    
    # 1. Проверка устройства
    device = next(model.parameters()).device
    print(f"Model device: {device}")
    
    # 2. Проверка pad_token
    if tokenizer.pad_token is None:
        issues.append("pad_token не установлен")
        tokenizer.pad_token = tokenizer.eos_token
        print("Исправлено: pad_token = eos_token")
    
    # 3. Проверка памяти
    if torch.cuda.is_available():
        mem_alloc = torch.cuda.memory_allocated() / 1e9
        mem_total = torch.cuda.get_device_properties(0).total_memory / 1e9
        print(f"VRAM: {mem_alloc:.2f} / {mem_total:.2f} GB")
        if mem_alloc / mem_total > 0.9:
            issues.append("VRAM почти заполнена, возможны OOM")
    
    # 4. Проверка шаблона чата
    try:
        test_messages = [{"role": "user", "content": "test"}]
        template = tokenizer.apply_chat_template(test_messages, tokenize=False)
        if "<|im_start|>" not in template:
            issues.append("chat_template может быть устаревшим")
    except Exception as e:
        issues.append(f"Ошибка apply_chat_template: {e}")
    
    # 5. Проверка dtype
    dtype = next(model.parameters()).dtype
    print(f"Model dtype: {dtype}")
    if dtype == torch.float32 and torch.cuda.is_available():
        issues.append("Используется FP32 на GPU, рекомендуется FP16")
    
    # 6. Тестовая генерация
    try:
        test_inputs = tokenizer("test", return_tensors="pt").to(device)
        with torch.no_grad():
            _ = model.generate(**test_inputs, max_new_tokens=5)
        print("Тестовая генерация успешна")
    except Exception as e:
        issues.append(f"Ошибка генерации: {e}")
    
    return issues

# Использование
issues = diagnose_model(model, tokenizer)
if issues:
    print("\nНайденные проблемы:")
    for issue in issues:
        print(f"  - {issue}")
```

### Логирование ошибок для production

```python
import logging
import traceback
from datetime import datetime

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler(f"qwen_errors_{datetime.now():%Y%m%d}.log"),
        logging.StreamHandler()
    ]
)
logger = logging.getLogger("qwen_local")

def safe_generate(model, tokenizer, prompt, **gen_kwargs):
    """Безопасная генерация с логированием ошибок"""
    try:
        logger.info(f"Генерация с параметрами: {gen_kwargs}")
        
        inputs = tokenizer(prompt, return_tensors="pt").to(model.device)
        
        with torch.no_grad():
            outputs = model.generate(**inputs, **gen_kwargs)
        
        response = tokenizer.decode(outputs[0][inputs.input_ids.shape[1]:], skip_special_tokens=True)
        logger.info(f"Успешная генерация, длина: {len(response)} символов")
        return response
        
    except RuntimeError as e:
        if "out of memory" in str(e):
            logger.error(f"OOM ошибка: {e}")
            # Автоматическое восстановление
            torch.cuda.empty_cache()
            return "Ошибка: недостаточно памяти. Попробуйте уменьшить max_new_tokens."
        else:
            logger.error(f"RuntimeError: {traceback.format_exc()}")
            raise
            
    except Exception as e:
        logger.error(f"Неожиданная ошибка: {traceback.format_exc()}")
        raise
```

### Чек-лист перед запуском

- [ ] `pip freeze | grep transformers` ≥ 4.37.0
- [ ] `torch.cuda.is_available()` → True (если есть GPU)
- [ ] `tokenizer.pad_token_id` установлен
- [ ] Для GGUF: файл .gguf существует и не повреждён
- [ ] Для VL: установлен `qwen-vl-utils`
- [ ] `max_new_tokens` не превышает лимиты модели
- [ ] При батчевой генерации: `padding_side="left"`

### Быстрое решение: минимальная конфигурация, которая всегда работает

```python
from transformers import AutoTokenizer, AutoModelForCausalLM, BitsAndBytesConfig
import torch

# Самая надёжная конфигурация для слабых GPU
bnb_config = BitsAndBytesConfig(load_in_4bit=True)
tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen2.5-3B-Instruct")
if tokenizer.pad_token is None:
    tokenizer.pad_token = tokenizer.eos_token

model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    quantization_config=bnb_config,
    device_map="auto"
)

def safe_chat(user_input):
    messages = [{"role": "user", "content": user_input}]
    prompt = tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
    inputs = tokenizer(prompt, return_tensors="pt").to(model.device)
    
    try:
        with torch.no_grad():
            outputs = model.generate(
                **inputs,
                max_new_tokens=256,
                do_sample=True,
                temperature=0.7,
                repetition_penalty=1.1
            )
        return tokenizer.decode(outputs[0][inputs.input_ids.shape[1]:], skip_special_tokens=True)
    except RuntimeError as e:
        if "out of memory" in str(e):
            torch.cuda.empty_cache()
            return "Ошибка: переполнение памяти. Перезапустите ядро."
        raise e
```

Эта конфигурация работает на любом GPU с 4+ ГБ VRAM и на CPU с 8+ ГБ RAM (медленно).

Следующая глава покажет, как оптимизировать инференс для production-нагрузок с помощью vLLM и TensorRT.

Скидывайте главу 23.



## 23. Оптимизация производительности и батчевая обработка

### Приоритетная стратегия: vLLM для production

Если ваша задача — обслуживать множество запросов (чат, API, пакетная обработка), **vLLM — единственное разумное решение**. Для Qwen2.5-3B на A100 он даёт прирост吞吐ности в 2–3 раза по сравнению со стандартными методами.

**Ключевые технологии vLLM:**

| Технология | Что даёт |
|------------|----------|
| PagedAttention | Экономия KV-кэша на 20-40% |
| Continuous Batching | Объединение запросов разной длины |
| Dynamic Batching | Автоматическая группировка |

**Базовый запуск vLLM сервера:**

```bash
pip install vllm

python -m vllm.entrypoints.api_server \
    --model Qwen/Qwen2.5-3B-Instruct \
    --tensor-parallel-size 1 \
    --max-num-batched-tokens 4096 \
    --max-num-seqs 256 \
    --gpu-memory-utilization 0.9
```

**Программный батчинг через vLLM:**

```python
from vllm import LLM, SamplingParams

llm = LLM(
    model="Qwen/Qwen2.5-3B-Instruct",
    max_model_len=4096,
    tensor_parallel_size=1,
    gpu_memory_utilization=0.9
)

sampling_params = SamplingParams(
    temperature=0.7,
    top_p=0.9,
    max_tokens=256
)

prompts = [
    "Explain Python decorators",
    "Write a fibonacci function",
    "What is RAG?"
]

outputs = llm.generate(prompts, sampling_params)
for output in outputs:
    print(output.outputs[0].text)
```

**Ожидаемые метрики (на A10G):**

| Метрика | Статический батчинг | vLLM оптимизация |
|---------|---------------------|------------------|
| Пропускная способность | 12.4 req/s | **28.6 req/s** |
| Средняя задержка | 320 ms | **138 ms** |
| Утилизация GPU | 65% | **92%** |

### Спеulative Decoding: ускорение для batch=1

Если вы обслуживаете одного пользователя (интерактивный чат, IDE-плагин), **speculative decoding** даёт ускорение до 1.76× без потери качества.

**Принцип:** Маленькая "черновая" модель (draft) генерирует 4–6 токенов за шаг, большая "целевая" модель (target) проверяет их все за один forward pass. При совпадении токенов мы получаем ускорение пропорционально длине последовательности.

**Для Qwen2.5-3B оптимальная пара:**

```bash
# Установка abyo-speculate (Rust библиотека)
cargo add abyo-speculate
```

```rust
use abyo_speculate::{SpeculateEngine, Method};
use abyo_speculate::model::qwen2_local::Qwen2Decoder;

// Target: Qwen2.5-3B, Draft: Qwen2.5-0.5B
let mut engine = SpeculateEngine::builder()
    .target_model("Qwen/Qwen2.5-3B-Instruct")
    .draft_model("Qwen/Qwen2.5-0.5B-Instruct")
    .method(Method::Vanilla)
    .draft_lookahead(4)  // 4 токена за шаг
    .build()?;

// Загрузка декодеров (CUDA поддерживается)
let target = Qwen2Decoder::from_paths(/* config, weights */)?;
let draft = Qwen2Decoder::from_paths(/* ... */)?;
engine = engine.with_target(target).with_draft(draft);

let prompt_ids = tokenizer.encode("Write a quick sort in Python");
let output_ids = engine.generate_tokens(&prompt_ids, 200)?;
```

**Измеренные ускорения (RTX 4070 Ti SUPER, BF16):**

| Задача | Обычный (tok/s) | Speculative (tok/s) | Ускорение |
|--------|-----------------|---------------------|-----------|
| Чат | 34.0 | 48.5 | **1.42×** |
| Код | 33.9 | 59.8 | **1.76×** |
| Перевод | 33.8 | 47.2 | **1.40×** |

**Почему код ускоряется сильнее всего:** Токены кода более предсказуемы, черновая модель чаще угадывает следующие токены.

### Батчевая обработка в transformers (если vLLM недоступен)

Если вы привязаны к `transformers` (например, для тонкой настройки или кастомных модификаций), используйте паддинг и `padding_side="left"`:

```python
from transformers import AutoTokenizer, AutoModelForCausalLM
import torch

tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen2.5-3B-Instruct")
tokenizer.padding_side = "left"  # Критически важно!
if tokenizer.pad_token is None:
    tokenizer.pad_token = tokenizer.eos_token

model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    torch_dtype=torch.float16,
    device_map="auto"
)

# Промпты разной длины
prompts = [
    "Hello",
    "What is the capital of France?",
    "Explain quantum computing in simple terms"
]

# Токенизация с паддингом
inputs = tokenizer(
    prompts,
    padding=True,
    truncation=True,
    max_length=512,
    return_tensors="pt"
).to(model.device)

# Батчевая генерация
with torch.no_grad():
    outputs = model.generate(
        **inputs,
        max_new_tokens=128,
        do_sample=True,
        temperature=0.7,
        pad_token_id=tokenizer.pad_token_id
    )

# Декодирование каждого ответа
responses = []
for i, input_ids in enumerate(inputs.input_ids):
    response = tokenizer.decode(
        outputs[i][len(input_ids):],
        skip_special_tokens=True
    )
    responses.append(response)
```

**Что произойдёт, если забыть `padding_side="left"`:** Модель будет генерировать токены, начиная с паддинг-токенов справа, что даст мусор на выходе.

### Flash Attention 2: экономия памяти

Flash Attention 2 снижает память для KV-кэша и ускоряет attention на 2-3×:

```bash
pip install flash-attn --no-build-isolation
```

```python
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    torch_dtype=torch.float16,
    attn_implementation="flash_attention_2",  # ключевой параметр
    device_map="auto"
)
```

**Когда Flash Attention даёт выигрыш:**
- Длинные контексты (>4096 токенов)
- Большие батчи (>4 промптов)
- Мульти-изображения в VL моделях

**Ограничения:** Требует GPU с compute capability 8.0+ (A100, RTX 3090/4090). Не работает со старыми картами.

### Квантизация как оптимизация

4-битная квантизация снижает размер модели с ~6 GB до ~2 GB VRAM с минимальной потерей качества (96-97% от FP16).

**Для vLLM (GPTQ/AWQ):**

```bash
# Загрузка предварительно квантизованной модели
vllm serve Qwen/Qwen2.5-3B-Instruct-AWQ \
    --port 8000 \
    --quantization awq
```

**Для transformers (bitsandbytes):**

```python
from transformers import BitsAndBytesConfig

bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_use_double_quant=True
)

model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    quantization_config=bnb_config,
    device_map="auto"
)
```

**Сравнение методов квантизации:**

| Метод | VRAM | Качество | Совместимость |
|-------|------|----------|---------------|
| FP16 | 6 GB | 100% | transformers, vLLM |
| 8-bit (bitsandbytes) | 3 GB | 98-99% | transformers |
| 4-bit NF4 | 2 GB | 96-97% | transformers |
| GPTQ-Int4 | 2 GB | 96-97% | vLLM, auto-gptq |
| AWQ | 2 GB | 96-97% | vLLM |

### Оптимизация длины последовательности

**Самое недооценённое правило:** Уменьшение длины контекста даёт квадратичный выигрыш в памяти, так как память attention растёт как O(n²).

```python
# Вместо 8192 используйте 4096, если это возможно
inputs = tokenizer(
    prompts,
    max_length=4096,  # снижение с 8192 → ~50% экономии VRAM
    truncation=True,
    padding=True
)
```

**Практическое правило:** Для Qwen2.5-3B, если 90% ваших данных умещается в 4096 токенов, используйте 4096. Не гонитесь за максимальным контекстом без необходимости.

### Мониторинг производительности

```python
import time
import torch

def benchmark_generation(model, tokenizer, prompts, batch_size=4, num_iterations=10):
    """Замер скорости генерации в батчевом режиме"""
    
    # Подготовка батчей
    all_outputs = []
    latencies = []
    total_tokens = 0
    
    for i in range(0, len(prompts), batch_size):
        batch = prompts[i:i+batch_size]
        
        # Токенизация
        inputs = tokenizer(
            batch,
            padding=True,
            truncation=True,
            max_length=512,
            return_tensors="pt"
        ).to(model.device)
        
        input_len = inputs.input_ids.shape[1]
        
        # Замер времени
        torch.cuda.synchronize()
        start = time.time()
        
        with torch.no_grad():
            outputs = model.generate(
                **inputs,
                max_new_tokens=128,
                do_sample=False
            )
        
        torch.cuda.synchronize()
        end = time.time()
        
        output_len = outputs.shape[1] - input_len
        total_tokens += output_len
        latencies.append(end - start)
        
        all_outputs.extend([
            tokenizer.decode(out[input_len:], skip_special_tokens=True)
            for out in outputs
        ])
    
    # Статистика
    total_time = sum(latencies)
    avg_latency = total_time / len(latencies)
    throughput = total_tokens / total_time
    
    print(f"Total time: {total_time:.2f}s")
    print(f"Avg latency: {avg_latency*1000:.1f}ms")
    print(f"Throughput: {throughput:.1f} tokens/s")
    
    return all_outputs

# Использование
test_prompts = ["Hello"] * 100  # 100 одинаковых запросов
benchmark_generation(model, tokenizer, test_prompts, batch_size=8)
```

### Сравнение подходов

| Сценарий | Рекомендация | Ожидаемая производительность |
|----------|--------------|------------------------------|
| **High-load API** (тысячи запросов/час) | vLLM + 4-bit AWQ | 30-40 req/s на A10G |
| **Интерактивный чат** (1 пользователь) | Speculative decoding (0.5B draft) | 1.76× ускорение |
| **Пакетная обработка** (оффлайн) | transformers + батчинг + Flash Attention | 50+ токен/с |
| **Edge устройство** (CPU, 8GB RAM) | GGUF Q4_K_M + llama.cpp | 5-10 токен/с |
| **Тонкая настройка** | QLoRA + gradient checkpointing | 2-4 GB VRAM |

### Чек-лист оптимизации (в порядке приоритета)

1. **Используйте vLLM для production** — самый высокий ROI
2. **Включите 4-битную квантизацию** — снижает VRAM на 60-70%
3. **Ограничьте длину контекста** — квадратичный выигрыш в памяти
4. **Для batch=1 используйте speculative decoding** — ускорение до 1.76×
5. **Включите Flash Attention 2** — ускорение attention на 2-3× (если GPU поддерживает)
6. **Настройте batch size под вашу VRAM** — максимальный без OOM

Следующая глава покажет, как деплоить модель в production с мониторингом, масштабированием и отказоустойчивостью.

Скидывайте главу 24.



## 24. Кэширование моделей и работа без интернета

### Автоматическое кэширование: как это работает

Hugging Face Transformers автоматически кэширует модели после первого использования. Стандартный путь кэша:

| ОС | Путь к кэшу |
|----|-------------|
| Linux/macOS | `~/.cache/huggingface/hub/` |
| Windows | `C:\Users\<username>\.cache\huggingface\hub\` |

**Важное уточнение:** Многие пользователи пытаются вручную управлять кэшем, но в большинстве случаев это не нужно. `from_pretrained()` автоматически скачивает модель при первом вызове и использует кэш при последующих .

```python
# Первый запуск — скачивание в кэш
model = AutoModelForCausalLM.from_pretrained("Qwen/Qwen2.5-3B-Instruct")
# Второй запуск — загрузка из кэша, сеть не используется
model = AutoModelForCausalLM.from_pretrained("Qwen/Qwen2.5-3B-Instruct")
```

### Способ 1: snapshot_download для полного контроля

Когда нужен полный контроль над путём загрузки — например, для переноса модели на другую машину или для точной настройки :

```python
from huggingface_hub import snapshot_download
import os

# Скачивание модели в указанную папку
model_path = snapshot_download(
    repo_id="Qwen/Qwen2.5-3B-Instruct",
    local_dir="./models/qwen-2.5-3b",  # путь к папке с моделью
    local_dir_use_symlinks=False       # прямое копирование, без симлинков
)

print(f"Модель сохранена в: {model_path}")
```

**Структура скачанной папки:**
```
./models/qwen-2.5-3b/
├── config.json
├── model.safetensors
├── tokenizer.json
├── tokenizer_config.json
└── generation_config.json
```

### Загрузка из локальной папки и offline-режим

После скачивания модель можно загружать без интернета:

```python
from transformers import AutoModelForCausalLM, AutoTokenizer

# Прямой путь к папке с моделью
local_path = "./models/qwen-2.5-3b"

tokenizer = AutoTokenizer.from_pretrained(local_path)
model = AutoModelForCausalLM.from_pretrained(
    local_path,
    local_files_only=True,  # запрещает любые сетевые запросы
    device_map="auto"
)
```

**Критическое замечание:** `local_files_only=True` требует **полной** модели в указанной папке. Если отсутствует хотя бы один файл (например, `config.json`), код выбросит `OSError` без попытки скачать недостающее .

### Переменные окружения для офлайн-режима

Современные версии Transformers и Hugging Face Hub используют единую переменную `HF_HUB_OFFLINE` :

```bash
# Установка перед запуском Python скрипта
export HF_HUB_OFFLINE=1
export HF_DATASETS_OFFLINE=1   # если используете datasets

python your_script.py
```

Или прямо в коде Python:

```python
import os
os.environ["HF_HUB_OFFLINE"] = "1"
os.environ["HF_DATASETS_OFFLINE"] = "1"
```

**Что дают эти переменные:**
| Переменная | Назначение | Альтернатива |
|------------|------------|--------------|
| `HF_HUB_OFFLINE=1` | Полностью отключает сетевые запросы | `TRANSFORMERS_OFFLINE=1` (устаревшая) |
| `local_files_only=True` | То же самое, но на уровне одного вызова | `model.from_pretrained(..., local_files_only=True)` |

### Решение проблемы "Incorrect path_or_model_id"

Ошибка из обращения пользователя возникает при неправильной передаче пути. Если вы скачали модель через `snapshot_download`, передавайте **путь к папке**, где лежат файлы модели, а не путь к `snapshots` подпапке внутри кэша .

```python
# ❌ Неправильно — путь указывает на snapshot подпапку внутри кэша
wrong_path = "D:/Qwen--Qwen2.5-3B-Instruct/.cache/models--Qwen--Qwen2.5-3B-Instruct/snapshots/aa8e72..."

# ✅ Правильно — указываем на корневую папку с моделью
model = AutoModelForCausalLM.from_pretrained(
    "./models/qwen-2.5-3b",  # папка, где лежат .safetensors файлы
    local_files_only=True
)
```

### Альтернатива: GGUF + Ollama для полной изоляции

Для полной изоляции от Python-зависимостей и интернета используйте GGUF версию через Ollama :

```bash
# 1. Скачивание модели через Ollama (или импорт .gguf файла)
ollama pull qwen2.5:3b

# 2. После скачивания работа полностью офлайн
ollama run qwen2.5:3b
```

**Преимущества GGUF подхода:**
- Один `.gguf` файл содержит все веса модели
- Не требует установки PyTorch и других тяжёлых зависимостей
- Легко переносится между компьютерами

### ModelScope: альтернативный источник моделей

Для пользователей в регионах с медленным доступом к Hugging Face:

```python
from modelscope.hub.snapshot_download import snapshot_download

# Скачивание через ModelScope
model_dir = snapshot_download(
    'qwen/Qwen2.5-3B-Instruct',
    cache_dir='./models/qwen2.5-3b'  # указание папки
)
print(f"Модель сохранена в: {model_dir}")

# Загрузка через transformers из скачанной папки
from transformers import AutoModelForCausalLM
model = AutoModelForCausalLM.from_pretrained(model_dir)
```

### Чек-лист офлайн-подготовки

**Перед отключением интернета:**
1. [ ] Загрузить модель через `snapshot_download` в выбранную папку
2. [ ] Проверить, что все файлы на месте: `ls ./models/qwen-2.5-3b/` должен показать `.safetensors`, `.json` конфиги
3. [ ] Протестировать загрузку с `local_files_only=True` пока интернет ещё есть
4. [ ] Сохранить скрипт с правильным путём к модели

**На целевой машине (без интернета):**
```python
# Полностью автономный запуск
import os
os.environ["HF_HUB_OFFLINE"] = "1"

from transformers import AutoTokenizer, AutoModelForCausalLM

# Абсолютный путь к скопированной модели
MODEL_PATH = "/absolute/path/to/models/qwen-2.5-3b"

tokenizer = AutoTokenizer.from_pretrained(MODEL_PATH)
model = AutoModelForCausalLM.from_pretrained(
    MODEL_PATH,
    device_map="auto",
    local_files_only=True
)
```

### Частые ошибки и решения

| Проблема | Решение |
|----------|---------|
| `OSError: Incorrect path_or_model_id` | Передаёте путь к snapshot подпапке, а не к корню модели  |
| `OSError: Cannot find file...` | В папке не хватает файлов. Скачайте заново через `snapshot_download`  |
| Все ещё идёт сетевой трафик после `local_files_only=True` | Установите `HF_HUB_OFFLINE=1` в переменных окружения  |
| `KeyError: 'auto_map'` | Модели Qwen требуют `trust_remote_code=True` (но для офлайн-режима он не нужен, если файлы скачаны)  |
| GGUF модель не запускается | Проверьте, что файл `.gguf` не повреждён, сравните размер с оригиналом  |



## 25. Лучшие практики и production-развёртывание

### Выбор движка инференса: vLLM vs SGLang

Для production-нагрузки выбор движка инференса — ключевое решение. Исследования на Qwen2.5-3B показывают, что современные движки дают значительное преимущество:

| Метрика | vLLM | SGLang (с кэшем) | Улучшение |
|---------|------|------------------|-----------|
| Пропускная способность (req/s) | 22.2 | **25.7** | +16.2%  |
| TTFT (средний) | 965 ms | **838 ms** | -13.2% |
| TPOT (средний) | 72 ms | **60 ms** | -17.2% |

**Вывод:** SGLang с включённым кэшированием обходит vLLM как по пропускной способности, так и по латентности. Это особенно заметно на мультимодальных моделях, но работает и на текстовых .

**Настройка SGLang для максимальной производительности:**

```bash
# Включение IPC кэша для ускорения мульти-входов
export SGLANG_USE_IPC_POOL_HANDLE_CACHE=1

# Запуск сервера
python -m sglang.launch_server \
    --model-path Qwen/Qwen2.5-3B-Instruct \
    --host 0.0.0.0 \
    --port 8000 \
    --tp 1 \
    --mem-fraction-static 0.9
```

### Production-деплой с TensorRT-LLM

Для максимальной производительности на NVIDIA GPU используйте TensorRT-LLM. Baseten показывает, что развёртывание сводится к простому `config.yaml` :

```yaml
# config.yaml для Baseten/TensorRT-LLM
model_metadata:
  tags:
    - openai-compatible
model_name: Qwen-2.5-3B
resources:
  accelerator: L4
  use_gpu: true
trt_llm:
  build:
    base_model: decoder
    checkpoint_repository:
      source: HF
      repo: "Qwen/Qwen2.5-3B-Instruct"
    max_seq_len: 8192
    quantization_type: fp8          # FP8 квантизация для баланса
    tensor_parallel_count: 1
    num_builder_gpus: 2             # Важно для FP8 сборки
```

**Важно:** FP8 квантизация в TensorRT-LLM требует 2 GPU на этапе сборки (один L4 не справляется), но на инференсе работает на одном .

### Мониторинг production-систем

Alibaba Cloud Model Studio рекомендует отслеживать следующие метрики :

| Метрика | Назначение | Целевое значение |
|---------|------------|------------------|
| **TTFT** (Time to First Token) | Воспринимаемая отзывчивость | <500 ms |
| **TPOT** (Time Per Output Token) | Скорость потока | <80 ms |
| **RPM** (Requests Per Minute) | Пропускная способность | Зависит от SLA |
| **TPM** (Tokens Per Minute) | Использование | Под нагрузкой |
| **Failure Rate** | Стабильность | <0.1% |

```python
# Пример сбора метрик в production
from prometheus_client import Counter, Histogram, Gauge
import time

requests_total = Counter('qwen_requests_total', 'Total requests')
request_duration = Histogram('qwen_request_duration_seconds', 'Request latency')
active_requests = Gauge('qwen_active_requests', 'Currently processing')

class MetricMiddleware:
    async def __call__(self, request, call_next):
        active_requests.inc()
        start = time.time()
        
        try:
            response = await call_next(request)
            request_duration.observe(time.time() - start)
            requests_total.inc()
            return response
        except Exception as e:
            requests_total.labels(status='error').inc()
            raise
        finally:
            active_requests.dec()
```

### Ключевой урок: host-side overhead убивает производительность

При профилировании SGLang на Qwen2.5-VL-3B обнаружили, что 13% времени CPU scheduler'а тратилось на повторное открытие одних и тех же CUDA IPC хендлов . Решение — простой `dict`-кэш:

```python
_pool_storage_cache: dict = {}
_pool_cache_lock = threading.Lock()

def _pool_handle_cache_get(pool_device_index, pool_handle):
    cache_key = (pool_device_index, tuple(pool_handle))
    storage = _pool_storage_cache.get(cache_key)
    if storage is None:
        with _pool_cache_lock:
            storage = _pool_storage_cache.get(cache_key)
            if storage is None:
                storage = torch.UntypedStorage._new_shared_cuda(*pool_handle)
                _pool_storage_cache[cache_key] = storage
    return storage
```

**Результат:** +16% пропускной способности и -17% TPOT .

**Урок для вашего кода:** Кэшируйте всё, что не меняется между запросами — токенизированные промпты, KV-кэш (через radix attention), вычислительные ресурсы.

### Оптимизация памяти через квантизацию

Согласно анализу DeepWiki, Qwen2.5 поддерживает три основных формата квантизации :

| Метод | Точность | Экономия памяти | Скорость | Use Case |
|-------|----------|-----------------|----------|----------|
| FP16 | 16-bit | Базово | 100% | Макс. качество |
| GPTQ Int8 | 8-bit | ~2x | Высокая | Баланс |
| GPTQ Int4 | 4-bit | ~4x | Высокая | Экономия |
| AWQ Int4 | 4-bit | ~4x | ~3x | Высокая скорость |
| GGUF | 4-8 bit | 2-4x | Вариативно | CPU/Edge |

**Рекомендация:** Для production на GPU — AWQ Int4 (лучшая скорость) или GPTQ Int4 (слегка лучшее качество). Для CPU — GGUF Q4_K_M .

### Тонкая настройка в production: GRPO + LoRA

Для дообучения Qwen2.5-3B в промышленных масштабах используйте **verl** (Volcano Engine Reinforcement Learning) — фреймворк от ByteDance для distributed RL .

**Базовая конфигурация GRPO + LoRA (из производственного примера):**

```bash
python3 -m verl.trainer.main_ppo \
    algorithm.adv_estimator=grpo \
    data.train_files=./data/gsm8k/train.parquet \
    data.train_batch_size=16 \
    data.max_prompt_length=512 \
    data.max_response_length=1024 \
    actor_rollout_ref.model.path=Qwen/Qwen2.5-3B-Instruct \
    actor_rollout_ref.model.lora_rank=64 \
    actor_rollout_ref.model.lora_alpha=32 \
    actor_rollout_ref.actor.optim.lr=3e-6 \
    actor_rollout_ref.rollout.name=vllm \
    actor_rollout_ref.rollout.n=5 \
    actor_rollout_ref.rollout.gpu_memory_utilization=0.6 \
    actor_rollout_ref.model.enable_gradient_checkpointing=True \
    trainer.n_gpus_per_node=4 \
    trainer.total_epochs=15
```

**Почему GRPO:** В отличие от PPO, GRPO не требует отдельной модели Value Function (Critic), что снижает VRAM на ~50% и упрощает обучение . Для Qwen2.5-3B это критично, так как 4×A100 становятся достаточными вместо 8×.

**Критический параметр:** `actor_rollout_ref.rollout.n=5` — количество ответов на промпт для групповой оценки. Слишком высокое значение увеличивает время обучения, слишком низкое — снижает качество оптимизации .

### 4 ключевых принципа production-деплоя

1. **Всегда используйте production-движок (vLLM/SGLang/TensorRT)**, а не `transformers` напрямую. Разница в 2-3× по throughput .
2. **Кэшируйте всё** — хендлы, KV-кэш, токенизированные промпты. Даже простой Python `dict` может дать +16% пропускной способности .
3. **Выбирайте квантизацию под железо:** на GPU — AWQ/GPTQ Int4; на CPU — GGUF Q4_K_M; для максимальной скорости на H100 — FP8 .
4. **Мониторинг ненадёжен без трёх метрик:** TTFT (восприятие), TPOT (поток), Request Rate (пропускная способность). Одна из них всегда будет узким местом .

### Чек-лист развёртывания

- [ ] SGLang или vLLM запущен с оптимизациями (`SGLANG_USE_IPC_POOL_HANDLE_CACHE=1`)
- [ ] Квантизация применена (AWQ Int4 или FP8 под TensorRT)
- [ ] Настроены метрики TTFT, TPOT, RPM
- [ ] Логирование включено (все запросы и ответы аудируются)
- [ ] Rate limiting настроен для защиты от DDoS/злоупотреблений
- [ ] Автоматическое переключение на fallback модель при сбоях

Это завершает главу. Если нужны дополнительные детали по конкретной подтеме — дайте знать.
