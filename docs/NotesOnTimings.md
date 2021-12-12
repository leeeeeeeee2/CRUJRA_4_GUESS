# Details

Timing were done on the Goethe-HLR super computer using LPJ-GUESS 4.0.1.  Setups were the standard benchmarking runs (global and crop_global), with the standard 5 patches etc.  The runs used a single dual CPU node featurning 2 x Xeon Skylake Gold 6148 CPUs, each with 20 cores, for a total of 40 cores.

'standard' means the standard 0.5 degree gridlist which is ordered from highest to lowest latitudes.  'shuffled' means the standard cf global gridlist which has been randomised. 'dealt' means the the standard 0.5 degree gridlist was 'dealt' out like a pack of cards to each node (via a modifcation to the submist script).  This ensures the the low producutivity (high latitude cells) and high productivity (tropical) gridcells are distributed evenly across the processes given much improved load balancing.  
计时是在歌德-HLR超级计算机上使用LPJ-GUESS 4.0.1完成的。 设置是标准的基准测试运行（global和crop_global），有标准的5个补丁等。 运行时使用了一个双CPU节点，其中有2个Xeon Skylake Gold 6148 CPU，每个有20个内核，总共有40个内核。

标准 "是指标准的0.5度网格表，从最高纬度到最低纬度排序。 洗牌 "是指经过随机处理的标准cf全球网格表。发牌 "意味着标准的0.5度网格表像一包牌一样被 "发 "到每个节点上（通过对submist脚本的修改）。 这确保了低生产率（高纬度单元）和高生产率（热带）的网格单元被均匀地分配到各个进程中，大大改善了负载平衡。 

# Learnings

Firstly, 'dealing' out the gridcells can reduce the run time to about 63% of the 'non-dealt' setup.

Secondly, reading a properly constructed netCDF file (ie dimensions re-ordered) is not significantly slower than a reading a binary archive (with some load balancing on each) **even though the netCDF files are daily data**.     

首先，"处理 "出网格单元可以将运行时间减少到 "非处理 "设置的63%左右。

其次，读取一个正确构建的netCDF文件（即尺寸重新排序）并不比读取一个二进制档案（每个文件都有一些负载平衡）慢很多**，尽管netCDF文件是每日数据。    
