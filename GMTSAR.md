---
title: GMTSAR
created: '2021-11-14T04:50:38.515Z'
modified: '2021-11-23T01:25:18.515Z'
---

# GMTSAR
关于GMTSAR有许多的步骤，在这些步骤中大致将他分成了以下几步，首先是数据准备，然后是配准

### 数据准备
一共需要准备Sentinel1数据，精轨数据，DEM数据，可以在以下网站进行下载
1. Sentinel数据
https://search.asf.alaska.edu
urs.earthdata.nasa.gov IDM 链接 ， 记得在连接里面选择https
2. 精轨数据
https://s1qc.asf.alaska.edu/aux_poeorb/
3. DEM数据
https://topex.ucsd.edu/gmtsar/demgen/

4. https://srtm.csi.cgiar.org/download/ SRTM数据  https://www.cnblogs.com/xionglee/articles/5688241.html SRTM说明

下载好这些数据之后建立一个文件夹，这个文件夹需要包括
- data : 文件夹，用于存放下载好的Sentinel-1数据，然后cd到data目录下用unzip_sentinel-1.csh /data ， 并且把精轨数据的文件也拷到data目录下面。
- orbit : 精轨数据文件夹，将下载好的精轨hmo数据放到该文件夹下面
- topo : DEM数据文件夹，将下载好的dem.grd放到该文件夹下面
- F1 ： 这个文件是用来存放IW1第一个burst的数据的。文件夹下面需要创建raw文件夹，topo文件夹，然后cd到raw文件夹下面，现在需要做的是把IW1数据软连接到raw文件目录下面，
1. 使用./link_S1.csh  [directory]  [num] ， directory为解压后的SAR数据目录。也就是上面的data目录，num为burst的序号，选择1，2，3。对于F1则为选1。软连接了数据之后，下一步需要软连接的是DEM数据，
2. 使用命令ln -s ../../topo/dem.grd。
3. 接下来需要将精确轨道数据进行软连接link_S1_orbits.csh csh 如 ./link_S1_orbits.csh /data/orbit/
这样则完成了整个F1文件夹的准备。
- F2 ： 文件与上述相同，F2文件夹是用来存放IW2的，所以注意上述命令要改为2
- F3 ： 文件与上述相同，F3文件夹是用来存放IW3文件的，所以注意将上面的命令改成3


对于128-3选择20200128为主影像


### 配准准备
1. preproc_batch_tops.csh data.in ../topo/dem.grd 1
  这一步是数据预处理的第一步，第一个参数data.in是上面./link_s1.ch 生成的数据列表文件放在F*/raw下面，../topo/dem.grd是DEM所在的路径，1代表的是第一步，这一步将会生成baseline_table.dat和baseline.ps（这是一种格式图片）,将baseline_table.dat移动到上一层目录去，使用mv baseline_table.dat ../命令，然后可以打开baseline.ps选取主影像，选取好的主影像后，在data.in将master对应的那一行移到第一行，然后F2,F3重复这个步骤。注意：这一步可能出现can't open tiff这个时候可能是解压有问题
2. preproc_batch_tops.csh data.in ../topo/dem.grd 2
  这一步是数据预处理的第二步，这一步直接运行就可以，然后会发现在raw文件夹中生成的SLC文件比较大

3. ls *ALL*PRM > prmlist ： 这一步是存放变量
  get_baseline_table.csh prmlist S1_20200527_ALL_F1.PRM  这一步是生成新的baseline_table.data，注意每一个文件夹后面S1_20200527_ALL_F1.PRM参数不相同
  做完这几步之后，现在需要得是修改baseline_table.data, 由于原来的baseline_table.data存在一定的问题，所以这个时候需要进行修改。


### 干涉图的生成
1. tcsh /GMTSAR/gmtsar/csh/select_pairs.csh baseline_table.dat 50 100
  /GMTSAR/gmtsar/csh/select_pairs.csh baseline_table.dat 50 100
  在F1目录下（注意此时的baseline_table.dat已经更新了），用这个命令，50指的是时间基线，100指的是垂直基线的长度，在这个过程中会生成intf.in文件，这里包含了干涉对的信息
