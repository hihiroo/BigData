# Spark Operations

![Untitled](/img/actions.png)

### **SparkContext**

- Spark cluster에 접근하기 위한 통로 역할, 각종 API 제공

```python
import pyspark
sc = pyspark.SparkContext()
```

### textFile()

- 디스크에 있는 파일을 메모리에 로드

```python
data_from_file = sc.textFile('textfile.txt')
```

### filter()

- lambda 식을 만족하는 element만 선택

```python
data_filtered = data_from_file.filter(lambda row: row[16]=="2014")
```

### parallelize()

- RDD를 생성
- sc.parallelize(data, 8)이면 data를 8개의 파티션으로 나누어 저장

```python
data = sc.parallelize([1,2,3,4,5,6,7,8,9,10],1)
```

### reduce()

- 매개변수로 들어가는 함수를 사용해 RDD를 단일객체가 될 때까지 줄인다.

```python
data = sc.parallelize([1,2,3,4,5,6,7,8,9,10],1)
data_reduce = data.reduce(lambda x,y: x+y)
print(data_reduce)
# 55
```

### collect()

- 데이터셋 전체를 리스트로 반환

```python
data = sc.parallelize([1,2,3,4,5],1)
data.collect()
# [1,2,3,4,5]
```

![Untitled](/img/collect.png)

### count()

- RDD의 element 개수 반환

```python
data.count()
# 5
```

### first()

- 처음 1개의 element 리턴

```python
data = sc.parallelize([1,2,3],1)
data.first()
# 1
```

### take(n)

- 처음 n개의 element를 리스트로 반환

```python
data = sc.parallelize([1,2,3,4,5],1)
data.take(3)
# [1,2,3]
```

### takeSample(withReplacement, num, seed=None)

- RDD에서 랜덤한 샘플을 뽑아 드라이버로 반환
- withReplacement : 한 번 뽑은 element를 다시 뽑을지 (True, False)
- num : sample의 크기

```python
data = sc.parallelize([1,2,3,4,5],1)
data.takeSample(Flase,3)
# [1, 4, 5]
```

### takeOrdered(n)

- rdd에서 작은 순서대로 n개 리턴 (데이터가 크면 sort() 사용 권장)

```python
data2 = sc.parallelize([10,9,8,7,6],1)
data2.takeOrdered(3)
# [6,7,8]
```

### saceAsTextFile(path)

- 데이터셋 파티션 하나당 하나의 파일로 저장 (각각의 element를 하나의 라인으로 저장)

```python
data = sc.parallelize([1,2,3,4,5],1)
data.saveAsTextFile("data")
data_reread = sc.textFile("data")
data_reread.collect()
# [1,2,3,4,5]
```

### countByKey()

- key-value 쌍으로 이루어진 RDD에서 작동하며, key를 기준으로 개수를 센다.
- key와 count 값으로 이루어진 dictionary 리턴

```python
data = sc.parallelize([('a',1),('b',2),('a',3)])
data.countByKey()
# defaultdict(int, {'a':2, 'b':1})
```

### map(func)

- RDD의 각 element에 함수를 적용하여 새로운 RDD 리턴

```python
data = sc.parallelize([1,2,3,4,5],1)
data = data.map(lambda x:x+1)
data.collect()
# [2,3,4,5,6]
```

### flatMap(func)

- element 하나를 여러개로 분해

```python
data = sc.parallelize([1,2,3,4],1)
data.map(lambda x: [x,x*x]).collect()
# [[1,1],[2,4],[3,9],[4,16]]
data.flatMap(lambda x: [x,x*x]).collect()
[1,1,2,4,3,9,4,16]
```

### mapPartitions(func)

- 각 파티션에 대해 map을 실행

```python
data = sc.parallelize([1,2,3,4,5,6,7,8,9,10],3)
def f(x):
	yield sum(x)
data.mapPartitions(f).collect()
# [6,15,34]
```

### mapValues(func)

- key-value 쌍 RDD에 대해 value에만 func 적용

```python
x = sc.parallelize([("a",["apple","banana","lemon"]), ("b", ["grapes"])])
def f(x): return len(x)
x.mapValues(f).collect()
[('a',3), ('b',1)]
```

### reduceByKey(func, [numPartitions])

- 같은 key를 가진 elements 끼리 func을 수행

