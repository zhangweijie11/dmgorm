## 达梦数据库（DM8）安装+Golang 使用

> 数据库名称+版本：达梦数据库 DM8
>
> 安装方式：Docker
>
> 编程语言+版本：Golang 1.20
>
> ORM+版本：gorm V1.25.9

​		随着国家对信创工程的日益重视，目前采用有自主知识产权的国产数据库将成为主流。因此公司要求使用的数据库需要由 MySQL 切换为达梦数据库。DM 数据库和 MySQL 体系结构上存在差异，SQL 语法也存在一定的差异。DM数据库针对 MySQL 做了部分兼容性。但由于有根本性的差异，兼容度不高。另外对于 Golang 语言的支持和Mac系统的支持也不够好，笔者也是详细阅读达梦的官方文档后，根据实际的操作整理出该文章。

## 一、简介

​		武汉达梦数据库股份有限公司成立于2000年，是国内领先的数据库产品开发服务商，国内数据库基础软件产业发展的关键推动者。公司为客户提供各类数据库软件及集群软件、云计算与大数据等一系列数据库产品及相关技术服务，致力于成为国际顶尖的全栈数据产品及解决方案提供商。DM8是达梦公司在总结DM系列产品研发与应用经验的基础上，坚持开放创新、简洁实用的理念，推出的新一代自研数据库。

​		DM8吸收借鉴当前先进新技术思想与主流数据库产品的优点，融合了分布式、弹性计算与云计算的优势，对灵活性、易用性、可靠性、高安全性等方面进行了大规模改进，多样化架构充分满足不同场景需求，支持超大规模并发事务处理和事务-分析混合型业务处理，动态分配计算资源，实现更精细化的资源利用、更低成本的投入。一个数据库，满足用户多种需求，让用户能更加专注于业务发展。

## 二、安装

**本次安装采用 Docker 方式安装**

### 下载 Docker 安装包

