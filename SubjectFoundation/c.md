# 配置

### vs2015配置opencv

1.将bin目录配置到环境变量，选用vc14，就配置vc14/bin的路径。

2.进入vs，点击view，打开solutin exporer。右键选择properties。

3在properties配置c/c++  ->include files，Linker->additional lib,Linker->input ->additional dependencies

# problem

### c++的字符串与字符数组

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

int main(int argc, char** argv) {
	createDatabase("C:\\Users\\guo\\Desktop\\traindatabase");
	
	
}
```

使用strcpy才有作用。直接赋值无用，例如：directory[1]=(char *)a.c_str();

### devcpp调试提示无调试信息及闪退

打开编辑器的工具选项，进入编译器选项。勾选编译时加入以下命令和在连接器命令行加入以下命令。并在其内容中都添加-g3。

### PCA识别

>用的vs2015,opencv 3.1.0。

```
#include <opencv2/opencv.hpp>
#include <opencv2/core.hpp>
#include <iostream>
#include <stdio.h>
#include <cstring>   
#include<cv.h>
#include<highgui.h>
#include<cmath>
using namespace cv;
Mat m;
Mat A;


Mat createDatabase(std::string path)
{
	Mat src, gray_src,reshape_src;
	Mat sum_src(98304, 0, CV_8UC1);
	std::string temp = "";
	for (int i = 1; i <= 20; i++)
	{
		temp = path+std::to_string(i)+".jpg";
		std::cout<<temp<<std::endl;

	    src = imread(temp);
		std::cout << "src rows , cols, channels:" << src.rows << "," << src.cols << "," << src.channels() << std::endl;
	
		if (src.empty())
		{
			printf("could not load image...\n");
			return sum_src;
		}
		
		cvtColor(src,gray_src,CV_BGR2GRAY);

		
		std::cout << "rows , cols, channels:" << gray_src.rows <<","<< gray_src.cols << ","<< gray_src.channels() <<std::endl;
	
		reshape_src = gray_src.reshape(0, gray_src.rows* gray_src.cols);

		
		std::cout << "rows , cols, channels:" << reshape_src.rows << "," << reshape_src.cols << "," << reshape_src.channels() << std::endl;
		
	
		hconcat(sum_src,reshape_src,sum_src);
		
		
		std::cout << "rows , cols, channels:" << sum_src.rows << "," << sum_src.cols << "," << sum_src.channels() << std::endl;

	}
	return sum_src;
	
}

Mat EigenfaceCore(Mat database)
{
	Mat float_mat = Mat::ones(98304, 20, CV_32FC1);
	database.convertTo(float_mat, CV_32FC1);
	std::cout << float_mat.type() << std::endl;

	int rows = float_mat.rows;
	int cols = float_mat.cols;
	Mat mean_mat = Mat::ones(0, 1, CV_32FC1);
	
	for (int row = 0; row < rows; row++)
	{
		float sum = 0;
		for (int col = 0; col < cols; col++)
		{
			sum = sum + float_mat.at<float>(row,col);
		}
		float mean = sum / cols;
		mean_mat.push_back(mean);
		
	}
	std::cout <<"mean_mat`s rows,cols:"<< mean_mat.rows<<","<<mean_mat.cols << std::endl;
	m = mean_mat;

	
	for (int i = 0; i < cols; i++)
	{
		float_mat.col(i) = float_mat.col(i) - mean_mat-0;
	}
	A = float_mat;
	//转置矩阵相乘
	Mat trans_mat = Mat::ones(20, 98304, CV_32FC1);
	trans_mat= float_mat.t();
	
	std::cout << "trans_mat`s rows,cols:" << trans_mat.rows << "," << trans_mat.cols <<","<<trans_mat.type()<< std::endl;
	std::cout << "float_mat`s rows,cols:" << float_mat.rows << "," << float_mat.cols <<","<< trans_mat.type()<< std::endl;
	
	Mat multi_mat = Mat::ones(20, 20, CV_32FC1);
	multi_mat = trans_mat*float_mat;
	


	Mat eigenValues, eigenVectors;
	cv::eigen(multi_mat,eigenValues,eigenVectors);
	

	Mat values_mat(eigenValues.cols,eigenValues.rows, CV_32FC1);
	values_mat=eigenValues.t();
	Mat vectors_mat(eigenVectors.cols, eigenVectors.rows, CV_32FC1);
	vectors_mat = eigenVectors.t();
	
	std::cout << "values_mat`s rows,cols:" << values_mat.rows << "," << values_mat.cols << "," << values_mat.type() << std::endl;
	std::cout << "vectors_mat`s rows,cols:" << vectors_mat.rows << "," << vectors_mat.cols << "," << vectors_mat.type() << std::endl;
	std::cout << "values_mat :" << std::endl << values_mat << std::endl;
	std::cout << "vectors_mat :" << std::endl << vectors_mat << std::endl;

	Mat L_eig_vec(20, 0, CV_32FC1);
	for (int i = 0; i < values_mat.cols - 1; i++)
	{
		if (values_mat.at<float>(0, i) > 1)
		{
			hconcat(L_eig_vec,vectors_mat.col(i), L_eig_vec);
		}
	}

	std::cout << "L_eig_vec :" << std::endl << L_eig_vec << std::endl;
	
	Mat Eigenfaces = float_mat*L_eig_vec;
	return Eigenfaces;
}

