# 算法模板

## 1 排序

### 1.1 快速排序

```c++
void quick_sort(int q[], int l, int r)
{
    if (l >= r) return;

    int i = l - 1, j = r + 1, x = q[l + r >> 1];
    while (i < j)
    {
        do i ++ ; while (q[i] < x);
        do j -- ; while (q[j] > x);
        if (i < j) swap(q[i], q[j]);
    }
    quick_sort(q, l, j), quick_sort(q, j + 1, r);
}
void quickSort2(int[] q, int l, int r) {
        if (l >= r) {
            return;
        }
        int i = l - 1;
        int j = r + 1;
        int x = q[l + r >> 1];
        while (i < j) {
            while (q[++i] < x) ;
            while (q[--j] > x) ;
            if (i < j) {
                swap(q, i, j);
            }
        }
        quickSort2(q, l, j);
        quickSort2(q, j + 1, r);
    }
```

### 1.2 归并排序