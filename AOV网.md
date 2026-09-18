---
created_at: 2026-03-20
updated_at: 2026-09-18
tags:
    - 计算机基础
archived: true
---

[数据结构](./数据结构.md) / 6. 图 / 6.6 拓扑排序

# AOV网

在一个表示工程的有向图中, 用顶点表示活动, 用弧表示活动之间的优先级, 这样的有向图为顶点表示活动的网, 我们称为AOV网(Activity On Vertex Network)

**拓扑排序是解决怎么遍历一张有向图的问题**

思路: 持续寻找入度为0的顶点, 然后入栈, 出栈

```C
#include <stdio.h>
#include <stdlib.h>

typedef int VertexType;
typedef int EdgeType;

#define MAXSIZE 100

typedef struct{
    VertexType vertex[MAXSIZE];
    EdgeType arc[MAXSIZE][MAXSIZE];
    int vertex_num;
    int edge_num;
}Mat_Graph;

typedef struct EdgeNode{
    int edge_vex;
    struct EdgeNode* next;
}EdgeNode;

typedef struct VertexNode{
    int in;
    VertexType data;
    EdgeNode* head; //firstEdge
}VertexNode;

typedef VertexNode Adj_List[MAXSIZE];

typedef struct{
    Adj_List adj_list;
    int vertex_num;
    int edge_num;
}Adj_Graph;

typedef Adj_Graph* Adj_List_Graph;

int top = -1;
int stack[MAXSIZE];

void push(int e){
    if(top > MAXSIZE){
        printf("满了\n");
        return;
    }
    top++;
    stack[top] = e;
}

int pop(){
    if(top == -1){
        return -1;
    }
    int elem = stack[top];
    top--;
    return elem;

}

int is_empty(){
    if(top == -1){
        return 0;
    }
    else{
        return 1;
    }
}

void create_graph(Mat_Graph* G){
    G->vertex_num = 14;
    G->edge_num = 20;

    for(int i = 0; i < G->vertex_num; i++){
        G->vertex[i] = i;
    }

    for(int i = 0; i < G->vertex_num; i++){
        for(int j = 0; j < G->vertex_num; j++){
            G->arc[i][j] = 0;
        }
    }

    G->arc[0][4] = 1;
    G->arc[0][5] = 1;
    G->arc[0][11] = 1;
    G->arc[1][2] = 1;
    G->arc[1][4] = 1;
    G->arc[1][8] = 1;
    G->arc[2][5] = 1;
    G->arc[2][6] = 1;
    G->arc[2][9] = 1;
    G->arc[3][2] = 1;
    G->arc[3][13] = 1;
    G->arc[4][7] = 1;
    G->arc[5][8] = 1;
    G->arc[5][12] = 1;
    G->arc[6][5] = 1;
    G->arc[8][7] = 1;
    G->arc[9][10] = 1;
    G->arc[9][11] = 1;
    G->arc[10][13] = 1;
    G->arc[12][9] = 1;
}

void create_adj_graph(Mat_Graph G, Adj_List_Graph* ALG){
    EdgeNode* e;

    *ALG = (Adj_List_Graph)malloc(sizeof(Adj_Graph));
    (*ALG)->vertex_num = G.vertex_num;
    (*ALG)->edge_num = G.edge_num;

    for(int i = 0; i < G.vertex_num; i++){
        (*ALG)->adj_list[i].in = 0;
        (*ALG)->adj_list[i].data = G.vertex[i];
        (*ALG)->adj_list[i].head = NULL;
    }

    for(int i = 0; i < G.vertex_num; i++){
        for(int j = 0; j < G.vertex_num; j++){
            if(G.arc[i][j] == 1){
                e = (EdgeNode*)malloc(sizeof(EdgeNode));
                e->edge_vex = j;
                e->next = (*ALG)->adj_list[i].head;
                (*ALG)->adj_list[i].head = e;
                (*ALG)->adj_list[j].in++;
            }
        }
    }
}

void topological_sort(Adj_List_Graph ALG){
    EdgeNode* e;
    int curr;
    int k;

    for(int i = 0; i < ALG->vertex_num; i++){
        if(ALG->adj_list[i].in == 0){
            push(i);
        }
    }

    while(is_empty() != 0){
        curr = pop();
        printf("V%d -> ", ALG->adj_list[curr].data);
        e = ALG->adj_list[curr].head;

        while(e != NULL){
            k = e->edge_vex;
            ALG->adj_list[k].in--;
            if(ALG->adj_list[k].in == 0){
                push(k);
            }
            e = e->next;
        }
    }
}

int main(){
    Mat_Graph G;
    Adj_List_Graph ALG;
    create_graph(&G);
    create_adj_graph(G, &ALG);
    topological_sort(ALG);
    return 0;
}
```

## 相关笔记

- 上一篇:[弗洛伊德(Floyd) 算法](./弗洛伊德算法.md)
- 下一篇:[AOE网和关键路径](./AOE网和关键路径.md)
- 返回索引:[数据结构](./数据结构.md)
