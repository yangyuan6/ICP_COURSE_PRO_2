# 代码设计部分

主要有两个文件，types.mo是一些类型定义，以及关于**Proposal**提议的操作函数。  

main.mo是主文件。  

```rust
  var proposals : Buffer.Buffer<Proposal> = Buffer.Buffer<Proposal>(0);
  var ownedCanisters : [Canister] = [];
  var ownerList : [Owner] = [];
```

**proposals**保存了所有**提议**的列表  

**ownCanisters**保存了所有创建的**Canister**的列表

**ownerList**保存了管理员列表

首先，需要调用`init()`函数，进行初始化，传入管理员Principal列表，以及阈值**M**  

阈值**M**需要大于0，小于等于Principal列表人数

任何一个管理员都可以发起提议`propose()`或者支持提议`approve()`

```rust
public type Proposal = {
    id: ID;
    proposer: Owner;
    wasm_code:  ?Blob; // valid only for install code type
    ptype: ProposalType;
    canister_id:  ?Canister; // can be null only for create canister case
    approvers: [Owner];
    finished: Bool;
};
```

提议包括的数据项：
- **id**，唯一的ID
- **proposal**，提议人
- 可选的**wsm_code**
- **ptype**，提议类型
- 可选的**canister_id**
- **approvers**，已经同意该提议的管理员列表
- **finished**，提议是否执行


```rust
public type ProposalType = {
    #installCode;
    #uninstallCode;
    #createCanister;
    #startCanister;
    #stopCanister;
    #deleteCanister;
};
```

提议类型包括：
- 创建canister
- 启动canister
- 停止canister
- 删除canister
- 在canister中安装代码
- 在canister中卸载代码

发起提议定义：
```rust
propose(ptype: ProposalType, canister_id: ?Canister, wasm_code: ?Blob) : async Proposal
```

发起提议时，需要根据提议类型，提供可选的**canister_id** 以及 **wasm_code**

当提议类型不是创建canister时，需要提供**canister_id**

当提议类型是安装代码时，需要提供**wasm_code**

发起提议，返回**proposal**

支持提议定义：
```rust
approve(id: ID) : async Proposal
```

支持提议时，只需提供提议**id**
支持提议，返回**proposal**

提议被创建后，提议的approvers为空，发起人还需要再次调用`approve()`去支持该提议

