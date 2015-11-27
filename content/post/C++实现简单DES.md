+++
date = "2015-11-27T15:35:46+08:00"
draft = false
title = "C++实现简单DES"
tags = ["C++"]

+++

# 用C++实现简单DES加密
学习密码知识遇到分组密码加密方式DES,尝试用C++写了一下程序，过程中遇到了很多问题，也学到了很多新知识

目标功能是输入十六位十六进制数，和十六位十六进制数的密钥经过加密输出密文和每一轮加密的密钥。首先要把十六进制数转化成二进制０１形式，然后开始做变换。开始的想法是用数组保存二进制数，后来发现变换时数组长度不确定，不易初始化。后来想到可以用string类保存０１序列。

过程中用到了string类的初始化`string str;`
insert，pushback，getInt， getString等方法过程还是挺有意思的此外还学回来如何快速地调试。代码写的比较麻烦，以后有时间再重构一下(实在太长)。
代码如下：
		// DES.cpp : Defines the entry point for the console application.
		//
	
		#include "stdafx.h"
		#include <iostream>
		#include <string>
		#include <typeinfo>
		#include <vector>
		using namespace std;
		string Etransform(string R);    //E变换
		string N16ToN2(string N16);    //十六进制数字转换成二进制字符。 ok
		string N2ToN16(string N2);     //二进制数转换成十六进制。
		void getC0D0(string k,string& C0,string& D0);		 //用初始密钥生成C0和D0
		string STransform(string k,string e);  //取异或 + S盒变换  返回S32个字节
		string PTransform(string S);             //P变换 +与Li取异或 返回32字节Ri
		int getInt(string str);					 //string转换成int "0001"-> 1
		string getString(int n);				 //int转换成string
		void Ftransform(string& L, string& R, string k16, int i);   //圈函数
		string getKey(string& Ci, string& Di, int i);     //由Ci，Di得到密钥Ki
	
	
		int _tmain(int argc, _TCHAR* argv[]){
		
		  	string M16;							//输入的十六进制数字
			string m16;
			string K16;
			string k16;
			string key;     //密钥48位
			string R0;      //初始化R0
			string L0;      //
			string C0;      //密钥扩展中的C0;
			string D0;		//密钥扩展中的D0;
			int i;
			cout << "输入一个十六位的十六进制的明文(以0x开头)" << endl;
			getline(cin, M16);
			while(M16.length() != 18){
				cout << "输入错误，重新输入";
				getline(cin, M16);
			}
			cout << M16 << endl;
			cout << "输入十六位密钥（以0x开头）" << endl;
			getline(cin, K16);
			while (K16.length() != 18){
				cout << "输入错误，重新输入" << endl;
				getline(cin, K16);
			}
			k16.insert(0,N16ToN2(K16));		//密钥对应的二进制数64
			m16.insert(0,N16ToN2(M16));		//明文对应的二进制数64
			L0.insert(0, m16, 0, 32);
			R0.insert(0, m16, 32, 32);
			getC0D0(k16, C0, D0);           //得到C0，D0
			for (i = 1; i < 16; i++){
				key.insert(0, getKey(C0, D0, i));
				Ftransform(L0, R0, key, i);
				cout <<"第"<<i<<"次加密"<< endl;
			}
			key.insert(0, getKey(C0, D0, i));
			Ftransform(R0, L0, key, i);
			cout << "第" <<16<< "次加密" << endl;
			system("pause");
			return 0;
		}
	
		string N16ToN2(string N16){
			string n16;							//转换成二进制数字，用字符串表示。
			int i;
			for (i = 2; i < 18; i++){
				int a;
				a = N16[i] - '0';
				switch (a)
				{
				case 0: n16 += "0000"; break;
				case 1: n16 += "0001"; break;
				case 2: n16 += "0010"; break;
				case 3: n16 += "0011"; break;
				case 4: n16 += "0100"; break;
				case 5: n16 += "0101"; break;
				case 6: n16 += "0110"; break;
				case 7: n16 += "0111"; break;
				case 8: n16 += "1000"; break;
				case 9: n16 += "1001"; break;
				case 17: n16 += "1010"; break;
				case 18: n16 += "1011"; break;
				case 19: n16 += "1100"; break;
				case 20: n16 += "1101"; break;
				case 21: n16 += "1110"; break;
				case 22: n16 += "1111"; break;
				}	//每个十六进制数可以表示成4个二进制数
			}
			return n16;
		} 
	
		string Etransform(string R){
			string temp;
			int i,j;
			for (i = 1; i < 8; i++){
				temp.push_back(R[4*i-1]);
				temp.push_back(R[4*i]);
			}
			temp.push_back(R[31]);
			temp.push_back(R[0]);
			for (j = 0; j < 7; j++){
				R.insert(6*j+5, temp, 2*j, 2);
			}
			R.push_back(temp[15]);
			R.insert(0, temp,14,1);   //E变换，36位R变成48位
			return R;
		}
		
		void getC0D0(string k, string& C0,string& D0){
			C0.push_back(k[56]);		//64位密钥k 经PC-1拣选变换得到C0，D0
			C0.push_back(k[48]);
			C0.push_back(k[40]);
			C0.push_back(k[32]);
			C0.push_back(k[24]);
			C0.push_back(k[16]);
			C0.push_back(k[8]);
			C0.push_back(k[0]);
			C0.push_back(k[57]);
			C0.push_back(k[49]);
			C0.push_back(k[41]);
			C0.push_back(k[33]);
			C0.push_back(k[25]);
			C0.push_back(k[17]);
			C0.push_back(k[9]);
			C0.push_back(k[1]);
			C0.push_back(k[58]);
			C0.push_back(k[50]);
			C0.push_back(k[42]);
			C0.push_back(k[34]);
			C0.push_back(k[26]);
			C0.push_back(k[18]);
			C0.push_back(k[10]);
			C0.push_back(k[2]);
			C0.push_back(k[59]);
			C0.push_back(k[51]);
			C0.push_back(k[43]);
			C0.push_back(k[35]);
			D0.push_back(k[62]);
			D0.push_back(k[54]);
			D0.push_back(k[46]);
			D0.push_back(k[38]);
			D0.push_back(k[30]);
			D0.push_back(k[22]);
			D0.push_back(k[14]);
			D0.push_back(k[6]);
			D0.push_back(k[61]);
			D0.push_back(k[53]);
			D0.push_back(k[45]);
			D0.push_back(k[37]);
			D0.push_back(k[29]);
			D0.push_back(k[21]);
			D0.push_back(k[13]);
			D0.push_back(k[5]);
			D0.push_back(k[60]);
			D0.push_back(k[52]);
			D0.push_back(k[44]);
			D0.push_back(k[36]);
			D0.push_back(k[28]);
			D0.push_back(k[20]);
			D0.push_back(k[12]);
			D0.push_back(k[4]);
			D0.push_back(k[27]);
			D0.push_back(k[19]);
			D0.push_back(k[11]);
			D0.push_back(k[3]);     //拣选结束 得到C0,D0
		} 
	
		string STransform(string k, string e){
			int i,j,a,b,y;
			string s;			//s为密钥K与e变换后的R取异或的结果
			string S;
			string m;		//对应矩阵横坐标
			string n;		//对应矩阵纵坐标
			char c;
			for (i = 0; i < 48; i++){
				if (k[i] == e[i]) c = '0';//异或操作
				else c = '1';
				s.push_back(c);    
			}
			int S1[4][16] = { { 14, 4, 13, 1, 2, 15, 11, 8, 3, 10, 6, 12, 5, 9, 0, 7 },
							{ 0, 15, 7, 4, 14, 2, 13, 1, 10, 6, 12, 11, 9, 5, 3, 8 },
							{ 4, 1, 14, 8, 13, 6, 2, 11, 15, 12, 9, 7, 3, 10, 5, 0 },			//S1
							{ 15, 12, 8, 2, 4, 9, 1, 7, 5, 11, 3, 15, 10, 0, 6, 13 } };
		
			int S2[4][16] = { { 15, 1, 8, 14, 6, 11, 3, 4, 9, 7, 2, 13, 12, 0, 5, 10 },
							{ 3, 13, 4, 7, 15, 2, 8, 14, 12, 0, 1, 10, 6, 9, 11, 5 },
							{ 0, 14, 7, 11, 10, 4, 13, 1, 5, 8, 12, 6, 9, 3, 2, 15 },			//S2
							{ 13, 8, 10, 1, 3, 15, 4, 2, 11, 6, 7, 12, 0, 5, 14, 9 } };
	
			int S3[4][16] = { { 10, 0, 9, 14, 6, 3, 15, 5, 1, 13, 12, 7, 11, 4, 2, 8 },
							{ 13, 7, 0, 9, 3, 4, 6, 10, 2, 8, 5, 14, 12, 11, 15, 1 },
							{ 13, 6, 4, 9, 8, 15, 3, 0, 11, 1, 2, 12, 5, 10, 14, 7 },			//S3
							{ 1, 10, 13, 0, 6, 9, 8, 7, 4, 15, 14, 3, 11, 5, 2, 12 } };
	
			int S4[4][16] = { { 7, 13, 14, 3, 0, 6, 9, 10, 1, 2, 8, 5, 11, 12, 4, 15 },
							{ 13, 8, 11, 5, 6, 15, 0, 3, 4, 7, 2, 12, 1, 10, 14, 9 },
							{ 10, 6, 9, 0, 12, 11, 7, 13, 15, 1, 3, 14, 5, 2, 8, 4 },			//S4
							{ 3, 15, 0, 6, 10, 1, 13, 8, 9, 4, 5, 11, 12, 7, 2, 14 } };
		
			int S5[4][16] = { { 2, 12, 4, 1, 7, 10, 11, 6, 8, 5, 3, 15, 13, 0, 14, 9 },
							{ 14, 11, 2, 12, 4, 7, 13, 1, 5, 0, 15, 10, 3, 9, 8, 6 },
							{ 4, 2, 1, 11, 10, 13, 7, 8, 15, 9, 12, 5, 6, 3, 0, 14 },			//S5
							{ 11, 8, 12, 7, 1, 14, 2, 13, 6, 15, 0, 9, 10, 4, 5, 3 } };
	
			int S6[4][16] = { { 12, 1, 10, 15, 9, 2, 6, 8, 0, 13, 3, 4, 14, 7, 5, 11 },
							{ 10, 15, 4, 2, 7, 12, 9, 5, 6, 1, 13, 14, 0, 11, 3, 8 },
							{ 9, 14, 15, 5, 2, 8, 12, 3, 7, 0, 4, 10, 1, 13, 11, 6 },			//S6
							{ 4, 3, 2, 12, 9, 5, 15, 10, 11, 14, 1, 7, 6, 0, 8, 13 } };
		
			int S7[4][16] = { { 4, 11, 2, 14, 15, 0, 8, 13, 3, 12, 9, 7, 5, 10, 6, 1 },
							{ 13, 0, 11, 7, 4, 9, 1, 10, 14, 3, 5, 12, 2, 15, 8, 6 },
							{ 1, 4, 11, 13, 12, 3, 7, 14, 10, 15, 6, 8, 0, 5, 9, 2 },			//S7
							{ 6, 11, 13, 8, 1, 4, 10, 7, 9, 5, 0, 15, 14, 2, 3, 12 } };
	
			int S8[4][16] = { { 13, 2, 8, 4, 6, 15, 11, 1, 10, 9, 3, 14, 5, 0, 12, 7 },
							{ 1, 15, 13, 8, 10, 3, 7, 4, 12, 5, 6, 11, 0, 14, 9, 2 },
							{ 7, 11, 4, 1, 9, 12, 14, 2, 0, 6, 10, 13, 15, 3, 5, 8 },			//S8
							{ 2, 1, 14, 7, 4, 10, 8, 13, 15, 12, 9, 0, 3, 5, 6, 11 } };
	
			for (j = 0; j < 8; j++){  //8个S-盒
				m.push_back(s[6 * j]);
				m.push_back(s[6 * j + 5]);
				n.insert(0, s, 6 * j, 4);
				a = getInt(m);		//矩阵横坐标
				b = getInt(n);		//矩阵纵坐标
				switch (j)
				{
				case 0: y = S1[a][b]; break;
				case 1: y = S2[a][b]; break;
				case 2: y = S3[a][b]; break;
				case 3: y = S4[a][b]; break;
				case 4: y = S5[a][b]; break;
				case 5: y = S6[a][b]; break;
				case 6: y = S7[a][b]; break;
				case 7: y = S8[a][b]; break;
				}
	
				S += getString(y);
				m.erase();
				n.erase();
			}
			return S;
		}
	
		int getInt(string str){
			int n,temp,a,b,c,d;
			temp = stoi(str);
			a = temp % 10;
			b = ((temp - a)/10) % 10;
			c = ((temp - a - 10 * b)/100) % 10;
			d = (temp - a - 10 * b - 100 * c) / 1000;
			n = a + 2 * b + 4 * c + 8 * d;
			return n;
		}
	
	
		string getString(int n){
			string str;
			switch (n)
			{
			case 0: str = "0000"; break;
			case 1: str = "0001"; break;
			case 2: str = "0010"; break;
			case 3: str = "0011"; break;
			case 4: str = "0100"; break;
			case 5: str = "0101"; break;
			case 6: str = "0110"; break;
			case 7: str = "0111"; break;
			case 8: str = "1000"; break;
			case 9: str = "1001"; break;
			case 10: str = "1010"; break;
			case 11: str = "1011"; break;
			case 12: str = "1100"; break;
			case 13: str = "1101"; break;
			case 14: str = "1110"; break;
			case 15: str = "1111"; break;
			}
			return str;
		}
	
		string PTransform(string S,string L){
			string strP;
			string Ri;
			int i,c;
			strP.push_back(S[15]);
			strP.push_back(S[6]);
			strP.push_back(S[19]);
			strP.push_back(S[20]);
			strP.push_back(S[28]);
			strP.push_back(S[11]);
			strP.push_back(S[27]);
			strP.push_back(S[16]);
			strP.push_back(S[0]);
			strP.push_back(S[14]);
			strP.push_back(S[22]);
			strP.push_back(S[25]);
			strP.push_back(S[4]);
			strP.push_back(S[17]);
			strP.push_back(S[30]);
			strP.push_back(S[9]);
			strP.push_back(S[1]);
			strP.push_back(S[7]);
			strP.push_back(S[23]);
			strP.push_back(S[13]);
			strP.push_back(S[31]);
			strP.push_back(S[26]);
			strP.push_back(S[2]);
			strP.push_back(S[8]);
			strP.push_back(S[18]);
			strP.push_back(S[12]);
			strP.push_back(S[29]);
			strP.push_back(S[5]);
			strP.push_back(S[21]);
			strP.push_back(S[10]);
			strP.push_back(S[3]);
			strP.push_back(S[24]);
			for (i = 0; i < 32; i++){
			//异或操作
				if (strP[i] == L[i]) c = '0';
				else c = '1';
				Ri.push_back(c);
			}
			return Ri;
		}
	
		void Ftransform(string& L, string& R,string key,int i){
			string temp;
			string SS;		//表示S-盒输出
			string EE;		//表示E变换输出
			string LR;		//加密一次的密文（二进制形式）
			string MW;     //输出的密文（十六进制形式）
			string KEY;    //输出的密钥（十六进制形式） 
			int j,m;
			temp.insert(0, R);
			EE.insert(0,Etransform(R));
			SS.insert(0,STransform(key, EE));
			R.erase();     //清空R
			R.insert(0, PTransform(SS, L));
			L.erase();       //清空L
			L.insert(0, temp);       //R变成L
			LR = L + R;
			cout << "密文是： ";
			for (j = 0; j < 16; j++){
				MW.erase();							//初始化MW
				MW.insert(0, LR, 4 * j, 4);     //LR每4位取出来成一个十六进制的字符
				cout << N2ToN16(MW);				//输出密文
			}
			cout << endl;
			cout << "密钥是：";
			for (m = 0; m < 12; m++){
				KEY.erase();
				KEY.insert(0, key, 4 * m, 4);
				cout << N2ToN16(KEY);
			}
			cout << endl;
		}
	
		string N2ToN16(string N2){
			string N16;
			int n; 
			n = getInt(N2);         //转换为整数
			switch (n)
			{
			case 0: N16.push_back('0'); break;
			case 1: N16.push_back('1'); break;
			case 2: N16.push_back('2'); break;
			case 3: N16.push_back('3'); break;
			case 4: N16.push_back('4'); break;
			case 5: N16.push_back('5'); break;
			case 6: N16.push_back('6'); break;
			case 7: N16.push_back('7'); break;
			case 8: N16.push_back('8'); break;
			case 9: N16.push_back('9'); break;
			case 10: N16.push_back('A'); break;
			case 11: N16.push_back('B'); break;
			case 12: N16.push_back('C'); break;
			case 13: N16.push_back('D'); break;
			case 14: N16.push_back('E'); break;
			case 15: N16.push_back('F'); break;
			}
			return N16;
		}
	
		string getKey(string& Ci, string& Di,int i){
			string C1;
			string D1;
			string key;
			if (i == 1 || i == 2 || i == 9 || i == 16){   //当i=1,2,9,16时左移一位
				C1.push_back(Ci[27]);
				D1.push_back(Di[27]);
				C1.insert(1, Ci, 0, 27);
				D1.insert(1, Di, 0, 27);
			}
			else{										//其他情况左移两位
				string temp1;
				string temp2;
				temp1.insert(0, Ci, 0, 2);
				temp2.insert(0, Di, 0, 2);
				C1.insert(0, temp1);
				D1.insert(0, temp2);
				C1.insert(2, Ci, 0, 26);
				D1.insert(2, Di, 0, 26);
			}
			key.push_back(C1[13]);			//PC-2拣选变换
			key.push_back(C1[16]);
			key.push_back(C1[10]);
			key.push_back(C1[23]);
			key.push_back(C1[0]);
			key.push_back(C1[4]);
			key.push_back(C1[2]);
			key.push_back(C1[27]);
			key.push_back(C1[14]);
			key.push_back(C1[5]);
			key.push_back(C1[20]);
			key.push_back(C1[9]);
			key.push_back(C1[22]);
			key.push_back(C1[18]);
			key.push_back(C1[11]);
			key.push_back(C1[3]);
			key.push_back(C1[25]);
			key.push_back(C1[7]);
			key.push_back(C1[15]);
			key.push_back(C1[6]);
			key.push_back(C1[26]);
			key.push_back(C1[19]);
			key.push_back(C1[12]);
			key.push_back(C1[1]);
			key.push_back(D1[12]);
			key.push_back(D1[23]);
			key.push_back(D1[2]);
			key.push_back(D1[8]);
			key.push_back(D1[18]);
			key.push_back(D1[26]);
			key.push_back(D1[1]);
			key.push_back(D1[11]);
			key.push_back(D1[22]);
			key.push_back(D1[16]);
			key.push_back(D1[4]);
			key.push_back(D1[19]);
			key.push_back(D1[15]);
			key.push_back(D1[20]);
			key.push_back(D1[10]);
			key.push_back(D1[27]);
			key.push_back(D1[5]);
			key.push_back(D1[24]);
			key.push_back(D1[17]);
			key.push_back(D1[13]);
			key.push_back(D1[21]);
			key.push_back(D1[7]);
			key.push_back(D1[0]);
			key.push_back(D1[3]);
	
			Ci.erase();
			Di.erase();
			Ci.insert(0, C1);
			Di.insert(0, D1);
			return key;
		}
	    
