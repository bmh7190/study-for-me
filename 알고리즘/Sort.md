
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