# IOA身份标识协议规范

Version 1.0.0

## 引言

IOA（Internet of Agents，智能体互联网）是基于区块链的 **W3C DID 方法 `did:ioa`**，为智能体互联网提供去中心化、权利下放且注重数据安全与隐私保护的数字身份服务。智能体、注册节点与组织等主体可通过 `did:ioa` 在链上登记 DID 文档，表达身份、能力与可达性，支撑跨节点的注册、发现与互联。

DID 文档侧重承载「主体是谁、具备何种能力、如何连接」等可解析信息（链上注册、发现与互联，见 §4）；可验证凭证（VC）由签发方按 [W3C VC 数据模型](https://www.w3.org/TR/vc-data-model/) 对主体属性（如注册状态、能力范围、所属单位等）作出可验证断言，供验签与策略判断。二者相互补充：`credentialSubject.id` 通常为主体 `did:ioa`，VC 根级 `id` 可为独立 URN，不必与 DID 路径相同；字段不必与 DID 文档一一镜像。

§4 以 [W3C DID Core](https://www.w3.org/TR/did-core/) 字段为文档主体；IOA 智能体与链上业务扩展置于 `extension`。链上注册 API 见 §5。有关 DID 与方法规范的背景，请参阅 [DID 入门](https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-fall2017/blob/master/topics-and-advance-readings/did-primer.md) 与 [DID 规范](https://www.w3.org/TR/did-core/)。

## 设计目标

| 目标 | 说明 | 规范落点 |
|------|------|----------|
| **注册（Registration）** | 智能体、组织或注册节点获得唯一、可验证的 `did:ioa`，并在链上登记 DID 文档。 | §3 标识符、§5.1 创建 |
| **发现（Discovery）** | 通过解析获取文档中的能力描述与服务端点，支持检索与匹配。 | §2.1、§5.2 Read、`service`（含子链解析） |
| **互联（Interconnection）** | 基于 `serviceEndpoint` 与协议能力（如 A2A）建立跨节点可信连接。 | §4.1.2 `AgentDescription` |
| **治理与安全（Governance & Security）** | 密钥控制、恢复、停用与隐私最小化。 | §5.3–5.5、§6 |

## 文档状态

本文档为 IOA 协议规范的 v1.0.0 版本。最新版本见本仓库 [中文版](IOA身份标识协议规范.md) 。

## 1. IOA 命名空间

- 标识此 `DID` 方法的 method name 是：`ioa`
- 使用此方法的 `DID` **必须** 以下前缀开头：`did:ioa`。此字符串 **必须** 为小写。在前缀之后的 DID 的剩余部分由特定算法生成。

## 2. 适用系统

IOA 方法适用于区块链网络，从该网络发布开始正式使用。

### 2.1 DID 解析

本方法的**标准解析**为 §5.2 **Read**：向链上注册节点发送 `operation: "read"` 请求，成功时从 `data.didDocument` 取得符合 §4 的 DID 文档（不含请求侧 `proof`）。

含 **acsn** 的 DID（如 `did:ioa:tele`）须先 Read 主链文档，从 `service` 中 `type` 为 `DIDSubResolver` 的项获取子链解析地址，再按该服务约定查询目标 DID；子链 Read 的请求/响应形态与 §5.2 相同，由子链注册节点实现。

## 3. IOA 标识符

### 3.1 IOA

- IOA 的组成结构如下：

<img src="image/ioa.png" style="zoom: 67%;" />

- **智能体及一般主体**（默认）：`did:ioa:` + **suffix**（22–42 个字母或数字，由公钥编码得到，见下文生成步骤）。例如 `did:ioa:sfHBuFa7asqWztQJ3nZUiHtXjSJsyqEp` 表示智能体标识。
- **可选 acsn**：4 个字母或数字的前缀段（见下文 ABNF），用于特殊类型。例如 `did:ioa:tele` 仅含 acsn、无 suffix，用于子链解析服务，对应 DID 文档内存放子链解析地址。

- IOA 生成方案由以下 ABNF 定义：

```plaintext
ioa-did = "did:ioa:" ioa-specific-identifier ; 固定的did:ioa前缀
ioa-specific-identifier = 0*1(acsn ":") suffix / acsn ":" 0*1(suffix)
acsn(可选):后缀 或者 acsn:后缀(可选)
acsn = 4(ALPHA / DIGIT); 4个字母或数字组合
suffix = (22,42)(ALPHA / DIGIT); 长度范围22-42的字母或数字组合
```

- 生成 IOA 地址的步骤如下定义：

1. 选择加密算法（如 SM2、ED25519 或 Secp256k1）。

2. 生成公私钥对。

3. 使用 Base58、Base64 或 Base32 编码方式对公钥进行编码。

4. 拼接前缀 `did:ioa:` 和编码后的公钥字符串，形成完整的 IOA（生成流程示意图见 `image/generateIOA.png`，资源待补充）。

<img src="image/generateIOA.png" style="zoom: 50%;" />

加密方法：

| 公私钥支持算法   | 加密类型 |
| --------- | ---- |
| SM2       | 'z'  |
| ED25519   | 'e'  |
| Secp256k1 | 's'  |

编码方法：

| 编码方式 | 编码类型 |
| -------- | -------- |
| Base58   | 'f'      |
| Base64   | 's'      |
| Base32   | 't'      |

## 4. IOA 文档规范

IOA DID 文档遵循 [DID Core](https://www.w3.org/TR/did-core/)：互操作字段位于文档顶层；IOA 方法扩展统一置于 `extension` 对象。链上 Create/Update 等请求可在 `didDocument` 外携带 `proof`（见 §5），**不**作为 §4.3 标准解析结果的一部分。

### 4.1 字段说明

#### 4.1.1 DID Core

- `@context`：**必填**。JSON-LD 上下文数组，**必须**包含 `https://www.w3.org/ns/did/v1`。使用 secp256k1 验证密钥时 **应** 增加 `https://w3id.org/security/suites/secp256k1-2019/v1`。
- `id`：**必填**。本文档对应的 `did:ioa` 标识符。
- `verificationMethod`：**必填**（`did:ioa:tele` 等系统解析 DID **MAY** 为空数组，见 §4.3）。验证方法数组，每项包含：
  - `id`：验证方法 ID（**必须**为 `{did}#{fragment}`）。
  - `type`：验证密钥类型，如 `EcdsaSecp256k1VerificationKey2019`（secp256k1）、或方法实现支持的其它 W3C 安全套件类型。
  - `controller`：控制该密钥的 DID，通常与文档 `id` 相同。
  - 密钥材料：如 `publicKeyCompressed`（secp256k1 压缩公钥，十六进制）等，依 `type` 与套件上下文而定。
- `authentication`：**必填**。`verificationMethod` 的 `id` 列表（含 `#fragment`），用于文档控制与链上操作授权。
- `version`：推荐。文档版本号字符串（如 `1.0.0`）。
- `service`：选填。服务端点数组，见 §4.1.2。
- `alsoKnownAs`：选填。关联标识数组，每项含 `type`、`id`。
- `created` / `updated`：选填。ISO-8601 时间戳。

#### 4.1.2 `service`

**类型 A：`AgentDescription`（智能体注册与发现）**

- `id`、`type`（固定为 `AgentDescription`）
- `name`、`description`：智能体名称与描述
- `capabilities`：
  - `tags`：**字符串数组**；每项为逗号分隔的一个或多个能力/任务 code（编码体系与 `extension.taskType` / 平台字典一致，见 §4.1.4）。
  - `skills`：技能名称字符串数组。
  - `protocol`：互联协议标识数组（如 `A2A`）。
- `serviceEndpoint`：**字符串数组**，可达 URI 列表（如 A2A 端点）

**类型 B：`DIDSubResolver`（子链解析）**

用于 `did:ioa:tele` 等场景：`id`、`type`（`DIDSubResolver`）、`version`、`serverType`、`protocol`、`serviceEndpoint`（字符串）、`port`。`serverType`、`protocol` 取值见 §4.1.4。

#### 4.1.3 `extension`（IOA 方法扩展）

| 字段 | 说明 | 是否可选 |
|------|------|----------|
| `taskType` | 智能体任务类型 code（逗号分隔） | 智能体文档推荐 |
| `applicationDomain` | 应用领域 code | 智能体文档推荐 |
| `companyDid` / `companyName` | 所属单位 DID 与名称 | 可选 |
| `registrationNodeDid` / `registrationNodeName` | 注册节点 DID 与名称 | 可选 |
| `recovery` | 恢复用 `verificationMethod` id 列表 | 可选 |
| `ttl` | 解析缓存时间（秒） | 链上注册时推荐 |
| `attributes` | 属性数组（`key`、`desc`、`encrypt`、`format`、`value`） | 可选 |
| `acsns` | 子链 AC 号列表 | 可选 |
| `verifiableCredentials` | 凭证引用列表（`id`、`type`） | 可选 |
| `delegateSign` | 委托签名（`signer`、`signatureValue`） | 可选 |
| `type` | 文档属性类型（整型） | 可选 |

#### 4.1.4 编码与枚举

**`DIDSubResolver.serverType`（解析地址类型）**

| 值 | 含义 |
|----|------|
| 1 | URL 地址（HTTP/HTTPS，与 `serviceEndpoint` 配合使用） |

**`DIDSubResolver.protocol`（传输协议）**

| 值 | 含义 |
|----|------|
| 1 | TCP |
| 2 | WebSocket |
| 3 | HTTP/HTTPS（与 §5 JSON POST API 一致） |

实现 **MAY** 扩展上表未列取值；客户端 **SHOULD** 忽略未知值。

**`extension.taskType` / `extension.applicationDomain`**

平台字典 code，在 `extension` 中以**逗号分隔字符串**表示（如 `"0503,0501"`）。具体 code 表由注册节点或平台字典维护，本规范不展开枚举。

**`extension.type`（文档属性类型，可选）**

| 值 | 含义 |
|----|------|
| 206 | 智能体 DID 文档（实现常用） |

**`alsoKnownAs.type`**

关联标识分类整型；语义由实现或业务字典定义。

**`extension.verifiableCredentials[].type`（凭证引用类型）**

| 值   | 含义                                                                                        |
| --- | ----------------------------------------------------------------------------------------- |
| 1   | 链上可验证声明引用（遗留实现）                                                                           |
| 2   | 外部 W3C VC 引用（`id` 为 VC 标识 URI/URN；由签发方按 VC 数据模型签发，`credentialSubject.id` 通常为主体 `did:ioa`） |

**`extension.delegateSign`（委托签名，可选）**

用于可信解析等场景：委托方对文档 `verificationMethod` 数组内容的完整性背书。

| 字段 | 说明 |
|------|------|
| `signer` | 签名方 verification method `id` |
| `signatureValue` | 对 `verificationMethod` 数组做 §5.0.2 同款待签序列化后，由 `signer` 对应私钥签名的 Base58 值 |

解析方 **MAY** 在缓存前校验 `delegateSign`；未携带时按常规 DID 解析流程处理。

### 4.2 结构体定义

**DIDDocument**

| 字段名称 | 类型 | 描述 | 是否可选 |
|----------|------|------|----------|
| Context | []string | JSON-LD `@context` | 必填 |
| ID | string | `did:ioa` 标识符 | 必填 |
| VerificationMethod | []VerificationMethod | 验证方法 | 必填 |
| Authentication | []string | verification method id 列表 | 必填 |
| Version | string | 文档版本 | 推荐 |
| Service | []Service | 服务数组 | 可选 |
| AlsoKnownAs | []AlsoKnownAs | 关联标识 | 可选 |
| Extension | Extension | IOA 扩展 | 推荐 |
| Created | string | 创建时间 | 可选 |
| Updated | string | 更新时间 | 可选 |

**VerificationMethod**

| 字段名称 | 类型 | 描述 | 是否可选 |
|----------|------|------|----------|
| ID | string | 验证方法 ID（含 fragment） | 必填 |
| Type | string | 如 `EcdsaSecp256k1VerificationKey2019` | 必填 |
| Controller | string | 控制方 DID | 必填 |
| PublicKeyCompressed | string | secp256k1 压缩公钥（hex） | 依 type |

**Extension** — 字段同 §4.1.3。

**AgentDescription（service）**

| 字段名称 | 类型 | 描述 | 是否可选 |
|----------|------|------|----------|
| ID | string | 服务 ID | 必填 |
| Type | string | `AgentDescription` | 必填 |
| Name | string | 名称 | 必填 |
| Description | string | 描述 | 推荐 |
| Capabilities | object | `tags`、`skills`、`protocol` | 推荐 |
| ServiceEndpoint | []string | URI 列表 | 必填 |

**DIDSubResolver（service）** — 字段见 §4.1.2 类型 B。

**Proof**（仅 §5 链上请求，非解析文档必选）

| 字段名称 | 类型 | 描述 |
|----------|------|------|
| Creator | string | verification method id |
| SignatureValue | string | 签名值 |

### 4.3 文档示例

**智能体 DID 文档**：

```json
{
    "@context": [
        "https://www.w3.org/ns/did/v1",
        "https://w3id.org/security/suites/secp256k1-2019/v1"
    ],
    "id": "did:ioa:sfHBuFa7asqWztQJ3nZUiHtXjSJsyqEp",
    "verificationMethod": [{
        "id": "did:ioa:sfHBuFa7asqWztQJ3nZUiHtXjSJsyqEp#secp256k1-6cda3f6e",
        "type": "EcdsaSecp256k1VerificationKey2019",
        "controller": "did:ioa:sfHBuFa7asqWztQJ3nZUiHtXjSJsyqEp",
        "publicKeyCompressed": "03bc5e9cea08abaa5d2062847064535ccab5a460ce70d32de403bc62caa5eb418e"
    }, {
        "id": "did:ioa:sfHBuFa7asqWztQJ3nZUiHtXjSJsyqEp#secp256k1-recovery01",
        "type": "EcdsaSecp256k1VerificationKey2019",
        "controller": "did:ioa:sfHBuFa7asqWztQJ3nZUiHtXjSJsyqEp",
        "publicKeyCompressed": "02aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
    }],
    "authentication": [
        "did:ioa:sfHBuFa7asqWztQJ3nZUiHtXjSJsyqEp#secp256k1-6cda3f6e"
    ],
    "version": "1.0.0",
    "service": [{
        "id": "did:ioa:sfHBuFa7asqWztQJ3nZUiHtXjSJsyqEp#agent-description",
        "type": "AgentDescription",
        "name": "法律风险评估智能体",
        "description": "专业的法律风险评估智能体，对合同、协议、法律文档等进行全面的法律风险评估，识别潜在的法律风险点，并提供专业的风险评估报告和风险规避建议。",
        "capabilities": {
            "tags": ["0503,0501", "0701,0702,0703,0704"],
            "skills": ["法律风险评估"],
            "protocol": ["A2A"]
        },
        "serviceEndpoint": ["http://127.0.0.1:9090"]
    }],
    "extension": {
        "taskType": "0503,0501",
        "applicationDomain": "0701,0702,0703,0704",
        "companyDid": "did:ioa:sfuCqVuDmycvFXb3pkTZ6fFW86sKDtn1",
        "companyName": "测试单位",
        "registrationNodeDid": "did:ioa:sfJmAbKzPNdYF9WNZ2Jv5rK1qn8VaHpm",
        "registrationNodeName": "智能体互联网注册节点",
        "recovery": [
            "did:ioa:sfHBuFa7asqWztQJ3nZUiHtXjSJsyqEp#secp256k1-recovery01"
        ]
    }
}
```

上例为解析/展示用金样例。链上 **Create** 时 `extension` **应**另含 `ttl`（§5.1），智能体文档 **宜**含 `extension.type`: `206`（§4.1.4）。

**子链解析服务示例**（`service.type` 为 `DIDSubResolver`）：

```json
{
    "@context": ["https://www.w3.org/ns/did/v1"],
    "id": "did:ioa:tele",
    "verificationMethod": [],
    "authentication": [],
    "version": "1.0.0",
    "service": [{
        "id": "did:ioa:tele#subResolve",
        "type": "DIDSubResolver",
        "version": "1.0.0",
        "serverType": 1,
        "protocol": 3,
        "serviceEndpoint": "https://resolver.ioa.org",
        "port": 8080
    }],
    "extension": {
        "ttl": 86400
    }
}
```

## 5. IOA 方法

§5 定义链上注册节点的 HTTP API。标准 DID 解析见 §2.1（即 §5.2 Read）。

### 5.0 通用约定

#### 5.0.1 传输层

所有 §5 操作经 **同一 HTTP 端点** 调用：

- **方法**：`POST`
- **`Content-Type`**：`application/json; charset=utf-8`
- **请求体**：JSON 对象，**必须**含 `id`、`operation`；Create/Update/Recovery **必须**含 `didDocument`；需授权的操作 **必须**含顶层 `proof`（不在 `didDocument` 内）
- **响应体**：JSON 对象，**应**含 `errorCode`、`message`；Read 成功时另含 `data.didDocument`（见 §5.6）

**示范**（注册节点基址由部署方提供）：

```http
POST /api/v1/ioa HTTP/1.1
Host: registry.example.ioa
Content-Type: application/json; charset=utf-8

{"id":"did:ioa:sfHBuFa7asqWztQJ3nZUiHtXjSJsyqEp","operation":"read"}
```

#### 5.0.2 链上请求 `proof` 与签名

[DID Core](https://www.w3.org/TR/did-core/) 将可验证证明用于校验控制关系；解析得到的 DID 文档（§4.3）**不包含**嵌入 `proof`（与 DID Core 1.0 一致）。链上 Create/Update/Deactivate/Recovery 在请求顶层携带 IOA 方法专用的简化 `proof`：

| 字段 | 说明 |
|------|------|
| `creator` | verification method `id`（含 `#fragment`） |
| `signatureValue` | 签名字符串 |

**权限**（与 DID Core `authentication` 关系一致）：

| 操作 | `proof.creator` **必须**属于 |
|------|------------------------------|
| Create / Update | 请求体 `didDocument.authentication` 所列 id |
| Deactivate | 链上**已存** DID 文档的 `extension.recovery` 所列 id（Deactivate 请求不含 `didDocument`） |
| Recovery | 请求体 `didDocument.extension.recovery` 所列 id |

对 Deactivate / Recovery，注册节点 **SHOULD** 以**执行操作前**链上已存文档判定 `proof.creator` 是否属于 `recovery` 列表。

**待签载荷**：自请求 JSON 中**移除** `proof` 键后，对其余对象做 UTF-8 **紧凑 JSON** 序列化（对象键按 Unicode 码点升序、无多余空白；数组与嵌套对象递归应用相同规则）。

**算法与编码**（与 `creator` 所指 `verificationMethod` 一致）：

- 类型为 `EcdsaSecp256k1VerificationKey2019` 时：对载荷做 **SHA-256**，使用 secp256k1 私钥 **ECDSA** 签名；`signatureValue` 为签名的 **Base58** 编码（无校验和）。验签使用文档中对应 `publicKeyCompressed`（十六进制）。
- 其它套件类型由实现定义，**SHOULD** 在部署文档中说明。

注册节点 **MUST** 验签通过且 `creator` 具备上表权限后，方可执行对应操作。

**与实现对齐**：上文序列化与编码为互操作**推荐**规则；若链上注册节点采用不同 JSON 规范化或签名编码，**MUST** 在部署文档中说明；客户端 **SHOULD** 以目标注册节点文档为准生成 `proof`。

### 5.1 创建（Create）

注册接口主要完成 IOA 文档的注册（传输与 `proof` 见 §5.0）。请求体中的 `didDocument` **应符合 §4**；顶层 `id` **必须**与 `didDocument.id` 相同。若文档已存在则不允许重复创建。

#### 请求参数

| 参数  | 字段类型   | 描述                     |
| ----------- | ------ | ------------------------------- |
| id          | String | 要创建的IOA         |
| operation   | String | "create"                        |
| proof       | Object | 签名，见 §5.0.2 |
| didDocument | Object | 要创建的IOA文档 |

#### 请求示例

`didDocument` 结构与 §4.3 智能体示例相同；链上创建时 `extension` **应**含 `ttl`，智能体文档 **宜**含 `extension.type`: `206`（§4.1.4）；`proof.creator` 为 `authentication` 中的主控密钥。

```json
{
    "id": "did:ioa:sfHBuFa7asqWztQJ3nZUiHtXjSJsyqEp",
    "operation": "create",
    "proof": {
        "creator": "did:ioa:sfHBuFa7asqWztQJ3nZUiHtXjSJsyqEp#secp256k1-6cda3f6e",
        "signatureValue": "2ExbMiHRpGSWxyZAwGgD4YnWUyXxhsp4F8mwGzE43VNrS3p3kru8JroVuox8AyXpyZrPhoepAUVtLwn3HyKnoXFcx1n"
    },
    "didDocument": { "...": "同 §4.3 智能体 DID 文档；extension 增加 ttl: 86400、可选 type: 206" }
}
```

#### 返回示例

```json
{
    "errorCode": 0,
    "message": "success"
}
```

### 5.2 读取（Read）

通过 IOA 查询对应的 DID 文档（**标准解析**，见 §2.1）。返回值为包含 `errorCode` 与 `data.didDocument` 的 JSON 对象。

#### 请求参数

| 参数      | 字段类型 | 描述          |
| --------- | -------- | ------------- |
| id        | String   | 要读取的 IOA |
| operation | String   | 固定为 `"read"` |

#### 请求示例

```json
{
    "id": "did:ioa:sfHBuFa7asqWztQJ3nZUiHtXjSJsyqEp",
    "operation": "read"
}
```

#### 返回数据

| 参数 | 字段类型 | 描述 |
|------|----------|------|
| errorCode | Int | 见 §5.6 |
| message | String | 可读说明（推荐） |
| data.didDocument | Object | 解析结果；字段定义见 §4.1（不含嵌入 `proof`） |

`extension.attributes` 数组元素结构如下：

| 参数                                   | 字段类型   | 描述                     |
| --------------------------------------------- | ------ | ------------------------------------- |
| data.didDocument.extension.attributes.key     | String | 属性的key                     |
| data.didDocument.extension.attributes.desc    | String | 属性的描述                           |
| data.didDocument.extension.attributes.encrypt | Int    | 是否加密，0非加密，1加密   |
| data.didDocument.extension.attributes.format  | String | image、text、video、mixture等数据类型 |
| data.didDocument.extension.attributes.value   | String | 属性自定义value  |

当 `service.type` 为 `AgentDescription` 或 `DIDSubResolver` 时，结构见 §4.1.2。

#### 返回示例

- 成功返回智能体文档：`data.didDocument` 结构同 §4.3 智能体示例（**不含** `proof`）。

```json
{
    "errorCode": 0,
    "message": "success",
    "data": {
        "didDocument": { "...": "同 §4.3 智能体 DID 文档" }
    }
}
```

- 成功返回子链解析服务文档：`data.didDocument` 结构同 §4.3 `DIDSubResolver` 示例。

```json
{
    "errorCode": 0,
    "message": "success",
    "data": {
        "didDocument": { "...": "同 §4.3 子链解析服务示例" }
    }
}
```

### 5.3 更新（Update）

更新接口主要完成 IOA 文档的更新（传输与 `proof` 见 §5.0）。**不得**修改 `authentication`；`proof.creator` 须为 `authentication` 中的 verification method `id`。

| 参数   | 字段类型   | 描述                    |
| ----------- | ------ | ------------------------------- |
| id          | String | 要更新的IOA     |
| operation   | String | "update"                        |
| proof       | Object | 签名，见 §5.0.2 |
| didDocument | Object | 更新后的IOA文档 |

#### 请求示例

`didDocument` 基于 §4.3 智能体示例；可更新 `version`、`service`、`extension` 等字段，`authentication` 保持不变。

```json
{
    "id": "did:ioa:sfHBuFa7asqWztQJ3nZUiHtXjSJsyqEp",
    "operation": "update",
    "proof": {
        "creator": "did:ioa:sfHBuFa7asqWztQJ3nZUiHtXjSJsyqEp#secp256k1-6cda3f6e",
        "signatureValue": "hLDyzcMbSpaV74RRqaN7bBqbN43Zm8FfqQdUmmGkZitVRiV5sHYv2JEkyxCtNLtw1iMJzSaUtKwgUrjop5JCGdCPwN"
    },
    "didDocument": { "...": "同 §4.3 智能体 DID 文档；version、service、extension 等可更新" }
}
```

#### 返回示例

```json
{
    "errorCode": 0,
    "message": "success"
}
```

### 5.4 停用（Deactivate）

停用接口撤销 IOA 文档（传输见 §5.0）。停用后链上记录更新为**停用文档**（见下文），不删除 DID 条目。`proof.creator` 须为链上**已存**文档的 `extension.recovery` 所列 verification method `id`；`operation` 为 `"deactivate"`。

| 参数 | 字段类型 | 描述 |
|------|----------|------|
| id | String | 要停用的 IOA |
| operation | String | `"deactivate"` |
| proof | Object | 签名，见 §5.0.2 |

**停用后链上文档形态**（Read 对已停用 DID 返回 `errorCode: 1`，见 §5.6）：

```json
{
    "@context": ["https://www.w3.org/ns/did/v1"],
    "id": "did:ioa:sfHBuFa7asqWztQJ3nZUiHtXjSJsyqEp",
    "verificationMethod": [],
    "authentication": []
}
```

仅保留 `id` 与空的验证关系；**不得**含 `service`、`extension` 等业务字段。已停用的 DID **不得**再执行 §5.5 Recovery。

#### 请求示例

```json
{
    "id": "did:ioa:sfHBuFa7asqWztQJ3nZUiHtXjSJsyqEp",
    "operation": "deactivate",
    "proof": {
        "creator": "did:ioa:sfHBuFa7asqWztQJ3nZUiHtXjSJsyqEp#secp256k1-recovery01",
        "signatureValue": "w9uH8kx5kyo2vdsWz8aELJZhsdYPokf9Rnh67Yra5Lo49KHteAGmF7hzXiVmJVbXR7jMkDmj1zZuWqvfiKehenirUg"
    }
}
```

#### 返回示例

```json
{
    "errorCode": 0,
    "message": "success"
}
```

### 5.5 恢复（Recovery）

恢复接口用于主控私钥丢失等场景，更新 `authentication` 与 `verificationMethod`（传输与 `proof` 见 §5.0）。**仅适用于未执行 §5.4 停用**、且链上仍存在含 `extension.recovery` 的活跃文档。`proof.creator` 须为请求体 `didDocument.extension.recovery` 所列 verification method `id`。

**不得**对已停用 DID 使用 Recovery：停用后链上无 `extension.recovery`，Read 对已停用标识返回 `errorCode: 1`（§5.6）。若业务需重新启用该 `did:ioa`，由注册节点实现或治理流程另行定义，超出本规范 §5.5。

| 参数       | 字段类型   | 描述               |
| ---------------- | ------ | --------------------------- |
| id               | String | 要恢复的IOA  |
| operation        | String | "recovery"                  |
| proof            | Object | 签名，见 §5.0.2 |
| didDocument | Object | 更新后的 did 文档 |

#### 请求示例

`didDocument` 替换 `authentication` 与 `verificationMethod` 为新主控密钥；`extension.recovery` 宜保持不变（见 §4.3）。

```json
{
    "id": "did:ioa:sfHBuFa7asqWztQJ3nZUiHtXjSJsyqEp",
    "operation": "recovery",
    "proof": {
        "creator": "did:ioa:sfHBuFa7asqWztQJ3nZUiHtXjSJsyqEp#secp256k1-recovery01",
        "signatureValue": "2ESSXREizoYDfkp7t5MnmqkaAtAykBi45ri2mkc3Tr1XHd1JQAnaMrxpnodqSFADw2SacNrHNmjf6KQb7BYBgyJbaKi"
    },
    "didDocument": { "...": "新 authentication/verificationMethod；extension.recovery 同 §4.3" }
}
```

#### 返回示例

```json
{
    "errorCode": 0,
    "message": "success"
}
```

### 5.6 响应码说明

所有 §5 接口响应 **SHOULD** 包含 `errorCode`（整型）与 `message`（字符串）。Read 成功时另含 `data.didDocument`。`errorCode` 为 0 表示成功；非 0 表示失败，**SHOULD** 在 `message` 中给出可读说明。

| errorCode | 说明 |
|-----------|------|
| 0 | 成功 |
| 1 | 文档不存在或已停用 |
| 2 | 文档已存在（Create 重复） |
| 3 | 签名无效或 `proof.creator` 无权限 |
| 4 | 请求参数无效 |
| 5 | 操作不允许（如 Update 修改 `authentication`） |
| 其他 | 实现自定义；**MUST** 在 `message` 中说明 |

#### 失败响应示例

```json
{
    "errorCode": 1,
    "message": "DID 文档不存在或已停用"
}
```

```json
{
    "errorCode": 3,
    "message": "签名无效或 proof.creator 无权限"
}
```

```json
{
    "errorCode": 5,
    "message": "更新不得修改 authentication"
}
```

---

以上是完整的 **IOA 方法** 部分，包括创建、读取、更新、停用、恢复操作及响应码说明。

## 6. 安全和隐私考虑

### 6.1 安全考虑

- **验证密钥完整性**：IOA 的 `verificationMethod` 存储在区块链上，确保其不可篡改。
- **私钥保护**：私钥必须安全存储，避免泄露给未经授权的第三方。
- **链上请求签名**：注册节点 **MUST** 按 §5.0.2 校验 `proof`；验签失败或 `proof.creator` 无对应操作权限时 **MUST** 拒绝请求。
- **恢复密钥权限**：`extension.recovery` 所列 verification method **仅**用于 Deactivate/Recovery（§5.4、§5.5），**不得**用于 Create/Update；**不得**对已停用 DID 使用 §5.5；恢复主控密钥后宜评估是否轮换 recovery 密钥。
- **防止中间人攻击**：通过加密传输（如 TLS）保护 IOA 文档的传输过程。
- **身份恢复机制**：主控私钥丢失且文档**未停用**时，通过 §5.5 Recovery 与 recovery 密钥重新设置 `authentication`（须事先在活跃文档中登记 `extension.recovery`）。

### 6.2 隐私考虑

- **数据最小化**：仅在区块链上存储必要的信息，如验证密钥与服务端点，避免存储敏感信息。
- **隐私保护**：使用加密技术（如同态加密）保护敏感数据，避免隐私泄露。
- **访问控制**：通过加密方法控制对 IOA 及其关联服务的访问权限。
- **匿名化技术**：在必要时使用匿名化技术（如零知识证明）保护用户隐私。
