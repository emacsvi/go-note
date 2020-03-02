---
title: filecoin技术归档
slug: filecoin-archieve
date: "2019-11-27"
description: filecoin技术归档
categories: 
- filecoin 
tags: 
- filecoin 
---


filecoin技术归档。
<!--more-->

## 技术归档

11月27日我和石化金开始研究lotus如何在本地集群搭建**私链**来测试，其中遇到的技术难点粗略地记录下来：

- 私链利用**脚本搭建**完成。私链实现的功能如下：

  - 利用`--lotus-make-random-genesis`创建随机的`devnet.car`种子文件。并且由这个随机种子来创建特定的**genesis**文件。在生成**genesis**文件之后，会**铸币**以及创建**初始块**以及创建**state**的**merkleDAG**信息。包括在**铸币**的时候对默认地址分配**100000**的余额等。完成之后将所有信息写入devnet.car文件里面进行打包。由lotus进行读取解析并且同步。在代码里面固定。如下所示：

    ```go
    func MakeGenesis(outFile string) func(bs dtypes.ChainBlockstore, w *wallet.Wallet) modules.Genesis {
    	return func(bs dtypes.ChainBlockstore, w *wallet.Wallet) modules.Genesis {
    		return func() (*types.BlockHeader, error) {
    			glog.Warn("Generating new random genesis block, note that this SHOULD NOT happen unless you are setting up new network")
    			gmc := &gen.GenMinerCfg{
    				Owners:  []address.Address{minerAddr},
    				Workers: []address.Address{minerAddr},
    				PeerIDs: []peer.ID{"peer ID 1"},
    			}
    
    			addrs := map[address.Address]types.BigInt{
    				minerAddr: types.FromFil(100000),
    			}
    
    			b, err := gen.MakeGenesisBlock(bs, addrs, gmc, uint64(time.Now().Unix()))
    			fmt.Println("GENESIS MINER ADDRESS: ", gmc.MinerAddrs[0].String())
    			offl := offline.Exchange(bs)
    			blkserv := blockservice.New(bs, offl)
    			dserv := merkledag.NewDAGService(blkserv)
    
    			if err := car.WriteCar(context.TODO(), dserv, []cid.Cid{b.Genesis.Cid()}, f); err != nil {
    				return nil, err
    			}
    		}
    	}
    }
    ```

    

  - 在**bootstrap目录**里面配置私链**种子节点**的配置信息。这些配置内容配置完成之后，重新编译lotus程序，将编译好的程序让到私链其他节点，让其他节点与本种子节点进行通信连接。如下内容：
  
    ```bash
    liwei@liwei:~/26Lotus/build/bootstrap$ cat bootstrappers.pi
    /ip4/10.0.48.82/tcp/1347/p2p/12D3KooWKuy1VYzDxCPVy2GLdiFf1a6eVa8jGToe6mN9Xae4efDi
    liwei@liwei:~/26Lotus/build/bootstrap$ cat root.pi
    /ip4/10.0.48.129/tcp/1347/p2p/12D3KooWLRUUrzoxk3DE9No6cJGgjMxYWdAaiddcNDAJVqg4otZW
    liwei@liwei:~/26Lotus/build/bootstrap$
    ```
  
  - **bootstrap.toml**配置本节点的监听端口。所有节点的监听端口必须统一以便p2p通信。
  
    ```toml
    [Libp2p]
    ListenAddresses = ["/ip4/0.0.0.0/tcp/1347"]
    ```
  
    
  
  - 初始化第一个**矿工角色**。注册该矿工到链上的过程。
  
    ```bash
    lotus-storage-miner init --genesis-miner --actor=t0101
    ```
  
  - 将编译好的`lotus`与`lotus-storage-miner`程序上传到私链其他节点，并且将`bootstrap.toml`上传到`.lotus/config.toml`文件中，表示让对方也同样监听在**1347**端口上进行数据的协商。对应的**shell命令**如下：
  
    ```bash
    scp scripts/bootstrap.toml "${host}:.lotus/config.toml"
    ssh "$host" "echo -e '[Metrics]\nNickname=\"Boot-$host\"' >> .lotus/config.toml"
    ```
  
    最终`.lotus/config.toml`内容在**种子节点**如下：
  
    ```toml
    [Libp2p]
    ListenAddresses = ["/ip4/0.0.0.0/tcp/1347"]
    [Metrics]
    Nickname="Boot-genesis"
    ```
  
    在**私链节点**如下：
  
    ```toml
    [Libp2p]
    ListenAddresses = ["/ip4/0.0.0.0/tcp/1347"]
    [Metrics]
    Nickname="Boot-ipfs@10.0.48.82"
    ```
  
    
  
  - 私链代码有问题，需要改`createminer.go`里面的一段代码。才能创建矿工。更改之后再次编译上传到各个节点。更改如下
  
    ```go
    		msg := &types.Message{
    			To:       actors.StoragePowerAddress,
    			From:     addr,
    			Method:   actors.SPAMethods.CreateStorageMiner,
    			Params:   params,
    			Value: types.NewInt(51), // Value:    types.NewInt(0),
    			GasPrice: types.NewInt(0),
    			GasLimit: types.NewInt(10000),
    		}
    ```
  
    **创建矿工**需要用到的命令如下：
  
    ```bash
    # 获取节点的peer id
    peerID = lotus net id
    # 获取地址，这里将worker地址与奖励地址设置为同一个
    address = lotus wallet list(随便一个地址作为奖励即可)
    ownerAddress = address
    # 创建矿工角色,sectorSize需要如下计算结果：
    var SectorSizes = []uint64{
    	16 << 20,
    	256 << 20,
    	1 << 30,
    }
    actorID = lotus createminer address ownerAddress sectorSize(16<<20 and so on.) peerID
    # 这里面的代码我更进发现，在创建矿工角色的时候会打开本地默认的钱包作抵押，并且在vm里面执行所有过程并且放到链上。
    # 下面初始化
    lotus-storage-miner init --actor=actorID --owner=ownerAddress
    # 运行挖矿程序
    lotus-storage-miner run
    ```
  
    
  
  - 私链可以完成转账，生成交易记录。生成矿工角色以及挖矿。但是连续提供算力依然会出错。这说明官方的代码本身是有问题的。因为私链的测试结果和连接官方的devnet链结果现象是一致的。
  
  - 将搭建私链也容器化，方便调试。
  
- 其他的研究讨论内容：

  - 由于Gpus对密码速度与PoSt的速度提升很大，可能需要用到2080ti显卡。
  - VDF没有了，现在在出块的过程中修改为SurprisePoSt和ElectionPoSt证明。这会对计算的速度要求比较高。所以按官方的意思，如果你即使拥有了出块权，但是你的速度太慢，出的块可能会延时最终成为了孤块。
  - vm的执行在第一个tipset里面由一个miner去执行一次。
  - 如果一个period challenge没有完成或者失败，你不能参与EC共识，但是你可以在你的有效算力清除前及时清除challenge.

## 参考文献

- [issue620](https://github.com/filecoin-project/specs/pull/620)
- [filecoinLotusImplementation](filecoinLotusImplementation)

