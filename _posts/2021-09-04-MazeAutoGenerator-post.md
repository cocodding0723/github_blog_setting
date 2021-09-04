---
title : "[알고리즘] 미로 자동 생성 (Eller's Algorithm)"
categories : 
    - cpp
    - algorithm
date : 2021-09-04
---

참고 사이트
[참고1](https://dev-nicitis.tistory.com/5)
[참고2](http://weblog.jamisbuck.org/2010/12/29/maze-generation-eller-s-algorithm)

미로를 랜덤하게 생성하는 알고리즘에는 재귀적 백트래킹, 비트 마스크 등등이 있다.   
그중 시간 복잡도가 O(N)인 알고리즘인 엘러 알고리즘에 매력을 느껴 직접 구현해 보았다.

방법은 다음과 같다.

1. 각 행에 집합을 할당한다. 만약 집합이 존재한다면 집합을 할당하지 않는다.
2. 랜덤하게 각 집합을 합친다.
3. 각 집합마다 1개이상 가지를 내린다.
4. 1~3을 마지막행까지 반복한다.
5. 마지막 행에 도착시 그 행을 순환하며 인접 집합이 다를시 벽을 허문다.

코드는 다음과 같다.

```cpp
#include <iostream>
#include <vector>
#include <unordered_map>
#include <chrono>
#include <time.h>

using namespace std;

struct Cell
{
    int col, row;
    int group;
    bool up, down, left, right;
};

struct Group{
    vector<Cell*> members;
    vector<Cell*> last_rows;
};

int input;
int group_count = 1;
int cur_col = 0;
int draw_array[200][200];
Cell cells[90][90];
unordered_map<int, Group> groups;

void set_cell_group(int group);

void init(){
    for (int i = 0; i < input; i++){
        for (int j = 0; j < input; j++){
            cells[i][j].row = j;
            cells[i][j].col = i;
        }
    }
}

void set_row()
{
    for (int i = 0; i < input; i++)
    {
        if (cells[cur_col][i].group == 0)           // 그룹이 아직 지정되지 않았다면 새로운 그룹 생성
        {
            cells[cur_col][i].col = cur_col;
            cells[cur_col][i].row = i;
            cells[cur_col][i].group = group_count;

            groups.insert(make_pair(group_count, Group()));
            groups[group_count].members.push_back(&cells[cur_col][i]);
            groups[group_count].last_rows.push_back(&cells[cur_col][i]);

            group_count++;
        }
    }
}

void rnd_union(){
    int cur_group = -1;
    int next_group = -1;
    int dir;
    bool flag;

    for (int i = 0; i < input; i++){
        cur_group = cells[cur_col][i].group;
        flag = false;

        switch(rand() % 3){
            case 0:                     // 오른쪽 합치기
                next_group = cells[cur_col][i + 1].group;

                if (i + 1 < input && cur_group != next_group){
                    cells[cur_col][i + 1].left = true;
                    cells[cur_col][i].right = true;

                    groups[cur_group].last_rows.push_back(&cells[cur_col][i + 1]);

                    flag = true;
                }
                break;
            case 1:                     // 왼쪽 합치기
                next_group = cells[cur_col][i - 1].group;

                if (i - 1 > -1 && cur_group != next_group){
                    cells[cur_col][i - 1].right = true;
                    cells[cur_col][i].left = true;

                    groups[cur_group].last_rows.push_back(&cells[cur_col][i - 1]);

                    flag = true;
                }
                break;
            case 2:                     // 합치지 않기
                break;
        }

        if (flag){
            groups[cur_group].members.insert(groups[cur_group].members.end(), groups[next_group].members.begin(), groups[next_group].members.end());
            groups[cur_group].last_rows.insert(groups[cur_group].last_rows.end(), groups[next_group].last_rows.begin(), groups[next_group].last_rows.begin());
            groups.erase(next_group);
            set_cell_group(cur_group);
        }
    }
}

void set_cell_group(int group){
    for (int i = 0; i < groups[group].members.size(); i++){
        groups[group].members[i]->group = group;
    }
}

void group_down(){
    unordered_map<int, Group>::iterator iter;
    vector<Cell*> dest;
    int down_count = 0;
    int cur_row;

    for (iter = groups.begin(); iter != groups.end(); iter++){
        down_count = 0;

        dest.clear();
        dest.assign(iter->second.last_rows.begin(), iter->second.last_rows.end());

        groups[iter->first].last_rows.clear();

        for (int i = 0; i < dest.size(); i++){
            if (rand() % 2){
                cur_row = dest[i]->row;

                groups[iter->first].members.push_back(&cells[cur_col + 1][cur_row]);
                groups[iter->first].last_rows.push_back(&cells[cur_col + 1][cur_row]);

                cells[cur_col + 1][cur_row].up = true;
                cells[cur_col][cur_row].down = true;

                set_cell_group(iter->first);
                down_count++;
            }
        }

        if (down_count == 0 && dest.size() != 0){
            cur_row = iter->second.last_rows[rand() * 100 % dest.size()]->row;

            groups[iter->first].members.push_back(&cells[cur_col + 1][cur_row]);
            groups[iter->first].last_rows.push_back(&cells[cur_col + 1][cur_row]);

            cells[cur_col + 1][cur_row].up = true;
            cells[cur_col][cur_row].down = true;

            set_cell_group(iter->first);
        }
    }
}

void last_line(){
    for (int i = 0; i < input - 1; i++){
        if (cells[cur_col][i + 1].group != cells[cur_col][i].group){
            cells[cur_col][i].right = true;
            cells[cur_col][i + 1].left = true;
        }
    }
}

void init_array()
{
    for (int i = 0; i < input * 2 + 1; i++)
    {
        for (int j = 0; j < input * 2 + 1; j++)
        {
            draw_array[i][j] = 1;
        }
    }
}

void set_array_block()
{
    for (int i = 0; i < input; i++)
    {
        for (int j = 0; j < input; j++)
        {
            draw_array[i * 2 + 1][j * 2 + 1] = 0;
            if (cells[i][j].up)
                draw_array[i * 2][j * 2 + 1] = 0;
            if (cells[i][j].down)
                draw_array[i * 2 + 2][j * 2 + 1] = 0;
            if (cells[i][j].left)
                draw_array[i * 2 + 1][j * 2] = 0;
            if (cells[i][j].right)
                draw_array[i * 2 + 1][j * 2 + 2] = 0;
        }
    }
}

void draw_maze()
{
    for (int i = 0; i < input * 2 + 1; i++)
    {
        for (int j = 0; j < input * 2 + 1; j++)
        {
            if (draw_array[i][j] > 0)
            {
                cout << "■" << ' ';
            }
            else
                cout << "  ";
        }
        cout << endl;
    }
}

void print_group()
{
    for (int i = 0; i < input; i++)
    {
        for (int j = 0; j < input; j++)
        {
            {
                printf("%4d", cells[i][j].group);
            }
        }
        cout << endl;
    }

    cout << endl;
}

void print_maze(){
    init_array();
    set_array_block();
    draw_maze();
    print_group();
}

int main()
{

    cin >> input;

    int seed = time(NULL);
    srand(seed);

    chrono::system_clock::time_point start = chrono::system_clock::now();

    init();

    for (int i = 0; i < input - 1; i++){
        set_row();
        rnd_union();
        group_down();

        cur_col++;
    }

    chrono::duration<double> sec = chrono::system_clock::now() - start;


    set_row();
    last_line();

    print_maze();

    cout << "random seed : " << seed << endl; 
    cout << "미로 생성시 걸리는 시간 : " << sec.count() << "second" << endl;

    return 0;
}
```