
# Insertion Sort

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

```psuedo
Algorithm partition(S, p)
	Input sequence S, position p of pivot
	Output subsequences L, E, G of the
	elements of S less than, equal to,
	or greater than the pivot, resp.
L, E, G <- empty sequences
x <- S.remove(p)
while S.isEmpty()
	y <- S.remove(S.first())
	if y < x
		L.insertLast(y)
	else if y = x
		E.insertLast(y)
	else { y > x }
		G.insertLast(y)
return L, E, G
```
