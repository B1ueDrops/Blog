---
title: 循环队列的设计与实现
categories: 算法
mathjax: true
---



## 循环队列的原理

* 首先, 一个队列先进先出, 但是如果队列是固定长度, 那么必须设计成循环队列.
* 其次, 维护两个指针`front`和`rear`:
  * `front`始终指向队头元素.
  * `rear`指向下一个`enqueue`的元素要进入队列的位置.
* 注意, 队列的大小需要设置成`MAX_SIZE + 1`, 需要留空一个元素区分队空和队满.
* 首先, 判空, 如果下一个元素进入队列的位置是队头, 那么就是空.
* 其次, 判满, 如果下一个元素进入队列的位置是预留的空位, 那么就是满了, 这个空位是特殊的标志位.
  * 这个空位就是队头前面的位置.
* 然后, 求循环队列中的元素:
  * 如果是顺序队列, 那么就是`rear - front`.
  * 如果是循环队列, 那么就在`% MAX_SIZE`的意义下进行: `int size = (rear - front + MAX_SIZE) % MAX_SIZE;`



## 循环队列代码

```c
const int MAX_SIZE = 5;

typedef struct {
	int front;
	// 指向下一个空的位置
	int rear;
	int data[MAX_SIZE + 1];
} CircularQueue;

void init(CircularQueue *queue) {
	queue->front = 0;
	queue->rear = 0;
}

int is_empty(CircularQueue *queue) {
	if (queue->front == queue->rear) return 1;
	return 0;
}

int is_full(CircularQueue *queue) {
	if ((queue->rear + 1) % MAX_SIZE == queue->front) return 1;
	return 0;
}

int get_size(CircularQueue *queue) {
  return (queue.rear - queue.front + MAXSIZE) % MAXSIZE;
}
/*
 * @param queue 循环队列
 * @param x 指定元素
 * @return 如果队列已满, 返回0, 表示不能入队, 否则返回1
 * */
int enqueue(CircularQueue *queue, int x) {
	if (is_full(queue)) return 0;
	queue->data[queue->rear] = x;
	queue->rear = (queue->rear + 1) % MAX_SIZE;
	return 1;
}
/* x保存出队元素 */
int dequeue(CircularQueue *queue, int *x) {
	if (is_empty(queue)) return 0;
	*x = queue->data[queue->front];
	queue->front = (queue->front + 1) % MAX_SIZE;
	return 1;
}

```

