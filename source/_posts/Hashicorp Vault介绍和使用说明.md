---
title: Hashicorp Vault介绍和使用说明
date: 2022-09-21 18:22:20
tags:
  - RockyLinux
  - Ubuntu
  - Hashicorp Vault
  - Linux
categories:
  - Hashicorp Vault
---

## 1.概述

在本文中，我们将探索Hashicorp的Vault - 一种用于在现代应用程序体系结构中安全地管理机密信息的流行工具。

我们将讨论的主要议题包括：

* Vault试图解决什么问题
* Vault的架构和主要概念
* 设置一个简单的测试环境
* 使用命令行工具与Vault交互

## 2.机密信息问题

在深入了解Vault之前，让我们试着了解它试图解决的问题：机密信息管理。

大多数应用程序需要访问机密数据才能正常工作。例如，电子商务应用程序可以在某处配置用户名/密码以便连接到其数据库。它还可能需要API密钥才能与其他服务提供商集成，例如支付网关，物流和其他业务合作伙伴。

数据库凭证和API密钥就是我们需要以安全的方式存储和提供给我们的应用程序的机密信息。

一个简单的解决方案是将这些信息存储在配置文件中，并在启动时读取它们。但是这种方法的问题显而易见：有权访问此文件的人共享我们的应用程序具有的数据库权限 - 通常可以完全访问所有存储的数据。

我们可以尝试通过加密这些文件来使事情变得更加困难。但是，这种方法在整体安全性方面不会增加太多。主要是因为我们的应用程序必须能够访问主密钥。当以这种方式使用时，加密仅是一种“错误”的安全感。

现代应用程序和云环境往往会增加一些额外的复杂性：分布式服务，多个数据库，消息传递系统等等，所有敏感信息都在各处传播，从而增加了安全漏洞的风险。

<!--more-->
## 3.什么是Vault？

Hashicorp Vault解决了管理敏感信息的问题 - 在Vault的用语中使用 “secret”。在这种情况下，“管理”意味着Vault控制敏感信息的所有方面：它的生成，存储，使用以及最后它的撤销。

Hashicorp提供两种版本的Vault。本文中使用的开源版本可以免费使用，即使在商业环境中也是如此。同时还提供付费版本，其中包括不同SLA的技术支持和其他功能，例如HSM（硬件安全模块）支持。

### 3.1 架构和主要特点

Vault的架构非常简单。其主要组成部分是：

* 持久性后端 - 存储所有机密
* 一种API服务器 - 用于处理客户端请求并对机密执行操作
* 许多secret引擎 - 支持不同的机密类型

通过将所有机密处理委派给Vault，我们可以缓解一些安全问题：
我们的应用程序不再需要存储它们 ，只需在需要时询问Vault并在使用后将其丢弃
我们可以短暂的使用机密数据，从而限制攻击者盗取秘密的“机会之窗”

Vault会在将所有数据写入存储之前使用加密密钥对所有数据进行加密。此加密密钥由另一个密钥加密 - 主密钥，主密钥仅在启动时使用。

Vault实现的一个关键点是它不会将主密钥存储在服务器中。 这意味着即使Vault也无法在启动后访问其保存的数据。 此时，Vault实例被称为处于“密封”状态。

稍后，我们将完成生成主密钥和解封Vault实例所需的步骤。

一旦启封，Vault就可以接受API请求了。当然，这些请求需要身份验证，这使我们控制Vault如何对客户端进行身份验证并决定他们可以做什么或不做什么。

### 3.2 认证

要访问Vault中的机密，客户端需要使用一种受支持的方法对自身进行身份验证。最简单的方法使用Tokens，它只是使用特殊HTTP头在每个API请求上发送的字符串。

最初安装时，Vault会自动生成“根令牌”。此令牌与Linux系统中的超级用户等效，因此应将其使用限制在最低限度。作为最佳实践，我们应该使用此根令牌来创建具有较少权限的其他令牌，然后撤消它。但这不是问题，因为我们以后可以使用unseal键生成另一个根令牌。

