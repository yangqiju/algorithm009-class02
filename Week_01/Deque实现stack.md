学习笔记


### 通过push、peek、pop的stack方法使用Deque

      Deque<String> deque = new LinkedList<>();
      deque.push("a");//push 内部实现是 addFirst
      deque.push("b");
      deque.push("c");
      System.out.println(deque);//[c, b, a]

      String str = deque.peek();
      System.out.println(str);
      System.out.println(deque);

      while (deque.size() > 0) {
          System.out.println(deque.pop()); //内部实现为  removeFirst
      }
      System.out.println(deque);

      //push 和 pop 操作的都是first node
      
### 通过addFirst、getFirst、pollFirst 替换stack操作
      Deque<String> deque = new LinkedList<>();
      deque.addFirst("a");
      deque.addFirst("b");
      deque.addFirst("c");
      System.out.println(deque);

      String str = deque.getFirst();
      System.out.println(str);
      System.out.println(deque);

      while (deque.size() > 0) {
          System.out.println(deque.pollFirst());
      }
      System.out.println(deque);

### 总结
1. 使用stack的时候通常会使用push和pop的操作,`Deque`能实现相应的功能,官方建议使用`Deque`来做替换(https://docs.oracle.com/javase/10/docs/api/java/util/Stack.html)

2. deque中使用offer与addLast实现相同,内部实现为linkLast(e),poll与pollFirst接口内容相同, 内部实现为unlinkFirst.
表现不同的是addFirst/removeFirst/getFirst处理失败会抛出exception，而offerFirst/pollFirst/peekFirst会
返回特殊的值，例如`LinkedBlockingDeque`会返回true/false(https://docs.oracle.com/javase/10/docs/api/java/util/Deque.html)
