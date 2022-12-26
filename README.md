
## 项目介绍
Havenask是阿里巴巴集团自研的搜索引擎，也是阿里巴巴内部广泛使用的大规模分布式检索系统，支持了包括淘宝、天猫、菜鸟、高德、饿了么、全球化在内整个阿里巴巴集团的搜索业务，为用户提供高性能、低成本、易用的搜索服务，同时具有灵活的定制和开发能力，支持算法快速迭代，帮助客户和开发者量身定做适合自身业务的智能搜索服务，助力业务增长。

此外，基于Havenask打造的行业AI搜索产品——[阿里云OpenSearch](https://www.aliyun.com/product/opensearch)，也将持续在阿里云上为企业级开发者提供全托管、免运维的一站式智能搜索服务，欢迎企业级开发者们试用。
## 核心能力
Havenask 的核心能力与优势，有以下几点：
* <strong>极致的工程性能</strong>：支持千亿级数据实时检索，百万QPS查询，百万TPS写入，毫秒级查询延迟与秒级数据更新。
* <strong>C++的底层构建</strong>：对性能、内存、稳定性有更高保障。
* <strong>SQL查询支持</strong>：支持SQL语法便捷查询，查询体验更友好。
* <strong>丰富的插件机制</strong>：支持各类业务插件，拓展性强。
* <strong>支持图化开发</strong>：实现算法分钟级快速迭代，定制能力丰富，在新一代智能检索场景下的支持效果优秀。
* <strong>支持向量检索</strong>：可通过与插件配合实现多模态搜索，满足更多场景的搜索服务搭建需求（待发布）。

## 相关文档
* Havenask Wiki: [https://github.com/alibaba/havenask/wiki](https://github.com/alibaba/havenask/wiki)

## 开始使用
使用前确保设备已经安装和启动Docker服务

### 启动容器
克隆仓库
```
git clone git@github.com:alibaba/havenask.git
cd havenask/docker
```
创建容器，其中CONTAINER_NAME为指定的容器名
```
docker pull havenask/ha3_runtime:0.2.0
./create_container.sh <CONTAINER_NAME> havenask/ha3_runtime:0.2.0
```
如果由于Docker Hub访问不稳定无法下载以上镜像，可以尝试阿里云镜像源
```
docker pull registry.cn-hangzhou.aliyuncs.com/havenask/ha3_runtime:0.2.0
./create_container.sh <CONTAINER_NAME> registry.cn-hangzhou.aliyuncs.com/havenask/ha3_runtime:0.2.0
```

登陆容器
```
./<CONTAINER_NAME>/sshme
```

### 测试索引构建

构建全量索引
```
cd ~/havenask/example/scripts
python build_demo_data.py /ha3_install
```

### 测试引擎查询
启动havenask引擎
```
python start_demo_searcher.py /ha3_install
```

引擎的默认查询端口为45800，使用脚本进行查询测试。下面是一些测试query。

```
python curl_http.py 45800 "query=select count(*) from in0"

python curl_http.py 45800 "query=select id,hits from in0 where MATCHINDEX('title', '搜索词典')"

python curl_http.py 45800 "query=select title, subject from in0_summary_ where id=1 or id=2"
```

## 联系我们
官方技术交流钉钉群：

![3293821693450208](https://user-images.githubusercontent.com/590717/206684715-5ab1df49-f919-4d8e-85ee-58b364edef31.jpg)


## 编译过程
进入提供的docker，在aios的各个目录下对模块进行单独编译，
```
scons
```

— 优先index_lib编译，index_lib是建立索引结构的服务 —
alog目录：/ha3_depends/usr/local/lib64
# modify by youhe.chen
env.Append(CPPPATH=['/ha3_depends/usr/local/include/','/ha3_depends/usr/include/'])
env.Append(LIBPATH=['/ha3_depends/usr/local/lib64','/ha3_depends/usr/local/lib','/ha3_depends/usr/lib64/'])
# ---
调整Sconscript里lib顺序

— build_service编译，build_service是离线建立索引的服务 —
Cd ~/havenask

Put Protoc and add exec into sys PATH
```
export PATH=$PATH:/ha3_depends/usr/local/bin
```
change ld path
```
sudo cat "/ha3_depends/usr/local/lib" > /etc/ld.so.conf.d/protobuf.conf
sudo ldconfig
```

在aios/build_service/build_service/tools/partition_split_merger/SConscript里添加了依赖库
env.aCheckLibrary('swift_client_minimal')

— plugin_platform/indexer_plugins/aitheta_indexer 编译 —
缺少文件aitheta/index_framework.h，无法编译

- ha3编译
