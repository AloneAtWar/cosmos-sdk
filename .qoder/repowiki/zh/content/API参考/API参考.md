# API参考

<cite>
**本文档中引用的文件**   
- [auth/v1beta1/query.proto](file://proto/cosmos/auth/v1beta1/query.proto)
- [auth/v1beta1/tx.proto](file://proto/cosmos/auth/v1beta1/tx.proto)
- [bank/v1beta1/query.proto](file://proto/cosmos/bank/v1beta1/query.proto)
- [bank/v1beta1/tx.proto](file://proto/cosmos/bank/v1beta1/tx.proto)
- [staking/v1beta1/query.proto](file://proto/cosmos/staking/v1beta1/query.proto)
- [distribution/v1beta1/query.proto](file://proto/cosmos/distribution/v1beta1/query.proto)
- [gov/v1/query.proto](file://proto/cosmos/gov/v1/query.proto)
- [gov/v1/tx.proto](file://proto/cosmos/gov/v1/tx.proto)
- [tx/v1beta1/service.proto](file://proto/cosmos/tx/v1beta1/service.proto)
- [base/node/v1beta1/query.proto](file://proto/cosmos/base/node/v1beta1/query.proto)
</cite>

## 目录
1. [引言](#引言)
2. [身份验证模块 (auth)](#身份验证模块-auth)
3. [银行模块 (bank)](#银行模块-bank)
4. [质押模块 (staking)](#质押模块-staking)
5. [分发模块 (distribution)](#分发模块-distribution)
6. [治理模块 (gov)](#治理模块-gov)
7. [交易服务 (tx)](#交易服务-tx)
8. [节点服务 (node)](#节点服务-node)
9. [gRPC-Gateway转换机制](#grpc-gateway转换机制)
10. [身份验证机制](#身份验证机制)

## 引言
本文档提供了Cosmos SDK中通过gRPC和REST暴露的公共API的详尽参考。文档涵盖了主要模块的查询服务（Query Service）和事务服务（Tx Service），包括`auth`、`bank`、`staking`、`distribution`、`gov`等核心模块。每个API端点都遵循RESTful API文档标准，详细说明了HTTP方法、URL路径模式、请求参数、请求体JSON Schema、响应状态码、响应体JSON Schema以及可能的错误码。此外，文档还解释了gRPC-Gateway如何将gRPC调用转换为HTTP/JSON，并说明了基于交易签名的身份验证机制。

**本文档中引用的文件**
- [auth/v1beta1/query.proto](file://proto/cosmos/auth/v1beta1/query.proto)
- [auth/v1beta1/tx.proto](file://proto/cosmos/auth/v1beta1/tx.proto)
- [bank/v1beta1/query.proto](file://proto/cosmos/bank/v1beta1/query.proto)
- [bank/v1beta1/tx.proto](file://proto/cosmos/bank/v1beta1/tx.proto)
- [staking/v1beta1/query.proto](file://proto/cosmos/staking/v1beta1/query.proto)
- [distribution/v1beta1/query.proto](file://proto/cosmos/distribution/v1beta1/query.proto)
- [gov/v1/query.proto](file://proto/cosmos/gov/v1/query.proto)
- [gov/v1/tx.proto](file://proto/cosmos/gov/v1/tx.proto)
- [tx/v1beta1/service.proto](file://proto/cosmos/tx/v1beta1/service.proto)
- [base/node/v1beta1/query.proto](file://proto/cosmos/base/node/v1beta1/query.proto)

## 身份验证模块 (auth)

### 查询服务 (Query Service)
身份验证模块的查询服务提供账户信息和系统参数的查询功能。

#### Accounts
查询所有现有账户。

- **HTTP方法**: GET
- **URL路径**: `/cosmos/auth/v1beta1/accounts`
- **请求参数**:
  - `pagination.key` (查询参数, string): 分页键
  - `pagination.offset` (查询参数, string): 偏移量
  - `pagination.limit` (查询参数, string): 限制数量
  - `pagination.count_total` (查询参数, boolean): 是否计算总数
  - `pagination.reverse` (查询参数, boolean): 是否反向排序
- **请求体**: 无
- **响应状态码**:
  - `200 OK`: 成功获取账户列表
  - `400 Bad Request`: 请求参数无效
  - `500 Internal Server Error`: 服务器内部错误
- **响应体JSON Schema**:
```json
{
  "accounts": [
    {
      "@type": "string",
      "value": "object"
    }
  ],
  "pagination": {
    "next_key": "string",
    "total": "string"
  }
}
```
- **错误码**: 无特定错误码

**本文档中引用的文件**
- [auth/v1beta1/query.proto](file://proto/cosmos/auth/v1beta1/query.proto#L14-L24)

#### Account
根据地址查询账户详情。

- **HTTP方法**: GET
- **URL路径**: `/cosmos/auth/v1beta1/accounts/{address}`
- **请求参数**:
  - `address` (路径参数, string): 账户地址
- **请求体**: 无
- **响应状态码**:
  - `200 OK`: 成功获取账户信息
  - `400 Bad Request`: 地址格式无效
  - `404 Not Found`: 账户不存在
  - `500 Internal Server Error`: 服务器内部错误
- **响应体JSON Schema**:
```json
{
  "account": {
    "@type": "string",
    "value": "object"
  }
}
```
- **错误码**: 无特定错误码

**本文档中引用的文件**
- [auth/v1beta1/query.proto](file://proto/cosmos/auth/v1beta1/query.proto#L26-L30)

#### Params
查询所有参数。

- **HTTP方法**: GET
- **URL路径**: `/cosmos/auth/v1beta1/params`
- **请求参数**: 无
- **请求体**: 无
- **响应状态码**:
  - `200 OK`: 成功获取参数
  - `500 Internal Server Error`: 服务器内部错误
- **响应体JSON Schema**:
```json
{
  "params": {
    "max_memo_characters": "string",
    "tx_sig_limit": "string",
    "tx_size_cost_per_byte": "string",
    "sig_verify_cost_ed25519": "string",
    "sig_verify_cost_secp256k1": "string"
  }
}
```
- **错误码**: 无特定错误码

**本文档中引用的文件**
- [auth/v1beta1/query.proto](file://proto/cosmos/auth/v1beta1/query.proto#L39-L43)

### 事务服务 (Tx Service)
身份验证模块的事务服务允许更新模块参数。

#### UpdateParams
更新x/auth模块的参数。

- **HTTP方法**: POST
- **URL路径**: `/cosmos/auth/v1beta1/tx/update_params`
- **请求参数**: 无
- **请求体JSON Schema**:
```json
{
  "type": "object",
  "properties": {
    "authority": {
      "type": "string"
    },
    "params": {
      "type": "object",
      "properties": {
        "max_memo_characters": {
          "type": "string"
        },
        "tx_sig_limit": {
          "type": "string"
        },
        "tx_size_cost_per_byte": {
          "type": "string"
        },
        "sig_verify_cost_ed25519": {
          "type": "string"
        },
        "sig_verify_cost_secp256k1": {
          "type": "string"
        }
      },
      "required": ["max_memo_characters", "tx_sig_limit", "tx_size_cost_per_byte", "sig_verify_cost_ed25519", "sig_verify_cost_secp256k1"]
    }
  },
  "required": ["authority", "params"]
}
```
- **响应状态码**:
  - `200 OK`: 参数更新成功
  - `400 Bad Request`: 请求体格式无效
  - `401 Unauthorized`: 签名验证失败
  - `403 Forbidden`: 权限不足
  - `500 Internal Server Error`: 服务器内部错误
- **响应体JSON Schema**:
```json
{
  "height": "string",
  "txhash": "string",
  "codespace": "string",
  "code": "integer",
  "data": "string",
  "raw_log": "string",
  "logs": [
    {
      "msg_index": "integer",
      "log": "string",
      "events": [
        {
          "type": "string",
          "attributes": [
            {
              "key": "string",
              "value": "string"
            }
          ]
        }
      ]
    }
  ],
  "info": "string",
  "gas_wanted": "string",
  "gas_used": "string",
  "tx": {
    "@type": "string",
    "value": "object"
  },
  "timestamp": "string"
}
```
- **错误码**:
  - `403`: 权限不足，只有授权地址可以更新参数

**本文档中引用的文件**
- [auth/v1beta1/tx.proto](file://proto/cosmos/auth/v1beta1/tx.proto#L18-L20)

## 银行模块 (bank)

### 查询服务 (Query Service)
银行模块的查询服务提供账户余额、总供应量和代币元数据的查询功能。

#### Balance
查询单个账户的单个币种余额。

- **HTTP方法**: GET
- **URL路径**: `/cosmos/bank/v1beta1/balances/{address}/by_denom`
- **请求参数**:
  - `address` (路径参数, string): 账户地址
  - `denom` (查询参数, string): 币种名称
- **请求体**: 无
- **响应状态码**:
  - `200 OK`: 成功获取余额
  - `400 Bad Request`: 请求参数无效
  - `404 Not Found`: 账户或币种不存在
  - `500 Internal Server Error`: 服务器内部错误
- **响应体JSON Schema**:
```json
{
  "balance": {
    "denom": "string",
    "amount": "string"
  }
}
```
- **错误码**: 无特定错误码

**本文档中引用的文件**
- [bank/v1beta1/query.proto](file://proto/cosmos/bank/v1beta1/query.proto#L17-L21)

#### AllBalances
查询单个账户的所有币种余额。

- **HTTP方法**: GET
- **URL路径**: `/cosmos/bank/v1beta1/balances/{address}`
- **请求参数**:
  - `address` (路径参数, string): 账户地址
  - `pagination.key` (查询参数, string): 分页键
  - `pagination.offset` (查询参数, string): 偏移量
  - `pagination.limit` (查询参数, string): 限制数量
  - `pagination.count_total` (查询参数, boolean): 是否计算总数
  - `pagination.reverse` (查询参数, boolean): 是否反向排序
  - `resolve_denom` (查询参数, boolean): 是否解析代币名称
- **请求体**: 无
- **响应状态码**:
  - `200 OK`: 成功获取所有余额
  - `400 Bad Request`: 请求参数无效
  - `404 Not Found`: 账户不存在
  - `500 Internal Server Error`: 服务器内部错误
- **响应体JSON Schema**:
```json
{
  "balances": [
    {
      "denom": "string",
      "amount": "string"
    }
  ],
  "pagination": {
    "next_key": "string",
    "total": "string"
  }
}
```
- **错误码**: 无特定错误码

**本文档中引用的文件**
- [bank/v1beta1/query.proto](file://proto/cosmos/bank/v1beta1/query.proto#L23-L30)

#### TotalSupply
查询所有币种的总供应量。

- **HTTP方法**: GET
- **URL路径**: `/cosmos/bank/v1beta1/supply`
- **请求参数**:
  - `pagination.key` (查询参数, string): 分页键
  - `pagination.offset` (查询参数, string): 偏移量
  - `pagination.limit` (查询参数, string): 限制数量
  - `pagination.count_total` (查询参数, boolean): 是否计算总数
  - `pagination.reverse` (查询参数, boolean): 是否反向排序
- **请求体**: 无
- **响应状态码**:
  - `200 OK`: 成功获取总供应量
  - `400 Bad Request`: 请求参数无效
  - `500 Internal Server Error`: 服务器内部错误
- **响应体JSON Schema**:
```json
{
  "supply": [
    {
      "denom": "string",
      "amount": "string"
    }
  ],
  "pagination": {
    "next_key": "string",
    "total": "string"
  }
}
```
- **错误码**: 无特定错误码

**本文档中引用的文件**
- [bank/v1beta1/query.proto](file://proto/cosmos/bank/v1beta1/query.proto#L54-L61)

#### DenomMetadata
查询指定币种的客户端元数据。

- **HTTP方法**: GET
- **URL路径**: `/cosmos/bank/v1beta1/denoms_metadata/{denom}`
- **请求参数**:
  - `denom` (路径参数, string): 币种名称
- **请求体**: 无
- **响应状态码**:
  - `200 OK`: 成功获取元数据
  - `400 Bad Request`: 币种名称无效
  - `404 Not Found`: 币种元数据不存在
  - `500 Internal Server Error`: 服务器内部错误
- **响应体JSON Schema**:
```json
{
  "metadata": {
    "description": "string",
    "denom_units": [
      {
        "denom": "string",
        "exponent": "integer",
        "aliases": ["string"]
      }
    ],
    "base": "string",
    "display": "string",
    "name": "string",
    "symbol": "string",
    "uri": "string",
    "uri_hash": "string"
  }
}
```
- **错误码**: 无特定错误码

**本文档中引用的文件**
- [bank/v1beta1/query.proto](file://proto/cosmos/bank/v1beta1/query.proto#L85-L88)

### 事务服务 (Tx Service)
银行模块的事务服务提供资金转移和参数更新功能。

#### Send
从一个账户向另一个账户发送资金。

- **HTTP方法**: POST
- **URL路径**: `/cosmos/bank/v1beta1/tx/send`
- **请求参数**: 无
- **请求体JSON Schema**:
```json
{
  "type": "object",
  "properties": {
    "from_address": {
      "type": "string"
    },
    "to_address": {
      "type": "string"
    },
    "amount": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "denom": "string",
          "amount": "string"
        },
        "required": ["denom", "amount"]
      }
    }
  },
  "required": ["from_address", "to_address", "amount"]
}
```
- **响应状态码**:
  - `200 OK`: 资金转移成功
  - `400 Bad Request`: 请求体格式无效
  - `401 Unauthorized`: 签名验证失败
  - `403 Forbidden`: 余额不足或权限不足
  - `500 Internal Server Error`: 服务器内部错误
- **响应体JSON Schema**:
```json
{
  "height": "string",
  "txhash": "string",
  "codespace": "string",
  "code": "integer",
  "data": "string",
  "raw_log": "string",
  "logs": [
    {
      "msg_index": "integer",
      "log": "string",
      "events": [
        {
          "type": "string",
          "attributes": [
            {
              "key": "string",
              "value": "string"
            }
          ]
        }
      ]
    }
  ],
  "info": "string",
  "gas_wanted": "string",
  "gas_used": "string",
  "tx": {
    "@type": "string",
    "value": "object"
  },
  "timestamp": "string"
}
```
- **错误码**:
  - `403`: 余额不足，无法完成转账
  - `403`: 发送方账户没有足够的权限

**本文档中引用的文件**
- [bank/v1beta1/tx.proto](file://proto/cosmos/bank/v1beta1/tx.proto#L18-L19)

#### MultiSend
从多个账户向多个账户进行任意的多输入、多输出资金转移。

- **HTTP方法**: POST
- **URL路径**: `/cosmos/bank/v1beta1/tx/multi_send`
- **请求参数**: 无
- **请求体JSON Schema**:
```json
{
  "type": "object",
  "properties": {
    "inputs": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "address": "string",
          "coins": {
            "type": "array",
            "items": {
              "type": "object",
              "properties": {
                "denom": "string",
                "amount": "string"
              },
              "required": ["denom", "amount"]
            }
          }
        },
        "required": ["address", "coins"]
      }
    },
    "outputs": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "address": "string",
          "coins": {
            "type": "array",
            "items": {
              "type": "object",
              "properties": {
                "denom": "string",
                "amount": "string"
              },
              "required": ["denom", "amount"]
            }
          }
        },
        "required": ["address", "coins"]
      }
    }
  },
  "required": ["inputs", "outputs"]
}
```
- **响应状态码**:
  - `200 OK`: 多重资金转移成功
  - `400 Bad Request`: 请求体格式无效
  - `401 Unauthorized`: 签名验证失败
  - `403 Forbidden`: 余额不足或权限不足
  - `500 Internal Server Error`: 服务器内部错误
- **响应体JSON Schema**:
```json
{
  "height": "string",
  "txhash": "string",
  "codespace": "string",
  "code": "integer",
  "data": "string",
  "raw_log": "string",
  "logs": [
    {
      "msg_index": "integer",
      "log": "string",
      "events": [
        {
          "type": "string",
          "attributes": [
            {
              "key": "string",
              "value": "string"
            }
          ]
        }
      ]
    }
  ],
  "info": "string",
  "gas_wanted": "string",
  "gas_used": "string",
  "tx": {
    "@type": "string",
    "value": "object"
  },
  "timestamp": "string"
}
```
- **错误码**:
  - `403`: 输入和输出的金额不匹配
  - `403`: 发送方账户没有足够的权限

**本文档中引用的文件**
- [bank/v1beta1/tx.proto](file://proto/cosmos/bank/v1beta1/tx.proto#L20-L21)

## 质押模块 (staking)

### 查询服务 (Query Service)
质押模块的查询服务提供验证者、委托和质押池信息的查询功能。

#### Validators
查询所有匹配给定状态的验证者。

- **HTTP方法**: GET
- **URL路径**: `/cosmos/staking/v1beta1/validators`
- **请求参数**:
  - `status` (查询参数, string): 验证者状态 (active, bonded, jailed等)
  - `pagination.key` (查询参数, string): 分页键
  - `pagination.offset` (查询参数, string): 偏移量
  - `pagination.limit` (查询参数, string): 限制数量
  - `pagination.count_total` (查询参数, boolean): 是否计算总数
  - `pagination.reverse` (查询参数, boolean): 是否反向排序
- **请求体**: 无
- **响应状态码**:
  - `200 OK`: 成功获取验证者列表
  - `400 Bad Request`: 请求参数无效
  - `500 Internal Server Error`: 服务器内部错误
- **响应体JSON Schema**:
```json
{
  "validators": [
    {
      "operator_address": "string",
      "consensus_pubkey": {
        "@type": "string",
        "value": "string"
      },
      "jailed": "boolean",
      "status": "string",
      "tokens": "string",
      "delegator_shares": "string",
      "description": {
        "moniker": "string",
        "identity": "string",
        "website": "string",
        "security_contact": "string",
        "details": "string"
      },
      "unbonding_time": "string",
      "commission": {
        "commission_rates": {
          "rate": "string",
          "max_rate": "string",
          "max_change_rate": "string"
        },
        "update_time": "string"
      },
      "min_self_delegation": "string"
    }
  ],
  "pagination": {
    "next_key": "string",
    "total": "string"
  }
}
```
- **错误码**: 无特定错误码

**本文档中引用的文件**
- [staking/v1beta1/query.proto](file://proto/cosmos/staking/v1beta1/query.proto#L16-L23)

#### Validator
根据验证者地址查询验证者信息。

- **HTTP方法**: GET
- **URL路径**: `/cosmos/staking/v1beta1/validators/{validator_addr}`
- **请求参数**:
  - `validator_addr` (路径参数, string): 验证者地址
- **请求体**: 无
- **响应状态码**:
  - `200 OK`: 成功获取验证者信息
  - `400 Bad Request`: 验证者地址无效
  - `404 Not Found`: 验证者不存在
  - `500 Internal Server Error`: 服务器内部错误
- **响应体JSON Schema**:
```json
{
  "validator": {
    "operator_address": "string",
    "consensus_pubkey": {
      "@type": "string",
      "value": "string"
    },
    "jailed": "boolean",
    "status": "string",
    "tokens": "string",
    "delegator_shares": "string",
    "description": {
      "moniker": "string",
      "identity": "string",
      "website": "string",
      "security_contact": "string",
      "details": "string"
    },
    "unbonding_time": "string",
    "commission": {
      "commission_rates": {
        "rate": "string",
        "max_rate": "string",
        "max_change_rate": "string"
      },
      "update_time": "string"
    },
    "min_self_delegation": "string"
  }
}
```
- **错误码**: 无特定错误码

**本文档中引用的文件**
- [staking/v1beta1/query.proto](file://proto/cosmos/staking/v1beta1/query.proto#L25-L29)

#### Delegation
查询验证者-委托人对的委托信息。

- **HTTP方法**: GET
- **URL路径**: `/cosmos/staking/v1beta1/validators/{validator_addr}/delegations/{delegator_addr}`
- **请求参数**:
  - `validator_addr` (路径参数, string): 验证者地址
  - `delegator_addr` (路径参数, string): 委托人地址
- **请求体**: 无
- **响应状态码**:
  - `200 OK`: 成功获取委托信息
  - `400 Bad Request`: 地址格式无效
  - `404 Not Found`: 委托关系不存在
  - `500 Internal Server Error`: 服务器内部错误
- **响应体JSON Schema**:
```json
{
  "delegation_response": {
    "delegation": {
      "delegator_address": "string",
      "validator_address": "string",
      "shares": "string"
    },
    "balance": {
      "denom": "string",
      "amount": "string"
    }
  }
}
```
- **错误码**: 无特定错误码

**本文档中引用的文件**
- [staking/v1beta1/query.proto](file://proto/cosmos/staking/v1beta1/query.proto#L51-L56)

#### Pool
查询质押池信息。

- **HTTP方法**: GET
- **URL路径**: `/cosmos/staking/v1beta1/pool`
- **请求参数**: 无
- **请求体**: 无
- **响应状态码**:
  - `200 OK`: 成功获取质押池信息
  - `500 Internal Server Error`: 服务器内部错误
- **响应体JSON Schema**:
```json
{
  "pool": {
    "not_bonded_tokens": "string",
    "bonded_tokens": "string"
  }
}
```
- **错误码**: 无特定错误码

**本文档中引用的文件**
- [staking/v1beta1/query.proto](file://proto/cosmos/staking/v1beta1/query.proto#L120-L124)

## 分发模块 (distribution)

### 查询服务 (Query Service)
分发模块的查询服务提供验证者奖励、委托人奖励和社区池的查询功能。

#### DelegationRewards
查询委托人因特定验证者而累积的总奖励。

- **HTTP方法**: GET
- **URL路径**: `/cosmos/distribution/v1beta1/delegators/{delegator_address}/rewards/{validator_address}`
- **请求参数**:
  - `delegator_address` (路径参数, string): 委托人地址
  - `validator_address` (路径参数, string): 验证者地址
- **请求体**: 无
- **响应状态码**:
  - `200 OK`: 成功获取委托奖励
  - `400 Bad Request`: 地址格式无效
  - `404 Not Found`: 委托关系不存在
  - `500 Internal Server Error`: 服务器内部错误
- **响应体JSON Schema**:
```json
{
  "rewards": [
    {
      "denom": "string",
      "amount": "string"
    }
  ]
}
```
- **错误码**: 无特定错误码

**本文档中引用的文件**
- [distribution/v1beta1/query.proto](file://proto/cosmos/distribution/v1beta1/query.proto#L45-L49)

#### DelegationTotalRewards
查询每个验证者的总奖励。

- **HTTP方法**: GET
- **URL路径**: `/cosmos/distribution/v1beta1/delegators/{delegator_address}/rewards`
- **请求参数**:
  - `delegator_address` (路径参数, string): 委托人地址
- **请求体**: 无
- **响应状态码**:
  - `200 OK`: 成功获取总奖励
  - `400 Bad Request`: 委托人地址无效
  - `500 Internal Server Error`: 服务器内部错误
- **响应体JSON Schema**:
```json
{
  "rewards": [
    {
      "validator_address": "string",
      "reward": [
        {
          "denom": "string",
          "amount": "string"
        }
      ]
    }
  ],
  "total": [
    {
      "denom": "string",
      "amount": "string"
    }
  ]
}
```
- **错误码**: 无特定错误码

**本文档中引用的文件**
- [distribution/v1beta1/query.proto](file://proto/cosmos/distribution/v1beta1/query.proto#L51-L55)

#### CommunityPool
查询社区池的代币。

- **HTTP方法**: GET
- **URL路径**: `/cosmos/distribution/v1beta1/community_pool`
- **请求参数**: 无
- **请求体**: 无
- **响应状态码**:
  - `200 OK`: 成功获取社区池信息
  - `500 Internal Server Error`: 服务器内部错误
- **响应体JSON Schema**:
```json
{
  "pool": [
    {
      "denom": "string",
      "amount": "string"
    }
  ]
}
```
- **错误码**: 无特定错误码

**本文档中引用的文件**
- [distribution/v1beta1/query.proto](file://proto/cosmos/distribution/v1beta1/query.proto#L69-L74)

## 治理模块 (gov)

### 查询服务 (Query Service)
治理模块的查询服务提供提案、投票和治理参数的查询功能。

#### Proposal
根据提案ID查询提案详情。

- **HTTP方法**: GET
- **URL路径**: `/cosmos/gov/v1/proposals/{proposal_id}`
- **请求参数**:
  - `proposal_id` (路径参数, string): 提案ID
- **请求体**: 无
- **响应状态码**:
  - `200 OK`: 成功获取提案信息
  - `400 Bad Request`: 提案ID无效
  - `404 Not Found`: 提案不存在
  - `500 Internal Server Error`: 服务器内部错误
- **响应体JSON Schema**:
```json
{
  "proposal": {
    "id": "string",
    "messages": [
      {
        "@type": "string",
        "value": "object"
      }
    ],
    "status": "string",
    "final_tally_result": {
      "yes_count": "string",
      "abstain_count": "string",
      "no_count": "string",
      "no_with_veto_count": "string"
    },
    "submit_time": "string",
    "deposit_end_time": "string",
    "total_deposit": [
      {
        "denom": "string",
        "amount": "string"
      }
    ],
    "voting_start_time": "string",
    "voting_end_time": "string",
    "metadata": "string",
    "title": "string",
    "summary": "string",
    "proposer": "string"
  }
}
```
- **错误码**: 无特定错误码

**本文档中引用的文件**
- [gov/v1/query.proto](file://proto/cosmos/gov/v1/query.proto#L18-L21)

#### Proposals
根据状态查询所有提案。

- **HTTP方法**: GET
- **URL路径**: `/cosmos/gov/v1/proposals`
- **请求参数**:
  - `proposal_status` (查询参数, string): 提案状态 (deposit_period, voting_period, passed, rejected等)
  - `voter` (查询参数, string): 投票人地址
  - `depositor` (查询参数, string): 存款人地址
  - `pagination.key` (查询参数, string): 分页键
  - `pagination.offset` (查询参数, string): 偏移量
  - `pagination.limit` (查询参数, string): 限制数量
  - `pagination.count_total` (查询参数, boolean): 是否计算总数
  - `pagination.reverse` (查询参数, boolean): 是否反向排序
- **请求体**: 无
- **响应状态码**:
  - `200 OK`: 成功获取提案列表
  - `400 Bad Request`: 请求参数无效
  - `500 Internal Server Error`: 服务器内部错误
- **响应体JSON Schema**:
```json
{
  "proposals": [
    {
      "id": "string",
      "messages": [
        {
          "@type": "string",
          "value": "object"
        }
      ],
      "status": "string",
      "final_tally_result": {
        "yes_count": "string",
        "abstain_count": "string",
        "no_count": "string",
        "no_with_veto_count": "string"
      },
      "submit_time": "string",
      "deposit_end_time": "string",
      "total_deposit": [
        {
          "denom": "string",
          "amount": "string"
        }
      ],
      "voting_start_time": "string",
      "voting_end_time": "string",
      "metadata": "string",
      "title": "string",
      "summary": "string",
      "proposer": "string"
    }
  ],
  "pagination": {
    "next_key": "string",
    "total": "string"
  }
}
```
- **错误码**: 无特定错误码

**本文档中引用的文件**
- [gov/v1/query.proto](file://proto/cosmos/gov/v1/query.proto#L23-L26)

#### TallyResult
查询提案投票的计票结果。

- **HTTP方法**: GET
- **URL路径**: `/cosmos/gov/v1/proposals/{proposal_id}/tally`
- **请求参数**:
  - `proposal_id` (路径参数, string): 提案ID
- **请求体**: 无
- **响应状态码**:
  - `200 OK`: 成功获取计票结果
  - `400 Bad Request`: 提案ID无效
  - `404 Not Found`: 提案不存在
  - `500 Internal Server Error`: 服务器内部错误
- **响应体JSON Schema**:
```json
{
  "tally": {
    "yes_count": "string",
    "abstain_count": "string",
    "no_count": "string",
    "no_with_veto_count": "string"
  }
}
```
- **错误码**: 无特定错误码

**本文档中引用的文件**
- [gov/v1/query.proto](file://proto/cosmos/gov/v1/query.proto#L53-L56)

### 事务服务 (Tx Service)
治理模块的事务服务提供提交提案、投票和存款功能。

#### SubmitProposal
提交包含任意消息的新提案。

- **HTTP方法**: POST
- **URL路径**: `/cosmos/gov/v1/tx/submit_proposal`
- **请求参数**: 无
- **请求体JSON Schema**:
```json
{
  "type": "object",
  "properties": {
    "messages": {
      "type": "array",
      "items": {
        "@type": "string",
        "value": "object"
      }
    },
    "initial_deposit": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "denom": "string",
          "amount": "string"
        },
        "required": ["denom", "amount"]
      }
    },
    "proposer": {
      "type": "string"
    },
    "metadata": {
      "type": "string"
    },
    "title": {
      "type": "string"
    },
    "summary": {
      "type": "string"
    },
    "expedited": {
      "type": "boolean"
    }
  },
  "required": ["messages", "initial_deposit", "proposer"]
}
```
- **响应状态码**:
  - `200 OK`: 提案提交成功
  - `400 Bad Request`: 请求体格式无效
  - `401 Unauthorized`: 签名验证失败
  - `403 Forbidden`: 存款不足或权限不足
  - `500 Internal Server Error`: 服务器内部错误
- **响应体JSON Schema**:
```json
{
  "height": "string",
  "txhash": "string",
  "codespace": "string",
  "code": "integer",
  "data": "string",
  "raw_log": "string",
  "logs": [
    {
      "msg_index": "integer",
      "log": "string",
      "events": [
        {
          "type": "string",
          "attributes": [
            {
              "key": "string",
              "value": "string"
            }
          ]
        }
      ]
    }
  ],
  "info": "string",
  "gas_wanted": "string",
  "gas_used": "string",
  "tx": {
    "@type": "string",
    "value": "object"
  },
  "timestamp": "string"
}
```
- **错误码**:
  - `403`: 初始存款不足，无法提交提案
  - `403`: 提案人账户没有足够的权限

**本文档中引用的文件**
- [gov/v1/tx.proto](file://proto/cosmos/gov/v1/tx.proto#L19-L22)

#### Vote
对特定提案投赞成、反对、弃权或否决票。

- **HTTP方法**: POST
- **URL路径**: `/cosmos/gov/v1/tx/vote`
- **请求参数**: 无
- **请求体JSON Schema**:
```json
{
  "type": "object",
  "properties": {
    "proposal_id": {
      "type": "string"
    },
    "voter": {
      "type": "string"
    },
    "option": {
      "type": "string",
      "enum": ["VOTE_OPTION_UNSPECIFIED", "VOTE_OPTION_YES", "VOTE_OPTION_ABSTAIN", "VOTE_OPTION_NO", "VOTE_OPTION_NO_WITH_VETO"]
    },
    "metadata": {
      "type": "string"
    }
  },
  "required": ["proposal_id", "voter", "option"]
}
```
- **响应状态码**:
  - `200 OK`: 投票成功
  - `400 Bad Request`: 请求体格式无效
  - `401 Unauthorized`: 签名验证失败
  - `403 Forbidden`: 无投票权或已投票
  - `500 Internal Server Error`: 服务器内部错误
- **响应体JSON Schema**:
```json
{
  "height": "string",
  "txhash": "string",
  "codespace": "string",
  "code": "integer",
  "data": "string",
  "raw_log": "string",
  "logs": [
    {
      "msg_index": "integer",
      "log": "string",
      "events": [
        {
          "type": "string",
          "attributes": [
            {
              "key": "string",
              "value": "string"
            }
          ]
        }
      ]
    }
  ],
  "info": "string",
  "gas_wanted": "string",
  "gas_used": "string",
  "tx": {
    "@type": "string",
    "value": "object"
  },
  "timestamp": "string"
}
```
- **错误码**:
  - `403`: 提案不在投票期，无法投票
  - `403`: 投票人已经对提案投过票

**本文档中引用的文件**
- [gov/v1/tx.proto](file://proto/cosmos/gov/v1/tx.proto#L30-L33)

## 交易服务 (tx)
交易服务提供交易模拟、广播和查询功能。

### Simulate
模拟执行交易以估算gas使用量。

- **HTTP方法**: POST
- **URL路径**: `/cosmos/tx/v1beta1/simulate`
- **请求参数**: 无
- **请求体JSON Schema**:
```json
{
  "type": "object",
  "properties": {
    "tx_bytes": {
      "type": "string",
      "format": "byte"
    }
  },
  "required": ["tx_bytes"]
}
```
- **响应状态码**:
  - `200 OK`: 模拟成功
  - `400 Bad Request`: 交易字节无效
  - `500 Internal Server Error`: 服务器内部错误
- **响应体JSON Schema**:
```json
{
  "gas_info": {
    "gas_wanted": "string",
    "gas_used": "string"
  },
  "result": {
    "data": "string",
    "log": "string",
    "events": [
      {
        "type": "string",
        "attributes": [
          {
            "key": "string",
            "value": "string"
          }
        ]
      }
    ],
    "msg_responses": [
      {
        "@type": "string",
        "value": "object"
      }
    ]
  }
}
```
- **错误码**: 无特定错误码

**本文档中引用的文件**
- [tx/v1beta1/service.proto](file://proto/cosmos/tx/v1beta1/service.proto#L16-L22)

### BroadcastTx
广播交易。

- **HTTP方法**: POST
- **URL路径**: `/cosmos/tx/v1beta1/txs`
- **请求参数**: 无
- **请求体JSON Schema**:
```json
{
  "type": "object",
  "properties": {
    "tx_bytes": {
      "type": "string",
      "format": "byte"
    },
    "mode": {
      "type": "string",
      "enum": ["BROADCAST_MODE_UNSPECIFIED", "BROADCAST_MODE_SYNC", "BROADCAST_MODE_ASYNC"]
    }
  },
  "required": ["tx_bytes"]
}
```
- **响应状态码**:
  - `200 OK`: 交易广播成功
  - `400 Bad Request`: 交易字节无效
  - `500 Internal Server Error`: 服务器内部错误
- **响应体JSON Schema**:
```json
{
  "tx_response": {
    "height": "string",
    "txhash": "string",
    "codespace": "string",
    "code": "integer",
    "data": "string",
    "raw_log": "string",
    "logs": [
      {
        "msg_index": "integer",
        "log": "string",
        "events": [
          {
            "type": "string",
            "attributes": [
              {
                "key": "string",
                "value": "string"
              }
            ]
          }
        ]
      }
    ],
    "info": "string",
    "gas_wanted": "string",
    "gas_used": "string",
    "tx": {
      "@type": "string",
      "value": "object"
    },
    "timestamp": "string"
  }
}
```
- **错误码**: 无特定错误码

**本文档中引用的文件**
- [tx/v1beta1/service.proto](file://proto/cosmos/tx/v1beta1/service.proto#L27-L33)

### GetTx
根据哈希获取交易。

- **HTTP方法**: GET
- **URL路径**: `/cosmos/tx/v1beta1/txs/{hash}`
- **请求参数**:
  - `hash` (路径参数, string): 交易哈希
- **请求体**: 无
- **响应状态码**:
  - `200 OK`: 成功获取交易
  - `400 Bad Request`: 哈希格式无效
  - `404 Not Found`: 交易不存在
  - `500 Internal Server Error`: 服务器内部错误
- **响应体JSON Schema**:
```json
{
  "tx": {
    "@type": "string",
    "value": "object"
  },
  "tx_response": {
    "height": "string",
    "txhash": "string",
    "codespace": "string",
    "code": "integer",
    "data": "string",
    "raw_log": "string",
    "logs": [
      {
        "msg_index": "integer",
        "log": "string",
        "events": [
          {
            "type": "string",
            "attributes": [
              {
                "key": "string",
                "value": "string"
              }
            ]
          }
        ]
      }
    ],
    "info": "string",
    "gas_wanted": "string",
    "gas_used": "string",
    "tx": {
      "@type": "string",
      "value": "object"
    },
    "timestamp": "string"
  }
}
```
- **错误码**: 无特定错误码

**本文档中引用的文件**
- [tx/v1beta1/service.proto](file://proto/cosmos/tx/v1beta1/service.proto#L23-L26)

## 节点服务 (node)
节点服务提供节点配置和状态的查询功能。

### Status
查询节点状态。

- **HTTP方法**: GET
- **URL路径**: `/cosmos/base/node/v1beta1/status`
- **请求参数**: 无
- **请求体**: 无
- **响应状态码**:
  - `200 OK`: 成功获取节点状态
  - `500 Internal Server Error`: 服务器内部错误
- **响应体JSON Schema**:
```json
{
  "earliest_store_height": "string",
  "height": "string",
  "timestamp": "string",
  "app_hash": "string",
  "validator_hash": "string"
}
```
- **错误码**: 无特定错误码

**本文档中引用的文件**
- [base/node/v1beta1/query.proto](file://proto/cosmos/base/node/v1beta1/query.proto#L16-L19)

## gRPC-Gateway转换机制
gRPC-Gateway是一个gRPC到JSON/HTTP的转换器，它允许通过HTTP/JSON访问gRPC服务。在Cosmos SDK中，gRPC-Gateway通过以下机制将gRPC调用转换为HTTP/JSON：

1. **Protobuf注解**: 每个gRPC服务方法都使用`google.api.http`注解来定义对应的HTTP映射。例如，在`auth/v1beta1/query.proto`中，`Account`方法的注解为：
```protobuf
rpc Account(QueryAccountRequest) returns (QueryAccountResponse) {
  option (google.api.http).get = "/cosmos/auth/v1beta1/accounts/{address}";
}
```

2. **路径参数映射**: Protobuf消息中的字段被映射到URL路径参数。例如，`QueryAccountRequest`消息中的`address`字段被映射到URL路径中的`{address}`占位符。

3. **查询参数映射**: 对于GET请求，消息中的其他字段被映射到查询参数。例如，分页参数被映射到`pagination.key`、`pagination.offset`等查询参数。

4. **请求体映射**: 对于POST请求，整个Protobuf消息被编码为JSON并作为请求体发送。

5. **响应转换**: gRPC响应被自动转换为JSON格式返回给客户端。

6. **错误处理**: gRPC错误状态码被转换为相应的HTTP状态码。例如，gRPC的`InvalidArgument`错误被转换为HTTP 400错误。

这种机制使得开发者可以使用熟悉的HTTP/JSON接口来与Cosmos链交互，同时保留了gRPC的类型安全和性能优势。

**本文档中引用的文件**
- [auth/v1beta1/query.proto](file://proto/cosmos/auth/v1beta1/query.proto)
- [bank/v1beta1/query.proto](file://proto/cosmos/bank/v1beta1/query.proto)
- [staking/v1beta1/query.proto](file://proto/cosmos/staking/v1beta1/query.proto)

## 身份验证机制
Cosmos SDK中的身份验证机制基于交易签名，确保只有授权用户才能执行敏感操作。主要机制如下：

1. **交易签名**: 所有事务操作（Tx Service）都需要交易签名。签名使用发送方的私钥对交易内容进行加密，证明发送方的身份和授权。

2. **签名验证**: 节点在处理交易前会验证签名的有效性。验证过程包括：
   - 使用发送方的公钥解密签名
   - 重新计算交易内容的哈希
   - 比较解密结果和计算的哈希是否一致

3. **多签支持**: 系统支持多签账户，需要多个签名才能授权交易。

4. **授权机制**: `authz`模块允许账户授权其他账户代表其执行特定操作，而无需共享私钥。

5. **费用支付**: 交易发送方需要支付gas费用，这通过从发送方账户扣除相应代币实现，进一步确保了交易的授权性。

6. **防重放攻击**: 每个交易包含一个序列号（sequence number），防止交易被重复提交。

这种基于签名的身份验证机制确保了系统的安全性和去中心化特性，是Cosmos生态中所有操作的基础。

**本文档中引用的文件**
- [auth/v1beta1/tx.proto](file://proto/cosmos/auth/v1beta1/tx.proto)
- [bank/v1beta1/tx.proto](file://proto/cosmos/bank/v1beta1/tx.proto)
- [gov/v1/tx.proto](file://proto/cosmos/gov/v1/tx.proto)