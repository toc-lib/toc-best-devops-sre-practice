# TOC DevOps/SRE 最佳实践



## 关于最佳实践

这里是TOC在不同项目的维护过程中总结的关于DevOps/SRE方面的最佳实践，我们将致力于在项目上尽最大的努力来推行这些最佳实践。我们希望这些最佳实践能对项目的稳定运营提供帮助，也希望刚接触DevOps/SRE的新人能通过学习这些最佳实践来提升自己在这方面的水平。

因为我们希望在项目上推行行业领先的流行理念和技术，所以这里整理的最佳实践也多基于此。虽然也会覆盖到一些传统的技术，但是我们建议尽早摒弃传统的维护思维和方式，避免落于人后。

最后，所谓“最佳实践”应该是**最适合自己**的实践，而**不一定是最先进**的，所以这里总结的最佳实践请适度取用，不要为了“最佳”而实践。



## 主题
- 用户与权限
- 敏感数据
- 云平台与网络
- 操作系统和服务
- 应用开发和部署
- 监控与可视化
- 数据与备份
- 故障与应急响应



## 用户与权限
#### 给每个用户建立独立的账号

  在任何的系统中，我们都强烈建议给每个用户(包括服务)建立独立的账号，而非使用共享账号。虽然这样会带来一些管理上的成本，但是独立用户账号所属和权责明确，更容易实施最小化权限管理，用户也可以有自己特性化的设定，独立的密码。另外，独立账号的使用也会大大减小账号泄露的风险。即使不慎发生账号泄漏，隔离泄漏账号所带来的影响范围更小，评估风险和操作审计等后续工作也会比较容易。 

#### 建立服务专用的账号

  一些自动化工具会需要和相关的系统/平台进行交互操作，大部分的系统/平台都会对类似的操作鉴权，因此这些工具也需要对应账号来完成相应的验证。我们建议给类似的需求建立服务专用的账号，为方便管理，可以给账号名称上加上一些表意的前后缀，比如`svc`或者`machine_user`等来区分账号的属性。

  如果需要使用的第三方服务并不需要每个团队成员都注册账号，我们建议使用一个管理专用而非个人所属邮箱等信息来注册，以避免团队成员变动带来的账号无法保留等影响。

#### 最小化权限原则

  最小化权限原则是指系统的每个程序或者用户都应该使用完成工作所需的最小权限工作。最小权限原则限制操作所需的权限，降低账号或者系统在被恶意利用时造成的损失。因此在给账号或者角色赋权时，尽可能的只赋予操作所需的权限，而非随意的扩大权限范围。即使我们需要给一些临时的操作赋权，也不要给予不必要的额外权限，并在操作完成之后清理临时权限。如有可能，我们也建议新建临时账号来完成此类操作，而非扩大原有账号的权限。

#### 使用强密码策略

  我们强烈建议使用强密码策略，一个安全的密码应该不少于12个字符，至少有三种不同的字符，如数字，标点，大小写字母。应避免在密码中包含个人信息，如出生日期或名字，宠物或乐队。还要避免歌词，伴侣和常用词组等。并且我们建议不要使用重复密码，尽可能的在不同的系统中使用不同的密码，并定期更换密码。

#### 使用多重验证(MFA)

  如果对应的系统支持多重验证，我们建议开启使用多重验证功能。不要把密码管理工具和MFA工具安装在在同一设备上。

