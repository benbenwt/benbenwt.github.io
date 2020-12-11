### 配置

##### devcpp配置opencv



### problem

##### c++的字符串与字符数组

```

#include <iostream>
#include <iomanip>
#include <string>
#include <stdio.h>
#include <stdlib.h>
#include <cstring>        // for strcat()
#include <io.h>
/* run this program using the console pauser or add your own getch, system("pause") or input loop */
using namespace std;

void dir(string path,char ** directory,int *size)
{
	long hFile = 0;
	struct _finddata_t fileInfo;
	string pathName, exdName;
 
	if ((hFile = _findfirst(pathName.assign(path).
		append("\\*").c_str(), &fileInfo)) == -1) {
		return;
	}
	do {
		if (fileInfo.attrib&_A_SUBDIR) {
			string fname = string(fileInfo.name);
			if (fname != ".." && fname != ".") {
				dir(path + "\\" + fname,directory,size);
			} 
		} else {
//			cout << path << "\\" << fileInfo.name << endl;
			
			directory[*size]=(char *)malloc(sizeof(char)*100);
			
			string str1=path+"\\"+fileInfo.name;
			
			strcpy(directory[*size],str1.c_str());
//			cout<<"diretory:"<<directory[size]<<endl;
//			cout<<size<<endl;
			*size=*size+1;
		}
	} while (_findnext(hFile, &fileInfo) == 0);
	_findclose(hFile);
	return;
}
 


void createDatabase(string trainDatabase)
{
	int size=0;
	
	char ** directory=(char **)malloc(sizeof(char *)*50);
	
	dir(trainDatabase,directory,&size);
	
	size=size-1;
	cout<<"size:"<<size<<endl;
	for(int i=size;i>=0;i--)
	{
		cout<<directory[i]<<endl;
	}
}


void eigenfaceCore()
{
	
}

void example()
{
	
}

void recognition()
{
	
}
int main(int argc, char** argv) {
	createDatabase("C:\\Users\\guo\\Desktop\\traindatabase");
	
	
}
```

使用strcpy才有作用。直接赋值无用，例如：directory[1]=(char *)a.c_str();

##### devcpp调试提示无调试信息及闪退

打开编辑器的工具选项，进入编译器选项。勾选编译时加入以下命令和在连接器命令行加入以下命令。并在其内容中都添加-g3。

临时：

```
#include <opencv2/opencv.hpp>
#include <opencv2/core.hpp>
#include <iostream>
#include <stdio.h>
#include <cstring>   
#include<cv.h>
#include<highgui.h>
using namespace cv;

void createDatabase(std::string path)
{
	Mat src, gray_src;
	std::string temp = "";
	for (int i = 1; i <= 20; i++)
	{
		temp = path+std::to_string(i)+".jpg";
		std::cout<<temp<<std::endl;

	    src = imread(temp);

		if (src.empty())
		{
			printf("could not load image...\n");
			return;
		}
		
		cvtColor(src,gray_src,CV_BGR2GRAY);

		namedWindow("gray_src", CV_WINDOW_AUTOSIZE);
		imshow("gray_src", src);
		waitKey(0);

		int rows = gray_src.rows;
		int cols = gray_src.cols;

		std::cout << "rows and cols:" <<rows<<","<<cols<< std::endl;
	}
	
}

int main(int args, char ** argv)
{
	createDatabase("C:\\Users\\guo\\Desktop\\traindatabase\\");
	return 0;
}
```



