---

mindmap-plugin: markdown

---
## 栈
```c
//定义栈结构
typedef struct{
    int data[MAXSIZE];//存储元素的数组
    int top;//栈顶指针
}SeqStack;
```

在数组栈中用: ** int top;而不是int *top;**

+ <font style="color:rgba(0, 0, 0, 0.85) !important;">链式栈必须用</font><font style="color:rgba(0, 0, 0, 0.85) !important;"> </font>`<font style="color:rgb(0, 0, 0);">Node* top</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">（地址指针），因为链表节点内存不连续，只能通过地址找到下一个节点；</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;">数组栈用 </font>`<font style="color:rgb(0, 0, 0);">int top</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">（下标），因为数组内存连续，下标能更高效地定位和管理栈顶</font>



<font style="color:rgba(0, 0, 0, 0.85) !important;">假设数组 </font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">data</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;"> 的起始地址是 </font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">0x100</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">（每个 </font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">int</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;"> 占 4 字节），当栈中存入元素 </font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">10</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">（存在 </font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">data[0]</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">）时：</font>

+ <font style="color:rgba(0, 0, 0, 0.85) !important;">栈顶元素的</font>**<font style="color:rgb(0, 0, 0) !important;">内存地址</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">是 </font>`<font style="color:rgb(0, 0, 0);">0x100</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">（</font>`<font style="color:rgb(0, 0, 0);">&data[0]</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">）</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;">栈顶元素的</font>**<font style="color:rgb(0, 0, 0) !important;">数组下标</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">是 </font>`<font style="color:rgb(0, 0, 0);">0</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">（即 </font>`<font style="color:rgb(0, 0, 0);">top=0</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">）</font>



<font style="color:rgba(0, 0, 0, 0.85);">数组的内存是连续固定的,通过 “下标 + 数组起始地址” 就能快速定位元素（</font>`<font style="color:rgba(0, 0, 0, 0.85);">data[top]</font>`<font style="color:rgba(0, 0, 0, 0.85);"> 等价于 </font>`<font style="color:rgba(0, 0, 0, 0.85);">*(data + top)</font>`<font style="color:rgba(0, 0, 0, 0.85);">），无需直接存储地址</font>



```c
//入栈
int Push(SeqStack *S,int e){

    if(S->top==MAXSIZE -1){
        printf("栈满，入栈失败！\n");
        return 0;
    }

    S->top++;
    S->data[S->top]=e;
    return 1;
}
```

S->data[**S->top**]=e;`<font style="color:rgba(0, 0, 0, 0.85);">S->top</font>`<font style="color:rgba(0, 0, 0, 0.85);"> 是栈顶的 “下标”（例如 </font>`<font style="color:rgba(0, 0, 0, 0.85);">top=2</font>`<font style="color:rgba(0, 0, 0, 0.85);"> 表示栈顶元素在数组的第 2 个位置）</font>

<font style="color:rgba(0, 0, 0, 0.85);"></font>

## <font style="color:rgba(0, 0, 0, 0.85);">队列</font>


```c
// 1. 初始化队列
void InitQueue(SqQueue *Q) {
    Q->front = 0;       // 队头指针初始化为0
    Q->rear = 0;        // 队尾指针初始化为0
}
```

为什么指针初始化为0，而不是-1





```c
// 3. 判断队列是否为满
bool IsFull(SqQueue *Q) {
    // (队尾指针+1) % 最大容量 == 队头指针，则队列满
    return ((Q->rear + 1) % MAXSIZE == Q->front);
}
```

区分是队空还是队满的不同处理方式