Vault还支持其他身份验证机制，如LDAP，JWT，TLS证书等。所有这些机制都建立在基本令牌机制之上：一旦Vault验证了我们的客户端，它将提供一个令牌，然后我们可以使用它来访问其他API。

令牌有一些与之相关的属性。主要属性是：

* 一组关联的策略
* 有效期
* 是否强制更新
* 最大使用次数

除非另有说明，否则由Vault创建的令牌将形成父子关系。子令牌最多可以与父令牌具有相同级别的权限。

通常的情况是，我们可以创建具有限制性策略的子令牌。关于此关系的另一个关键点：当我们使令牌无效时，所有子令牌及其后代也会失效。

### 3.3 策略

策略确切地定义了客户端可以访问哪些秘密以及它们可以使用它们执行哪些操作。让我们看看一个简单的策略是如何定义的：

```sh
path "secret/accounting"{
capabilities = [ "read"]
}
```

在这里，我们使用HCL（Hashicorp的配置语言）语法来定义我们的策略。Vault还支持JSON格式，但我们将在示例中坚持使用HCL，因为它更易于阅读。

Vault中的策略是“默认拒绝”。附加到此示例策略的令牌将访问secret/accounting下存储的秘密，而不是其他内容。在创建时，令牌可以附加到多个策略。这非常有用，因为它允许我们创建和测试较小的策略，然后根据需要应用它们。

策略的另一个重要方面是他们使用懒惰评估。这意味着我们可以更新给定的策略，所有令牌都会立即受到影响。

到目前为止描述的策略也称为访问控制列表策略或ACL策略。Vault还支持两种其他策略类型：EGP和RGP策略。这些仅在付费版本中可用，并使用Sentinel支持扩展基本策略语法。

扩展基本策略允许我们在策略中考虑其他属性，例如一天中的时间，多个身份验证因素，客户端网络来源等。例如，我们可以定义一个允许仅在工作时间访问某指定机密的策略。

我们可以在Vault的文档中找到有关策略语法的更多详细信息 。

## 4.Secret类型

Vault支持一系列不同的秘密类型，可以解决不同的用例：

* Key-Value： 简单的静态键值对
* 动态生成的凭据：由Vault根据客户端请求生成
* 加密密钥：用于使用客户端数据执行加密功能

每种秘密类型由以下属性定义：

* 一个挂载点，定义了它的REST API前缀
* 通过相应的API公开的一组操作
* 一组配置参数

可以通过路径访问给定的秘密实例 ，非常类似于文件系统中的目录树。此路径的第一个组件对应于此类型的所有机密所在的安装点。

例如，字符串 secret/my-application 对应于我们可以在其中找到 my-application 的键值对的路径 。

### 4.1 Key-Value Secrets

顾名思义，键值Secrets是指在给定路径下可用的简单键值对。例如，我们可以在 path/secret/my-application 下 存储 foo=bar 对 。

稍后，我们使用相同的路径来检索相同的一对或多对 - 多对可以存储在同一路径下。

Vault支持三种Key-Value机密：

* 非版本化键值对，其中更新替换现有值
* 版本化键值对，可保持可配置数量的旧版本
* Cubbyhole，一种特殊类型的非版本化密钥对，其值的范围限定为给定的访问令牌（稍后将详细介绍）。

密钥值秘密本质上是静态的，因此没有到期的概念与它们相关联。这种秘密的主要使用场景是存储凭证以访问外部系统，例如API密钥。

在这种情况下，凭据更新是半手动过程，通常需要有人获取新凭据并使用Vault的命令行或其UI来输入新值。

### 4.2 动态生成的Secrets

当应用程序请求时，Vault会动态生成动态机密。Vault支持多种类型的动态机密，包括以下几种：

* 数据库凭证
* SSH密钥对
* X.509证书
* AWS凭证
* Google Cloud服务帐户
* Active Directory帐户