请在达梦数据库官网下载 [Docker 安装包](https://eco.dameng.com/download/)。

### 导入安装包

拷贝安装包到 /opt 目录下，执行以下命令导入安装包：

```
docker load -i dm8_20230808_rev197096_x86_rh6_64_single.tar
```

结果显示如下：

![企业微信截图_16928403528979.png](https://eco.dameng.com/eco-file-server/file/eco/preview/20230824092603MJ1W3VSZSVT3VNC1TQ)

导入完成后，可以使用 `docker images` 查看导入的镜像。结果显示如下：

![企业微信截图_16928404063815.png](https://eco.dameng.com/eco-file-server/file/eco/preview/20230824092656BQKXXECFVXCVEIL7VE)

### 启动容器

镜像导入后，使用 `docker run` 启动容器，启动命令如下：

```shell
Copydocker run -d -p 30236:5236 --restart=always --name dm8 --privileged=true -e PAGE_SIZE=16 -e LD_LIBRARY_PATH=/opt/dmdbms/bin -e  EXTENT_SIZE=32 -e BLANK_PAD_MODE=1 -e LOG_SIZE=1024 -e UNICODE_FLAG=1 -e LENGTH_IN_CHAR=1 -e INSTANCE_NAME=dm8 -v /data/dm8_test:/opt/dmdbms/data dm8_single:dm8_20230808_rev197096_x86_rh6_64
```

结果显示如下：

![企业微信截图_16928404765536.png](https://eco.dameng.com/eco-file-server/file/eco/preview/20230824092812KJJQ9A1887JM0PLLKR)

容器启动完成后，使用 `docker ps` 查看镜像的启动情况，结果显示如下：

![image.png](https://eco.dameng.com/eco-file-server/file/eco/preview/20220901112856M6H3O8KNQ9PNTFAXPV)

启动完成后，可通过日志检查启动情况，命令如下：

```shell
docker logs -f  dm8
或
docker logs -f 58deb28d1209
```

结果显示如下：

![企业微信截图_1692841166824.png](https://eco.dameng.com/eco-file-server/file/eco/preview/20230824093938HEWY0M7PLO45GMHD4Q)

### 启动/停止数据库

停止数据库命令如下：

```shell
Copydocker stop  dm8
```

启动数据库命令如下：

```shell
Copydocker start  dm8
```

重启命令如下：

```shell
Copydocker restart  dm8
```

> 注意
>
> 1.如果使用 docker 容器里面的 disql，进入容器后，先执行 source /etc/profile 防止中文乱码。
> 2.新版本 Docker 镜像中数据库默认用户名/密码为 SYSDBA/SYSDBA001。

### 进入容器 查看

```
# 进入容器
docker exec -it dm8 bash

# 登录
[dmdba@centos7_6_33 ~]$ cd /opt/dmdbms/bin
[dmdba@centos7_6_33 bin]$ ./disql SYSDBA/SYSDBA@192.168.6.33:5236

服务器 [192.168.6.33:5236]: 处于普通打开状态
登录使用时间: 2.341(毫秒)
disql V8
```

## 三、使用

使用共分为两种方式，一种是自己安装驱动包和依赖包，增加必要的文件后开始使用；第二种是直接导入`go get gitlab.example.com/zhangweijie/dmgorm `包开始使用

首先介绍第一种方式：

### 第一种方式：自主安装使用

#### 安装 dm 驱动包

将提供的 dm 驱动包（<a href="https://download.dameng.com/eco/adapter/resource/go/go-20230418.zip" rel="nofollow">Go 驱动源码</a>）下载。驱动源码为`go-20230418.zip`压缩包，解压缩后会出现名为 `go` 的文件夹，文件夹下有三个压缩包，分别为

`dm-go-driver.zip`：Go 的驱动源码,解压后会出现名为 `dm`的文件夹，将文件夹放到GOPATH 的 src 目录下

> 注意
>
> 由于该驱动不支持Mac系统，所以需要在`/dm/security`目录下增加名为`zzg_drawin.go`文件
>
> ```
> /*
>  * Copyright (c) 2000-2018, 达梦数据库有限公司.
>  * All rights reserved.
>  */
> 
> package security
> 
> import "plugin"
> 
> var (
> 	dmCipherEncryptSo           *plugin.Plugin
> 	cipherGetCountProc          plugin.Symbol
> 	cipherGetInfoProc           plugin.Symbol
> 	cipherEncryptInitProc       plugin.Symbol
> 	cipherGetCipherTextSizeProc plugin.Symbol
> 	cipherEncryptProc           plugin.Symbol
> 	cipherCleanupProc           plugin.Symbol
> 	cipherDecryptInitProc       plugin.Symbol
> 	cipherDecryptProc           plugin.Symbol
> )
> 
> func initThirdPartCipher(cipherPath string) (err error) {
> 	if dmCipherEncryptSo, err = plugin.Open(cipherPath); err != nil {
> 		return err
> 	}
> 	if cipherGetCountProc, err = dmCipherEncryptSo.Lookup("cipher_get_count"); err != nil {
> 		return err
> 	}
> 	if cipherGetInfoProc, err = dmCipherEncryptSo.Lookup("cipher_get_info"); err != nil {
> 		return err
> 	}
> 	if cipherEncryptInitProc, err = dmCipherEncryptSo.Lookup("cipher_encrypt_init"); err != nil {
> 		return err
> 	}
> 	if cipherGetCipherTextSizeProc, err = dmCipherEncryptSo.Lookup("cipher_get_cipher_text_size"); err != nil {
> 		return err
> 	}
> 	if cipherEncryptProc, err = dmCipherEncryptSo.Lookup("cipher_encrypt"); err != nil {
> 		return err
> 	}
> 	if cipherCleanupProc, err = dmCipherEncryptSo.Lookup("cipher_cleanup"); err != nil {
> 		return err
> 	}
> 	if cipherDecryptInitProc, err = dmCipherEncryptSo.Lookup("cipher_decrypt_init"); err != nil {
> 		return err
> 	}
> 	if cipherDecryptProc, err = dmCipherEncryptSo.Lookup("cipher_decrypt"); err != nil {
> 		return err
> 	}
> 	return nil
> }
> 
> func cipherGetCount() int {
> 	ret := cipherGetCountProc.(func() interface{})()
> 	return ret.(int)
> }
> 
> func cipherGetInfo(seqno, cipherId, cipherName, _type, blkSize, khSIze uintptr) {
> 	ret := cipherGetInfoProc.(func(uintptr, uintptr, uintptr, uintptr, uintptr, uintptr) interface{})(seqno, cipherId, cipherName, _type, blkSize, khSIze)
> 	if ret.(int) == 0 {
> 		panic("ThirdPartyCipher: call cipher_get_info failed")
> 	}
> }
> 
> func cipherEncryptInit(cipherId, key, keySize, cipherPara uintptr) {
> 	ret := cipherEncryptInitProc.(func(uintptr, uintptr, uintptr, uintptr) interface{})(cipherId, key, keySize, cipherPara)
> 	if ret.(int) == 0 {
> 		panic("ThirdPartyCipher: call cipher_encrypt_init failed")
> 	}
> }
> 
> func cipherGetCipherTextSize(cipherId, cipherPara, plainTextSize uintptr) uintptr {
> 	ciphertextLen := cipherGetCipherTextSizeProc.(func(uintptr, uintptr, uintptr) interface{})(cipherId, cipherPara, plainTextSize)
> 	return ciphertextLen.(uintptr)
> }
> 
> func cipherEncrypt(cipherId, cipherPara, plainText, plainTextSize, cipherText, cipherTextBufSize uintptr) uintptr {
> 	ret := cipherEncryptProc.(func(uintptr, uintptr, uintptr, uintptr, uintptr, uintptr) interface{})(cipherId, cipherPara, plainText, plainTextSize, cipherText, cipherTextBufSize)
> 	return ret.(uintptr)
> }
> 
> func cipherClean(cipherId, cipherPara uintptr) {
> 	cipherEncryptProc.(func(uintptr, uintptr))(cipherId, cipherPara)
> }
> 
> func cipherDecryptInit(cipherId, key, keySize, cipherPara uintptr) {
> 	ret := cipherDecryptInitProc.(func(uintptr, uintptr, uintptr, uintptr) interface{})(cipherId, key, keySize, cipherPara)
> 	if ret.(int) == 0 {
> 		panic("ThirdPartyCipher: call cipher_decrypt_init failed")
> 	}
> }
> 
> func cipherDecrypt(cipherId, cipherPara, cipherText, cipherTextSize, plainText, plainTextBufSize uintptr) uintptr {
> 	ret := cipherDecryptProc.(func(uintptr, uintptr, uintptr, uintptr, uintptr, uintptr) interface{})(cipherId, cipherPara, cipherText, cipherTextSize, plainText, plainTextBufSize)
> 	return ret.(uintptr)
> }
> 
> ```
>
>

`gorm_v1_dialect.zip`：适配 gorm v1版本，暂时不用

``gorm_v2_dialect.zip`：适配 gorm v2版本，解压后也会出现名为`dm`的文件夹，将文件夹放到项目能够找到的路径下即可，后续会使用

#### 安装依赖包

```plaintext
go get golang.org/x/text

go get github.com/golang/snappy
```

#### 开始使用

```
// 展示 GormV2 版本的用法
package main
import (
	"dm" // DM驱动包路径
	"gorm.io/gorm"
)

type Product struct {
	gorm.Model
	Code  string
	Price uint
}

func main() {
	db, err := gorm.Open(dm.Open("dm://SYSDBA:SYSDBA@10.99.60.8"), &gorm.Config{})
	if err != nil {
		panic("failed to connect database")
	}

	// 迁移 schema
	db.AutoMigrate(&Product{})

	// Create
	db.Create(&Product{Code: "D42", Price: 100})

	// Read
	var product Product
	db.First(&product, 1) // 根据整型主键查找
	db.First(&product, "code = ?", "D42") // 查找 code 字段值为 D42 的记录

	// Update - 将 product 的 price 更新为 200
	db.Model(&product).Update("Price", 200)
	// Update - 更新多个字段
	db.Model(&product).Updates(Product{Price: 200, Code: "F42"}) // 仅更新非零值字段
	db.Model(&product).Updates(map[string]interface{}{"Price": 200, "Code": "F42"})

	// Delete - 删除 product
	db.Delete(&product, 1)
}
```

### 第二种：导包使用

#### 导入现有包

```
go get gitlab.example.com/zhangweijie/dmgorm 
# 或者
go get github.com/zhangweijie/dmgorm 
```

#### 开始使用

```
package main

import (
	"fmt"
	"gitlab.example.com/zhangweijie/dmgorm/dmgorm"
	"gorm.io/gorm"
)

type Product struct {
	gorm.Model
	Code  string
	Price uint
}

func main() {
	db, err := gorm.Open(dmgorm.Open("dm://SYSDBA:SYSDBA001@10.99.60.8:5236"), &gorm.Config{})
	if err != nil {
		panic("failed to connect database")
	}

	// 迁移 schema
	db.AutoMigrate(&Product{})

	// Create
	db.Create(&Product{Code: "D42", Price: 100})

	// Read
	var product Product
	db.First(&product, 2) // 根据整型主键查找
	fmt.Println("------------>", product)
	db.First(&product, "code = ?", "F42") // 查找 code 字段值为 D42 的记录
	fmt.Println("------------>", product.ID, product.Price, product.Code)

	// Update - 将 product 的 price 更新为 200
	db.Model(&product).Update("Price", 200)
	// Update - 更新多个字段
	db.Model(&product).Updates(Product{Price: 200, Code: "F42"}) // 仅更新非零值字段
	db.Model(&product).Updates(map[string]interface{}{"Price": 200, "Code": "F42"})

	// Delete - 删除 product
	db.Delete(&product, 1)
}
```
