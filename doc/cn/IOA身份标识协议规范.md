# IOA身份标识协议规范

Version 1.0.0

## 关于

IOA(Internet of Agents，简称IOA)是基于区块链技术构建的去中心化身份标识系统，旨在为用户提供安全、可信、隐私保护的身份标识服务。IOA 是一种分布式身份标识符（DID），适用于区块链网络，支持个人、企业、设备和数字对象的身份标识与认证。有关DID和DID方法规范的更多信息，请参阅[DID入门](https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-fall2017/blob/master/topics-and-advance-readings/did-primer.md)和[DID规范](https://www.w3.org/TR/did-core/)。

## 摘要

IOA 为个人、企业、设备和数字对象等提供基于区块链技术数字身份服务，旨在构建一套去中心化的、权利下放的、数据安全、隐私得到保护的标识符系统。通过 IOA，可以实现可信连接、交互和互操作，推动数字经济的发展。

## 文档状态

本文档为 IOA 协议规范的 v1.0.0版本。文档的最新版本可在 [IOA 官方文档仓库](https://github.com/teleinfo-bif/ioa/blob/main/doc/en/IOA%20Protocol%20Specification.md) 获取。

## 1. IOA 命名空间

- 标识此 `DID` 方法的 method name 是：`ioa`
- 使用此方法的` DID` **必须** 以下前缀开头：`did:ioa`。此字符串 **必须** 为小写。在前缀之后的 DID 的剩余部分由特定算法生成。

## 2. 适用系统

IOA 方法适用于区块链网络，从该网络发布开始正式使用。

## 3. IOA 标识符

### 3.1 IOA

- IOA 的组成结构如下：

<img src="image\ioa.png" alt="../" style="zoom:67%;" />

- `did:ioa:tele`(ascn号) 这样的 `IOA` 是一类特殊的IOA, 存放子链解析服务，只有前三个部分，不包含后缀。 对应的 `IOA` 文档里存放子链解析地址。

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

4. 拼接前缀 `did:ioa:` 和编码后的公钥字符串，形成完整的 IOA。

   <img src="image\generateIOA.png" style="zoom: 50%;" />

加密方法：

| 公私钥支持算法 | 加密类型 |
| -------------- | -------- |
| SM2            | 'z'      |
| ED25519        | 'e'      |
| Secp256k1      | 's'      |

编码方法：

| 编码方式 | 编码类型 |
| -------- | -------- |
| Base58   | 'f'      |
| Base64   | 's'      |
| Base32   | 't'      |

## 4. IOA 文档规范

### 4.1 IOA 规范说明
IOA 文档遵循 DID Document 规范，并在之基础上做了一定的扩展。IOA 文档字段说明如下：

- `context`：必填字段。一组解释 JSON-LD 文档的规则，遵循 DID 规范，用于实现不同 DID Document 的互操作，必须包含 https://www.w3.org/ns/did/v1 。
- `version`：必填字段。文档的版本号，用于文档的版本升级。
- `id`：必填字段。文档的 IOA。
- `publicKey`：选填字段。一组公钥，包含：
  - `id`，公钥的 ID。
  - `type`，字符串，代表公钥的加密算法类型，支持 SM2、Ed25519 和Secp256k1 三种。
  - `controller`，一个 IOA，表明此公钥的归属。
  - `publicKeyHex`，公钥的十六进制编码。
- `authentication`：必填字段。一组公钥的 IOA，表明此 IOA 的归属，拥有此公钥对应私钥的一方可以控制和管理此 IOA 文档。
- `alsoKnownAs`：选填字段。一组和本 IOA 相关联的其他 ID，包括：
  - `type`，关联标识的类型。
  - `id`，关联的标识。
- `extension`：IOA 扩展字段。包含如下字段：
  - `recovery`，选填字段。一组公钥 ID，在 authentication 私钥泄漏或者丢失的情况下用来恢复对文档的控制权。
  - `ttl`，必填字段。Time-To-Live，即如果解析使用缓存的话缓存生效的时间，单位秒。
  - `delegateSign`，选填字段。第三方对 publicKey 的签名，可信解析使用。包括：
    - `signer`，签名者，这里是一个公钥的 ID。
    - `signatureValue`，使用相应私钥对 publicKey 字段的签名。
  - `type`，IOA 文档的属性类型。
- `attributes`：必填字段。一组属性。包含如下字段：

| 参数    | 描述                                        |
| ------- | ------------------------------------------- |
| key     | 属性的关键字                                |
| desc    | 选填。属性描述                              |
| encrypt | 选填。是否加密，0非加密，1加密              |
| format  | 选填。image、text、video、mixture等数据类型 |
| value   | 选填。属性自定义value                       |

- `acsns`：选填字段。一组子链 `AC` 号，只有 `IOA` 文档类型不是凭证类型且文档是主链上的 `IOA` 文档才可能有该字段，存放当前IOA拥有的所有 `AC` 号。
- `verifiableCredentials`：选填字段。凭证列表，包含：
  - `id`，可验证声明的 IOA
  - `type`，凭证类型
- `service`：选填字段。一组服务地址，包含：
  - `id`，服务地址的 ID
  - `type`，字符串，代表服务的类型
  - `serviceEndPoint`，一个 URI 地址
- `created`：必填字段。创建时间
- `updated`：必填字段。上次的更新时间
- `proof`：选填字段。文档所有者对文档内容的签名，包含：
  - `creator`，proof 的创建者，这里是一个公钥的 ID
  - `signatureValue`，使用相应私钥对除 proof 字段的整个 IOA 文档签名

### 4.2 IOA 结构体定义

**DIDDocument**

| 字段名称       | 类型          | 描述                                                  | 是否可选 |
| -------------- | ------------- | ----------------------------------------------------- | -------- |
| Context        | []string      | DID 文档的上下文信息，通常包含 JSON-LD 的上下文 URL。 | 必填     |
| Version        | string        | DID 文档的版本号。                                    | 必填     |
| ID             | string        | DID 的唯一标识符。                                    | 必填     |
| PublicKey      | []PublicKey   | 公钥信息数组，包含与 DID 关联的公钥。                 | 必填     |
| Authentication | []string      | 用于身份验证的公钥 ID 列表。                          | 必填     |
| AlsoKnownAs    | []AlsoKnownAs | 与当前 DID 相关联的其他 DID 列表。                    | 可选     |
| Extension      | Extension     | 扩展字段，包含额外的元数据或配置信息。                | 必填     |
| Service        | []Service     | 与 DID 关联的服务信息数组。                           | 可选     |
| Created        | string        | DID 文档的创建时间。                                  | 必填     |
| Updated        | string        | DID 文档的最后更新时间。                              | 必填     |
| Proof          | Proof         | 签名信息，用于验证 DID 文档的完整性和来源。           | 可选     |

**PublicKey** 

| 字段名称     | 类型   | 描述                           | 是否可选 |
| ------------ | ------ | ------------------------------ | -------- |
| ID           | string | 公钥的唯一标识符。             | 必填     |
| Type         | string | 公钥的类型，例如 "SECP256K1"。 | 必填     |
| Controller   | string | 控制该公钥的 DID。             | 必填     |
| PublicKeyHex | string | 公钥的十六进制表示。           | 必填     |

**AlsoKnownAs** 

| 字段名称 | 类型   | 描述             | 是否可选 |
| -------- | ------ | ---------------- | -------- |
| Type     | int    | 关联 ID 的类型。 | 必填     |
| ID       | string | 关联的 DID。     | 必填     |

**Extension** 

| 字段名称              | 类型                   | 描述                                  | 是否可选 |
| --------------------- | ---------------------- | ------------------------------------- | -------- |
| Recovery              | []string               | 用于恢复 DID 的公钥 ID 列表。         | 可选     |
| TTL                   | uint                   | 缓存时间，单位为秒。                  | 必填     |
| DelegateSign          | DelegateSign           | 委托签名信息。                        | 可选     |
| Type                  | uint                   | 扩展字段的类型。                      | 必填     |
| Attributes            | []Attribute            | 属性列表，包含与 DID 相关的额外信息。 | 必填     |
| VerifiableCredentials | []VerifiableCredential | 可验证凭证列表。                      | 可选     |

**DelegateSign** 

| 字段名称       | 类型   | 描述            | 是否可选 |
| -------------- | ------ | --------------- | -------- |
| Signer         | string | 签名公钥的 ID。 | 必填     |
| SignatureValue | string | 签名的值。      | 必填     |

**Attribute** 

| 字段名称 | 类型   | 描述                                 | 是否可选 |
| -------- | ------ | ------------------------------------ | -------- |
| Key      | string | 属性的键。                           | 必填     |
| Desc     | string | 属性的描述。                         | 可选     |
| Encrypt  | uint   | 是否加密，0 表示非加密，1 表示加密。 | 可选     |
| Format   | string | 数据类型，例如 "text"、"image" 等。  | 可选     |
| Value    | string | 属性的值。                           | 可选     |

**VerifiableCredential** 

| 字段名称 | 类型   | 描述               | 是否可选 |
| -------- | ------ | ------------------ | -------- |
| ID       | string | 凭证的唯一标识符。 | 必填     |
| Type     | uint   | 凭证的类型。       | 必填     |

**Service**

如果Service ，当 `type` 为子链解析服务时，service为以下结构：

| 字段名称        | 类型   | 描述                 | 是否可选 |
| --------------- | ------ | -------------------- | -------- |
| ID              | string | 服务的唯一标识符。   | 必填     |
| Type            | string | DIDSubResolver       | 必填     |
| Version         | string | 服务的版本号。       | 必填     |
| ServerType      | uint   | 服务的服务地址类型。 | 必填     |
| Protocol        | uint   | 服务支持的协议类型。 | 必填     |
| ServiceEndpoint | string | 服务的端点 URL。     | 必填     |
| Port            | uint   | 服务的端口号。       | 必填     |

否则为

| 字段名称        | 类型   | 描述               | 是否可选 |
| --------------- | ------ | ------------------ | -------- |
| ID              | string | 服务的唯一标识符。 | 必填     |
| Type            | string | 服务的类型。       | 必填     |
| ServiceEndpoint | string | 服务的端点 URL。   | 必填     |

**Proof** 

| 字段名称       | 类型   | 描述                | 是否可选 |
| -------------- | ------ | ------------------- | -------- |
| Creator        | string | 创建签名的公钥 ID。 | 必填     |
| SignatureValue | string | 签名的值。          | 必填     |

### 4.3 IOA 文档示例：

```json
{
    "context": ["https://www.w3.org/ns/did/v1"],
    "version": "1.0.0",
    "id": "did:ioa:efnVUgqQFfYeu97ABf6sGm3WFtVXHZB2",
    "publicKey": [{
        "id": "did:ioa:efnVUgqQFfYeu97ABf6sGm3WFtVXHZB2#key-1",
        "type": "Ed25519",
        "controller": "did:ioa:efnVUgqQFfYeu97ABf6sGm3WFtVXHZB2",
        "publicKeyHex": "b9906e1b50e81501369cc777979f8bcf27bd1917d794fa6d5e320b1ccc4f48bb"
    }],
    "authentication": ["did:ioa:efnVUgqQFfYeu97ABf6sGm3WFtVXHZB2#key-1"],
    "extension": {
        "recovery": ["did:ioa:efnVUgqQFfYeu97ABf6sGm3WFtVXHZB2#key-2"],
        "ttl": 86400,
        "delegateSign": {
            "signer": "did:ioa:efJgt44mNDewKK1VEN454R17cjso3mSG#key-1",
            "signatureValue": "eyJhbGciOiJSUzI1NiIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19"
        },
        "type": 206
    },
    "service": [{
        "id": "did:ioa:ef24NBA7au48UTZrUNRHj2p3bnRzF3YCH#subResolve",
        "type": "IDPointerResolve",
        "serviceEndpoint": "https://resolver.ioa.org"
    }],
    "created": "2021-05-10T06:23:38Z",
    "updated": "2021-05-10T06:23:38Z",
    "proof": {
        "creator": "did:ioa:efJgt44mNDewKK1VEN454R17cjso3mSG#key-1",
        "signatureValue": "9E07CD62FE6CE0A843497EBD045C0AE9FD6E1845414D0ED251622C66D9CC927CC21DB9C09DFF628DC042FCBB7D8B2B4901E7DA9774C20065202B76D4B1C15900"
    }
}
```

## 5. IOA 方法

### 5.1 创建（Create）

注册接口主要完成IOA 文档的注册,支持HTTP POST 方法。创建 <code>IOA</code> 文档时 <code>proof</code> 字段的签名者需要是 <code>authentication</code> 字段里的 <code>公钥</code> 才有权限创建成功，如果IOA文档存在，则不允许重复创建。

#### 请求参数

| 参数  | 字段类型   | 描述                     |
| ----------- | ------ | ------------------------------- |
| id          | String | 要创建的IOA         |
| operation   | String | "create"                        |
| didDocument | Object | 要创建的IOA文档 |

#### 请求示例

```json
{
    "id": "did:ioa:sf24eYrmwXt6nx4fig3XJm7n9UP6PNRJ3",
    "operation": "create",
    "didDocument": {
        "context": [
            "https://www.w3.org/ns/did/v1"
        ],
        "version": "1.0.0",
        "id": "did:ioa:sf24eYrmwXt6nx4fig3XJm7n9UP6PNRJ3",
        "publicKey": [
            {
                "id": "did:ioa:sf24eYrmwXt6nx4fig3XJm7n9UP6PNRJ3",
                "type": "SECP256K1",
                "controller": "did:ioa:sf24eYrmwXt6nx4fig3XJm7n9UP6PNRJ3",
                "publicKeyHex": "04730dd9bd6256a6ffc03766e2d5b349f3734b7410116750b188b6d26b2ba092cf8c1ff07e6d61529a535681cb82f6e60501ca9ee2c1a672df631ff9a72a21c26b"
            },
            {
                "id": "did:ioa:sf6hQeifqQ56d2DBwFfYb4ByHtubkLLX",
                "type": "SECP256K1",
                "controller": "did:ioa:sf6hQeifqQ56d2DBwFfYb4ByHtubkLLX",
                "publicKeyHex": "04611176548da4a758187950dc2708a58ff7b4978c38cd9ae0961b889b74fa2f7be8592d69592bb2ffbc5315f8a563f479671ecf422d2ebd38d38c84c0d451b65c"
            },
            {
                "id": "did:ioa:sfXcZzrTnceK7PC67Pub2GD5Ps7EFqh4",
                "type": "SECP256K1",
                "controller": "did:ioa:sfXcZzrTnceK7PC67Pub2GD5Ps7EFqh4",
                "publicKeyHex": "04dd05980091d890191b128f29f46fce235b3e10f46cc9faab11de70d9f808e86769a9a955683eac7f3d40edc369d8c08492637493a0e72cc2838939980cfdb068"
            },
            {
                "id": "did:ioa:sfS6wbcHxZ9a9d4u1iMmmPNGSMWTpSJP",
                "type": "SECP256K1",
                "controller": "did:ioa:sfS6wbcHxZ9a9d4u1iMmmPNGSMWTpSJP",
                "publicKeyHex": "04a1617f65ae9683820f8c9bd73d1cb85a98969359ddafe1c99de3a36d00d156968aedceeec71b98c9a9727aa63a55a5349f830dbc7ced5b9518c04e4bb4c76328"
            }
        ],
        "authentication": [
            "did:ioa:sf24eYrmwXt6nx4fig3XJm7n9UP6PNRJ3",
            "did:ioa:sf6hQeifqQ56d2DBwFfYb4ByHtubkLLX"
        ],
        "extension": {
            "recovery": [
                "did:ioa:sfXcZzrTnceK7PC67Pub2GD5Ps7EFqh4"
            ],
            "ttl": 86400,
            "type": 206,
            "attributes": [
                {
                    "key": "201"
                }
            ]
        },
        "service": [
            {
                "id": "did:ioa:sfS6wbcHxZ9a9d4u1iMmmPNGSMWTpSJP",
                "type": "IDPointerResolve",
                "serviceEndpoint": "127.0.0.1"
            }
        ],
        "created": "2025-03-22T09:37:16Z",
        "updated": "2025-03-22T09:37:16Z",
        "proof": {
            "creator": "did:ioa:sf24eYrmwXt6nx4fig3XJm7n9UP6PNRJ3",
            "signatureValue": "2ExbMiHRpGSWxyZAwGgD4YnWUyXxhsp4F8mwGzE43VNrS3p3kru8JroVuox8AyXpyZrPhoepAUVtLwn3HyKnoXFcx1n"
        }
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

### 5.2 读取（Read）

通过 IOA 查询对应的 IOA 文档信息，支持 HTTP GET 方法。返回值为 <code>IOA</code> 文档的 <code>JSON</code> 字符串。

#### 请求参数

| 参数      | 字段类型 | 描述          |
| --------- | -------- | ------------- |
| id        | String   | 要读取的IOA |
| operation | String   | "read"        |

#### 请求示例

```plaintext
{
    "id": "did:ioa:sf24eYrmwXt6nx4fig3XJm7n9UP6PNRJ3",
    "operation": "read"
}
```
#### 返回数据

| 参数                                                   | 字段类型       | 描述                                                          |
| ------------------------------------------------------ | ------------- | ------------------------------------------------------------ |
| errorCode                                              | Int           | 见响应码说明                                               |
| data.didDocument                                       | Object        | 解析结果                                            |
| data.didDocument.context                              | Array         | 一组url数组                                                         |
| data.didDocument.version                               | String        | IOA文档的版本                             |
| data.didDocument.id                                    | String        | 解析的IOA                                          |
| data.didDocument.publicKey                             | Array(Object) | 公钥                                                   |
| data.didDocument.publicKey.id                          | String        | 公钥id                                              |
| data.didDocument.publicKey.type                        | String        | 公钥算法类型                                   |
| data.didDocument.publicKey.controller                  | String        | 一个IOA,表明此公钥的归属               |
| data.didDocument.publicKey.publicKeyHex                | String        | 十六进制公钥                                            |
| data.didDocument.authentication                        | Array         | 一组公钥id                                                |
| data.didDocument.alsoKnownAs                           | Array(Object) | 关联id                                              |
| data.didDocument.alsoKnownAs.type                      | Int           | 关联id的类型                                             |
| data.didDocument.alsoKnownAs.id                        | String        | 关联id                                              |
| data.didDocument.extension                             | Object        | 扩展字段                                            |
| data.didDocument.extension.recovery                    | Array         | 一组公钥id                                               |
| data.didDocument.extension.ttl                         | long          | 缓存时间，单位秒                                       |
| data.didDocument.extension.delegateSign                | Object        | 第三方对publicKey的签名，可信解析使用                          |
| data.didDocument.extension.delegateSign.signer         | String        | 签名公钥id                                 |
| data.didDocument.extension.delegateSign.signatureValue | String        | 签名的base58编码                                  |
| data.didDocument.extension.type                        | Int           | 属性类型                                                 |
| data.didDocument.extension.attributes                  | Array(Object) | 一组属性,属性结构见下文说明 |
| data.didDocument.extension.acsns                       | Array(Object) | AC号列表                                                |
| data.didDocument.extension.verfiableCredentials        | Array(Object) | 凭证列表，只有主链非凭证类型的BID文档才可能有本字段 |
| data.didDocument.extension.verfiableCredentials.id     | String        | 凭证ID                                               |
| data.didDocument.extension.verifiableCredentials.type  | Int           | 凭证类型                                              |
| data.didDocument.service                               | Array(Object) | 一组服务地址，结构见下表                                            |
| data.didDocument.service.id                            | String        | 服务地址的ID                                        |
| data.didDocument.service.type                          | String        | 字符串，代表服务的类型                                         |
| data.didDocument.service.serviceEndpoint               | String        | 服务的URL地址                                                |
| data.didDocument.created                               | String        | 创建时间                                          |
| data.didDocument.updated                               | String        | 上次的更新时间                                          |
| data.didDocument.proof                                 | Object        | 签名信息                                      |
| data.didDocument.proof.creator                         | String        | 签名公钥id                                      |
| data.didDocument.proof.signatureValue                  | String        | 签名的base58编码                                  |

其中 <code>attributes</code> 结构如下：
| 参数                                   | 字段类型   | 描述                     |
| --------------------------------------------- | ------ | ------------------------------------- |
| data.didDocument.extension.attributes.key     | String | 属性的key                     |
| data.didDocument.extension.attributes.desc    | String | 属性的描述                           |
| data.didDocument.extension.attributes.encrypt | Int    | 是否加密，0非加密，1加密   |
| data.didDocument.extension.attributes.format  | String | image、text、video、mixture等数据类型 |
| data.didDocument.extension.attributes.value   | String | 属性自定义value  |

当 <code>service.type</code> 为子链解析服务时,service结构如下：

| 参数                              | 字段类型   | 描述                                       |
| ---------------------------------------- | ------ | ---------------------------------------------------- |
| data.didDocument.service.id              | String | 服务地址的ID                              |
| data.didDocument.service.type            | String | 字符串，代表服务的类型                             |
| data.didDocument.service.version         | String | 解析服务支持的IOA协议版本 |
| data.didDocument.service.protocol        | Int    | 解析服务支持的传输协议   |
| data.didDocument.service.serverType      | Int    | 解析地址类型                           |
| data.didDocument.service.serviceEndpoint | String | 解析地址                           |
| data.didDocument.service.port            | Int    | 解析端口                                |

#### 返回示例

- 成功返回普通 `IOA` 文档：

```json
{
    "errorCode": 0,
    "data": {
        "didDocument": {
            "context": [
                "https://www.w3.org/ns/did/v1"
            ],
            "version": "1.0.0",
            "id": "did:ioa:sf24eYrmwXt6nx4fig3XJm7n9UP6PNRJ3",
            "publicKey": [
                {
                    "id": "did:ioa:sf24eYrmwXt6nx4fig3XJm7n9UP6PNRJ3",
                    "type": "SECP256K1",
                    "controller": "did:ioa:sf24eYrmwXt6nx4fig3XJm7n9UP6PNRJ3",
                    "publicKeyHex": "04730dd9bd6256a6ffc03766e2d5b349f3734b7410116750b188b6d26b2ba092cf8c1ff07e6d61529a535681cb82f6e60501ca9ee2c1a672df631ff9a72a21c26b"
                },
                {
                    "id": "did:ioa:sf6hQeifqQ56d2DBwFfYb4ByHtubkLLX",
                    "type": "SECP256K1",
                    "controller": "did:ioa:sf6hQeifqQ56d2DBwFfYb4ByHtubkLLX",
                    "publicKeyHex": "04611176548da4a758187950dc2708a58ff7b4978c38cd9ae0961b889b74fa2f7be8592d69592bb2ffbc5315f8a563f479671ecf422d2ebd38d38c84c0d451b65c"
                },
                {
                    "id": "did:ioa:sfXcZzrTnceK7PC67Pub2GD5Ps7EFqh4",
                    "type": "SECP256K1",
                    "controller": "did:ioa:sfXcZzrTnceK7PC67Pub2GD5Ps7EFqh4",
                    "publicKeyHex": "04dd05980091d890191b128f29f46fce235b3e10f46cc9faab11de70d9f808e86769a9a955683eac7f3d40edc369d8c08492637493a0e72cc2838939980cfdb068"
                },
                {
                    "id": "did:ioa:sfS6wbcHxZ9a9d4u1iMmmPNGSMWTpSJP",
                    "type": "SECP256K1",
                    "controller": "did:ioa:sfS6wbcHxZ9a9d4u1iMmmPNGSMWTpSJP",
                    "publicKeyHex": "04a1617f65ae9683820f8c9bd73d1cb85a98969359ddafe1c99de3a36d00d156968aedceeec71b98c9a9727aa63a55a5349f830dbc7ced5b9518c04e4bb4c76328"
                }
            ],
            "authentication": [
                "did:ioa:sf24eYrmwXt6nx4fig3XJm7n9UP6PNRJ3",
                "did:ioa:sf6hQeifqQ56d2DBwFfYb4ByHtubkLLX"
            ],
            "extension": {
                "recovery": [
                    "did:ioa:sfXcZzrTnceK7PC67Pub2GD5Ps7EFqh4"
                ],
                "ttl": 86400,
                "type": 206,
                "attributes": [
                    {
                        "key": "201"
                    }
                ]
            },
            "service": [
                {
                    "id": "did:ioa:sfS6wbcHxZ9a9d4u1iMmmPNGSMWTpSJP",
                    "type": "IDPointerResolve",
                    "serviceEndpoint": "127.0.0.1"
                }
            ],
            "created": "2025-03-22T09:37:16Z",
            "updated": "2025-03-22T09:37:16Z",
            "proof": {
                "creator": "did:ioa:sf24eYrmwXt6nx4fig3XJm7n9UP6PNRJ3",
                "signatureValue": "2ExbMiHRpGSWxyZAwGgD4YnWUyXxhsp4F8mwGzE43VNrS3p3kru8JroVuox8AyXpyZrPhoepAUVtLwn3HyKnoXFcx1n"
            }
        }
    }
}
```

- 成功返回包含子链解析服务地址的 `IOA` 文档示例：

  ```json
  {
      "errorCode": 0,
      "data": {
          "didDocument": {
              "context": [
                  "https://www.w3.org/ns/did/v1"
              ],
              "version": "1.0.0",
              "id": "did:ioa:sf24eYrmwXt6nx4fig3XJm7n9UP6PNRJ3",
              "publicKey": [
                  {
                      "id": "did:ioa:sf24eYrmwXt6nx4fig3XJm7n9UP6PNRJ3",
                      "type": "SECP256K1",
                      "controller": "did:ioa:sf24eYrmwXt6nx4fig3XJm7n9UP6PNRJ3",
                      "publicKeyHex": "04730dd9bd6256a6ffc03766e2d5b349f3734b7410116750b188b6d26b2ba092cf8c1ff07e6d61529a535681cb82f6e60501ca9ee2c1a672df631ff9a72a21c26b"
                  },
                  {
                      "id": "did:ioa:sf6hQeifqQ56d2DBwFfYb4ByHtubkLLX",
                      "type": "SECP256K1",
                      "controller": "did:ioa:sf6hQeifqQ56d2DBwFfYb4ByHtubkLLX",
                      "publicKeyHex": "04611176548da4a758187950dc2708a58ff7b4978c38cd9ae0961b889b74fa2f7be8592d69592bb2ffbc5315f8a563f479671ecf422d2ebd38d38c84c0d451b65c"
                  },
                  {
                      "id": "did:ioa:sfXcZzrTnceK7PC67Pub2GD5Ps7EFqh4",
                      "type": "SECP256K1",
                      "controller": "did:ioa:sfXcZzrTnceK7PC67Pub2GD5Ps7EFqh4",
                      "publicKeyHex": "04dd05980091d890191b128f29f46fce235b3e10f46cc9faab11de70d9f808e86769a9a955683eac7f3d40edc369d8c08492637493a0e72cc2838939980cfdb068"
                  },
                  {
                      "id": "did:ioa:sfS6wbcHxZ9a9d4u1iMmmPNGSMWTpSJP",
                      "type": "SECP256K1",
                      "controller": "did:ioa:sfS6wbcHxZ9a9d4u1iMmmPNGSMWTpSJP",
                      "publicKeyHex": "04a1617f65ae9683820f8c9bd73d1cb85a98969359ddafe1c99de3a36d00d156968aedceeec71b98c9a9727aa63a55a5349f830dbc7ced5b9518c04e4bb4c76328"
                  }
              ],
              "authentication": [
                  "did:ioa:sf24eYrmwXt6nx4fig3XJm7n9UP6PNRJ3",
                  "did:ioa:sf6hQeifqQ56d2DBwFfYb4ByHtubkLLX"
              ],
              "extension": {
                  "recovery": [
                      "did:ioa:sfXcZzrTnceK7PC67Pub2GD5Ps7EFqh4"
                  ],
                  "ttl": 86400,
                  "type": 206,
                  "attributes": [
                      {
                          "key": "201"
                      }
                  ]
              },
              "service": [
                  {
                      "id": "did:ioa:sfS6wbcHxZ9a9d4u1iMmmPNGSMWTpSJP",
                      "type": "DIDSubResolve",
                      "serviceEndpoint": "127.0.0.1",
                      "version": "1.0.0",
                      "serverType": 1,
                      "protocol": 3,
                      "port": 8080
                  }
              ],
              "created": "2025-03-22T09:37:16Z",
              "updated": "2025-03-22T09:37:16Z",
              "proof": {
                  "creator": "did:ioa:sf24eYrmwXt6nx4fig3XJm7n9UP6PNRJ3",
                  "signatureValue": "2ExbMiHRpGSWxyZAwGgD4YnWUyXxhsp4F8mwGzE43VNrS3p3kru8JroVuox8AyXpyZrPhoepAUVtLwn3HyKnoXFcx1n"
              }
          }
      }
  }
  ```

### 5.3 更新（Update）

更新接口主要完成IOA 文档的更新，支持HTTP POST 方法。更新操作不允许更新 <code>authentication</code> 字段，更新 <code>IOA</code> 文档时 <code>proof</code> 字段的签名者需要是 <code>authentication</code> 字段里的 <code>公钥</code> 才有权限更新成功。

| 参数   | 字段类型   | 描述                    |
| ----------- | ------ | ------------------------------- |
| id          | String | 要更新的IOA     |
| operation   | String | "update"                        |
| didDocument | Object | 更新后的IOA文档 |

#### 请求示例

```json
{
    "id": "did:bid:sf24eYrmwXt6nx4fig3XJm7n9UP6PNRJ3",
    "operation": "update",
    "didDocument": {
        "context": [
            "https://www.w3.org/ns/did/v1"
        ],
        "version": "1.0.0",
        "id": "did:ioa:sf24eYrmwXt6nx4fig3XJm7n9UP6PNRJ3",
        "publicKey": [
            {
                "id": "did:ioa:sf24eYrmwXt6nx4fig3XJm7n9UP6PNRJ3",
                "type": "SECP256K1",
                "controller": "did:ioa:sf24eYrmwXt6nx4fig3XJm7n9UP6PNRJ3",
                "publicKeyHex": "04730dd9bd6256a6ffc03766e2d5b349f3734b7410116750b188b6d26b2ba092cf8c1ff07e6d61529a535681cb82f6e60501ca9ee2c1a672df631ff9a72a21c26b"
            },
            {
                "id": "did:ioa:sf6hQeifqQ56d2DBwFfYb4ByHtubkLLX",
                "type": "SECP256K1",
                "controller": "did:ioa:sf6hQeifqQ56d2DBwFfYb4ByHtubkLLX",
                "publicKeyHex": "04611176548da4a758187950dc2708a58ff7b4978c38cd9ae0961b889b74fa2f7be8592d69592bb2ffbc5315f8a563f479671ecf422d2ebd38d38c84c0d451b65c"
            },
            {
                "id": "did:ioa:sfXcZzrTnceK7PC67Pub2GD5Ps7EFqh4",
                "type": "SECP256K1",
                "controller": "did:ioa:sfXcZzrTnceK7PC67Pub2GD5Ps7EFqh4",
                "publicKeyHex": "04dd05980091d890191b128f29f46fce235b3e10f46cc9faab11de70d9f808e86769a9a955683eac7f3d40edc369d8c08492637493a0e72cc2838939980cfdb068"
            },
            {
                "id": "did:ioa:sfS6wbcHxZ9a9d4u1iMmmPNGSMWTpSJP",
                "type": "SECP256K1",
                "controller": "did:ioa:sfS6wbcHxZ9a9d4u1iMmmPNGSMWTpSJP",
                "publicKeyHex": "04a1617f65ae9683820f8c9bd73d1cb85a98969359ddafe1c99de3a36d00d156968aedceeec71b98c9a9727aa63a55a5349f830dbc7ced5b9518c04e4bb4c76328"
            },
            {
                "id": "did:ioa:sf5iXTcoPvqKikeAMtvCCNHWaQoqA96t",
                "type": "SECP256K1",
                "controller": "did:ioa:sf5iXTcoPvqKikeAMtvCCNHWaQoqA96t",
                "publicKeyHex": "043f404ab3bf64c30526b5e588e89d59f304b8b8ba53204e002b1cea6a7827ccd64389cd351605b100e4a2e950a0a5a121d8fa4d3c3b4ab15ed94bf54770cd2a3c"
            }
        ],
        "authentication": [
            "did:ioa:sf24eYrmwXt6nx4fig3XJm7n9UP6PNRJ3",
            "did:ioa:sf6hQeifqQ56d2DBwFfYb4ByHtubkLLX"
        ],
        "extension": {
            "recovery": [
                "did:ioa:sfXcZzrTnceK7PC67Pub2GD5Ps7EFqh4"
            ],
            "ttl": 86400,
            "type": 206,
            "attributes": [
                {
                    "key": "201"
                }
            ]
        },
        "service": [
            {
                "id": "did:ioa:sfS6wbcHxZ9a9d4u1iMmmPNGSMWTpSJP",
                "type": "IDPointerResolve",
                "serviceEndpoint": "127.0.0.1"
            },
            {
                "id": "did:ioa:sf5iXTcoPvqKikeAMtvCCNHWaQoqA96t",
                "type": "IDHubResolve",
                "serviceEndpoint": "192.168.2.15"
            }
        ],
        "created": "2025-03-22T09:44:16Z",
        "updated": "2025-03-22T09:44:25Z",
        "proof": {
            "creator": "did:ioa:sf24eYrmwXt6nx4fig3XJm7n9UP6PNRJ3",
            "signatureValue": "hLDyzcMbSpaV74RRqaN7bBqbN43Zm8FfqQdUmmGkZitVRiV5sHYv2JEkyxCtNLtw1iMJzSaUtKwgUrjop5JCGdCPwN"
        }
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

### 5.4 停用（Deactivate）

<code>Deactivate</code> 接口主要完成对 <code>IOA</code> 文档撤销，支持 <code>HTTP POST</code> 方法。撤销后的 <code>IOA</code> 文档更新为空而不是删除, 停用 <code>IOA</code> 文档时 <code>proof</code> 字段的签名者需要是 <code>recovery</code> 字段里的 <code>公钥</code> 才有权限停用成功。

| 参数 | 字段类型   | 描述                              |
| --------- | ------ | ---------------------------------------- |
| id        | String | 要停用的IOA          |
| operation | String | "delete"                                 |
| proof     | Object | recovery中的公钥对应的私钥签名 |


#### 请求示例

```json
{
    "id": "did:ioa:sf24eYrmwXt6nx4fig3XJm7n9UP6PNRJ3",
    "operation": "deactivate",
    "proof": {
        "creator": "did:ioa:sfXcZzrTnceK7PC67Pub2GD5Ps7EFqh4",
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

<code>Recovery</code> 接口主要完成对 <code>IOA</code> 文档里的 <code>authentication</code> 和 <code>publicKey</code> 字段里的内容修改，支持 <code>HTTP POST</code> 方法。更换密钥时 <code>proof</code> 字段的签名者需要是 <code>recovery</code> 字段里的 <code>公钥</code> 才有权限更换成功。

| 参数       | 字段类型   | 描述               |
| ---------------- | ------ | --------------------------- |
| id               | String | 要恢复的IOA  |
| operation        | String | "recovery"                  |
| didDocumentation | Object | 更新后的did文档 |

#### 请求示例

```json
{
    "id": "did:ioa:sf24eYrmwXt6nx4fig3XJm7n9UP6PNRJ3",
    "operation": "recovery",
    "didDocument": {
        "context": [
            "https://www.w3.org/ns/did/v1"
        ],
        "version": "1.0.0",
        "id": "did:ioa:sf24eYrmwXt6nx4fig3XJm7n9UP6PNRJ3",
        "publicKey": [
            {
                "id": "did:ioa:sf24eYrmwXt6nx4fig3XJm7n9UP6PNRJ3",
                "type": "SECP256K1",
                "controller": "did:ioa:sf24eYrmwXt6nx4fig3XJm7n9UP6PNRJ3",
                "publicKeyHex": "04730dd9bd6256a6ffc03766e2d5b349f3734b7410116750b188b6d26b2ba092cf8c1ff07e6d61529a535681cb82f6e60501ca9ee2c1a672df631ff9a72a21c26b"
            },
            {
                "id": "did:ioa:sfXcZzrTnceK7PC67Pub2GD5Ps7EFqh4",
                "type": "SECP256K1",
                "controller": "did:ioa:sfXcZzrTnceK7PC67Pub2GD5Ps7EFqh4",
                "publicKeyHex": "04dd05980091d890191b128f29f46fce235b3e10f46cc9faab11de70d9f808e86769a9a955683eac7f3d40edc369d8c08492637493a0e72cc2838939980cfdb068"
            },
            {
                "id": "did:ioa:sfS6wbcHxZ9a9d4u1iMmmPNGSMWTpSJP",
                "type": "SECP256K1",
                "controller": "did:ioa:sfS6wbcHxZ9a9d4u1iMmmPNGSMWTpSJP",
                "publicKeyHex": "04a1617f65ae9683820f8c9bd73d1cb85a98969359ddafe1c99de3a36d00d156968aedceeec71b98c9a9727aa63a55a5349f830dbc7ced5b9518c04e4bb4c76328"
            }
        ],
        "authentication": [
            "did:ioa:sf24eYrmwXt6nx4fig3XJm7n9UP6PNRJ3"
        ],
        "extension": {
            "recovery": [
                "did:ioa:sfXcZzrTnceK7PC67Pub2GD5Ps7EFqh4"
            ],
            "ttl": 86400,
            "type": 206,
            "attributes": [
                {
                    "key": "201"
                }
            ]
        },
        "service": [
            {
                "id": "did:ioa:sfS6wbcHxZ9a9d4u1iMmmPNGSMWTpSJP",
                "type": "IDPointerResolve",
                "serviceEndpoint": "127.0.0.1"
            }
        ],
        "created": "2025-03-22T09:46:13Z",
        "updated": "2025-03-22T09:46:18Z",
        "proof": {
            "creator": "did:ioa:sfXcZzrTnceK7PC67Pub2GD5Ps7EFqh4",
            "signatureValue": "2ESSXREizoYDfkp7t5MnmqkaAtAykBi45ri2mkc3Tr1XHd1JQAnaMrxpnodqSFADw2SacNrHNmjf6KQb7BYBgyJbaKi"
        }
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

---

以上是完整的 **IOA 方法** 部分，包括创建、读取、更新、停用和恢复操作的详细描述和示例。

## 6. 安全和隐私考虑

### 6.1 安全考虑

- **公钥完整性**：IOA 的公钥存储在区块链上，确保其不可篡改。
- **私钥保护**：私钥必须安全存储，避免泄露给未经授权的第三方。
- **防止中间人攻击**：通过加密传输（如 TLS）保护 IOA 文档的传输过程。
- **身份恢复机制**：在私钥丢失的情况下，通过恢复密钥重新设置主密钥。
-  **防止 `DDOS` 攻击 **： `IOA` 系统基于区块链技术构建，天然防止 `DDOS` 攻击。
-  **防止隐私数据盗窃攻击 **： 在 `IOA` 系统中，所有与用户隐私相关的数据都存储在本地。仅加密算法生成的 `hash` 或字符串在链上是公开的，攻击者无法通过哈希值或字符串导出隐私数据。

### 6.2 隐私考虑

- **数据最小化**：仅在区块链上存储必要的信息，如公钥和服务端点，避免存储敏感信息。
- **隐私保护**：使用加密技术（如同态加密）保护敏感数据，避免隐私泄露。
- **访问控制**：通过加密方法控制对 IOA 及其关联服务的访问权限。
- **匿名化技术**：在必要时使用匿名化技术（如零知识证明）保护用户隐私。