所有这些都遵循相同的使用模式。首先，我们使用连接到关联服务所需的详细信息配置secret引擎。然后，我们定义一个或多个角色， 描述实际的secret创建。

我们以数据库secret引擎为例。首先，我们必须使用所有用户数据库连接详细信息配置Vault，包括来自具有管理员权限的预先存在的用户的凭据以创建新用户。

然后，我们创建一个或多个角色（Vault角色，而不是数据库角色），其中包含用于创建新用户的实际SQL语句。这些通常不仅包括用户创建语句，还包括访问模式对象（表，视图等）以及所有必需的 grant 语句。

当客户端访问相应的API时，Vault将使用提供的语句在数据库中创建新的临时用户并返回其凭据。然后，客户端可以使用这些凭据在请求角色的生存时间属性定义的时间段内访问数据库。

凭证到达其到期时间后，Vault将自动撤消与此用户关联的任何权限。客户端还可以请求Vault续订这些凭据。只有在特定数据库驱动程序支持且相关策略允许的情况下，才会进行续订过程。

### 4.3 加密密钥

这个类型的秘密引擎处理加密，解密，签名等加密功能。所有这些操作都使用Vault内部生成和存储的加密密钥。除非明确告知，否则Vault将永远不会公开给定的加密密钥。

关联的API允许客户端发送Vault纯文本数据并接收其加密版本。相反的情况也是可能的：我们可以发送加密数据并取回原始文本。

目前，只有一种这种类型的引擎：Transit引擎。此引擎支持流行的密钥类型，如RSA和ECDSA，还支持 Convergent Encryption。使用此模式时，给定的明文值始终会产生相同的密文结果，这种属性在某些应用程序中非常有用。

例如，我们可以使用此模式加密事务日志表中的信用卡号。使用收敛加密，每次插入新事务时，加密的信用卡值都是相同的，因此允许使用常规SQL查询进行报告，搜索等。

## 5.Vault设置

在本节中，我们将创建一个本地测试环境，以便测试Vault的功能。

Vault的部署很简单：只需下载与我们的操作系统对应的软件包，并将其可执行文件（Windows上的vault 或vault.exe ）提取到PATH上的某个目录。此可执行文件包含服务器，也是标准客户端。

Vault支持一种development 模式，适用于某些快速测试并习惯其命令行工具，但对于实际使用情况来说太简单了：重启时所有数据都丢失，API访问使用普通HTTP。

相反，我们将使用基于文件的持久存储和设置HTTPS，以便我们可以探索一些可能导致问题的真实配置细节。

### 5.1 启动Vault Server

Vault使用HCL或JSON格式的配置文件。以下文件定义了使用文件存储和自签名证书启动服务器所需的所有配置：

```sh
// Enable UI
ui = true

// Filesystem storage
storage "file" {
  path    = "./vault-data"
}

// TCP Listener using a self-signed certificate
listener "tcp" {
  address     = "127.0.0.1:8200"
  tls_cert_file = "localhost.cert"
  tls_key_file = "localhost.key"
}
```

现在，让我们运行Vault。打开命令shell，转到包含我们的配置文件的目录并运行以下命令：

```sh
vault server -config ./vault-test.hcl
```

Vault将启动并显示一些初始化消息。它们将包括其版本，一些配置详细信息以及API可用的地址。就是这样我们的Vault服务器启动并运行。

### 5.2 Vault初始化

我们的Vault服务器现在正在运行，但由于这是它的第一次运行，我们需要初始化它。

让我们打开一个新的shell并执行以下命令来实现这个目的：

```sh
export VAULT_ADDR=https://localhost:8200
export VAULT_CACERT=localhost.cert
vault operator init
```

这里我们定义了一些环境变量，因此我们不必每次都将它们作为参数传递给Vault：

* VAULT_ADDR：我们的API服务器将为其提供请求的基URI
* VAULT_CACERT：服务器证书公钥的路径