```python
data = sc.parallelize([('a',1),('b',2),('c',3),('a',4)])
print(data.reduceByKey(lambda x,y:x+y).collect())
# [('b',2),('c',3),('a',5)]
```

### groupByKey([numPartitions])

- (key,value) 쌍의 RDD에서 같은 key가 같은 쌍만 모아 (key, Iterable<value>) 쌍의 RDD 반환
- numPartitions(optional) : 반환되는 새 RDD가 나누어질 파티션 개수

```python
data = sc.parallelize([('a',1), ('b',2), ('a',4)])
print(data.groupByKey().mapValues(list).collect())
# [('b',[2]), ('a',[1,4])]
```

### sortByKey([ascending=True],[numPartitions])

- key 값을 기준으로 오름차순 혹은 내림차순으로 정렬한 결과 RDD를 리턴

```python
data = sc.parallelize([(3,'a'), (4,'b'), (1,'c'), (2,'d')])
print(data.sortByKey().collect())
# [(1,'c'),(2,'d'),(3,'a'),(4,'b')]
```

### glom()

- 각 파티션을 하나의 리스트에 나열

```python
data = sc.parallelize([1,2,3,4,5],3)
data.glom().collect()
# [[1],[2,3],[4,5]]
```

### coalesce(numPartitions, shuffle=Flase)

- numPartitions 개의 파티션으로 파티션 개수를 줄인 새 RDD를 생성

```python
data = sc.parallelize([1,2,3,4,5], 3)
data.glom().collect()
# [[1], [2,3], [4,5]]
data.coalesce(1).glom().collect()
# [[1,2,3,4,5]]
```

### repartition(numPartitions)

- numPartitions 만큼의 partition이 있는 새 RDD 리턴
- RDD의 데이터를 무작위로 재구성

```python
data = sc.parallelize([1,2,3,4,5,6],3)
data.glom().collect()
# [[1,2],[3,4],[5,6]]
data.repartition(2).glom().collect()
# [[1,2,3],[4,5,6]]
```

### sample(withReplacement, fraction, seed)

- 데이터의 일부를 샘플링
- withReplacement : 중복 추출 여부
- Fraction : 리턴할 데이터셋과 전체 데이터셋 간의 크기 비율

```python
data = sc.parallelize([1,2,3,4,5,6,7,8,9,10],3)
sample = data.sample(False, 0.5, 777)
print(sample.collect())
# [2,3,4,5,9]
```

### distinct([numPartitions])

- RDD에서 중복된 값을 제거한 새로운 RDD 리턴

```python
data = sc.parallelize([1,2,3,4,5,4,5],1)
print(data.distinct().collect())
# [1,2,3,4,5]
```

### union(otherDataset)

- 두 RDD의 element들을 모두 합쳐 새로운 RDD 생성
- 중복 제거 안됨

```python
data1 = sc.parallelize([1,2,3],1)
data2 = sc.parallelize([3,4,5],1)
print(data1.union(data2).collect())
# [1,2,3,3,4,5]
```

### intersection(otherDataset)

- 두 RDD의 교집합에 해당하는 새로운 RDD 생성

```python
data1 = sc.parallelize([1,2,3],1)
data2 = sc.parallelize([3,4,5],1)
print(data1.intersection(data2).collect())
# [3]
```

### cogroup(otherDataset, [numPartitions])

- key-value 쌍 유형의 RDD에서 사용
- (a,b) RDD와 (a,c) RDD를 cogroup하면 (a, (Iterable \<b>, iterable \<c>)) 튜플 반환

```python
data1 = sc.parallelize([(1,'a'), (2,'b')])
data2 = sc.parallelize([(1,'c'), (2,'d')])
group = data1.cogroup(data2).mapValues(lambda t: (list(t[0]), list(t[1]))).collect()
group
# [(1,(['a'],['c'])), (2,(['b'],['d']))]
```

### join(otherDataset, numPartitions=None)

- key-value 쌍 유형의 RDD에서 매칭되는 키에 대해 value를 튜플로 조합하여 RDD로 반환
- self에 (k,w)가 있고 other에 (k,v)가 있으면 (k, (w,v))로 조합

```python
x = sc.parallelize([('a',1), ('b', 4)])
y = sc.parallelize([('a',2), ('a', 3)])
sorted(x.join(y).collect())
# [('a',(1,2)), ('a',(1,3))]
```