#### 使用角色而非用户账号

  一些平台，比如AWS，支持角色使用临时认证进行获取操作权限([参考1](https://docs.aws.amazon.com/zh_cn/general/latest/gr/aws-access-keys-best-practices.html#use-roles))，所以我们建议在你的业务或者操作支持的情况下，使用角色而非用户账号来完成对应的操作。

#### 开启审计日志

  如果你的系统支持记录审计日志，我们建议请开启审计日志并保存至少半年的记录。审计日志本身是安全合规性检查的必备材料之一。审计日志也可以帮助我们进行疑难问题的定位，而定期的检查和分析也可以进一步发现可能存在的安全隐患。

#### 减少使用特权(root)账号

  在任何系统的日常管理工作中，在非必要的情况下，我们强烈建议不要使用特权(root)账号来做操作。特权账号具有系统所有权限，疏忽和不慎的操作有可能带来极大的损失。如果是在多人管理的情况下，也会增加账号泄漏的风险。我们强烈建议对于特权账号施行一切必要的安全管理，比如强密码，开启多重验证，及时的操作审计等。

## 敏感数据
#### 不在代码库中明文保存敏感信息和数据

  在代码库中不要明文保存任何的敏感信息和数据，比如数据库的密码，API验证使用的Token等。如果条件限制无法通过使用专用密码管理来获取密码，可以考虑在代码库保存加密过的数据，请[使用公开可靠的加密算法和工具](#使用公开可靠的加密算法和工具)来加密对应的数据。如果敏感信息和数据不慎被提交到远端的代码库，如果代码库为公开模式，立即设置代码库为私有模式，及时[清理历史提交记录](https://docs.github.com/en/github/authenticating-to-github/removing-sensitive-data-from-a-repository)。

#### 加密存储和传输数据

  主流的云服务厂商提供了存储加密功能，比如AWS的RDS，S3，EBS等都支持服务器端加密。我们建议开启加密功能，如成本允许或有更高级别的安全需求，可以使用自持有的密钥来加密。这样可以最大限度的减小存储介质丢失的情况下数据被泄漏的风险。如果服务提供商不能提供服务端加密，可以采用客户端加密的方式，即在本地[使用公开可靠的加密算法和工具](#使用公开可靠的加密算法和工具)加密需要保护的数据，然后保存在远端。在传输数据的过程中，我们建议使用加密的方式来传输，比如使用安全套接字层/传输层安全协定 (SSL/TLS) 保护传输中的数据。如果是工作交流中需要发送敏感的信息，我们推荐交换GPG公钥加密需要传输内容，而不是通过聊天软件或者邮件直接明文发送这些信息。

#### 使用密码管理工具管理账号和密码

  无论是何种类型的账号和密码，我们都建议使用密码管理工具来进行管理，都应极力避免使用文本文件或电子表格来保存。一些需要共享的账号，比如服务专用账号，特权管理账号等也通过密码管理工具来进行信息共享，并根据需要分级别控制读取和管理权限。如果管理工具支持，可以开启存储加密，定期密码更新等策略。

#### 使用公开可靠的加密算法和工具

  在加密数据的过程中，[不要使用保密的密码算法](https://www.ituring.com.cn/book/miniarticle/129179)，这是非常不安全的行为，我们强烈建议使用公开可靠的加密算法和工具，并且在所使用的算法不可靠的时候，及时的更新加密的算法和工具。

## 云平台与网络

#### 基于账号的环境隔离

  在使用云平台的时候，我们强烈建议给每个产品的每个环境都建立独立的账号，以达到最大程度的隔离。诚然这样会带来一些额外的成本，但是相对于带来的好处，比如说在不同账号中通过不同的权限来控制资源的操作，以降低对重要环境相关资源误操作的风险；避免某一个账号的验证信息泄漏导致所有资源面临风险；需要清理某一个废弃产品所占用的所有资源等；这个成本是非常值得的。有一些云服务提供商比如AWS可以把所拥有的一个账号升级为Orgnization账号，通过它对组织所有的子账号进行分组，账单关联和基于账号级别的策略设置等，也有助于我们更好的管理拥有的账号。

#### 分层网络设计

  我们建议在初建一个VPC网络时，至少包含以下四种类型的子网。1. 公网网络(Public Subnet)，这个子网的资源在配有一个公网的IP时，可被外部网络直接访问到。一般我们会把负载均衡，防火墙等需要直接对外暴漏的资源放置在这个子网内，这个子网内的资源一般不不会限制访问来源。2. 地址转换网络(NAT Subnet)，这个子网的资源通过地址转换之后可以访问外部网络，但是不能被外部网络直接访问，即使资源配有一个公网IP。我们一般把需要访问外部的业务和应用部署在这个网络内，但是这些业务在被外部网络访问的时候，需要通过负载均衡或者防火墙转发流量。3. 私网网络(Private Subnet)，这个子网的资源不可以被外部网络访问，同时也不可以访问外部网络。一般我们会把数据库等比较重要的资源和服务放在这里网络里面。4. 服务网络(Service Subnet)，这个子网和公网网络一样，可以直接被外网访问，我们一般在这个网络里面部署VPN，堡垒机等内部通讯使用的资源和服务，子网内的资源会被严格限制只允许特定的来源才能访问。除此之外，你可以根据自己的情况增加其他类型的子网。

#### 只开放必要的端口

  我们建议在使用访问控制列表(ACL)和安全组(Security Group)的时候，只开放必要的端口给需要访问的网络，避免使用宽泛的访问策略，防止一些服务的端口意外暴露在外部网络中，最大限度的保障网络和业务的安全。

## 操作系统和服务

#### 不要直接暴漏资源的地址给使用者

  由服务商提供的资源地址，不论是域名还是IP，都不要直接暴露给使用者。我们建议用另外一个可被管理的域名，使用CNAME（如果你的域名提供商和资源地址支持Alias则更好）的方式映射到原资源，把我们管理的域名提供给使用者来使用。这样带来的好处是如果我们需要对资源本身做更改的话，不需要使用者也做信息上的更新。有一些服务商提供的资源名称带有证书，映射后的域名会碰到证书不匹配的问题，这种情况下可以使用一个支持证书的负载均衡设备来代理对应的服务，使用者和负载均衡设备之间使用我们管理的域名证书通讯，负载均衡设备和后端的资源之间使用资源的证书通讯。

#### 只安装必要的依赖和工具

  在业务运行的系统中，无论是虚拟机还是容器中，只安装必要的依赖和工具，这样不但可以保证较小的系统大小，也能最大化避免因为工具和服务的漏洞造成被攻克的风险。如果你需要对一个有问题的业务进行排错，在隔离相应的虚拟机或者容器后安装需要的工具。在盘排错结束之后，及时[销毁这台虚拟机或者容器](#使用牲口模式来设计业务)。

#### 只运行必要的服务和端口

  对应的，在系统中只运行必要的服务，不要运行不相关的服务并开放对外访问权限。如果服务本身的业务端口和管理端口分离，根据需要限制两种端口的访问权限。

#### 加强SSH服务安全

  我们建议在你的堡垒机等需要提供SSH服务并对外部网络开放的设备上，使用非标准的端口来提供SSH服务，以减少暴力破解的风险。关闭root账号登陆，尽量使用密钥验证来登陆服务器。尽量避免使用SSH Agent，如有可能，关闭Agent转发功能。如果你不需要在设备上使用隧道转发功能，关闭X11转发和TCP转发。如果对于安全有进一步的要求，开启MFA登陆验证。

## 应用开发和部署

#### 尽可能遵循12-factors

  在业务设计和开发过程中，尽可能的遵循[12-factors](https://12factor.net/zh_cn/)。虽然这是一套SaaS服务开发的方法论，但是对于云平台下的软件开发设计也非常有借鉴意义。遵循这些原则，有助于我们开发出高质量和高维护性的应用。

#### 使用牲口模式

  我们建议使用[牲口模式](https://www.youtube.com/watch?v=zWgq6sd1Ols)来设计和维护你的应用。请参考12-factors的[进程](https://12factor.net/zh_cn/processes)部分设计业务，使逻辑和数据分离，除了缓存数据之外，逻辑所消费和产生的数据不应依赖于自身的存储。在维护的过程中，服务实例在其生命周期中不应有任何变化，和其最初产生时的状态一致。服务所在的实例可以被随时终止，这样当集群中的某个实例出现问题，都可以马上使用一个新的实例来替代。

#### 使用弹性伸缩

  我们建议给你的业务启用弹性伸缩，弹性伸缩可以根据需要业务的实际消耗情况自动调整所需资源的数量，在增强业务健壮性的同时，也可以提升资源的有效使用率。你可以根据业务的性质使用CPU，内存使用率抑或是访问量等单一或者组合指标来作为伸缩的依据。如果你的业务对于资源的消耗具有规律性，你也可以使用计划性的伸缩来应对周期性的变化。在有某些特殊需要的时候，你也可以通过手工调整弹性伸缩的配置来控制所需要的资源数量。

#### 让业务升级向前兼容

  在做业务升级时候，设计你的业务具有向前兼容的能力，以应对升级失败时某一功能模块或者依赖无法随之回滚的风险。比如说在有数据库字段变化的升级中，在正式对数据库做变动之前，基于旧的业务流程做代码层面更新，使其可以兼容数据库将要发生的改动并加以部署。在数据库升级完成之后，如果新的业务流程上线后不幸出现重大的问题等情况需要回滚时，回滚之后的代码仍然可以兼容数据库的变化，而不用对数据库也进行回滚。

#### 使用唯一性标识给镜像打标签

  当生成容器镜像时，使用一些唯一性标识来给镜像打标签，比如git commit hash等关联性比较强的标识。虽然时间戳也是一个唯一性比较强的标识，但是关联性相对较差，如果长度不足，也有一定的几率产生碰撞。我们不建议使用pipeline的build号来作为镜像的标签，因为如果需要更换工具或者为此应用重新生成pipeline时，这个号码将会被重置，将会覆盖之前生成的容器镜像。使用具有唯一性标识的镜像可以确保部署所对应的源代码版本的准确性，这样在出现问题溯源的时候可以对应到唯一的变化而不是可能存在的多个变化。关于生成容器镜像的更多实践，请参考[爱扯淡 - docker build的好实践和坏实践](https://blog.toc-platform.com/2020/05/21/ichegg-docker-build-practice/)。

#### 管理脚本和业务脚本分离

 我们的应用中一般都会有一些脚本来做一些辅助性的工作。一般来说，这些脚本分为两种：一种是用来做应用打包，部署等管理相关工作工作，这种类型的脚本也是无需打包进业务运行所需的产出物中的。另外一种就是辅助业务运行相关的，比如说初始化环境和配置，结束时的清理工作等，这种脚本则无需打包到产出物中。我们建议除了一些有特殊要求的脚本外，不要把脚本放在根目录。并且把这两种不用类型的脚本存放在不同的目录中。对于管理脚本，我们建议使用[auto/Action](https://github.com/toc-lib/OFC/blob/master/auto-ACTION-scripts.md)模式来编写和管理。

#### 减少脚本/工具对环境的依赖

  一般情况下，脚本都会或多或少的使用到一些外部工具。而我们的脚本很有可能会运行在不同的环境中，不同环境中提供的工具也会有版本和用法的差异。我们建议尽可能的减少所使用的工具对环境的依赖，尤其是系统不会默认安装的工具。另外在编写脚本的时候，也尽量避免使用只有某些版本才有的语法特性。这些情况都会导致管理脚本有可能出现一些不可预期的结果，我们建议使用容器化工具或者容器化环境管理工具如[Batect](https://batect.dev/)来替代对应的需求。

#### 使用容器并定期的更新操作系统

  容器技术非常好的解耦了应用和操作系统的之间的依赖关系，使得业务和操作系统之间可以独立的升级。所以我们极力推荐你把自己的应用容器化，并定期的更新容器的操作系统和容器运行的操作系统来保障业务的安全性。

#### 定期的检查和升级依赖包

  我们建议定期的对应用的依赖包做更新和安全检查，并升级到一个合适的版本。我们建议在应用的pipeline中加入这些检查任务，并在常规的开发过程中及时的升级。如果应用已经处于维护阶段，我们也建议定期执行这些检查并在需要的时候加以升级。这样可以让应用的安全性和代码的可用性都有保障。而不是使应用处在长期的无人问津状态，之后在某一天需要进行改动的时候，面临大量的依赖包过期无法获取和版本升级造成的接口变化。这时就需要投入非常高的成本来让代码重新变得可用，甚至完全无法更新而变成遗留系统。

#### 定期的重新部署维护阶段的应用

  如果一个应用已经处于交付之后的维护阶段，不再有日常的开发部署。即使因为一些原因无法做定期的应用依赖升级，我们也建议你定期的重新部署这个应用，以应对平台等更底层的变化带来的部署失败的风险。定期部署可以确保你的应用在新的平台环境中也可以正常的部署且。因为平台升级都是分阶段分批进行的，如果应用无法在新的环境部署，你也会有一个缓冲期来制订应对策略，而不是在平台完成升级之后的某一天，应用发生了问题才发现已经无法部署。

## 监控与可视化

#### 使用存活检查和健康检查

  我们建议你的应用中至少实现两种类型的监控接口，我们称作存活检查和健康检查。存活检查只检查应用自身的运行状态，如果应用自身能够正常启动并运行，接口就返回正常的状态。健康检查除了检查自身的状态之外，还需要检查业务所依赖的其他的服务，比如数据库是否能够正常连接。结合我们的[牲口设计模式](#使用牲口模式)，存活检查可以让管理工具尝试重启或是杀死对应的实例来保证正常运行所需的实例数量。而健康检查则是给前端的负载均衡把对应的实例中服务集群中移除。我们并不建议两种检查使用同一个接口，因为当依赖出现问题时，重启或者杀死实例并不是一个有效的解决方案。具体请参考[应用监控接口](https://github.com/toc-lib/OFC/blob/master/Application-Supported-Monitoring.md)中`/diagnostic/liveness`和`/diagnostic/readiness`。

#### 给你的应用添加分析接口

  我们建议给你的应用加上服务和依赖分析接口，这个接口应该检查到业务相关的所有依赖服务和资源的状态情况，并以可读性比较高的方式呈现结果。如果某一个依赖对外暴漏了多个节点，那么在这种接口中应该对每一个节点都去做检查。具体请参考[应用监控接口](https://github.com/toc-lib/OFC/blob/master/Application-Supported-Monitoring.md)中`/diagnostic/diagnosis`

#### 在输出中提供应用版本和主机信息

  我们建议在你的应用的每次输出中都加上应用版本和主机的信息，尤其是你的应用是以集群的方式运行的情况下。在碰到一些特殊情况，比如存活检查和健康检查遗漏了某些检查内容，导致有问题的实例没有被重启或移除，抑或滚动更新时有一台实例没有成功升级到新版本但是显示更新完成了。这些输出将有助于我们定位出问题的具体实例和应用版本，以及是否和特定的环境有关系。

#### 分级报警信息

  我们会对应用做各种类型的监控并配置报警，但是这些信息的重要程度和需要响应的紧急程度也是不一样的，比如说数据库的连接数量超过上限的90%和磁盘使用空间超过90%，显然后者我们可以延缓处理，而前者如果不及时处理就有可能会造成网站不能正常响应。我们建议对于报警信息至少分为两类：一类是需要马上响应的请求，在接到报警的时候我们就应该放下当前的工作立即去处理。另外一种是可以被滞后处理的，即使我们之后忘记了，在收到第二次报警再处理也不会对业务有什么影响。值守人员配备的工具上，应该只收到第一种报警，而不是所有类型的报警信息都能收到，避免过多的非重要报警信息导致的懈怠而造成疏忽了重要消息而没有及时处理。第二类消息我们建议记录在一些可供查阅的渠道，作为每日的巡检环节之一并加以处理。你可以根据具体的情况再次细分，但是我们并不建议做过多的分级，因为会带来比较高的管理和配置成本。

#### 避免报警范围外延

  如果应用的某些依赖不属于我们可控制范围的，比如第三方的服务，或者外部网络故障。我们可以设置监控，但是对应的故障信息记录在一些可查阅的渠道，以辅助我们在业务受到影响时分析即可。不要把这类信息纳入报警信息范围，因为即使你接受到了，很大程度上可能也做不了什么，尤其是外部网络波动。

## 数据与备份

#### 不要在关系型数据库中存储富文本数据

  虽然关系型数据库都支持存储富文本类型的数据，比如视频，图片等。但是在关系型数据库中存储富文本数据是一种反模式设计。我们强烈反对在关系型数据库中保存此类数据。因为这种数据会让数据库变得很大，在备份和恢复的时候都会消耗大量的时间。而且此类数据一般也很难做查询和关联，无法很好的利用关系型数据库的优点。我们建议把这类文件存在在外部存储，比如s3等，然后把访问地址等以原数据的形式存储在数据库中。

#### 使用适当的存储类型保存数据

  各大云服务提供商提供了不同细分类型的存储来满足不同场景的需求，这些存储类型从价格，可靠性，恢复时长等有不小的差异。我们建议合理根据数据的特征和业务的要求使用适当类型存储来保存数据，来最大化的优化存储的成本。

#### 定期备份数据和检查备份

  数据是信息时代业务的命脉，如果重要的数据不慎丢失或损坏而无法恢复，可能会对一个企业带来灾难性的后果，所以数据的备份至关重要。我们建议根据业务制定的RPO(Recovery Point Objective)定期的备份相关的数据，并且周期性的对备份的数据做完整性检查和恢复测试，以保证所备份的数据是有效的。

## 故障与应急响应

#### 建立应急响应计划

  故障的发生是不可避免的，所以我们极力建议团队应该切合自身的实际情况制定一套应急响应计划。在问题发生的时候按照流程及时的调动需要的资源来应对由此产生的风险，让损失降低到最低的程度。一个合理的应急响应流程应该包含以下的核心要素：1. 明确的事故风险等级评估，以此来判定问题影响的范围和应对的方式；2. 何种情况下由什么级别的人员来介入，以及对应的人员联系方式；3. 合理的团队组合人员分工，比如谁来负责沟通协调，谁来做具体问题解决和处理等；4. 明确的沟通渠道和进展汇报机制，让受到影响的利益相关者都能了解到最新的情况。5. 完整的事故溯源和应对措施，事故分析可以帮助我们评估造成的损失，了解问题产生的原因，从中学习并制定相应的措施来避免同类事件的再一次发生。

#### 应急响应培训

  在制定了应急响应计划，我们需要让团队的成员熟悉计划的内容。除此之外，我们还应告知响应流程中需要合作的非团队人员，以确保他们清楚的了解自己在何种情况下会需要一起配合应对发生的问题和自己的职责。如果条件允许，可以定期的做应急响应的培训和演练，尤其是对刚加入团队的新成员。以确保在问题发生的时候，成员知道如何合理的应对，而不是置之不理甚至使用错误的对应方式应对，导致问题的进一步恶化或者关键信息丢失无法溯源等，造成更大的损失。

#### 非问责式事故分析

  事故分析除了帮助我们评估造成的损失，还是一次很好的学习机会，我们可以在分析中了解问题发生的原因，制定相应的措施来避免同类事件的再一次发生。但是很多的团队在做事故分析的时候往往倾向于把问题原因归结到导致问题发生的当事人身上并加以惩罚，这是非常不好的一种方式，会对团队成员的责任心产生极大的打击，长期以往会造成一种做多错多，然后不求有功，但求无过的做事方式。我们建议采用非问责式的事故分析，相信团队的每一个人都是做了在当时的能力的范围内最合理的决定，事故的发生并不是谁故意造成的。在做事故溯源时，不要使用成员的名字，而是使用团队来代替，让团队成员认识到团队本身是一个整体，该为这件事情来负责，而不是那个不幸做了这件事情的人来负责。