在我们的例子中，我们使用VAULT_CACERT， 因此我们可以使用HTTPS访问Vault的API。我们需要这个，因为我们使用的是自签名证书。对于我们通常可以访问CA签名证书的制作环境而言，这不是必需的。

发出上述命令后，我们应该看到如下消息：

```sh
Unseal Key 1: kHcMDI6lwIfX1xdcThrDF5jR00gDOc1a7t6kr+fCXWcN
Unseal Key 2: 6xsSLornxUqfMUSoq7XPIb9PLIFnzPPMR+UMTzWP0m0o
Unseal Key 3: pgrhB6wtrnINh3SY4mS2cWjEzmuiMRWmg+SIp50/4DQs
Unseal Key 4: BpR1Lzuogn9AJYJADxXuCit5yxGYtM2qF0BxHn1g/Zw8
Unseal Key 5: Vj7W6HYcCZvtkCnjNdR+4fuw+aCmlfQ+IZMTc70cyzIh

Initial Root Token: 2HgD0nKyaDjI5TICz9pObW2T
```

前5行是我们稍后用于解开Vault存储的主密钥。 请注意，Vault仅在初始化期间显示解封密钥 - 以后永远不会出现。记下并安全地存储它们，否则在服务器重启时我们将无法访问我们的机密信息！

另外，请注意根令牌，因为我们稍后会需要它。与unseal键不同，root令牌可以在以后轻松生成，因此一旦完成所有配置任务就可以安全地销毁它。由于我们稍后将发出需要身份验证令牌的命令，因此我们现在将根令牌保存在环境变量中：

```sh
export VAULT_TOKEN= 2HgD0nKyaDjI5TICz9pObW2T
```

现在让我们看看我们的服务器状态，我们已经使用以下命令对其进行了初始化：

```sh
$ vault status
Key                Value
---                -----
Seal Type          shamir
Sealed             true
Total Shares       5
Threshold          3
Unseal Progress    0/3
Unseal Nonce       n/a
Version            0.11.3
HA Enabled         false
```

我们可以看到Vault仍然是密封的。Unseal Progress ：“0/3”意味着Vault需要三个，但到目前为止还没有开启。

### 5.3 Vault Unseal

我们现在开启Vault，以便我们可以开始使用其秘密服务。为了完成开封过程，我们需要提供五个关键股中的任意三个：

```sh
vault operator unseal kHcMDI6lwIfX1xdcThrDF5jR00gDOc1a7t6kr+fCXWcN
vault operator unseal 6xsSLornxUqfMUSoq7XPIb9PLIFnzPPMR+UMTzWP0m0o
vault operator unseal pgrhB6wtrnINh3SY4mS2cWjEzmuiMRWmg+SIp50/4DQs
```

发出每个命令后，Vault将打印启封进度，包括它需要多少个共享。发送最后一个密钥共享后，我们会看到如下消息：

```sh
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    5
Threshold       3
Version         0.11.3
Cluster Name    vault-cluster-6c90f68f
Cluster ID      a1259a6b-aa24-b02a-1d36-b64907ea3570
HA Enabled      false
```

在这种情况下，“密封”属性为“假”，这意味着Vault已准备好接受命令。

## 6.测试Vault

在本节中，我们将使用其支持的两种Secret类型测试我们的Vault设置：键值对和数据库。我们还将展示如何创建附加了特定策略的新令牌。

### 6.1 使用键/值Secret

首先，让我们存储秘密的键值对并将其读回。
假设用于初始化Vault的命令shell仍处于打开状态，我们使用以下命令将这些键值对存储在secret / fakebank路径下：

```sh
vault kv put secret/fakebank api_key=abc1234 api_secret=1a2b3c4d
```

我们现在可以使用以下命令随时读取这些键值对：

```sh
$ vault kv get secret/fakebank
======= Data =======
Key           Value
---           -----
api_key       abc1234
api_secret    1a2b3c4d
```

