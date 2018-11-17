﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿**CCAOP**>endow your c++ code the ability of AOP, without even altering or recompling it.**Introduction**---With the prosperity of Internet, Spring has became very popular among the Java developers. AOP, as  a pillar of Spring's key features, has been used massively in Java as a design pattern that C++, by and large, has not.This is a tool to generate boilerplate C++ code﻿, which can hook any C++ class member function( not inline ) you specify.You can write your own code, run before or after the function you hook without even altering or recompiling the program you want to hook. This endows you the ability of AOP in C++ world, coarsely though.**Getting Started**---In the sample folder, there are some cpp and head files:> foo.h```#include <iostream>#include <string>using namespace std;class Foo {	 public:	 	void showMessage();};```---> foo.cpp```#include "foo.h"void Foo::showMessage(){	cout<<"Hello World!"<<endl;}```---_using this command to complie the dynamic library:_> gcc foo.cpp -shared -fPIC -o libfoo.so -g   _you will get libfoo.so in your working directory_---> test.cpp```#include "foo.h"int main(){	Foo x;	x.showMessage();	return 0;}```_using this command to complie the executable file:_>  g++ test.cpp -o ccaoptest -lfoo -L./> export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:./> ./ccaoptest_you can get the legendary "Hello World" in your console:_> [wuzk@localhost wuzk]$ ./ccaoptest> Hello World!The code above is a simple c++ code, with a class Foo, and it's member function showMessage.Now we are going to hook it with other code,Here we go:---copy ccaop.py to your directory, using commad as bellow: > python ccaop.py -h foo.h -f "void Foo::showMessage()" -l ./libfoo.so -o bar.cpp> > Generating the boilerplate code for your aop library...> Using these options:> ** function     : void Foo::showMessage()> ** namespace    :  > ** libname      : ./libfoo.so> ** headfile     : foo.h> ** cppname      : bar.cpp> Are you gonna hook this function ?> Foo::showMessage()> if it is the right function press y,else press other key:yPress Y if it is  the function you want to hook, press other key if it is not.CCAOP will show you all the function overloaded, choose the one you really want to hook.After you press Y, ccaop will generate the boilerplate code:> bar.cpp```//code generated by ccaop//https://github.com/wzk784533/CCAOP#include <iostream>#include <string>/*you can comment the header files above,if you don't need it in you aop library*/#include <dlfcn.h>#include "foo.h"/*In some complex system, you might need some more other head files,which defines the class in your function paramater.Before compling this code,you must include these head files, avoiding type declariton error complains in complier.*/using namespace std;void Foo::showMessage(){   typedef void (*pFunc)(Foo *);   static void *handle = NULL;   static void *func = NULL;   if(!func)   {      handle = dlopen("./libfoo.so", RTLD_LAZY);      func=dlsym(handle,"_ZN3Foo11showMessageEv");      dlclose(handle);   }   /* AOP before */   /* your code starts here */   cout<<"Before hook"<<endl;			   //real function call   pFunc realFunc=(pFunc)(func);   realFunc(this);		   /* AOP after */   /* your code starts here */    cout<<"after hook"<<endl;   return;}```_using this command to complie the  hook library:_> g++ bar.cpp -ldl -fPIC -shared -o libbar.so> export LD_PRELOAD=/home/wuzk/wuzk/libbar.soNow we check the result:> ./ccaoptest> [wuzk@localhost wuzk]$ ./ccaoptest> Before hook> Hello World!> after hookThe code we add, just run, without recompling the executable file ccaoptest