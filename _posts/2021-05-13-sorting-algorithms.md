---
title: "[CS] 여러가지 정렬 알고리즘"

categories:
  - Computer Science
tags:
  - [Algorithm, Sort]

toc: true
toc_sticky: true

date: 2021-05-13
last_modified_at: 2021-05-13
---

# 빅오 표기법

빅오 표기법(big-O notation)은 점근표기법 중 하나로, 알고리즘의 성능을 상한선을 기준으로 하여 수학적으로 표기하는 방법이다. 알고리즘의 시간 복잡도(시간 효율성)과 공간 복잡도(메모리 공간 효율성)를 나타낼 수 있다. 여기서 알고리즘의 시간 복잡도는 실제 알고리즘의 러닝타임을 의미하는 것이 아닌, 데이터나 사용자 수의 증가에 따른 알고리즘의 성능 변화를 예측하는 것이 목적이다.

<br>

빅오 표기법의 정의는 아래와 같다.

```
💡 어떤 함수 𝑓(𝑛)의 빅오 표기법이 𝑂(𝑔(𝑛))이라는 것은,
𝑛 > 𝑛₀ 을 만족하는 충분히 큰 모든 𝑛에 대해 𝑓(𝑛) ≤ 𝑐𝑔(𝑛) 인
양수 𝑐와 𝑛₀가 존재한다는 것이다.
```

## 𝑂(1)

상수형. 입력 데이터의 크기와 상관 없이 항상 일정한 연산 횟수를 가진다.

```c
for(int i = 0; i < 10; i++)
```

## 𝑂(𝚕𝚘𝚐𝑛)

로그형. (ex : 이진 탐색)

```c
for(int i = n; i > 0; i/=2)
```

## 𝑂(𝑛)

선형. 입력 데이터의 크기와 연산 횟수가 비례한다. (ex : 최댓값 찾기)

```c
for(int i = 0; i < n; i++)
```

## 𝑂(𝑛𝚕𝚘𝚐𝑛)

선형로그형. (ex : 퀵 정렬, 합병 정렬)

```c
for(int i = 0; i < n; i++)
  for(int j = i; j > 0; j/=2)
```

## 𝑂(𝑛²)

평방형. (ex : 버블 정렬, 삽입 정렬)

```c
for(int i = 0; i < n; i++)
  for(int j = 0; j < n; j++)
```

## 𝑂(𝑛³)

입방형.

```c
for(int i = 0; i < n; i++)
  for(int j = 0; j < n; j++)
		for(int k = 0; k < n; k++)
```

## 𝑂(2ⁿ)

지수형. (ex : 피보나치수열)

```c
int fibonacci(int n) {
  if(n == 0 || n == 1)
    return 1;
  return fibonacci(n-1) + fibonacci(n-2);
}
```

<br>

---

# 정렬 알고리즘

## 선택 정렬

제자리 정렬 알고리즘의 하나이다. 첫번째 자료와 두 번째부터 n번째 자료까지 전부 비교하여 가장 크거나 작은 값을 찾아 첫번째 자리에 놓는다. 이후에는 두 번째부터 n-1번째 자료까지 같은 작업을 반복하면 정렬이 완료된다. 𝑂(𝑛²)의 시간 복잡도를 갖는다.

```c
// 5개 정수를 오름차순으로 선택 정렬하여 출력하는 코드

#include <stdio.h>

int main() {

	int arr[5] = { 7, 9, 1, 4, 3 };
	int n = 5;

	for (int i = 0; i < n - 1; i++) {
		for (int j = i; j < n; j++) {
			if (arr[i] > arr[j]) {
				int tmp = arr[i];
				arr[i] = arr[j];
				arr[j] = tmp;
			}
		}
	}

	for (int i = 0; i < n; i++) printf("%d ", arr[i]);

	return 0;
}
```

## 삽입 정렬

두 번째 자료부터 시작하여 그 왼쪽의 이미 정렬된 자료들과 비교하여 삽입할 위치를 정하고 그 위치보다 오른쪽에 있던 자료들을 한 칸씩 오른쪽으로 옮긴 후 원하는 자리에 자료를 삽입한다. 이 작업을 두 번째 자료부터 n번째 자료까지 반복하면 정렬이 완료된다. 𝑂(𝑛²)의 시간 복잡도를 갖는다.

```c
// 5개 정수를 오름차순으로 삽입 정렬하여 출력하는 코드

#include <stdio.h>

int main() {

	int arr[5] = { 7, 9, 1, 4, 3 };
	int n = 5;

	for (int i = 1; i < n; i++) {
		int curnum = arr[i], isput = 0;
		for (int j = i - 1; j >= 0 && isput == 0; j--) {
			if (curnum < arr[j]) arr[j + 1] = arr[j];
			else {
				arr[j + 1] = curnum;
				isput = 1;
			}
		}
		if (isput == 0) arr[0] = curnum;
	}

	for (int i = 0; i < n; i++) printf("%d ", arr[i]);

	return 0;
}
```

