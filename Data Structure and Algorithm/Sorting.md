---
title: Sorting
categories:
- 算法
---
***基于《算法导论》中的伪代码 以及部分网络资源***



# ***INSERT_SORT***

```c++
void INSERT_SORT(int *original_data, int data_size)
{
    for (int j = 1; j < data_size; j++)
    {
        int key = original_data[j];
        int i = j - 1;
        while (i >= 0 && original_data[i] > key)
        {
            original_data[i + 1] = original_data[i];
            i = i - 1;
        }
        original_data[i + 1] = key;
    }
}
```

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# ***MERGE_SORT***

```c++
void MERGE(int *original_data, int p, int q, int r)
{
    // Create two sub-arr
    int n1 = q - p + 1;
    int n2 = r - q;
    int *left_arr = new int[n1 + 1];
    int *right_arr = new int[n2 + 1];
    for (int i = 0; i < n1; i++)
    {
        left_arr[i] = original_data[p + i];
    }
    for (int j = 0; j < n2; j++)
    {
        right_arr[j] = original_data[q + j + 1];
    }
    //The last element of two arr must be infinite big
    left_arr[n1] = 10000000;
    right_arr[n2] = 10000000;

    // Sort
    int i = 0;
    int j = 0;
    for (int k = p; k <= r; k++)
    {
        if (left_arr[i] <= right_arr[j])
        {
            original_data[k] = left_arr[i];
            i++;
        }
        else
        {
            original_data[k] = right_arr[j];
            j++;
        }
    }

    //Never forget to free memory
    delete[] left_arr;
    delete[] right_arr;
}
```



**《算法导论》2.3-2 ：不使用哨兵的MERGE**

```c++
void MERGE_WZOUT_FLAG(int *original_data, int p, int q, int r)
{
    // Create two sub-arr
    int n1 = q - p + 1;
    int n2 = r - q;
    int *left_arr = new int[n1 + 1];
    int *right_arr = new int[n2 + 1];
    for (int i = 0; i < n1; i++)
    {
        left_arr[i] = original_data[p + i];
    }
    for (int j = 0; j < n2; j++)
    {
        right_arr[j] = original_data[q + j + 1];
    }
    //The last element of two arr must be infinite big
    left_arr[n1] = 10000000;
    right_arr[n2] = 10000000;

    // Sort
    int i = 0;
    int j = 0;
    int k = p;
    while (i < n1 && j < n2)
    {
        if (left_arr[i] <= right_arr[j])
        {
            original_data[k] = left_arr[i];
            k++;
            i++;
        }
        else
        {
            original_data[k] = right_arr[j];
            k++;
            j++;
        }
    }

    while (i < n1)
    {
        original_data[k] = left_arr[i];
        k++;
        i++;
    }
    while (i < n2)
    {
        original_data[k] = right_arr[j];
        k++;
        j++;
    }

    //Never forget to free memory
    delete[] left_arr;
    delete[] right_arr;
}
```



**MERGE_SORT**

```c++
void MERGE_SORT(int *original_data, int p, int r)
{
    if (p < r)
    {
        int q = (p + r) / 2;
        MERGE_SORT(original_data, p, q);
        MERGE_SORT(original_data, q + 1, r);
        //Two method
        MERGE(original_data, p, q, r);
        MERGE_WZOUT_FLAG(original_data, p, q, r);
    }
}
```

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


# ***BUBBLE_SORT***


```c++
void BUBBLE_SORT(int *original_data, int size)
{
    for (int i = 0; i < size - 1; i++)
    {
        for (int j = size - 1; j > i; j--)
        {
            if (original_data[j] < original_data[j - 1])
            {
                //Swap
                int temp = original_data[j - 1];
                original_data[j - 1] = original_data[j];
                original_data[j] = temp;
            }
        }
    }
}
```

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# ***QUICKSORT***

```c++
int PARTITION(int *arr, int p, int r)
{
    int x = arr[r];
    int i = p - 1;
    for (int j = p; j <= r - 1; j++)
    {
        if (arr[j] <= x)
        {
            i++;
            swap(arr[i], arr[j]);
        }
    }
    swap(arr[i + 1], arr[r]);
    return i + 1;
}
```

```c++
void QUICKSORT(int *arr, int p, int r)
{
    if (p < r)
    {
        int q = PARTITION(arr, p, r);
        QUICKSORT(arr, p, q - 1);
        QUICKSORT(arr, q + 1, r);
    }
}
```
随机化版本
```c++
int RANDOMIZED_PARTITION(int *arr, int p, int r)
{
    int i = p + rand() % (r - p + 1);
    swap(arr[r], arr[i]);
    return PARTITION(arr, p, r);
}
```
```c++
void RANDOMIZED_QUICKSORT(int *arr, int p, int r)
{
    if (p < r)
    {
        int q = RANDOMIZED_PARTITION(arr, p, r);
        RANDOMIZED_QUICKSORT(arr, p, q - 1);
        RANDOMIZED_QUICKSORT(arr, q + 1, r);
    }
}
```

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# ***COUNTING_SORT***

```c++
int *COUNTING_SORT(int *arr, int max_value, int len)
{
    //initialization
    int *res = new int[len];
    int *temp_arr = new int[max_value + 1];
    for (int i = 0; i < max_value + 1; i++)
    {
        temp_arr[i] = 0;
    }

    /*let temp_arr[i] contains the number of elements equal to i
    ie. arr={2,5,3,0,2,3,0,3} then temp_arr={2,0,2,3,0,1}
    */
    for (int i = 0; i < len; i++)
    {
        temp_arr[arr[i]]++;
    }

    /*let temp_arr[i] contains the number of elements less than or equal to i
    ie. arr={2,5,3,0,2,3,0,3} then temp_arr={2,2,4,7,7,8}
    */
    for (int i = 1; i < max_value + 1; i++)
    {
        temp_arr[i] += temp_arr[i - 1];
    }

    //sort
    for (int i = len - 1; i >= 0; i--)
    {
        res[temp_arr[arr[i]] - 1] = arr[i];
        //elements may be same
        temp_arr[arr[i]]--;
    }

    delete[] temp_arr;
    return res;
}
```

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------