2. cp inif.in ../F2
3. cp inif.in ../F3
4. 到F2目录下，vi intf.in :%s/F1/F2/g , 然后到F3目录下vi intf.in :%s/F1/F3/g
5. cp /GMTSAR/gmtsar/csh/batch_tops.config ./ ：拷贝batch_tops.config到每个文件夹


### 在服务器上跑的实验
1. 128-3 选的主影像是20200527  S1_20200527_ALL_F1
2. 激活docker 环境sudo docker run -i -v /mnt/raidDisk/128-3/:/data -t nbearson/gmtsar /bin/bash


### 补充文档
1. D:\zlc\GMTSAR\gmtsar脚本\link_S1.csh  用于软链接Sentinel-1数据
2. D:\zlc\GMTSAR\gmtsar脚本\link_S1_orbits.csh 用于软连接精轨数据
3. D:\zlc\GMTSAR\gmtsar脚本\cor.csh 用于批处理配准脚本


### GMT画.PS文件
1. pyGMT安装 https://www.pygmt.org/dev/install.html
2. 激活conda activate pygmt
3. 将.ps文件转换为.jpg文件gmt psconvert -A baseline.ps
4. >gmt grdimage phase.grd -JX6i+ -I+d -jpg map 
5. gdalinfo dem.grd ：可以查看.grd图像的信息

### docker 命令
- sudo docker images : 查看镜像
- sudo docker ps : 查看正在运行的container
- sudo docker ps -a : 查看所有的container
- sudo docker stop $(docker ps -q) & docker rm $(docker ps -aq) : 停用并删除所有的容器
- sudo docker stop ID : ID为关闭容器的ID
- sudo docker rm ID : ID为删除容器的ID
- systemctl status docker : 查看docker 运行情况
- systemctl start docker ： 启动docker
- systemctl stop docker : 关闭docker
- apt-get update  : 先更新再安装
- apt-get install sudo ： linux 安装sudo
- apt-get install vim : 安装vim
- docker rmi ： docker 删除images
- sudo docker exec -i -t 1238f6e5c82f /bin/bash : 进已经run的环境


