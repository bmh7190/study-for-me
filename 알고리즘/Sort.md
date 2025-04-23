
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



머지 소트는 정렬되어있는 두 배열을 병합하는 전략이다. 정렬되어있는 두 배열에서 하나 씩 원소를 제거해가면서, 대소비교를 통해 하나의 배열에 넣는다. 그럼 최종적으로 정렬된 두 배열이 정렬된 하나의 배열이 되는 것이다. 이 점을 이용한다

우선 배열의 원소가 1이 될 때까지 divide한다. ㅇ


---
# **HeapSort**

```c
heapSort(E, n) // Outline
construct H from E, the set of n elements to be sorted
for (i = n; i ≥ 1; i--)
	curMax = getMax(H)
	deleteMax(H);
	E[i] = curMax;
```

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