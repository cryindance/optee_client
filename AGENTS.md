# OP-TEE Client 知识库

**组件**: 客户端库与工具
**规模**: ~300文件
**功能**: 非安全世界(REE)与TEE通信接口
**主要语言**: C

---

## 目录结构

```
optee_client/
├── libteec/              # TEE客户端API库
│   ├── src/              # 源文件
│   └── include/          # 公共头文件
├── libckteec/            # PKCS#11接口库
├── libseteec/            # 安全元素接口
├── tee-supplicant/       # 用户空间守护进程
└── public/               # 公共头文件
```

---

## 核心组件

### libteec - 客户端API
主要API函数：
- `TEEC_InitializeContext()` - 初始化TEE连接
- `TEEC_OpenSession()` - 打开TA会话
- `TEEC_InvokeCommand()` - 调用TA命令
- `TEEC_CloseSession()` - 关闭会话
- `TEEC_FinalizeContext()` - 释放资源

### tee-supplicant - 守护进程
- 处理REE文件系统访问请求
- 管理RPMB(Replay Protected Memory Block)
- 加载TA二进制文件
- 执行插件调用

### libckteec - PKCS#11
- 提供标准PKCS#11接口
- 支持加密操作、密钥管理
- 用于智能卡、HSM集成

---

## 关键文件

| 文件 | 说明 |
|------|------|
| `libteec/src/tee_client_api.c` | 核心API实现 |
| `tee-supplicant/src/tee_supplicant.c` | 守护进程主程序 |
| `libckteec/src/pkcs11_api.c` | PKCS#11 API实现 |
| `public/tee_client_api.h` | 公共API头文件 |

---

## 构建命令

```bash
# 构建客户端库
make -C optee_client

# 安装到指定目录
make -C optee_client install DESTDIR=/path/to/install

# 交叉编译
make CROSS_COMPILE=aarch64-linux-gnu-
```

---

## 使用示例

```c
#include <tee_client_api.h>

TEEC_Context ctx;
TEEC_Session sess;
TEEC_UUID uuid = TA_HELLO_WORLD_UUID;

// 初始化
TEEC_InitializeContext(NULL, &ctx);

// 打开会话
TEEC_OpenSession(&ctx, &sess, &uuid, TEEC_LOGIN_PUBLIC, NULL, NULL, NULL);

// 调用命令
TEEC_Operation op;
memset(&op, 0, sizeof(op));
op.paramTypes = TEEC_PARAM_TYPES(TEEC_VALUE_INOUT, TEEC_NONE, TEEC_NONE, TEEC_NONE);
op.params[0].value.a = 42;
TEEC_InvokeCommand(&sess, CMD_INC_VALUE, &op, NULL);

// 清理
TEEC_CloseSession(&sess);
TEEC_FinalizeContext(&ctx);
```

---

## 注意事项

1. **线程安全**: libteec是线程安全的，但会话不能跨线程共享
2. **超时处理**: 默认超时可能被TEE覆盖
3. **内存管理**: 共享内存需要正确注册和释放
4. **错误处理**: 检查返回值，使用`TEEC_ERROR_*`宏
