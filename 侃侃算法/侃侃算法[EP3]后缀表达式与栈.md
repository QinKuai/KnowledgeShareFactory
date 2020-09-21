## 侃侃算法EP3·后缀表达式与栈

### 1. 前言

- 这个板块旨在记录一些日常中或是面试中会问到的算法和数据结构相关的内容，更多是给自己总结和需要的人分享。在内容部分可能由于我的阅历和实战经历不足，会有忽视或是写错的点，还望轻喷。



### 2. 内容

- 日常生活中常使用的表达式是**中缀表达式**，也就是**标准的四则运算表达式**，例如9+(3-1)*3+10/2这类
- 但计算机并不能很好地理解中缀表达式，对于计算机**后缀表达式**更容易理解一些。

#### 2.1 什么是后缀表达式

  - 后缀表达式也称**逆波兰式**，是通过中缀表达式通过一定规则转化而来的。
  - 规则如下：
  - 从左到右遍历中缀表达式的每一个数字和符号，
      - 若是数字就输出，成为后缀表达式的一部分。
      - 若是符号，则判断该符号与栈顶符号的优先级，
          - 若是**右括号**或是**优先级不高于栈顶符号**（乘除优于加减），则将栈顶元素依次出栈并输出，然后将当前符号入栈

  ```java
private static Map<Character, Integer> priorityMap = new HashMap<>();
	
static {
	priorityMap.put('+', 0);
	priorityMap.put('-', 0);
	priorityMap.put('*', 1);
	priorityMap.put('/', 1);		
}

// 测试代码
public static void main(String[] args){
	String line = "9+(3-1)*3+10/2";
    // 输出结果为931-3*+102/+
    // 关于多位数的判定问题，实际上可以通过分隔符解决
    // 但不是这里谈论的内容，我就不细说了
	System.out.println(rPN(line));
}
	
// 以整数四则为例
// 输入为包含中缀表达式的字符串
// 输出为包含输入对应的后缀表达式的字符串
public static String rPN(String line) {
	Deque<Character> stack = new ArrayDeque<>();
	StringBuffer sBuffer = new StringBuffer();
	char[] chars = line.toCharArray();
	char nowChar;
		
	for (int i = 0; i < chars.length; i++) {
		nowChar = chars[i];
		if(isDigit(nowChar)) {
			sBuffer.append(nowChar);
		}else {
			if (stack.isEmpty()) {
					stack.push(nowChar);
			}else {
				if(nowChar == '(') {
					stack.push(nowChar);
				}else if (nowChar == ')') {
					while (stack.peek() != '(') {
						sBuffer.append(stack.pop());
					}
					stack.pop();
				}else {
					Character top = stack.peek();
					if(top != '(' && isNotHigher(nowChar, top)){
						while (top != null && top != '(') {
							sBuffer.append(stack.pop());
							top = stack.peek();
						}
					}
					stack.push(nowChar);
				}
			}
		}
	}
		
	while (!stack.isEmpty()) {
		sBuffer.append(stack.pop());
	}
		
	return sBuffer.toString();
}
	
public static boolean isDigit(char c) {
	return (c >= '0' && c <= '9') ? true : false;
}
	
public static boolean isNotHigher(char oper1, char oper2) {
	return priorityMap.get(oper1) <= priorityMap.get(oper2) ? true : false;
}
  ```

