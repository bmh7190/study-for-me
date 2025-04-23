
# **Insertion Sort**

```c
void insertionSort (Element[] E, int n){
	int xindex;
	for(xindex = 1; xindex<n; xindex++){
		Element current = E[xindex];
		key x = current.key;
		int xLoc = shiftVacRec(E, xindex, x);
		E[xLoc] = current;
	}
}
```

첫 인덱스부터 그 인덱스를 기준으로 shiftVacRec 수행한다. 그걸 수행하면, 인덱스의 정렬된 위치가 나오고, 그 위치에 미리 저장해둔 인덱스의 값을 옮긴다. 

```c
int shiftVacRec(Element[] E, int vacant, Key x){
	int xLoc;
	if (vacant == 0)
		xLoc = vacant;
	else if (E[vacant-1].key ≤ x)
		xLoc = vacant;
	else
		E[vacant] = E[vacant-1];
		xLoc = shiftVacRec(E, vacant-1, x);
	return xLoc;
}
```

`vacant == 0` : 처음 입력이니까 바로 xLoc(x의 위치 결정)

`E[vacant-1].key ≤ x` : 전 인덱스의 원소보다 크기 때문에 정렬되어 xLoc 결정

`E[vacant-1].key > x` : 전 인덱스의 원소보다 작기 때문에 정렬이 필요!

일단 전 위치에 있던 원소를 지금 위치로 옮긴다. 
그리고 전 인덱스부터 x를 넣고, `shiftVacRect()` 다시 수행한다. 

---
# *Divide and Conquer*

문제를 하나의 큰 단위로 해결하는 것보다 작은 단위로 나누어  해결하는 것이 더 쉽다. 

더 작은 형태의 같은 문제로 문제를 나눈다.
재귀적으로 작은 경우에 해결한다. 
원래 입력값에 대한 해결책을 얻기 위해서 작은 문제에서 나온 해결책들을 합친다. 

---
# **Quick-Sort**

```c
Algorithm partition(S, p)
	Input sequence S, position p of pivot
	Output subsequences L, E, G of the
	elements of S less than, equal to,
	or greater than the pivot, resp.
	
L, E, G <- empty sequences
x <- S.remove(p)

while ㄱS.isEmpty()
	y <- S.remove(S.first())
	if y < x
		L.insertLast(y)
	else if y = x
		E.insertLast(y)
	else { y > x }
		G.insertLast(y)
		
return L, E, G
```

퀵정렬은 divide and conquer을 기반으로 랜덤한 정렬 알고리즘이다.

Divde: 랜덤한 원소 x, pivot을 설정하고, 그 x 보다 작은면 L그룹에 똑같으면, E, 더 크면 G그룹으로 나눈다.
계속 Divide한다면 결국 작은 원소와 큰 원소가 결정되고, combine 하면서 정렬이 완료된다. 

S 배열에서 원소를 빼서 각각의 구역으로 보내는 것은 O(1) 이 걸리지만 모든 원소에 대해 수행해야 하므로 O(n) 이 걸린다. 즉 divide만 O(n)이 걸리고, 다시 합치는건 O(log n )이 걸리기 때문에 O(nlogn)의 수행시간을 가진다. 

여기서 가장 최악의 수행 시간을 가지는 경우는 pivot이 최솟값이 최댓값에 있을 경우이다. 이 때는 O(n<sup>2</sup>)
의 수행 시간이 걸린다.

위에서는 그냥 새로운 배열을 만들고 계속 넣어줬기 때문에, 공간 복잡도가 증가했다. 하지만 공간복잡도를 늘리지 않고, 한 배열 안에서 수행할 수 있는 in-place 방식으로 퀵정렬을 구현할 수 있다. 

```c++
Alogirthm inPlaceQuickSort(S, l, r){
	if(l>=r) return;
	int i = a random integer between l and r;
	x = S.elemAtRank(i);
	(h, k) = inPlacePartition(x);
	inPlaceQuickSort(S, l, h-1);
	inPlaceQuickSort(S, k+1, r);
}
```

우선 base 조건은 l>= r 일 경우다.

처음 피봇인 x에 대해서 inplacepartition(x)을 수행하는데, 
j와 k는 양 쪽 끝에서 시작하고 j는 피봇보다 클 경우 k는 피봇보다 작을 경우 각각 멈추고, 둘 다 멈추면 j와 k의 원소를 swap한다. 계속 진행하다가 j와 k가 만나는 지점이 생기면  피봇과 원소를 swap하고  j와 k의 인덱스를 반환한다. 반환된 h와 k에 대해서 각각 inplacequicksort를 수행해주면 된다!

---
## **Merge Sorted Sequences**

머지 소트는 정렬되어 있는 두 배열을 병합하는 전략을 이용한 정렬 알고리즘이다. 정렬된 두 배열에서 하나씩 원소를 꺼내면서 대소 비교를 통해 더 작은 값을 새로운 배열에 넣는 방식이다. 이렇게 하면 최종적으로 두 배열이 하나의 정렬된 배열로 합쳐진다. 이 원리를 이용하는 게 머지 소트다.