这个简单的测试向我们展示了Vault正在按预期工作。我们现在可以测试一些额外的功能。

### 6.2 创建新令牌

到目前为止，我们已经使用了根令牌来验证我们的请求。由于根令牌 的方式太强大了，它被认为是使用较少的特权和更短的时间生存令牌的最佳实践。

让我们创建一个新的令牌，我们可以像根令牌一样使用它，但在一分钟后过期：

```sh
$ vault token create -ttl 1m
Key                  Value
---                  -----
token                Luc1pGxZxKkNEwpCLD3Ea4VX
token_accessor       1Zzp7g1FJYW5nvqACZFQtRRx
token_duration       1m
token_renewable      true
token_policies       ["root"]
identity_policies    []
policies             ["root"]
```

让我们测试一下这个令牌，用它来读取我们之前创建的键/值对：

```sh
$ export VAULT_TOKEN=Luc1pGxZxKkNEwpCLD3Ea4VX
$ vault kv get secret/fakebank
======= Data =======
Key           Value
---           -----
api_key       abc1234
api_secret    1a2b3c4d
```

如果我们等待一分钟并尝试重新发出此命令，我们会收到一条错误消息：

```sh
$ vault kv get secret/fakebank
Error making API request.
URL: GET https://localhost:8200/v1/sys/internal/ui/mounts/secret/fakebank
Code: 403. Errors:
* permission denied
```

该消息表明我们的令牌不再有效，这正是我们所期望的。

### 6.3 测试策略

我们在上一节中创建的示例令牌虽是短暂存在，但仍然非常强大。现在让我们使用策略来创建更多受限制的令牌。

例如，让我们定义一个策略，该策略只允许对我们之前使用的 secret/fakebank 路径进行读访问：

```sh
$ cat> sample-policy.hcl <
path "secret/fakebank"{
capabilities = ["read"]
}
EOF
$ export VAULT_TOKEN=2HgD0nKyaDjI5TICz9pObW2T
$ vault policy write fakebank-ro ./sample-policy.hcl
Success! Uploaded policy: fakebank-ro
```

现在，我们使用以下命令使用此策略创建令牌：

```sh
$ vault token create -policy=fakebank-ro
Key                  Value
---                  -----
token                <token value>
token_accessor       <token accessor value>
token_duration       768h
token_renewable      true
token_policies       ["default""fakebank-ro"]
identity_policies    []
policies             ["default""fakebank-ro"]
```

正如我们之前所做的那样，让我们​​使用此标记读取我们的秘密值：

```sh
$ export VAULT_TOKEN=<token value>
$ vault kv get secret/fakebank
======= Data =======
Key           Value
---           -----
api_key       abc1234
api_secret    1a2b3c4d
```

到现在为止还挺好。我们可以按预期读取数据。让我们看看当我们尝试更新这个秘密时会发生什么：

```sh
$ vault kv put secret/fakebank api_key=foo api_secret=bar
Error writing data to secret/fakebank: Error making API request.
URL: PUT https://127.0.0.1:8200/v1/secret/fakebank
Code: 403. Errors:
* permission denied
```

由于我们的策略未授予允许写入，因此Vault将返回403 - 拒绝访问状态代码。

### 6.4 使用动态数据库凭据

作为本文的最后一个示例，让我们使用Vault的数据库秘密引擎来创建动态凭据。我们假设我们在本地有一个MySQL服务器，我们可以使用“root”权限访问它。我们还将使用一个非常简单的模式，该模式由一个表 - 帐户组成。

用于创建此架构的SQL脚本和特权用户可在此处获得。

```sh
--
-- Sample schema for testing vault database secrets
--
create schema fakebank;
use fakebank;
create table account( 
  id decimal(16,0), 
  name varchar(30),
  branch_id decimal(16,0),
  customer_id decimal(16,0),
  primary key (id));
  
--
-- MySQL user that will be used by Vault to create other users on demand
--
create user 'fakebank-admin'@'%' identified by 'Sup&rSecre7!';
grant all privileges on fakebank.* to 'fakebank-admin'@'%' with grant option;
grant create user on *.* to 'fakebank-admin' with grant option;

flush privileges;
```

