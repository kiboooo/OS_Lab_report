##  实验一: 线程管理    
####  一． 实验目的   
   >  * 通过观察、分析实验现象，深入理解线程及线程在调度执行和内存空间等方面的特点
   >  * 掌握线程与进程的区别。
   >   * 掌握在POSIX 规范中pthread_create() 函数的功能和使用方法。 
####  二．实验要求  
##### 2.1 实验环境要求 
1. 硬件  
	 > *  主机：Pentium III 以上
    > *  内存：128MB 以上
    > *  显示器：VGA 或更高
    > * 硬盘空间：至少100MB 以上剩余空间

2. 软件  
> * Linux 操作系统，内核2.4.26 以上，预装有X-Window 、vi、gcc、gdb 和任 意web 浏览器。 
3. 实验前的准备工作  阅读参考资料，了解线程的创建等相关系统调用。  
#### 三、实验内容   
##### 3.1 补充POSIX 下进程控制的残缺版实验程序 
##### 3.2回答下列问题：  
1. 你最初认为前三列数会相等吗？最后一列斜杠两边的数字是相等，还是大于或者  小于关系？  
2.  最后的结果如你所料吗？有什么特点？试对原因进行分析。  
3.  thread 的CPU 占用率是多少？为什么会这样？   
4.  thread_worker()内是死循环，它是怎么退出的？你认为这样退出好吗？
#### 四、实验结果    
##### 4.1 补充完全的源程序   
	#include<stdio.h>
	#include<sys/types.h> 
	#include<unistd.h>  
	#include<ctype.h> 
	#include<pthread.h>  
	#define MAX_THREAD 3/* 线程的个数 */  
	unsigned long long main_counter,counter[MAX_THREAD]; 
	 /* unsigned long long是比long还长的整数 */ 
	void* thread_worker(void*); 
	  int main(int argc,char argv[]){  
			int i,rtn;  
		   char ch;   
		  pthread_t pthread_id[MAX_THREAD]={0};/* 存放每个线程的id */  
		   for(i=0;i<MAX_THREAD;i++){ 
     pthread_create(&pthread_id[i],NULL,thread_worker,(void*)i);  
     /*用pthread_create建一个普通的线程， 线程id存入pthread_id[i]， 线程执行的函数是thread_worker，并i作为参数传递给线程 */  
     }  /* 用户按一次回车执行下面的循环体一次。按q退出 */  
     do{
        unsigned long long sum=0;
        for(i=0;i<MAX_THREAD;i++){    
        sum+=counter[i]; /* 求所有线程的counter的和 */
        printf("counter[%d]=%llu\n",i,counter[i]);  
        }
		printf("main_counter=%llu/sum=%llu\n",main_counter,sum);  }while((ch=getchar())!='q'); 
		return 0; 
	 }  
		 void* thread_worker(void* p){  
		 int thread_num;   
		 thread_num=(int)p; /*把main中的i的值传递给thread_num */  
		 for(;;){   
		 main_counter++;   
	 counter[thread_num]++;    
	  } 
	}
##### 4.3 回答上述实验内容中的问题  
 1． 试验运行前我认为前三列数不会相等，因为三个线程运行次数是随机的，结果不可预料，当然counter[i]值不会一定相等。而我认为main_counter与sum值应该是相等的。因为都是三个线程的counter之和。   

2．而实验结果是前三列数确实不相等。不过`main_counter`与`sum`的值也不相等，`main_counter < sum`，经分析讨论得出解释：因为三个线程在共同争取运行`thread_worker()`函数，比如`main_counter`初值为0，`pthread_id[0]`执行之后`main_counter+1`，此时还未来得及将值赋给`main_counter`，这时的`main_counter`还是0；`pthread_id[1]`也执行这个函数，`main_counter+1`，若此时在1号线程将`main_counter+1`的值还未赋给`main_counter`，即这时的`main_counter`还是0，`pthread_id[2]`也来执行这个函数，`main_counter+1`，此时三个线程才将加完之后的值赋给`main_counter`，则`main_counter=0+1=1`，而真正执行次数`sum=0+1+1+1=3`。`main_counter < sum` 

 3．`thread`的CPU占用率在我的机子上执行结果是181，因为三个线程是无
 限循环的运行，使得cpu占用率很高。   
 
4． `thread_worker()`函数内是死循环，退出时因为主函数中设置的输入q时循环退出。输入q时主进程执行退出，`return `退出程序，则子线程也强制退出。 这样退出不好。
#### 五、 程序代码：
```cpp
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <signal.h>
#include <ctype.h>
#include <stdlib.h>

#define MAX_CHILD_NUMBER 10

#define SLEEP_INTERVAL 2
int proc_number = 0;
void do_something();


int main(int argc , char* argv[])
{
        int child_proc_number = MAX_CHILD_NUMBER;
        int i,ch;
        pid_t child_pid;
        pid_t pid[10] = {0};
        if(argc>1)
        {
                child_proc_number = atoi(argv[1]);
                child_proc_number = (child_proc_number > 10)? 10:child_proc_number;
        }
        for(i= 0 ; i<child_proc_number;i++)
        {
                printf("This is :%d\n",i);
                child_pid = fork();
                if(child_pid == -1){
                perror("creat error!\n");
                return 1;
                }else if(child_pid>0)
                {
                        pid[i] = child_pid;
                }else{
                proc_number =i;
                do_something();
                }
        }

        while((ch=getchar())!='q'){
        if(isdigit(ch)){
        kill(pid[ch-'0'],SIGTERM);

        }
    }
        kill(0,SIGTERM);
return 0;
}

void do_something(){
        for(;;){
        printf("This is process NO %d and its pid is %d\n",proc_number,getpid());
        sleep(SLEEP_INTERVAL);
        break;
        }
}

```