当提议的支持者数量，达到阈值**M**后，会立即执行该提议。这部分使用到了ic management API里面的内容。具体参考[这里](https://github.com/alexxuyang/icp_course_H_2/blob/408d46f2c8eb7ecb3c8a80e320b901c5edf04f68/src/icp_course_H_2/main.mo#L81)。

# 测试部分

由于测试与流程比较复杂，我们使用了[ic-repl](https://github.com/chenyan2002/ic-repl)工具。

在代码根目录有三个identity文件，id1、id2、id3，我们会使用这三个id文件。

测试的多签模式是：2/3，总共有三人，当有2人同意时，就会执行提议。

首先需要在代码根目录执行：

```shell
dfx start --clean
```

然后部署智能合约：

```shell
dfx depoly
```

执行测试案例一：

```shell
ic-repl ./create-install-start-call.sh
```

测试案例一，包括了创建canister、安装代码、启动canister、调用canister。

除了调用canister，其它三步，都是通过提议、多签方式进行提案与审批的。

最后可以看到调用到了动态生成的智能合约的结果（greet合约）：

```shell
"Hello, world!"
```

在dfx服务端，也有类似输出可以看到：

```shell
 May 19 18:30:35.498 INFO Log Level: INFO
 May 19 18:30:35.498 INFO Starting server. Listening on http://127.0.0.1:8000/
[Canister rrkah-fqaaa-aaaaa-aaaaq-cai] ("Caller: ", ndb4h-h6tuq-2iudh-j3opo-trbbe-vljdk-7bxgi-t5eyp-744ga-6eqv6-2ae, ". Iint with owner list: ", [cnh44-cjhoh-yyoqz-tcp2t-yto7n-6vlpk-xw52p-zuo43-rrlge-4ozr5-6ae, ndb4h-h6tuq-2iudh-j3opo-trbbe-vljdk-7bxgi-t5eyp-744ga-6eqv6-2ae, lzf3n-nlh22-cyptu-56v52-klerd-chdxu-t62na-viscs-oqr2d-kyl44-rqe], "M=", 2, "N=", 3)
[Canister rrkah-fqaaa-aaaaa-aaaaq-cai] (ndb4h-h6tuq-2iudh-j3opo-trbbe-vljdk-7bxgi-t5eyp-744ga-6eqv6-2ae, "PROPOSED", #createCanister, "Proposal ID", 0)
[Canister rrkah-fqaaa-aaaaa-aaaaq-cai] ()
[Canister rrkah-fqaaa-aaaaa-aaaaq-cai] (ndb4h-h6tuq-2iudh-j3opo-trbbe-vljdk-7bxgi-t5eyp-744ga-6eqv6-2ae, "APPROVED", #createCanister, "Proposal ID", 0, "Executed", false)
[Canister rrkah-fqaaa-aaaaa-aaaaq-cai] ()
[Canister rrkah-fqaaa-aaaaa-aaaaq-cai] (cnh44-cjhoh-yyoqz-tcp2t-yto7n-6vlpk-xw52p-zuo43-rrlge-4ozr5-6ae, "APPROVED", #createCanister, "Proposal ID", 0, "Executed", true)
[Canister rrkah-fqaaa-aaaaa-aaaaq-cai] ()
[Canister rrkah-fqaaa-aaaaa-aaaaq-cai] (ndb4h-h6tuq-2iudh-j3opo-trbbe-vljdk-7bxgi-t5eyp-744ga-6eqv6-2ae, "PROPOSED", #installCode, "Proposal ID", 1)
[Canister rrkah-fqaaa-aaaaa-aaaaq-cai] ()
[Canister rrkah-fqaaa-aaaaa-aaaaq-cai] (ndb4h-h6tuq-2iudh-j3opo-trbbe-vljdk-7bxgi-t5eyp-744ga-6eqv6-2ae, "APPROVED", #installCode, "Proposal ID", 1, "Executed", false)
[Canister rrkah-fqaaa-aaaaa-aaaaq-cai] ()
[Canister rrkah-fqaaa-aaaaa-aaaaq-cai] (cnh44-cjhoh-yyoqz-tcp2t-yto7n-6vlpk-xw52p-zuo43-rrlge-4ozr5-6ae, "APPROVED", #installCode, "Proposal ID", 1, "Executed", true)
[Canister rrkah-fqaaa-aaaaa-aaaaq-cai] ()
[Canister rrkah-fqaaa-aaaaa-aaaaq-cai] (ndb4h-h6tuq-2iudh-j3opo-trbbe-vljdk-7bxgi-t5eyp-744ga-6eqv6-2ae, "PROPOSED", #startCanister, "Proposal ID", 2)
[Canister rrkah-fqaaa-aaaaa-aaaaq-cai] ()
[Canister rrkah-fqaaa-aaaaa-aaaaq-cai] (ndb4h-h6tuq-2iudh-j3opo-trbbe-vljdk-7bxgi-t5eyp-744ga-6eqv6-2ae, "APPROVED", #startCanister, "Proposal ID", 2, "Executed", false)
[Canister rrkah-fqaaa-aaaaa-aaaaq-cai] ()
[Canister rrkah-fqaaa-aaaaa-aaaaq-cai] (cnh44-cjhoh-yyoqz-tcp2t-yto7n-6vlpk-xw52p-zuo43-rrlge-4ozr5-6ae, "APPROVED", #startCanister, "Proposal ID", 2, "Executed", true)
[Canister rrkah-fqaaa-aaaaa-aaaaq-cai] ()
```

执行测试案例二：

```shell
ic-repl ./create-install-start-call-stop-uninstall-delete.sh
```

测试案例二，包括了创建canister、安装代码、启动canister、调用canister、停止canister、删除代码、删除canister。

除了调用canister，其它六步，都是通过提议、多签方式进行提案与审批的。