现在，让我们配置Vault以使用此数据库。默认情况下未启用数据库密钥引擎，因此我们必须先解决此问题，然后才能继续：

```sh
$ vault secrets enable database
Success! Enabled the database secrets engine at: database/
```

我们现在创建一个数据库配置资源：

```sh
vault write database/config/mysql-fakebank plugin_name=mysql-legacy-database-plugin connection_url="{{username}}:{{password}}@tcp(127.0.0.1:3306)/fakebank" allowed_roles="*" username="fakebank-admin" password="Sup&rSecre7!"
```

路径前缀 database/config 是必须存储所有数据库配置的地方。我们选择名称 mysql-fakebank， 以便我们可以轻松找出此配置所指的数据库。至于配置键：

* plugin_name：定义将使用哪个数据库插件。Vault的文档中描述了可用的插件名称
* connection_url：这是插件在连接数据库时使用的模板。请注意{{username}}和{{password}}模板占位符。连接到数据库时，Vault将按实际值替换这些占位符
* allowed_roles：定义哪些Vault角色（上面讨论过）可以使用此配置。在我们的例子中，我们使用“*”，因此它可用于所有角色
用户名和密码：这是Vault用于执行数据库操作的帐户，例如创建新用户和撤消其权限

**Vault数据库角色设置:**

最终配置任务是创建包含创建用户所需的SQL命令的Vault数据库角色资源。根据我们的安全要求，我们可以根据需要创建任意数量的角色。

在这里，我们创建一个角色，授予对fakebank模式的所有表的只读访问权限：

```sh
vault write database/roles/fakebank-accounts-ro db_name=mysql-fakebank creation_statements="CREATE USER '{{name}}'@'%' IDENTIFIED BY '{{password}}';GRANT SELECT ON fakebank.* TO '{{name}}'@'%';"
```

数据库引擎将路径前缀数据库/角色 定义为存储角色的位置。 fakebank-accounts-ro是我们稍后在创建动态凭据时使用的角色名称。我们还提供以下属性：

* db_name：现有数据库配置的名称。对应于我们在创建配置资源时使用的路径的最后部分
* creation_statements： Vault将用于创建新用户的SQL语句模板列表

**创建动态凭据:**

一旦我们准备好数据库角色及其相应的配置，我们就会使用以下命令生成新的动态凭据：

```sh
$ vault read database/creds/fakebank-accounts-ro
Key                Value
---                -----
lease_id           database/creds/fakebank-accounts-ro/t4RQeXdyAVq75jNmbaQfPM6n
lease_duration     768h
lease_renewable    true
password           A1a-24NNCeaazWCp2EE7
username           v-fake-60fSchsFo       
```

该数据库 /creds 前缀用于生成可用角色凭据。由于我们使用了 fakebank-accounts-ro 角色，因此返回的用户名/密码将仅限于选择操作。有效期为1个小时。

我们可以通过使用提供的凭据连接到数据库然后执行一些SQL命令来验证这一点：

```sh
$ mysql -h 127.0.0.1 -u v-fake-60fSchsFo -p fakebank
Enter password:
mysql> select * from account;
Empty set (0.00 sec)

mysql> delete from account;
ERROR 1142 (42000): DELETE command denied to user 'v-fake-60fSchsFo'@'localhost' for table 'account'
```

我们可以看到第一个select成功完成，但是我们无法执行delete语句。最后，如果我们等待一个小时并尝试使用相同的凭据进行连接，我们将无法再连接到数据库。Vault已自动撤消此用户的所有权限。

转发：

[Hashicorp Vault介绍和使用说明](https://blog.csdn.net/peterwanghao/article/details/83181932)