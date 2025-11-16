# REST客户端

<cite>
**本文档中引用的文件**  
- [client/query.go](file://client/query.go)
- [client/grpc_query.go](file://client/grpc_query.go)
- [client/rpc/block.go](file://client/rpc/block.go)
- [client/rpc/tx.go](file://client/rpc/tx.go)
- [client/rpc/validators.go](file://client/rpc/validators.go)
- [server/api/server.go](file://server/api/server.go)
- [client/docs/config.json](file://client/docs/config.json)
- [client/docs/embed.go](file://client/docs/embed.go)
</cite>

## 目录
1. [简介](#简介)
2. [REST API路由设计](#rest-api路由设计)
3. [查询工具函数](#查询工具函数)
4. [gRPC查询转换](#grpc查询转换)
5. [区块数据查询](#区块数据查询)
6. [交易信息查询](#交易信息查询)
7. [验证者集查询](#验证者集查询)
8. [Swagger UI集成](#swagger-ui集成)
9. [使用示例](#使用示例)
10. [性能考虑与缓存策略](#性能考虑与缓存策略)

## 简介
Cosmos SDK提供了通过HTTP接口查询区块链数据的REST客户端功能。该功能允许开发者通过标准的HTTP请求来获取区块链的区块详情、交易信息、账户状态等数据。REST API基于gRPC-Gateway实现，将HTTP/JSON请求转换为底层的gRPC调用，从而提供了一种简单易用的区块链数据查询方式。

## REST API路由设计
Cosmos SDK的REST API路由设计遵循模块化原则，通过gRPC-Gateway将gRPC服务映射为HTTP RESTful接口。API服务器在`server/api/server.go`中实现，使用Gorilla Mux路由器处理HTTP请求。

API路由的注册在应用级别完成，通过`RegisterAPIRoutes`方法为所有模块注册相应的REST端点。例如，在`simapp/app.go`中可以看到：

```go
func (app *SimApp) RegisterAPIRoutes(apiSvr *api.Server, apiConfig config.APIConfig) {
    clientCtx := apiSvr.ClientCtx
    authtx.RegisterGRPCGatewayRoutes(clientCtx, apiSvr.GRPCGatewayRouter)
    cmtservice.RegisterGRPCGatewayRoutes(clientCtx, apiSvr.GRPCGatewayRouter)
    nodeservice.RegisterGRPCGatewayRoutes(clientCtx, apiSvr.GRPCGatewayRouter)
    app.BasicModuleManager.RegisterGRPCGatewayRoutes(clientCtx, apiSvr.GRPCGatewayRouter)
    
    if err := server.RegisterSwaggerAPI(apiSvr.ClientCtx, apiSvr.Router, apiConfig.Swagger); err != nil {
        panic(err)
    }
}
```

这种设计使得每个模块都可以独立定义自己的gRPC服务，并自动获得相应的REST接口，实现了良好的模块化和可扩展性。

**Section sources**
- [server/api/server.go](file://server/api/server.go#L31-L237)
- [simapp/app.go](file://simapp/app.go#L790-L854)
- [runtime/app.go](file://runtime/app.go#L190-L271)

## 查询工具函数
`client/query.go`文件提供了核心的查询工具函数，这些函数封装了与CometBFT节点的交互逻辑，为上层应用提供了简洁的查询接口。

主要查询函数包括：
- `Query`：通过路径向CometBFT节点发起查询
- `QueryWithData`：带数据负载的查询
- `QueryStore`：针对特定存储的查询
- `QueryABCI`：直接执行ABCI查询

这些函数通过`Context`结构体中的`Client`字段与CometBFT节点通信，实现了从应用层到共识层的查询通道。查询结果经过验证后返回，确保了数据的可靠性。

```go
func (ctx Context) Query(path string) ([]byte, int64, error) {
    return ctx.query(path, nil)
}

func (ctx Context) QueryWithData(path string, data []byte) ([]byte, int64, error) {
    return ctx.query(path, data)
}
```

**Section sources**
- [client/query.go](file://client/query.go#L1-L171)

## gRPC查询转换
`client/grpc_query.go`文件实现了将REST请求转换为底层gRPC调用的核心逻辑。`Context`结构体实现了gRPC的`ClientConn`接口，通过`Invoke`方法处理gRPC调用。

当收到查询请求时，系统会根据配置决定是直接调用gRPC客户端还是通过ABCI查询。如果配置了gRPC客户端，则直接转发请求；否则，将请求序列化后通过ABCI接口查询。

```go
func (ctx Context) Invoke(grpcCtx gocontext.Context, method string, req, reply any, opts ...grpc.CallOption) (err error) {
    if reqProto, ok := req.(*tx.BroadcastTxRequest); ok {
        // 广播交易
        broadcastRes, err := TxServiceBroadcast(grpcCtx, ctx, reqProto)
        // ...
        return err
    }

    if ctx.GRPCClient != nil {
        // 调用gRPC
        return ctx.GRPCClient.Invoke(grpcCtx, method, req, reply, opts...)
    }

    // 通过ABCI查询
    reqBz, err := ctx.gRPCCodec().Marshal(req)
    // ...
    abciReq := abci.RequestQuery{
        Path:   method,
        Data:   reqBz,
        Height: ctx.Height,
    }
    
    res, err := ctx.QueryABCI(abciReq)
    // ...
    return ctx.gRPCCodec().Unmarshal(res.Value, reply)
}
```

这种设计实现了REST API与gRPC服务的无缝集成，既保持了gRPC的高效性，又提供了REST的易用性。

**Section sources**
- [client/grpc_query.go](file://client/grpc_query.go#L1-L143)

## 区块数据查询
`client/rpc/block.go`文件提供了查询区块链数据的具体实现，包括获取链高度、查询区块和格式化区块结果等功能。

### 获取链高度
`GetChainHeight`函数通过CometBFT的RPC接口获取当前区块链的高度：

```go
func GetChainHeight(clientCtx client.Context) (int64, error) {
    node, err := clientCtx.GetNode()
    if err != nil {
        return -1, err
    }

    status, err := node.Status(clientCtx.GetCmdContextWithFallback())
    if err != nil {
        return -1, err
    }

    height := status.SyncInfo.LatestBlockHeight
    return height, nil
}
```

### 查询区块
`QueryBlocks`函数支持通过自定义查询条件搜索区块，查询语法基于CometBFT的事件系统：

```go
func QueryBlocks(clientCtx client.Context, page, limit int, query, orderBy string) (*sdk.SearchBlocksResult, error) {
    node, err := clientCtx.GetNode()
    if err != nil {
        return nil, err
    }

    resBlocks, err := node.BlockSearch(clientCtx.GetCmdContextWithFallback(), query, &page, &limit, orderBy)
    if err != nil {
        return nil, err
    }

    blocks, err := formatBlockResults(resBlocks.Blocks)
    if err != nil {
        return nil, err
    }

    result := sdk.NewSearchBlocksResult(int64(resBlocks.TotalCount), int64(len(blocks)), int64(page), int64(limit), blocks)
    return result, nil
}
```

查询语法支持多种条件组合，例如：
- `tm.event = 'NewBlock'`：查询新区块事件
- `tm.event = 'Tx' AND tx.hash = 'XYZ'`：查询特定交易
- `tx.height = 5`：查询第五个区块的所有交易

**Section sources**
- [client/rpc/block.go](file://client/rpc/block.go#L1-L135)

## 交易信息查询
`client/rpc/tx.go`文件提供了交易相关的查询功能，包括等待交易确认和解析交易响应等。

### 等待交易确认
`WaitTxCmd`命令允许客户端等待特定交易被包含在区块中，这对于需要确认交易状态的应用场景非常有用：

```go
func WaitTxCmd() *cobra.Command {
    cmd := &cobra.Command{
        Use:     "wait-tx [hash]",
        Short:   "等待交易被包含在区块中",
        Long:    `订阅CometBFT WebSocket连接并等待具有给定哈希的交易事件。`,
        Args: cobra.MaximumNArgs(1),
        RunE: func(cmd *cobra.Command, args []string) error {
            clientCtx, err := client.GetClientTxContext(cmd)
            if err != nil {
                return err
            }

            timeout, err := cmd.Flags().GetDuration(TimeoutFlag)
            if err != nil {
                return err
            }

            c, err := rpchttp.New(clientCtx.NodeURI, "/websocket")
            if err != nil {
                return err
            }
            if err := c.Start(); err != nil {
                return err
            }
            defer c.Stop()

            ctx, cancel := context.WithTimeout(context.Background(), timeout)
            defer cancel()

            // 订阅交易事件
            query := fmt.Sprintf("%s='%s' AND %s='%X'", tmtypes.EventTypeKey, tmtypes.EventTx, tmtypes.TxHashKey, hash)
            eventCh, err := c.Subscribe(ctx, subscriber, query)
            // ...
        },
    }

    cmd.Flags().Duration(TimeoutFlag, 15*time.Second, "等待交易被包含在区块中的最大时间")
    flags.AddQueryFlagsToCmd(cmd)
    return cmd
}
```

### 交易响应格式化
系统提供了多个辅助函数来格式化交易响应，包括`newTxResponseCheckTx`、`newTxResponseDeliverTx`和`newResponseFormatBroadcastTxCommit`，这些函数将CometBFT的响应转换为SDK标准的`TxResponse`格式。

**Section sources**
- [client/rpc/tx.go](file://client/rpc/tx.go#L1-L227)

## 验证者集查询
`client/rpc/validators.go`文件提供了查询验证者集的功能，允许获取指定高度的完整验证者集合。

```go
func ValidatorCommand() *cobra.Command {
    cmd := &cobra.Command{
        Use:     "comet-validator-set [height]",
        Aliases: []string{"cometbft-validator-set", "tendermint-validator-set"},
        Short:   "获取给定高度的完整CometBFT验证者集",
        Args:    cobra.MaximumNArgs(1),
        RunE: func(cmd *cobra.Command, args []string) error {
            clientCtx, err := client.GetClientQueryContext(cmd)
            if err != nil {
                return err
            }

            var height *int64
            if len(args) > 0 {
                val, err := strconv.ParseInt(args[0], 10, 64)
                if err != nil {
                    return err
                }

                if val > 0 {
                    height = &val
                }
            }

            page, _ := cmd.Flags().GetInt(flags.FlagPage)
            limit, _ := cmd.Flags().GetInt(flags.FlagLimit)

            response, err := cmtservice.ValidatorsOutput(cmd.Context(), clientCtx, height, page, limit)
            if err != nil {
                return err
            }

            return clientCtx.PrintProto(response)
        },
    }

    cmd.Flags().String(flags.FlagNode, "tcp://localhost:26657", "<host>:<port> to CometBFT RPC interface for this chain")
    cmd.Flags().StringP(flags.FlagOutput, "o", "text", "Output format (text|json)")
    cmd.Flags().Int(flags.FlagPage, query.DefaultPage, "Query a specific page of paginated results")
    cmd.Flags().Int(flags.FlagLimit, 100, "Query number of results returned per page")
    return cmd
}
```

该功能通过`cmtservice.ValidatorsOutput`函数实现，支持分页查询，便于处理大型验证者集合。

**Section sources**
- [client/rpc/validators.go](file://client/rpc/validators.go#L1-L60)

## Swagger UI集成
Cosmos SDK集成了Swagger UI来提供API的可视化文档，使得开发者可以方便地探索和测试REST API。

### Swagger配置
Swagger文档的配置在`client/docs/config.json`文件中定义，该文件指定了所有模块的Swagger JSON文件位置：

```json
{
  "swagger": "2.0",
  "info": {
    "title": "Cosmos SDK - gRPC Gateway docs",
    "description": "A REST interface for state queries.",
    "version": "1.0.0"
  },
  "apis": [
    {
      "url": "./tmp-swagger-gen/cosmos/auth/v1beta1/query.swagger.json",
      "operationIds": {
        "rename": {
          "Params": "AuthParams"
        }
      }
    },
    {
      "url": "./tmp-swagger-gen/cosmos/bank/v1beta1/query.swagger.json",
      "operationIds": {
        "rename": {
          "Params": "BankParams"
        }
      }
    }
    // ... 其他模块
  ]
}
```

### Swagger UI嵌入
Swagger UI界面通过Go的embed包嵌入到二进制文件中，确保了文档的便携性和一致性：

```go
package docs

import "embed"

//go:embed swagger-ui
var SwaggerUI embed.FS
```

### API文档注册
在应用启动时，通过`server.RegisterSwaggerAPI`函数注册Swagger API路由，使得Swagger UI可以通过HTTP访问：

```go
if err := server.RegisterSwaggerAPI(apiSvr.ClientCtx, apiSvr.Router, apiConfig.Swagger); err != nil {
    panic(err)
}
```

这使得开发者可以通过浏览器访问`/swagger/`路径来查看和测试所有可用的REST API端点。

**Section sources**
- [client/docs/config.json](file://client/docs/config.json#L1-L179)
- [client/docs/embed.go](file://client/docs/embed.go#L1-L7)
- [server/api/server.go](file://server/api/server.go#L31-L237)

## 使用示例
### curl命令示例
#### 查询账户余额
```bash
curl -X GET "http://localhost:1317/cosmos/bank/v1beta1/balances/{address}" \
  -H "accept: application/json"
```

#### 查询区块详情
```bash
curl -X GET "http://localhost:1317/cosmos/base/tendermint/v1beta1/blocks/latest" \
  -H "accept: application/json"
```

#### 查询交易
```bash
curl -X GET "http://localhost:1317/cosmos/tx/v1beta1/txs/{hash}" \
  -H "accept: application/json"
```

### Go客户端代码示例
```go
// 创建客户端上下文
clientCtx := client.Context{
    Client:     rpcClient,
    GRPCClient: grpcClient,
    Codec:      codec,
}

// 查询账户
queryClient := types.NewQueryClient(clientCtx)
res, err := queryClient.Account(context.Background(), &types.QueryAccountRequest{
    Address: "cosmos1...",
})
if err != nil {
    log.Fatal(err)
}

// 查询区块
block, err := rpc.GetBlockByHeight(clientCtx, &height)
if err != nil {
    log.Fatal(err)
}
```

## 性能考虑与缓存策略
### 性能优化
1. **gRPC优先**：当gRPC客户端可用时，优先使用gRPC而不是ABCI查询，因为gRPC通常具有更好的性能。
2. **连接复用**：保持HTTP和gRPC连接的持久性，避免频繁建立和关闭连接的开销。
3. **批量查询**：对于需要获取多个相关数据的场景，考虑使用批量查询接口减少网络往返次数。

### 缓存策略
1. **客户端缓存**：在客户端实现适当的缓存机制，对于不经常变化的数据（如账户余额、验证者集）可以设置合理的缓存有效期。
2. **ETag支持**：利用HTTP的ETag机制，通过条件请求减少不必要的数据传输。
3. **分页查询**：对于大型数据集，使用分页查询避免一次性获取过多数据导致性能下降。

通过合理的设计和使用这些REST API，开发者可以高效地构建基于Cosmos SDK的区块链应用，同时保持良好的性能和用户体验。