### bash 命令
1. bash select_pairs.sh baseline_table.dat 50 100
2. rm -rf ./*.zip ： 删除该文件夹里面的所有后缀为.zip的文件

###  tcsh
1. 以后所有的gmtsar都在tcsh上面运行


### inif.csh需要用上
1. cleanup.csh
2. dem2topo_ra.csh


### 自己写的东西
1. ls -d disp_* > dislist  ， 把此文件夹里面包含disp地全部放到dislist 里面去
2. 当出现permisission denied : chmod 777 /usr/local/GMTSAR/bin/unzip_sentinel-1.csh 


### ALOS实验
1. 选取的主影像为IMG-HH-ALOS2251300700-190117-FBDR1.1__A
2. http://gmt.soest.hawaii.edu/boards/6/topics/6265 参考的这篇文献
3. 运行命令p2p_processing.csh ALOS2 IMG-HH-ALOS2011986990-140813-HBQR1.1__A IMG-HH-ALOS2014056990-140827-HBQR1.1__A config.ALOS2.txt
4. 已经完成190117-190228 ，190117-190523
5. p2p_processing.csh ALOS2 IMG-HH-ALOS2251300700-190117-FBDR1.1__A IMG-HH-ALOS2257510700-190228-FBDR1.1__A config.ALOS2.txt   （这个命令是190117-190228） 已经完成
6. p2p_processing.csh ALOS2 IMG-HH-ALOS2251300700-190117-FBDR1.1__A IMG-HH-ALOS2269930700-190523-FBDR1.1__A config.ALOS2.txt   （这个命令是190117-190523） 已经完成
7. p2p_processing.csh ALOS2 IMG-HH-ALOS2251300700-190117-FBDR1.1__A IMG-HH-ALOS2278210700-190718-FBDR1.1__A config.ALOS2.txt   （这个命令是190117-190523） 正在运行

### ALOS1实验
1. 下载北麓河08年到10年的数据，unzip ALPSRP182280690-H1.1__A.zip -d raw/ ;
2. ALPSRP074920690 20070621
3. ALPSRP081630690 20070806
4. ALPSRP088340690 20070921
5. ALPSRP135310690 20080808
6. ALPSRP142020690 20080923  这个是双极化数据
7. ALPSRP148730690 20081108  这个是单极化数据
8. ALPSRP155440690 20081224
9. ALPSRP162150690 20090208
10. ALPSRP182280690 20090626
11. ALPSRP195700690 20090926
12. ALPSRP235960690 20100629 
13. FBS ALOS单极化数据，FBD ： ALOS 双极化数据
14. 在raw文件夹下面 ls *PRM > prmlis
15. IMG-HH-ALPSRP148730690-H1.1__A.PRM 是主影像
16. https://github.com/gmtsar/gmtsar/blob/master/gmtsar/csh/p2p_processing.csh 这个地方的321行需要注意

### 命令总结
## 攀枝花26-11轨道数据处理一共117景数据
1. 到data文件夹 unzip_sentinel-1.csh /data/panzhihua-26-11  11/23/20：00  ~ 11/23/22：00 ， 花了两个小时
2. cd ../F1; mkdir raw; cd raw; link_S1.csh ../../data/ 1; link_S1_orbits.csh ../../orbit/; ln -s ../../topo/dem.grd
3. cd ../../F2; mkdir raw; cd raw; link_S1.csh ../../data/ 2; link_S1_orbits.csh ../../orbit/; ln -s ../../topo/dem.grd
4. cd ../../F3; mkdir raw; cd raw; link_S1.csh ../../data/ 3; link_S1_orbits.csh ../../orbit/; ln -s ../../topo/dem.grd
5. cd ../../F1/raw; preproc_batch_tops.csh data.in dem.grd 1; mv baseline_table.dat ../; 接下来修改data.in，把master放在第一个s1a-iw1-slc-vv-20180811t111556;  preproc_batch_tops.csh data.in dem.grd 2     11/23/22.30 ~ 11/24/13.30 花了14个小时
6. cd ../../F2/raw; preproc_batch_tops.csh data.in dem.grd 1; mv baseline_table.dat ../; 接下来修改data.in，把master放在第一个s1a-iw2:-slc-vv-20180811t111556; preproc_batch_tops.csh data.in dem.grd 2
7. cd ../../F3/raw; preproc_batch_tops.csh data.in dem.grd 1; mv baseline_table.dat ../;  接下来修改data.in，把master放在第一个s1a-iw3:-slc-vv-20180811t111556;preproc_batch_tops.csh data.in dem.grd 2
8. cd ../../F1/raw; ls *ALL*PRM > prmlist; get_baseline_table.csh prmlist S1_20180811_ALL_F1.PRM;
9. cd ../../F2/raw; ls *ALL*PRM > prmlist; get_baseline_table.csh prmlist S1_20180811_ALL_F2.PRM;
10. cd ../../F3/raw; ls *ALL*PRM > prmlist; get_baseline_table.csh prmlist S1_20180811_ALL_F3.PRM;
11. cd ../../F1; rm -rf baseline_table.dat; cp raw/baseline_table.dat ./; 
12. select_pairs.csh baseline_table.dat 50 100
13. cp intf.in ../F2; cp intf.in ../F3;
14. cd ../F2; vi intf.in; :%s/F1/F2/g 这一步可能需要手动; cd ../F3; vi intf.in; :%s/F1/F3/g; 
15. 接下里需要修改batch_topo.config : master_image = S1_20180508_ALL_F1 ; threshold_geocode = 0 每个F* 文件都需要
16. head -1 intf.in > one.in; mkdir topo ; cp ../topo/dem.grd topo/ ; intf_tops.csh one.in batch_tops.config
17. cd ../F1; head -1 intf.in > one.in; mkdir topo ; cp ../topo/dem.grd topo/; intf_tops.csh one.in batch_tops.config;修改set proc_stage = 2;intf_tops_parallel.csh intf.in batch_tops.config 40   花费一个小时
18. cd ../F2; head -1 intf.in > one.in; mkdir topo ; cp ../topo/dem.grd topo/; intf_tops.csh one.in batch_tops.config;修改set proc_stage = 2;intf_tops_parallel.csh intf.in batch_tops.config 40   花费一个小时
19. cd ../F3; head -1 intf.in > one.in; mkdir topo ; cp ../topo/dem.grd topo/;intf_tops.csh one.in batch_tops.config;修改set proc_stage = 2; intf_tops_parallel.csh intf.in batch_tops.config 40   花费一个小时
20. cd ../; mkdir merge; cd merge; cp ../F1/intf.in ./; ls ../F1/intf_all/ > intflist ; create_merge_input.csh intflist .. 0 > merge_list; 修改merge_list, 把包含主影像放到第一行
21. cp ../F2/batch_tops.config ./; ln -s ../topo/dem.grd ./; merge_batch.csh merge_list batch_tops.config 花费两个小时
22. 在merge文件夹下面 unwrap_parallel.csh intflist 40; 16.35  需要花费一天左右
23. cd ..; mkdir sbas; cd sbas ; cp ../merge/intf.in ./; cp ../F1/baseline_table.dat ./;prep_sbas.csh intf.in baseline_table.dat ../merge unwrap.grd corr.grd; 这一步将会生成scene.tab 和 intf.tab; 
24. prep_sbas.csh intf.in baseline_table.dat ../merge unwrap_gacos_corrected_detrended.grd corr.grd ： 这个命令可以用来生成利用GACOS矫正后的结果用来做SBAS
25. gmt grdinfo ../merge/2021134_2021182/unwrap.grd 查看x n_columns: 8547 , y n_rows: 6765; sbas intf.tab scene.tab 387 117 8547 6765 ; 387是干涉对的数量，117是多少景图，后面两个参数是前面查到的

### 自动匹配精轨数据
1. X:\S1PreOrb\S1A 这个地方存放了所有的精轨数据
2. F:\GMTSAR\myscript\find_orbit_copy.py : 这个是自动从数据库中查找精轨数据
3. fing_orbit_copy.py /media/wangjing/data/greece/datadir /media/wangjing/data/greece/orbit S1A
4. ren * *.EOF 这个命令可以把所有的xml格式转换为EOF格式

### GMT命令
1. plot_png.csh phasefilt ./png/wrap 在merge里面创建png文件，将phasefilt.grd文件转换为png
2. merge# plot_png.csh unwrap ./png/unwrap 画解缠后的图
3. df  -lh : 可以查看所有盘的内存
4. https://blog.csdn.net/wade1010/article/details/83271104 安装ghostscript
6. F:\GMTSAR\myscript\plot_ll.csh 画形变速率图  plot_ll.csh vel_ll.grd
7. https://docs.gmt-china.org/latest/tutorial/get-started/windows/ 这个是GMT的中文官方文档
### GACOS 下载
1. http://www.gacos.net/ 网站 ， 记得要选择binary grid
2. 128-3 轨道 100.5 104 35.0 37.5
3. 26-11 轨道 100.5 103.6 26.0 28.5

## 折多山实验，折多山的经纬度 101°47′48.09″ 30°06′47.63″  26-93 可以直接在ASF中的filter里面的最后一行查找，26是orbit, 93是Frames
1. 20200109是主影像

## SBAS原理
ax^{2} + by^{2} + c = 0 

### 142-4-2020数据 一共31景数据
1. 数据存放在 W:/Subsidence/HuaBei/Sentinel-1/Orbit-142/142-4/142-4-2020/data
2. 
3. 寻找精轨数据 ，orbit文件夹一定要创建在data目录下面 打开prompt , F: , cd F:\GMTSAR\myscript; python find_orbit_copy.py W:/Subsidence/HuaBei/Sentinel-1/Orbit-142/142-4/142-4-2020/data X:/S1PreOrb/ S1A
4. 下一步查询DEM数据， 114.4，117.8 , 37.2, 39.5
5. cd /data/142-4-2020/; cd data/; unzip.py 即在进行并行的解压
6. cd ../F1; mkdir raw; cd raw; link_S1.csh ../../data/ 1; link_S1_orbits.csh ../../orbit/; ln -s ../../topo/dem.grd
8. cd ../../F2; mkdir raw; cd raw; link_S1.csh ../../data/ 2; link_S1_orbits.csh ../../orbit/; ln -s ../../topo/dem.grd
9. cd ../../F3; mkdir raw; cd raw; link_S1.csh ../../data/ 3; link_S1_orbits.csh ../../orbit/; ln -s ../../topo/dem.grd
10. cd ../../F1/raw; preproc_batch_tops.csh data.in dem.grd 1; mv baseline_table.dat ../; 
11. 选择s1a-iw1-slc-vv-20200609t101324 为主影像；接下来修改data.in，把master放在第一个s1a-iw1-slc-vv-20180811t111556;  
12. cd ../../F2/raw; preproc_batch_tops.csh data.in dem.grd 1;mv baseline_table.dat ../;接下来修改data.in，把master放在第一个s1a-iw1-slc-vv-20200609t101324; preproc_batch_tops.csh data.in dem.grd 2
13. cd ../../F1/raw; ls *ALL*PRM > prmlist; get_baseline_table.csh prmlist S1_20200609_ALL_F1.PRM; 这一步生成新的baseline_table
14. cd ../../F2/raw; ls *ALL*PRM > prmlist; get_baseline_table.csh prmlist S1_20200609_ALL_F2.PRM;
15. cd ../../F3/raw; ls *ALL*PRM > prmlist; get_baseline_table.csh prmlist S1_20200609_ALL_F3.PRM;
16. cd ../../F1; rm -rf baseline_table.dat; cp raw/baseline_table.dat ./; select_pairs.csh baseline_table.dat 50 100; 这一步生成intf.in
17. cp intf.in ../F2; cp intf.in ../F3;
18. cd ../F2; vi intf.in; :%s/F1/F2/g 这一步可能需要手动; cd ../F3; vi intf.in; :%s/F1/F3/g; 
19. cp /usr/local/GMTSAR/bin/batch_tops.config ../; cd ../ 到主目录下面;  cp batch_tops.config F1/;cp batch_tops.config F2/; cp batch_tops.config F3/;
20. cd F1/; head -1 intf.in > one.in; mkdir topo ; cp ../topo/dem.grd topo/; 接下里需要修改batch_topo.config : master_image = S1_20200609_ALL_F1 ; threshold_geocode = 0; intf_tops.csh one.in batch_tops.config; 修改set proc_stage = 2;intf_tops_parallel.csh intf.in batch_tops.config 40
21. cd ../F2/; head -1 intf.in > one.in; mkdir topo ; cp ../topo/dem.grd topo/ ;接下里需要修改batch_topo.config : master_image = S1_20200609_ALL_F2 ; threshold_geocode = 0; intf_tops.csh one.in batch_tops.config;修改set proc_stage = 2;intf_tops_parallel.csh intf.in batch_tops.config 40
22. cd ../F3/;head -1 intf.in > one.in; mkdir topo ; cp ../topo/dem.grd topo/ ;接下里需要修改batch_topo.config : master_image = S1_20200609_ALL_F3 ; threshold_geocode = 0; intf_tops.csh one.in batch_tops.config;修改set proc_stage = 2;intf_tops_parallel.csh intf.in batch_tops.config 40
23. 到此为止则生成完毕了干涉对， 接下来是把IW1,IW2,IW3进行merge，把他们合成一步
24. cd ../; mkdir merge; cd merge; cp ../F1/intf.in ./; ls ../F1/intf_all/ > intflist ; create_merge_input.csh intflist .. 0 > merge_list; 修改merge_list, 把包含主影像放到第一行
25. cp ../F2/batch_tops.config ./; ln -s ../topo/dem.grd ./; merge_batch.csh merge_list batch_tops.config 花费两个小时
26. 到此为止则完成了merge的过程，接下来要进行的是解缠的过程
27. 在merge文件夹下面 unwrap_parallel.csh intflist 40; 这个地方很玄学，有时候很快有时候很慢，需要考虑一下原因，是不是SWAP得原因
28. 到此为止则完成了Unwrap的过程，接下来要进行的SBAS的过程
30. cd ..; mkdir sbas; cd sbas ; cp ../merge/intf.in ./; cp ../F1/baseline_table.dat ./;prep_sbas.csh intf.in baseline_table.dat ../merge unwrap.grd corr.grd; 这一步将会生成scene.tab 和 intf.tab
31. gmt grdinfo ../merge/2020148_2020196/unwrap.grd ; x n_columns: 8548; y n_rows: 6532; 干涉对数量91；31景图像； sbas intf.tab scene.tab 91 31 8548 6532 ; 



### 下载数据
1. 上传到NASA上的X:\worldCity
2. Z:\newDataRequie\北美