## 버블 정렬

첫 번째 원소와 두 번째 원소를 비교해 교환하고, 두 번째 원소와 세 번째 원소를 비교해 교환한다. 이 작업을 n-1번째 원소와 n번째 원소까지 반복하면 가장 크거나 작은 값이 n번 자리에 있게 된다. 전체 작업을 반복하면 그 다음으로 크거나 작은 값이 순서대로 n-1, n-2, ... 번째 자리에 있게 되어 정렬이 완료된다. 𝑂(𝑛²)의 시간 복잡도를 갖는다.

```c
// 5개 정수를 오름차순으로 버블 정렬하여 출력하는 코드

#include <stdio.h>

int main() {

	int arr[5] = { 7, 9, 1, 4, 3 };
	int n = 5;

	for (int i = n - 1; i >= 0; i--) {
		for (int j = 0; j < i; j++) {
			if (arr[j] > arr[j + 1]) {
				int tmp = arr[j];
				arr[j] = arr[j + 1];
				arr[j + 1] = tmp;
			}
		}
	}

	for (int i = 0; i < n; i++) printf("%d ", arr[i]);

	return 0;
}
```

## 합병 정렬

하나의 리스트를 균등한 크기의 두 개의 리스트로 분할한 다음 분할된 부분 리스트를 정렬한 뒤, 두 개의 정렬된 부분 리스트를 합하여 정렬된 전체 리스트가 되게 하는 방법이다. 분할 정복 알고리즘의 하나이다. 𝑂(𝑛𝚕𝚘𝚐𝑛)의 시간복잡도를 가진다.

```c
// 8개 정수를 오름차순으로 합병 정렬하여 출력하는 코드
// https://jaehee-developer.tistory.com/12 의 코드 참고하여 작성함

#include <stdio.h>

void divide(int arr[], int, int);
void merge(int arr[], int, int, int);

int main() {

	int myarr[8] = { 12, 3, 7, 9, 1, 14, 3, 10 };
	int n = 8;
	divide(myarr, 0, n - 1);

	for (int i = 0; i < n; i++) printf("%d ", myarr[i]);

	return 0;
}

void divide(int arr[], int l, int r) {
	int m;

	//배열을 반으로 나눈 뒤 재귀적으로 작은 배열부터 merge
	if (l < r) {
		m = (l + r) / 2;
		divide(arr, l, m);
		divide(arr, m + 1, r);
		merge(arr, l, m, r);
	}
}
void merge(int arr[], int a, int b, int c) {
	int i = a, j = b + 1, k = a;
	int tmparr[8];

	//두 배열 중 하나가 끝나기 전까지 정렬하며 병합
	while (i <= b && j <= c) {
		if (arr[i] < arr[j]) tmparr[k++] = arr[i++];
		else tmparr[k++] = arr[j++];
	}

	//나머지 값들도 복사
	while (i <= b) tmparr[k++] = arr[i++];
	while (j <= c) tmparr[k++] = arr[j++];

	for (int s = a; s <= c; s++) arr[s] = tmparr[s];
}
```

## 퀵 정렬

기준값(pivot)을 설정한 후, pivot보다 작은 값을 왼쪽에, pivot보다 큰 값을 오른쪽에 몰아서 배열한다. 각 부분에서 앞의 작업을 반복해주면 오름차순 배열이 완료된다. 최악의 경우 𝑂(𝑛²)의 시간복잡도를, 평균적으로 𝑂(𝑛𝚕𝚘𝚐𝑛)의 시간복잡도를 가진다.

```c
// 8개 정수를 오름차순으로 퀵 정렬하여 출력하는 코드
// https://jaehee-developer.tistory.com/14 의 코드 참고하여 작성함

#include <stdio.h>

void divide(int arr[], int, int);
int getPivot(int arr[], int, int);

int main() {

	int myarr[8] = { 12, 3, 7, 9, 1, 14, 3, 10 };
	int n = 8;
	divide(myarr, 0, n - 1);

	for (int i = 0; i < n; i++) printf("%d ", myarr[i]);

	return 0;
}

void divide(int arr[], int start, int end) {
	if (start < end) {
		int pivot = getPivot(arr, start, end);
		divide(arr, start, pivot - 1);
		divide(arr, pivot + 1, end);
	}
}
int getPivot(int arr[], int start, int end) {
	int i = start - 1, j, pivot = arr[end], tmp;

	// pivot보다 작은 요소들을 배열의 앞에 모으기
	for (int j = start; j < end; j++) {
		if (arr[j] <= pivot) {
			tmp = arr[++i];
			arr[i] = arr[j];
			arr[j] = tmp;
		}
	}

	// pivot을 pivot보다 작은 요소들의 바로 오른쪽에 배치
	tmp = arr[i + 1];
	arr[i + 1] = arr[end];
	arr[end] = tmp;
	return i + 1;
}
```
