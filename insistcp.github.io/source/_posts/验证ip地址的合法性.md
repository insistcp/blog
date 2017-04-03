---
title: 验证ip地址的合法性
date: 2017-03-14 14:11:01
tags: [算法]
categories: 数学 #文章文类
---
### 问题描述：
	请解析IP地址和对应的掩码，进行分类识别。要求按照A/B/C/D/E类地址归类，不合法的地址和掩码单独归类。
<!-- more -->，
	所有的IP地址划分为 A,B,C,D,E五类
	A类地址1.0.0.0~126.255.255.255;
	B类地址128.0.0.0~191.255.255.255;
	C类地址192.0.0.0~223.255.255.255;
	D类地址224.0.0.0~239.255.255.255；
	E类地址240.0.0.0~255.255.255.255
	私网IP范围是：
	10.0.0.0～10.255.255.255
	172.16.0.0～172.31.255.255
	192.168.0.0～192.168.255.255
	子网掩码为前面是连续的1，然后全是0。（例如：255.255.255.32就是一个非法的掩码）
	本题暂时默认以0开头的IP地址是合法的，比如0.1.1.2，是合法地址
	输入描述:
	多行字符串。每行一个IP地址和掩码，用~隔开。
	输出描述:
	统计A、B、C、D、E、错误IP地址或错误掩码、私有IP的个数，之间以空格隔开。
	输入例子:
	10.70.44.68~255.254.255.0
	1.0.0.1~255.0.0.0
	192.168.0.2~255.255.255.0
	19..0.~255.255.255.0
	输出例子:
	1 0 1 0 0 2 1
### 解题思路
+ 1：现将我们输入的字符截断（“~”）,分别存于一个字符串数组中
+ 2：判断子网掩码的合法性，
+ 3：判断IP地址的属于哪一类别（当然这里要验证每一个IP地址的合法性），需注意这里每一个IP地址合法而且他们的子网掩码也要合法才算是一个合适的算法
+ 4：根据上面的判断做出相应的分类
### 上代码
```java 
package day1;

import java.util.Scanner;

public class Main {
	/**
	 * 存储各个类别的数量
	 */
	private static int A=0;
	private static int B=0;
	private static int C=0;
	private static int D=0;
	private static int E=0;
	private static int falseIP=0;
	private static int PriCount=0;
	/**
	 * 验证所有IP地址和子网掩码的各个数字是否在0-255之间
	 * @param netIP
	 * @return
	 */
	public static boolean verfityIPRange(String netIP){
		if(netIP==null || "".equals(netIP)){
			return false;
		}
		String[] netSub = netIP.split("\\.");
		int len = netSub.length;
		if(len!=4){
			return false;
		}else{
			for(int i = 0;i<len;i++){
				String temp1 = netSub[i].trim();
				if("".equals(temp1)){
					return false;
				}
				int temp = Integer.parseInt(temp1);
				if(temp<0||temp>255){
					return false;
				}
			}
		}
		return true;
	}
	/**
	 * 将子网掩码转换成二进制
	 * @param str
	 * @return
	 */
	public static String VerfitymaskCode(String str){
		if(null==str||"".equals(str)){
			return null;
		}
		StringBuilder sb = new StringBuilder();
		String temp1 = str.trim();
		if("".equals(temp1)){
			return null;
		}
		int temp  = Integer.parseInt(temp1);
		String value;
		for(int i=0;i<8;i++){
			int flag = 128;
			value = (flag&temp)==0?"0":"1"; 
			//注意这里的移位运算：意思是不断的让待转换数向左移一位：然和和一作比较，如果相等则
			temp<<=1;
			sb.append(value);
		}
		return  sb.toString();
	}
	/**
	 * 验证子网掩码的合法性
	 * @param str
	 * @return
	 */
	public static boolean verityIllegal(String str){
		if(!verfityIPRange(str)){
			return false;
		}
		StringBuilder maskCode = new StringBuilder();
		String[] temp = str.split("\\.");
		for(int i=0;i<4;i++){
			maskCode.append(VerfitymaskCode(temp[i]));
		}
		int oneIndex = maskCode.lastIndexOf("1");
		int zeroIndex = maskCode.indexOf("0");
		if(zeroIndex<oneIndex){
			return false;
		}
		return true;
	}
	/**
	 * 分类
	 * @param arrIP
	 */
	public static void classfity(String[] arrIP){
		if(arrIP.length!=2){
			return;
		}
		String frontNet = arrIP[0];
		String endNet = arrIP[1];
		/**
		 * 一旦范围和子网掩码都符合那么它一定是一个好的IP地址
		 */
		if(verfityIPRange(frontNet) && verityIllegal(endNet)){
			String[] tempIP = frontNet.split("\\.");
			int index1 = Integer.parseInt(tempIP[0]);
			int index2 = Integer.parseInt(tempIP[1]);
			if(0<index1&&index1<=126){
				if(index1==10){
					PriCount++;
				}
				A++;
			}
			if(128<=index1&&index1<=191){
				if(index1==172){
					if(16<=index2&&index2<32){
						PriCount++;
					}
				}
				B++;
			}
			if(192<=index1&&index1<=223){
				if(index1==192){
					if(index2==168){
						PriCount++;
					}
				}
				C++;
			}
			if(224<=index1&&index1<=239){
				D++;
			}
			if(240<=index1&&index1<=255){
				E++;
			}
		}else{
			falseIP++;
		}
	}
	public static void main(String[] args) {
		Scanner in = new Scanner(System.in);
		String[] arrIP = new String[2];
		while(in.hasNextLine()){
			String str = in.nextLine();
			arrIP = str.split("\\~");
			classfity(arrIP);
		}
		in.close();
		System.out.println(A+" "+B+" "+C+" "+D+" "+E+" "+falseIP+" "+PriCount);
	}
}

```