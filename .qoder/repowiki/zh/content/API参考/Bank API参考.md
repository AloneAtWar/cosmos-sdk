# Bank API参考

<cite>
**本文档中引用的文件**
- [query.proto](file://proto/cosmos/bank/v1beta1/query.proto)
- [tx.proto](file://proto/cosmos/bank/v1beta1/tx.proto)
- [bank.proto](file://proto/cosmos/bank/v1beta1/bank.proto)
- [coin.go](file://types/coin.go)
- [dec_coin.go](file://types/dec_coin.go)
- [pagination.go](file://types/query/pagination.go)
- [grpc.go](file://tests/e2e/bank/grpc.go)
- [pagination_test.go](file://types/query/pagination_test.go)
</cite>

## 目录
1. [简介](#简介)
2. [查询服务API](#查询服务api)
3. [交易服务API](#交易服务api)
4. [数据类型定义](#数据类型定义)
5. [分页查询](#分页查询)
6. [gRPC流式响应](#grpc流式响应)
7. [完整示例](#完整示例)
8. [错误处理](#错误处理)

## 简介

Cosmos SDK的Bank模块提供了完整的代币管理和转账功能。该模块支持多种查询操作（如余额查询、总供应量查询）和交易操作（如发送代币、多重发送）。Bank模块采用gRPC协议提供高性能的API接口，同时支持RESTful API调用。

## 查询服务API

Bank模块提供以下查询服务端点：

### 平衡查询

#### 1. 单个代币余额查询

**HTTP方法**: `GET`  
**路径**: `/cosmos/bank/v1beta1/balances/{address}/by_denom`  
**gRPC方法**: `Balance`

| 参数 | 类型 | 描述 |
|------|------|------|
| `address` | string | 账户地址 |
| `denom` | string | 代币标识符 |

**请求示例**:
```bash
curl -X GET \
  'http://localhost:1317/cosmos/bank/v1beta1/balances/cosmos1qyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyq/btc' \
  -H 'Accept: application/json'
```

**响应结构**:
```json
{
  "balance": {
    "denom": "btc",
    "amount": "1000000"
  }
}
```

#### 2. 所有代币余额查询

**HTTP方法**: `GET`  
**路径**: `/cosmos/bank/v1beta1/balances/{address}`  
**gRPC方法**: `AllBalances`

| 参数 | 类型 | 描述 |
|------|------|------|
| `address` | string | 账户地址 |
| `pagination` | PageRequest | 分页参数 |
| `resolve_denom` | bool | 是否解析代币为人类可读形式 |

**请求示例**:
```bash
curl -X GET \
  'http://localhost:1317/cosmos/bank/v1beta1/balances/cosmos1qyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyq?pagination.limit=10' \
  -H 'Accept: application/json'
```

**响应结构**:
```json
{
  "balances": [
    {
      "denom": "btc",
      "amount": "1000000"
    },
    {
      "denom": "eth",
      "amount": "500000"
    }
  ],
  "pagination": {
    "next_key": "Y29zbW9zMTJhZGQ=",
    "total": "2"
  }
}
```

### 总供应量查询

#### 3. 总供应量查询

**HTTP方法**: `GET`  
**路径**: `/cosmos/bank/v1beta1/supply`  
**gRPC方法**: `TotalSupply`

| 参数 | 类型 | 描述 |
|------|------|------|
| `pagination` | PageRequest | 分页参数 |

**请求示例**:
```bash
curl -X GET \
  'http://localhost:1317/cosmos/bank/v1beta1/supply?pagination.limit=5' \
  -H 'Accept: application/json'
```

**响应结构**:
```json
{
  "supply": [
    {
      "denom": "btc",
      "amount": "1000000000"
    },
    {
      "denom": "eth",
      "amount": "500000000"
    }
  ],
  "pagination": {
    "next_key": "",
    "total": "2"
  }
}
```

#### 4. 单一代币供应量查询

**HTTP方法**: `GET`  
**路径**: `/cosmos/bank/v1beta1/supply/by_denom`  
**gRPC方法**: `SupplyOf`

| 参数 | 类型 | 描述 |
|------|------|------|
| `denom` | string | 代币标识符 |

**请求示例**:
```bash
curl -X GET \
  'http://localhost:1317/cosmos/bank/v1beta1/supply/by_denom?denom=btc' \
  -H 'Accept: application/json'
```

**响应结构**:
```json
{
  "amount": {
    "denom": "btc",
    "amount": "1000000000"
  }
}
```

### 可花费余额查询

#### 5. 可花费余额查询

**HTTP方法**: `GET`  
**路径**: `/cosmos/bank/v1beta1/spendable_balances/{address}`  
**gRPC方法**: `SpendableBalances`

| 参数 | 类型 | 描述 |
|------|------|------|
| `address` | string | 账户地址 |
| `pagination` | PageRequest | 分页参数 |

**请求示例**:
```bash
curl -X GET \
  'http://localhost:1317/cosmos/bank/v1beta1/spendable_balances/cosmos1qyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyq?pagination.limit=10' \
  -H 'Accept: application/json'
```

#### 6. 单一代币可花费余额查询

**HTTP方法**: `GET`  
**路径**: `/cosmos/bank/v1beta1/spendable_balances/{address}/by_denom`  
**gRPC方法**: `SpendableBalanceByDenom`

| 参数 | 类型 | 描述 |
|------|------|------|
| `address` | string | 账户地址 |
| `denom` | string | 代币标识符 |

**请求示例**:
```bash
curl -X GET \
  'http://localhost:1317/cosmos/bank/v1beta1/spendable_balances/cosmos1qyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyq/by_denom?denom=btc' \
  -H 'Accept: application/json'
```

### 其他查询

#### 7. 链参数查询

**HTTP方法**: `GET`  
**路径**: `/cosmos/bank/v1beta1/params`  
**gRPC方法**: `Params`

**请求示例**:
```bash
curl -X GET \
  'http://localhost:1317/cosmos/bank/v1beta1/params' \
  -H 'Accept: application/json'
```

#### 8. 代币元数据查询

**HTTP方法**: `GET`  
**路径**: `/cosmos/bank/v1beta1/denoms_metadata`  
**gRPC方法**: `DenomsMetadata`

**请求示例**:
```bash
curl -X GET \
  'http://localhost:1317/cosmos/bank/v1beta1/denoms_metadata?pagination.limit=10' \
  -H 'Accept: application/json'
```

#### 9. 单一代币所有者查询

**HTTP方法**: `GET`  
**路径**: `/cosmos/bank/v1beta1/denom_owners/{denom}`  
**gRPC方法**: `DenomOwners`

| 参数 | 类型 | 描述 |
|------|------|------|
| `denom` | string | 代币标识符 |
| `pagination` | PageRequest | 分页参数 |

**请求示例**:
```bash
curl -X GET \
  'http://localhost:1317/cosmos/bank/v1beta1/denom_owners/btc?pagination.limit=10' \
  -H 'Accept: application/json'
```

**响应结构**:
```json
{
  "denom_owners": [
    {
      "address": "cosmos1qyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyq",
      "balance": {
        "denom": "btc",
        "amount": "1000000"
      }
    }
  ],
  "pagination": {
    "next_key": "",
    "total": "1"
  }
}
```

**Section sources**
- [query.proto](file://proto/cosmos/bank/v1beta1/query.proto#L16-L127)
- [grpc.go](file://tests/e2e/bank/grpc.go#L1-L283)

## 交易服务API

Bank模块提供以下交易服务端点：

### 发送代币交易

**HTTP方法**: `POST`  
**路径**: `/cosmos/bank/v1beta1/transactions/send`  
**gRPC方法**: `Send`

**请求结构**:
```json
{
  "from_address": "cosmos1qyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyq",
  "to_address": "cosmos1qyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyq",
  "amount": [
    {
      "denom": "btc",
      "amount": "1000"
    }
  ]
}
```

**响应结构**:
```json
{
  "tx_response": {
    "txhash": "A1B2C3D4E5F6...",
    "code": 0,
    "raw_log": "[]"
  }
}
```

### 多重发送交易

**HTTP方法**: `POST`  
**路径**: `/cosmos/bank/v1beta1/transactions/multisend`  
**gRPC方法**: `MultiSend`

**请求结构**:
```json
{
  "inputs": [
    {
      "address": "cosmos1qyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyq",
      "coins": [
        {
          "denom": "btc",
          "amount": "1000"
        }
      ]
    }
  ],
  "outputs": [
    {
      "address": "cosmos1qyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyq",
      "coins": [
        {
          "denom": "btc",
          "amount": "500"
        }
      ]
    },
    {
      "address": "cosmos1qyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyq",
      "coins": [
        {
          "denom": "btc",
          "amount": "500"
        }
      ]
    }
  ]
}
```

**响应结构**:
```json
{
  "tx_response": {
    "txhash": "A1B2C3D4E5F6...",
    "code": 0,
    "raw_log": "[]"
  }
}
```

**Section sources**
- [tx.proto](file://proto/cosmos/bank/v1beta1/tx.proto#L14-L36)

## 数据类型定义

### Coin类型

`Coin`是表示单个代币的基本类型，包含代币数量和标识符。

**JSON序列化格式**:
```json
{
  "denom": "btc",
  "amount": "1000000"
}
```

**字段说明**:
- `denom`: 代币标识符，必须符合验证规则
- `amount`: 代币数量，以整数字符串表示

### DecCoin类型

`DecCoin`是表示带小数位的代币类型。

**JSON序列化格式**:
```json
{
  "denom": "btc",
  "amount": "1000.500"
}
```

**字段说明**:
- `denom`: 代币标识符
- `amount`: 带小数位的代币数量，以字符串表示

### Coins类型

`Coins`是多个`Coin`的集合，按代币标识符排序。

**JSON序列化格式**:
```json
[
  {
    "denom": "btc",
    "amount": "1000000"
  },
  {
    "denom": "eth",
    "amount": "500000"
  }
]
```

### DecCoins类型

`DecCoins`是多个`DecCoin`的集合，按代币标识符排序。

**JSON序列化格式**:
```json
[
  {
    "denom": "btc",
    "amount": "1000.500"
  },
  {
    "denom": "eth",
    "amount": "50.250"
  }
]
```

**Section sources**
- [coin.go](file://types/coin.go#L1-L912)
- [dec_coin.go](file://types/dec_coin.go#L1-L679)

## 分页查询

Bank模块的所有列表查询都支持分页功能，包括余额查询、总供应量查询等。

### 分页参数

| 参数 | 类型 | 描述 | 默认值 |
|------|------|------|--------|
| `limit` | uint64 | 每页最大条目数 | 100 |
| `offset` | uint64 | 跳过的条目数 | 0 |
| `key` | bytes | 下一页的起始键 | nil |
| `count_total` | bool | 是否计算总数 | true |
| `reverse` | bool | 是否反向排序 | false |

### 分页示例

#### 基本分页
```bash
# 获取第一页（前10条）
curl -X GET \
  'http://localhost:1317/cosmos/bank/v1beta1/balances/cosmos1qyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyq?pagination.limit=10' \
  -H 'Accept: application/json'

# 获取第二页
curl -X GET \
  'http://localhost:1317/cosmos/bank/v1beta1/balances/cosmos1qyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyq?pagination.key=Y29zbW9zMTJhZGQ=&pagination.limit=10' \
  -H 'Accept: application/json'
```

#### 反向分页
```bash
# 最后一页（反向）
curl -X GET \
  'http://localhost:1317/cosmos/bank/v1beta1/balances/cosmos1qyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyq?pagination.reverse=true&pagination.limit=10' \
  -H 'Accept: application/json'
```

#### 计算总数
```bash
# 不计算总数以提高性能
curl -X GET \
  'http://localhost:1317/cosmos/bank/v1beta1/balances/cosmos1qyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyq?pagination.count_total=false' \
  -H 'Accept: application/json'
```

### 分页响应结构

```json
{
  "balances": [...],
  "pagination": {
    "next_key": "Y29zbW9zMTJhZGQ=",
    "total": "100"
  }
}
```

**字段说明**:
- `next_key`: 下一页的起始键，如果为空则表示已到达最后一页
- `total`: 总条目数，当`count_total`为true时返回

**Section sources**
- [pagination.go](file://types/query/pagination.go#L1-L161)
- [pagination_test.go](file://types/query/pagination_test.go#L87-L357)

## gRPC流式响应

Bank模块支持通过gRPC流式响应实时监控余额变化。

### 流式余额查询

```bash
# 使用gRPC客户端连接
grpcurl -plaintext localhost:9090 cosmos.bank.v1beta1.Query/SpendableBalances
```

### 实时余额监听

```javascript
// JavaScript示例：监听余额变化
const grpc = require('@grpc/grpc-js');
const protoLoader = require('@grpc/proto-loader');

const PROTO_PATH = './proto/cosmos/bank/v1beta1/query.proto';
const packageDefinition = protoLoader.loadSync(PROTO_PATH);
const protoDescriptor = grpc.loadPackageDefinition(packageDefinition);
const bank = protoDescriptor.cosmos.bank.v1beta1;

const client = new bank.Query('localhost:9090', grpc.credentials.createInsecure());

// 监听特定账户的余额变化
const call = client.SpendableBalances({
  address: 'cosmos1qyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyq',
  pagination: { limit: 100 }
});

call.on('data', function(response) {
  console.log('余额更新:', response);
});

call.on('end', function() {
  console.log('流结束');
});

call.on('status', function(status) {
  console.log('状态:', status);
});
```

### 流式响应优势

1. **实时性**: 立即响应余额变化
2. **高效性**: 减少轮询开销
3. **可靠性**: 连接断开后自动重连
4. **灵活性**: 支持多种过滤条件

## 完整示例

### 1. 查询账户余额

```bash
#!/bin/bash
# 查询账户所有余额

ACCOUNT_ADDRESS="cosmos1qyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyq"
API_URL="http://localhost:1317"

# 获取所有余额
response=$(curl -s "$API_URL/cosmos/bank/v1beta1/balances/$ACCOUNT_ADDRESS?pagination.limit=10")

# 解析响应
balances=$(echo $response | jq '.balances')
total=$(echo $response | jq '.pagination.total')

echo "账户地址: $ACCOUNT_ADDRESS"
echo "总余额数量: $total"
echo "余额详情:"
echo "$balances" | jq -r '.[] | "\(.denom): \(.amount)"'
```

### 2. 发送代币交易

```bash
#!/bin/bash
# 发送代币到另一个账户

FROM_ADDRESS="cosmos1qyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyq"
TO_ADDRESS="cosmos1qyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyq"
AMOUNT="1000btc"
FEE="10stake"

# 构建交易
tx_json=$(cat <<EOF
{
  "from_address": "$FROM_ADDRESS",
  "to_address": "$TO_ADDRESS",
  "amount": [
    {
      "denom": "btc",
      "amount": "1000"
    }
  ]
}
EOF
)

# 发送交易
response=$(curl -s -X POST \
  "$API_URL/cosmos/bank/v1beta1/transactions/send" \
  -H "Content-Type: application/json" \
  -d "$tx_json")

# 检查结果
tx_hash=$(echo $response | jq -r '.tx_response.txhash')
code=$(echo $response | jq -r '.tx_response.code')

if [ "$code" -eq 0 ]; then
  echo "交易成功！TX哈希: $tx_hash"
else
  echo "交易失败: $(echo $response | jq -r '.tx_response.raw_log')"
fi
```

### 3. 分页查询示例

```bash
#!/bin/bash
# 分页查询代币所有者

DENOM="btc"
API_URL="http://localhost:1317"
PAGE_SIZE=5
TOTAL_COUNT=0
NEXT_KEY=""

while true; do
  if [ -z "$NEXT_KEY" ]; then
    # 第一页
    response=$(curl -s "$API_URL/cosmos/bank/v1beta1/denom_owners/$DENOM?pagination.limit=$PAGE_SIZE&pagination.count_total=true")
  else
    # 后续页面
    response=$(curl -s "$API_URL/cosmos/bank/v1beta1/denom_owners/$DENOM?pagination.limit=$PAGE_SIZE&pagination.key=$NEXT_KEY")
  fi
  
  owners=$(echo $response | jq -r '.denom_owners[] | "\(.address): \(.balance.amount) \(.balance.denom)"')
  next_key=$(echo $response | jq -r '.pagination.next_key')
  total=$(echo $response | jq -r '.pagination.total')
  
  echo "所有者 ($((TOTAL_COUNT + 1))-$((TOTAL_COUNT + $(echo "$owners" | wc -l)))):"
  echo "$owners"
  echo ""
  
  TOTAL_COUNT=$((TOTAL_COUNT + $(echo "$owners" | wc -l)))
  
  if [ "$next_key" = "null" ] || [ "$TOTAL_COUNT" -ge "$total" ]; then
    break
  fi
  
  NEXT_KEY=$next_key
done

echo "总计找到 $TOTAL_COUNT 个所有者"
```

### 4. gRPC流式监听示例

```python
#!/usr/bin/env python3
# Python gRPC流式监听示例

import grpc
import json
from google.protobuf import json_format
from cosmos.bank.v1beta1 import query_pb2, query_pb2_grpc

def listen_balance_changes(address):
    """监听指定账户的余额变化"""
    
    # 创建gRPC连接
    channel = grpc.insecure_channel('localhost:9090')
    stub = query_pb2_grpc.QueryStub(channel)
    
    # 构建请求
    request = query_pb2.QuerySpendableBalancesRequest(
        address=address,
        pagination=query_pb2.PageRequest(limit=100)
    )
    
    try:
        # 发起流式调用
        print(f"开始监听账户 {address} 的余额变化...")
        for response in stub.SpendableBalances(request):
            print(f"\n时间: {datetime.now()}")
            print("余额变化:")
            
            # 格式化输出
            balances = []
            for coin in response.balances:
                balances.append({
                    'denom': coin.denom,
                    'amount': coin.amount
                })
            
            print(json.dumps(balances, indent=2))
            
    except grpc.RpcError as e:
        print(f"gRPC错误: {e}")
    finally:
        channel.close()

if __name__ == "__main__":
    account_address = "cosmos1qyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyq"
    listen_balance_changes(account_address)
```

## 错误处理

### 常见错误码

| 错误码 | 错误类型 | 描述 |
|--------|----------|------|
| 3 | InsufficientFunds | 余额不足 |
| 5 | InvalidAddress | 地址无效 |
| 7 | InvalidCoins | 代币数量无效 |
| 11 | Unauthorized | 权限不足 |
| 12 | NotFound | 资源未找到 |

### 错误响应格式

```json
{
  "code": 5,
  "message": "invalid address",
  "details": []
}
```

### 错误处理最佳实践

1. **输入验证**: 在发送请求前验证地址和金额格式
2. **重试机制**: 对于临时性错误实现指数退避重试
3. **超时设置**: 设置合理的请求超时时间
4. **日志记录**: 记录详细的错误信息用于调试
5. **用户友好提示**: 将技术错误转换为用户可理解的提示

### 示例错误处理

```bash
#!/bin/bash
# 带错误处理的余额查询

query_balance() {
    local address=$1
    local denom=$2
    
    response=$(curl -s -w "%{http_code}" -o /tmp/response.json \
        "http://localhost:1317/cosmos/bank/v1beta1/balances/$address/by_denom?denom=$denom")
    
    if [ "$response" -ne 200 ]; then
        echo "查询失败: HTTP $response"
        cat /tmp/response.json | jq -r '.message'
        rm -f /tmp/response.json
        return 1
    fi
    
    # 检查JSON响应
    if ! jq empty /tmp/response.json 2>/dev/null; then
        echo "响应格式错误"
        cat /tmp/response.json
        rm -f /tmp/response.json
        return 1
    fi
    
    # 提取余额
    amount=$(jq -r '.balance.amount // "0"' /tmp/response.json)
    rm -f /tmp/response.json
    
    echo "$amount"
    return 0
}

# 使用示例
balance=$(query_balance "cosmos1qyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyqszqgpqyq" "btc")
if [ $? -eq 0 ]; then
    echo "BTC余额: $balance"
else
    echo "无法获取余额"
fi
```

**Section sources**
- [grpc.go](file://tests/e2e/bank/grpc.go#L1-L283)
- [pagination_test.go](file://types/query/pagination_test.go#L87-L357)