void recognition(Mat Eigenfaces,std::string path)
{
	Mat Eigenfaces_trans = Eigenfaces.t();

	Mat ProjectedImages(Eigenfaces.cols, 0, CV_32FC1);
	ProjectedImages = Eigenfaces_trans * A;/*19*9000  9000*20*/
	std::cout << "ProjectedImages :" << std::endl << ProjectedImages << std::endl;


	Mat InputImage = imread(path);

	if (InputImage.empty())
	{
		printf("could not load image...\n");
		return;
	}

	Mat gray_src;
	cvtColor(InputImage, gray_src, CV_BGR2GRAY);
	Mat reshape_src = gray_src.reshape(0, gray_src.rows*gray_src.cols);
	std::cout <<"reshape:"<< reshape_src.rows <<","<< reshape_src.cols<<"," <<reshape_src.type()<< std::endl;
	Mat double_mat  ;
	reshape_src.convertTo(double_mat, CV_32FC1);
	std::cout <<"double:"<< double_mat.rows << "," << double_mat.cols << "," << double_mat.type() << std::endl;

	for (int i = 0; i < double_mat.cols; i++)
	{
		double_mat.col(i) = double_mat.col(i) - m-0;
	}

	Mat ProjectedTestImage = Eigenfaces_trans*double_mat;
	std::cout << "ProjectedTestImage:"<<ProjectedTestImage.rows << "," << ProjectedTestImage.cols << "," << ProjectedTestImage.type() << std::endl<< ProjectedTestImage<<std::endl;

	Mat Euc_dist;
	for (int i = 0; i < Eigenfaces.cols; i++)
	{
		Mat q = ProjectedImages.col(i);
		Mat temp = ProjectedTestImage - q;
		Euc_dist.push_back(pow(norm(temp),2));
	}
	Mat Euc_dist_mat=Euc_dist.t();
	std::cout <<"Euc_dist_mat:"<<std::endl<< Euc_dist_mat.rows << "," << Euc_dist_mat.cols << "," << Euc_dist_mat.type() << std::endl<< Euc_dist_mat<<std::endl;
	
	double min_values = Euc_dist_mat.at<double>(0, 0);
	int  min_index = -1;
	for (int i =1; i < Euc_dist_mat.cols; i++)
	{
		if (Euc_dist_mat.at<double>(0, i) < min_values)
		{
			min_values = Euc_dist_mat.at<double>(0, i);
			min_index = i;
		}
	}
	
	std::cout << "the result is " << min_index + 1 << ".jpg" << std::endl;
	
	namedWindow("test_image",CV_WINDOW_AUTOSIZE);
	imshow("test_image",InputImage);

	Mat result_image = imread(".\\traindatabase\\"+std::to_string(min_index+1)+".jpg");
	
	imshow("result_image", result_image);
}

int main(int args, char ** argv)
{
	Mat database;
	database=createDatabase(".\\traindatabase\\");
	std::cout << "database finish" << std::endl;
	Mat Eigenfaces=EigenfaceCore(database);
	
	recognition(Eigenfaces,".\\testdatabase\\8.jpg");
	
	waitKey(0);
	return 0;
}
```

##### 使用过程中的问题

1,mat1.col(1)=mat1.col(2)不生效，写作mat1.col(1)=mat1.col(2)-0。即可生效。

2,矩阵存储数值类型，及其与type编号对应关系。注意mat1.at<float>（row,col）保持一致。

```
+--------+----+----+----+----+------+------+------+------+
|        | C1 | C2 | C3 | C4 | C(5) | C(6) | C(7) | C(8) |
+--------+----+----+----+----+------+------+------+------+
| CV_8U  |  0 |  8 | 16 | 24 |   32 |   40 |   48 |   56 |
| CV_8S  |  1 |  9 | 17 | 25 |   33 |   41 |   49 |   57 |
| CV_16U |  2 | 10 | 18 | 26 |   34 |   42 |   50 |   58 |
| CV_16S |  3 | 11 | 19 | 27 |   35 |   43 |   51 |   59 |
| CV_32S |  4 | 12 | 20 | 28 |   36 |   44 |   52 |   60 |
| CV_32F |  5 | 13 | 21 | 29 |   37 |   45 |   53 |   61 |
| CV_64F |  6 | 14 | 22 | 30 |   38 |   46 |   54 |   62 |
+--------+----+----+----+----+------+------+------+------+
```

3，Mat float_mat = Mat::ones(98304, 20, CV_32FC1)，此形式声明的矩阵就收convertTo不会出错。

