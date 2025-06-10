thread
---
## 병렬 실행
#### 데이터 병렬실행
- 큰 데이터를 여러 조각으로 나눠서 같은 작업을 동시에 수행

```python
import threading

# 큰 데이터 리스트
numbers = list(range(1000))  # [0, 1, 2, ..., 999]
results = [0] * 10  # 10개 스레드의 결과를 저장할 리스트

def multiply_chunk(start, end, index):
    # 100개씩 나눈 데이터에 대해 같은 작업(곱하기) 수행
    chunk = numbers[start:end]
    results[index] = [num * 2 for num in chunk]

# 1000개의 데이터를 100개씩 10개로 나누어 처리
threads = []
for i in range(10):
    start = i * 100
    end = (i + 1) * 100
    t = threading.Thread(target=multiply_chunk, args=(start, end, i))
    threads.append(t)
    t.start()

for t in threads:
    t.join()

# 모든 결과 합치기
final_result = []
for chunk_result in results:
    final_result.extend(chunk_result)

print(f"처리된 데이터 수: {len(final_result)}")  # 1000
```

#### 태스크 병렬실행
- 같은 데이터에 대해 서로 다른 작업들을 동시에 수행

```python
import threading

numbers = [1, 2, 3, 4, 5]
results = {
    'multiply': [],
    'subtract': []
}

def multiply_by_two():
    # 곱하기 작업
    for num in numbers:
        results['multiply'].append(num * 2)

def subtract_one():
    # 빼기 작업
    for num in numbers:
        results['subtract'].append(num - 1)

# 같은 데이터(numbers)에 대해 서로 다른 작업을 동시에 수행
t1 = threading.Thread(target=multiply_by_two)
t2 = threading.Thread(target=subtract_one)

t1.start()
t2.start()

t1.join()
t2.join()

print("곱하기 결과:", results['multiply'])
print("빼기 결과:", results['subtract'])

# 출력 결과:
# 곱하기 결과: [2, 4, 6, 8, 10]
# 빼기 결과: [0, 1, 2, 3, 4]
```
## 정리
- **데이터 병렬**: 데이터 분할 → 같은 작업 동시 수행
- **태스크 병렬**: 같은 데이터 → 다른 작업들 동시 수행