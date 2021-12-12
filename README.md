# CRUJRA_4_GUESS
Process CRU-JRA55 (2019, v2.0) climate data for reading by LPJ-GUESS.

## Overview
这是一个简单的软件包，用于捆绑脚本（和文档！），以处理LPJ-GUESS读取的气候数据，任务相对简单。 对于其他植被模型（特别是运行TRENDY模拟的模型）的用户来说，这也可能是有用的，可能需要做一些小的修改。

## Aim
1. 把下载的CRUJRA数据（netCDF，次日时间分辨率，每年每个变量一个文件）处理成日值，每个变量一个netCDF文件，数据和元数据准备好给LPJ-GUESS的'cf'输入模块。 请注意，为了在LPJ-GUESS中快速读取数据，应该对尺寸进行重新排序（见下文）。

2. 计算 "派生 "气候量，如风速和相对湿度（用于SIMFIRE-BLAZE和SPITFIRE火灾模型）。 注意在一些高纬度的网格单元中，相对湿度可以超过100%（有时甚至是1000%），所以它的上限是99.99%。 我相信这只是一个数字假象，在如此低的温度下，实际蒸汽压力和饱和蒸汽压力都是非常小的数字。 由于各种潜在的原因（例如，饱和蒸汽压力的经验近似值的错误或精度影响和/或数据中非常小的不一致），实际蒸汽压力超过了饱和蒸汽压力。 虽然这个正差非常小，但与非常小的蒸汽值相比，它是比较大的，因此明显是 "超饱和 "的空气。

3. 为了方便和测试，制作每月的文件。


## Requirements

### Hardware
在每天的分辨率下，最终的文件每个大约占42Gb（未压缩）。 因此，我在Senckenberg BiK-F这里的一台大型内存机器上处理数据，这样就可以在内存中处理整个文件。 要重复这种处理，可能需要有一台具有许多千兆字节（也许是50？ 如果不需要尺寸重排步骤（ncpdq命令），那么可能就不需要这么大的内存了。

### Software
Both `cdo` and `nco` for processing the netcdf files, also `bash` for the scripting.

## Details and Strategy
The requirements for the 'cf' input module of LPJ-GUESS is fairly strict in terms of meta-data (see documentation in /docs) but fairly loose in terms of the underlying structure of the data.  So there are decision to make here.  It is important to note that the 'standard'/'CF convention' data structure is *very* slow when read by LPJ-GUESS because this structre is optimised for reading data which continuous in **space** whereas LPJ-GUESS requires data that is continuous in **time**.  There are three options to handle this:
LPJ-GUESS对'cf'输入模块的要求在元数据方面相当严格（见/docs中的文档），但在数据的基本结构方面却相当宽松。 所以在这里要做一些决定。 需要注意的是，"标准"/"CF惯例 "的数据结构在被LPJ-GUESS读取时是**慢的，因为这个结构是为读取**空间的连续数据而优化的，而LPJ-GUESS需要的是**时间的连续数据。 有三种方法可以处理这个问题。

1. **Re-ordering** the dimensions (lon, lat and time) so that time comes last.  This has the disadvantage that some programs will be confused by this non-standard ordering (eg. `ncview`).  It also doesn't appear to work well for the netCDF4 format (see below). 维度（lon, lat and time），所以时间在最后。 这样做的缺点是一些程序会被这种非标准的排序所迷惑（例如`ncview`）。 对于netCDF4格式，它似乎也不能很好地工作（见下文）。

2. **Chunking**  the data into chuncks which are much longer in time than in lon/lat.  This optimises the data for reading of time series, but it is slower for reading spatial slices (ie looking at the maps in a viewer program).  See these two useful posts from Unidata (developers of netCDF) on chunking: [Chunking Data: Why it Matters](https://www.unidata.ucar.edu/blogs/developer/entry/chunking_data_why_it_matters) and [Chunking Data: Choosing Shapes](https://www.unidata.ucar.edu/blogs/developer/en/entry/chunking_data_choosing_shapes).他的数据被分成几块，这些数据在时间上要比长/纬度上长得多。 这对读取时间序列的数据进行了优化，但对读取空间片断（即在查看器程序中查看地图）来说则比较慢。 请参阅Unidata公司（netCDF的开发者）关于分块的两篇有用的文章。[分块数据：为什么重要](https://www.unidata.ucar.edu/blogs/developer/entry/chunking_data_why_it_matters) 和[分块数据：选择形状](https://www.unidata.ucar.edu/blogs/developer/en/entry/chunking_data_choosing_shapes)。
3. **Reduced Lon/Lat Grid** This involves melting the lon and lat into a single dimension (with variables to look up the longitude and latitude).  This is the most effective in terms of disk space (and I think maybe also read time) but the format is the least convenient to use and view from a human perspective. 这涉及到将lon和lat融为一个单一的维度（用变量来查询经度和纬度）。 就磁盘空间而言，这是最有效的（我想也许还有读取时间），但从人的角度来看，这种格式在使用和查看方面是最不方便的。


For simplicity, here I choose option 1., re-order the dimensions, however the scripts also make a files with the standard ordering for checking and visualisation (at the cost of writing another 42 Gb file per variable).

为了简单起见，我在这里选择了选项1，对尺寸进行重新排序，但是脚本也会以标准的排序方式生成一个文件，用于检查和可视化（代价是每个变量要再写42Gb的文件）。


## Note on netCDF formats and chunking for netCDF

Whilst re-ordering 'classic' netCDF files produces files for which time series for one gridcell can be read perfectly efficiently with LPJ-GUESS, using the netCDF4 data format seems to mess this up.  I think the reason is that netCDF4 files are *always* chunked due to their underlying data structure, and `cdo` chucks the data for optimal reading of spatial slices (see the Unidata links above for that is important).  Therefore when using netCDF4 formatted files for LPJ-GUESS, appropriate chunking becomes essential.  Note that this is currently inferred from model performance, not conclusively tested.  In summary, the safest thing to do is to stick with netCDF 'classic' with re-ordered dimensions as is done here.  Using netCDF4 with appropriately chunked data is in some way optimal (access advanced features of netCDF4 and fiels can be easily viewed), but not fully investigated yet.
虽然重新排序的 "经典 "netCDF文件产生的文件可以用LPJ-GUESS非常有效地读取一个网格单元的时间序列，但使用netCDF4数据格式似乎会把它弄乱。 我认为原因是netCDF4文件由于其底层数据结构，总是*分块的，而`cdo'分块数据是为了优化空间片的读取（见上面的Unidata链接，这很重要）。 因此，当使用netCDF4格式的文件进行LPJ-GUESS时，适当的分块是必不可少的。 请注意，这只是从模型的性能中推断出来的，并没有经过确凿的测试。 总之，最安全的做法是坚持使用netCDF的 "经典 "格式，像这里所做的那样重新排序维度。 使用netCDF4和适当的分块数据在某种程度上是最优的（访问netCDF4的高级功能和fiels可以很容易地查看），但还没有充分研究。

## Timings

The /docs folder also has a couple of notes and run timings for LPG-GUESS.  This is probably not translatable to other models.

/docs文件夹中还有一些LPG-GUESS的注释和运行时间。 这可能无法转化为其他模型。



Questions? Contact matthew.forrest@senckenberg.de