먼저 배열의 원소가 하나가 될 때까지 계속 반으로 나눠가며 divide 과정을 반복하고, 그런 다음 다시 병합을 수행한다. 나누는 횟수는 대략 $logn$번이고, 각 병합 단계에서는 전체 원소 수인 $n$만큼의 연산이 필요하므로, 전체 수행 시간은 $nlogn$이 된다.

```c
void mergeSort(Element[] E, int first, int last){
	if (first < last)
		int mid = (first+last)/2;
		mergeSort(E, first, mid);
		mergeSort(E, mid+1, last);
		merge(E, first, mid, last);
	return;
}
```

기본적인 mergeSort 함수는 `first >= last`일 경우, 즉 정렬할 구간이 하나 이하일 때 재귀를 종료한다.  
그 외의 경우에는 먼저 중간 지점을 구한 뒤,

- **왼쪽 구간(first ~ mid)**
    
- **오른쪽 구간(mid+1 ~ last)**

각각에 대해 `mergeSort`를 재귀적으로 호출하고,  마지막에 `merge` 함수를 호출하여 두 정렬된 부분 배열을 하나로 합친다. 이처럼 `merge`가 **재귀 호출 이후에 실행**되므로,  전체 구조는 **postorder traversal(후위 순회)** 방식이라 할 수 있다.
 
```c
merge(A, B, C)
	if (A is empty)
		rest of C = rest of B
	else if (B is empty)
		rest of C = rest of A
	else if (first of A ≤ first of B)
		first of C = first of A
		merge (rest of A, B, rest of C)
	else
		first of C = first of B
		merge (A, rest of B, rest of C)
return
```

함수 `merge(A, B, C)`는 정렬된 두 배열 A와 B를 하나의 정렬된 배열 C로 병합하는 함수이다.  먼저 A 또는 B 중 하나가 비어 있으면, 나머지를 C에 그대로 붙인다.  그 외의 경우에는 A의 첫 번째 원소와 B의 첫 번째 원소를 비교하여,  A의 원소가 더 작거나 같으면 그 값을 C에 넣고,  A의 나머지, B, 그리고 C의 다음 위치를 대상으로 `merge`를 재귀 호출한다.  반대로 B의 원소가 더 작으면, B의 첫 원소를 C에 넣고 나머지를 merge한다.

이처럼 `merge` 함수 자체는 한 번 호출될 때 비교와 대입을 한 번만 수행하므로 시간 복잡도는 **O(1)** 이지만,  이 과정이 **모든 원소에 대해 재귀적으로 반복** 되기 때문에  전체 병합 과정의 **시간 복잡도는 O(n)** 이다.


---
# **HeapSort**

Heap은 이진트리에서 특별히 2가지 특성을 만족해야 한다.

> [!note]
> 1. 힙 구조를 가져야 한다. ->complete binary tree
> 2. 부모와 자식의 관계가 일관되어야 한다.


```c
heapSort(E, n) // Outline
construct H from E, the set of n elements to be sorted
for (i = n; i ≥ 1; i--)
	curMax = getMax(H)
	deleteMax(H);
	E[i] = curMax;
```

heapsort는 기본적으로 getmax를 통해서 최대값을 얻고, deleteMax를 통해 최댓값을 제거한다. 이때 진짜 최댓값이 제거되는게 아니라 맨 뒤의 원소와 자리를 바꾼 후 맨 뒤의 원소를 제거하게 된다. 이런 방식으로 힙 구조를 유지 시킬 수 있다. 그러나 부모와 자식 간의 순서 관계가 깨질 수도 있으므로, fixheap을 통해서 순서 관계를 만족시킨다.

```c
deteleMax(H) // Outline
	copy the rightmost element of the lowest level of H into K
	delete the rightmost element on the lowest level of H
	fixHeap(H, K);
```

```c
fixHeap(H, K){ // Outline
	if (H is a leaf)
		insert K in root(H);
	else
		set largerSubHeap to leftSubtree(H) or rightSubtree(H), whichever 
		has larger key at its root. This involves one key comparison. 
	
	if (K.key ≥ root(largerSubHeap.key)
		insert K in root(H);
	else
		insert root(largerSubHeap) in root(H);
		fixHeap(largerSubHeap, K);
	return;
}
```

fixheap 자체는 h 즉 logn 만 큼 수행하게 되는데 자식 들 간의 비교하고, 그 다음 부모와의 비교도 하기 때문에 총 2번의 비교가 발생한다. 그래서 2h 만큼의 수행시간이 필요하다. 



---
# **Accelerated Heapsort**

```c
void bubbleUpHeap (Element[] E, int root, Element K, int vacant){
	if (vacant == root)
		E[vacant] = K;
	else
		int parent = vacant / 2;
		if (K.key ≤ E[parent].key)
			E[vacant] = K;
		else
			E[vacant] = E[parent];
			bubbleUpHeap (E, root, K, parent);
}
```