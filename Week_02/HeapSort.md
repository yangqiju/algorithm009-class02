### HeapSort
  
    public class HeapSort {
    
    // Driver program
    public static void main(String args[]) {
        int arr[] = {12, 11, 13, 5, 6, 7};
        int n = arr.length;

        HeapSort ob = new HeapSort();
        ob.sort(arr);

        System.out.println("Sorted array is");
        System.out.println(Arrays.toString(arr));
    }

    public void sort(int arr[]) {
        int len = arr.length;
        //使用大顶堆
        for (int i = len / 2 - 1; i >= 0; i--) {
            this.buildheap(arr, len, i);
        }
        //排好堆后,从末尾开始替换,将最大的替换到数组后,替换到最后就是排序号的内容
        for (int i = len - 1; i > 0; i--) {
            int swap = arr[i];
            arr[i] = arr[0];
            arr[0] = swap;

            this.buildheap(arr, i, 0);
        }

    }

    private void buildheap(int arr[], int n, int rootIndex) {
        int leftIndex = rootIndex * 2 + 1;
        int rightIndex = rootIndex * 2 + 2;

        int tmpIndex = rootIndex;
        if (leftIndex < n && arr[leftIndex] > arr[rootIndex]) {
            tmpIndex = leftIndex;
        }
        if (rightIndex < n && arr[rightIndex] > arr[tmpIndex]) {
            tmpIndex = rightIndex;
        }

        if (tmpIndex != rootIndex) {
            //需要换

            int swap = arr[tmpIndex];
            arr[tmpIndex] = arr[rootIndex];
            arr[rootIndex] = swap;

            this.buildheap(arr, n, tmpIndex);
        }
    }


}
