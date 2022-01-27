## 1. 如果你的fig文件中的图像每个像素都有数据

可以通过以下方式获得图像每个点的值，输出data是矩阵，大小是图像像素的行列数

```matlab
open('figname.fig');
h=get(gca,'Children');
data=get(h,'Cdata');
```

 

## 2. 如果你的fig文件中 图像是由单条曲线绘制而成

比如说plot命令生成的，通过以下方式输出横坐标，纵坐标的取值

```matlab
open('figname.fig');
h_line=get(gca,'Children');%get line handles
xdata=get(h_line,'Xdata');
ydata=get(h_line,'Ydata');
```

 

## 3. 如果你的fig文件中 图像是由三维单条曲线绘制而成

比如说plot3命令生成的，通过以下方式输出x，y，z 坐标的取值

```matlab
open('figname.fig');
h_line=get(gca,'Children');%get line handles
xdata=get(h_line,'Xdata');
ydata=get(h_line,'Ydata');
zdata=get(h_line,'Zdata');
```

 

## 4. 如果你的fig文件中 图像是由多条曲线绘制而成

比如说plot命令生成的，通过以下方式输出横坐标，纵坐标的取值

```matlab
open('figname.fig');
lh = findall(gca, 'type', 'line');% 如果图中有多条曲线，lh为一个数组
xc = get(lh, 'xdata');      	% 取出x轴数据，xc是一个元胞数组
yc = get(lh, 'ydata');      	% 取出y轴数据，yc是一个元胞数组

% 如果想取得第2条曲线的x，y坐标

x2=xc{2};
y2=yc{2};
```

