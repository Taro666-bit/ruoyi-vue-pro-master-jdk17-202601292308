# 国内电商视频生成 SaaS 网站 - 技术方案

## 一、项目概述

基于 **ruoyi-vue-pro** 开源项目，快速搭建一个面向国内电商的 AI 视频生成 SaaS 平台，首期接入 Sora-2 模型，后期可扩展更多视频生成模型。

---

## 二、需要开启的模块

### 2.1 必须开启的核心模块

| 模块 | 路径 | 用途 | 代码完整性 |
|------|------|------|-----------|
| **yudao-module-system** | `/yudao-module-system/` | 用户管理、权限、租户、认证、短信、邮件 | ✅ 完整 |
| **yudao-module-infra** | `/yudao-module-infra/` | 文件存储、代码生成、定时任务、WebSocket | ✅ 完整 |
| **yudao-module-member** | `/yudao-module-member/` | 会员用户、等级、积分、签到 | ✅ 完整 |
| **yudao-module-pay** | `/yudao-module-pay/` | 支付宝/微信支付、钱包、充值、退款 | ✅ 完整 |
| **yudao-module-ai** | `/yudao-module-ai/` | AI 模型管理、API Key 管理、异步任务 | ✅ 完整（需扩展视频功能） |

### 2.2 可选模块（根据业务需要）

| 模块 | 用途 | 建议 |
|------|------|------|
| yudao-module-mp | 微信公众号 | 如需公众号登录/推送可开启 |
| yudao-module-report | 报表大屏 | 如需数据分析可开启 |

### 2.3 不需要开启的模块

- yudao-module-mall（商城） - 不需要传统商品交易
- yudao-module-bpm（工作流） - 不需要审批流程
- yudao-module-crm（CRM） - 不需要客户管理
- yudao-module-erp（ERP） - 不需要进销存
- yudao-module-iot（物联网） - 不相关

---

## 三、模块开启方式

修改 `yudao-server/pom.xml`，取消相关模块的注释：

```xml
<!-- 必须开启 -->
<dependency>
    <groupId>cn.iocoder.boot</groupId>
    <artifactId>yudao-module-system</artifactId>
</dependency>
<dependency>
    <groupId>cn.iocoder.boot</groupId>
    <artifactId>yudao-module-infra</artifactId>
</dependency>

<!-- 取消注释以开启 -->
<dependency>
    <groupId>cn.iocoder.boot</groupId>
    <artifactId>yudao-module-member</artifactId>
</dependency>
<dependency>
    <groupId>cn.iocoder.boot</groupId>
    <artifactId>yudao-module-pay</artifactId>
</dependency>
<dependency>
    <groupId>cn.iocoder.boot</groupId>
    <artifactId>yudao-module-ai</artifactId>
</dependency>
```

---

## 四、Controller 目录结构说明（admin vs app）

### 4.1 现有目录结构

```
yudao-module-ai/src/main/java/.../controller/
├── admin/           # 管理员后台接口（已实现）
│   ├── chat/        # AI 聊天
│   ├── image/       # AI 图片
│   ├── music/       # AI 音乐
│   ├── model/       # 模型管理
│   └── ...
└── app/             # 普通用户/C端接口（仅占位，待实现）
    └── package-info.java
```

### 4.2 admin 和 app 的区别

| 特性 | admin | app |
|------|-------|-----|
| **用途** | 管理员后台 | 普通用户/C端 |
| **权限** | 需要管理员登录 + @PreAuthorize | 只需用户登录 |
| **对应前端** | yudao-ui-admin-vue3 | yudao-ui-mall-uniapp 或自建 |
| **API 前缀** | `/admin/ai/xxx` | `/app/ai/xxx` |

### 4.3 开发策略（已确定）

**采用"先 admin 后 app"的开发顺序**：

| 阶段 | 接口位置 | 前端 | 目的 |
|------|----------|------|------|
| **Phase 1** | `controller/admin/video/` | yudao-ui-admin-vue3 | 功能开发 + 完整测试 |
| **Phase 2** | `controller/app/video/` | e-commerce-video-generation-ui | 用户端正式上线 |

**这样做的好处**：
1. 管理后台权限完善，便于开发调试
2. 可以快速验证 API 对接是否正确
3. 测试通过后再开放给用户，避免线上问题
4. admin 和 app 接口可以复用同一个 Service 层

### 4.4 普通用户访问方案（Phase 2 实现）

**后端**：在 `controller/app/` 下新建接口

```
controller/app/video/                    # 【Phase 2 新建】C端视频接口
├── AppAiVideoController.java            # 普通用户视频 API
└── vo/
    ├── AppAiVideoGenerateReqVO.java     # 生成请求
    └── AppAiVideoRespVO.java            # 响应
```

**前端**：新建独立 Vue3 项目 `e-commerce-video-generation-ui`

---

## 五、新建功能（在 AI 模块中扩展）

### 5.1 需要新建的代码

```
yudao-module-ai/src/main/java/cn/iocoder/yudao/module/ai/
│
├── controller/
│   ├── admin/video/                       # 【新建】管理后台视频接口
│   │   ├── AiVideoController.java         # 管理员视频管理 API
│   │   └── vo/
│   │       ├── AiVideoGenerateReqVO.java
│   │       ├── AiVideoRespVO.java
│   │       └── AiVideoPageReqVO.java
│   │
│   └── app/video/                         # 【新建】C端视频接口
│       ├── AppAiVideoController.java      # 普通用户视频 API
│       └── vo/
│           ├── AppAiVideoGenerateReqVO.java
│           └── AppAiVideoRespVO.java
│
├── service/video/                         # 【新建】视频生成 Service
│   ├── AiVideoService.java
│   └── AiVideoServiceImpl.java
│
├── dal/
│   ├── dataobject/video/                  # 【新建】视频数据对象
│   │   └── AiVideoDO.java
│   └── mysql/video/
│       └── AiVideoMapper.java
│
├── framework/ai/core/model/sora/          # 【新建】Sora API 客户端
│   └── api/
│       └── SoraApi.java
│
├── job/video/                             # 【新建】视频同步任务
│   └── AiVideoSyncJob.java
│
└── enums/video/                           # 【新建】视频枚举
    └── AiVideoStatusEnum.java
```

### 5.2 需要新建的数据库表

```sql
-- 视频生成任务表
CREATE TABLE ai_video (
    id BIGINT AUTO_INCREMENT PRIMARY KEY COMMENT '视频ID',
    user_id BIGINT NOT NULL COMMENT '用户ID',

    -- 生成参数
    prompt VARCHAR(2000) NOT NULL COMMENT '提示词',
    model VARCHAR(64) NOT NULL COMMENT '模型（sora-2/sora-2-pro/sora-2-all）',
    duration INT NOT NULL DEFAULT 10 COMMENT '视频时长（秒）',
    orientation VARCHAR(20) NOT NULL DEFAULT 'landscape' COMMENT '方向（portrait竖屏/landscape横屏）',
    size VARCHAR(20) NOT NULL DEFAULT 'large' COMMENT '分辨率（small约720p/large约1080p）',

    -- 图片输入（sora-2-all 图生视频时使用）
    input_image_url VARCHAR(1000) COMMENT '输入图片URL',

    -- 生成结果
    status TINYINT NOT NULL DEFAULT 10 COMMENT '状态（10进行中 20成功 30失败）',
    task_id VARCHAR(200) COMMENT '云雾平台任务ID',
    video_url VARCHAR(1000) COMMENT '生成的视频URL',
    enhanced_prompt VARCHAR(2000) COMMENT '平台优化后的提示词',
    error_message VARCHAR(500) COMMENT '错误信息',
    finish_time DATETIME COMMENT '完成时间',

    -- 计费相关
    price INT DEFAULT 0 COMMENT '消费金额（分）',

    -- 公共字段
    creator VARCHAR(64) DEFAULT '' COMMENT '创建者',
    create_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updater VARCHAR(64) DEFAULT '' COMMENT '更新者',
    update_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    deleted BIT NOT NULL DEFAULT 0 COMMENT '是否删除',
    tenant_id BIGINT NOT NULL DEFAULT 0 COMMENT '租户ID',

    INDEX idx_user_id (user_id),
    INDEX idx_task_id (task_id),
    INDEX idx_status (status),
    INDEX idx_tenant_id (tenant_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='AI视频生成任务表';
```

---

## 六、云雾 Sora-2 API 对接（已核实）

### 6.1 API 信息汇总

**基础信息**：
- Base URL: `https://yunwu.ai`
- API Key: `sk-lLqS0EeXrwzieSmF6KqOBwyHAalENKG9FFnMKn0NG7NtlOkW`

### 6.2 三个模型对比

| 模型 | 用途 | 时长支持 | 分辨率 | 文档 |
|------|------|----------|--------|------|
| **sora-2** | 文生视频（纯文字提示词） | 10秒 | small(720p)/large | api-358068995 |
| **sora-2-all** | 图生视频（图片+提示词） | 10/15秒 | small/large | api-358068907 |
| **sora-2-pro** | 高清文生视频 | 15/25秒 | large(1080p) | api-358742580 |

### 6.3 创建视频接口

**URL**: `POST https://yunwu.ai/v1/video/create`

**请求参数**：

| 参数 | 类型 | 必需 | 说明 |
|------|------|------|------|
| model | string | 是 | sora-2 / sora-2-all / sora-2-pro |
| prompt | string | 是 | 提示词 |
| images | array | 否 | 图片URL数组（sora-2-all 必需） |
| orientation | string | 是 | portrait(竖屏) / landscape(横屏) |
| size | string | 是 | small(720p) / large(1080p) |
| duration | int | 是 | 时长（10/15/25秒，取决于模型） |
| watermark | boolean | 否 | 默认true，false=强制无水印 |
| private | boolean | 否 | 默认false，true=视频隐藏 |

**请求示例**：

```json
// sora-2 文生视频
{
  "model": "sora-2",
  "prompt": "一个穿着红色连衣裙的女孩在樱花树下跳舞",
  "orientation": "portrait",
  "size": "large",
  "duration": 10,
  "watermark": false
}

// sora-2-all 图生视频
{
  "model": "sora-2-all",
  "images": ["https://example.com/input.jpg"],
  "prompt": "make animate",
  "orientation": "portrait",
  "size": "large",
  "duration": 15,
  "watermark": false
}
```

**响应示例**：

```json
{
  "id": "sora-2:task_01k9008rhbefnt3rb1g9szxdwr",
  "status": "pending",
  "status_update_time": 1762010621323
}
```

### 6.4 查询任务接口

**URL**: `GET https://yunwu.ai/v1/video/query`

**请求参数**：

| 参数 | 位置 | 类型 | 必需 | 说明 |
|------|------|------|------|------|
| id | query | string | 是 | 任务ID，如 `sora-2:task_01kbfq03gpe0wr9ge11z09xqrj` |

**请求示例**：

```
GET https://yunwu.ai/v1/video/query?id=sora-2:task_01k9008rhbefnt3rb1g9szxdwr
Authorization: Bearer sk-xxx
```

**响应示例**：

```json
{
  "id": "sora-2:task_01k9008rhbefnt3rb1g9szxdwr",
  "status": "completed",
  "video_url": "https://xxx.com/video.mp4",
  "enhanced_prompt": "优化后的提示词...",
  "status_update_time": 1762010700000
}
```

**状态值**：
- `pending` - 排队中
- `in_progress` - 生成中
- `completed` - 完成
- `failed` - 失败

### 6.5 SoraApi 客户端实现

```java
@Slf4j
public class SoraApi {

    private final String baseUrl;
    private final String apiKey;
    private final RestClient restClient;

    public SoraApi(String baseUrl, String apiKey) {
        this.baseUrl = baseUrl;
        this.apiKey = apiKey;
        this.restClient = RestClient.builder()
                .baseUrl(baseUrl)
                .defaultHeader("Authorization", "Bearer " + apiKey)
                .defaultHeader("Content-Type", "application/json")
                .build();
    }

    /**
     * 创建视频任务
     */
    public VideoCreateResponse createVideo(VideoCreateRequest request) {
        return restClient.post()
                .uri("/v1/video/create")
                .body(request)
                .retrieve()
                .body(VideoCreateResponse.class);
    }

    /**
     * 查询任务状态
     */
    public VideoQueryResponse queryVideo(String taskId) {
        return restClient.get()
                .uri(uriBuilder -> uriBuilder
                        .path("/v1/video/query")
                        .queryParam("id", taskId)
                        .build())
                .retrieve()
                .body(VideoQueryResponse.class);
    }

    // ========== 数据结构 ==========

    @Data
    public static class VideoCreateRequest {
        private List<String> images;    // 图片URL列表（sora-2-all 必需）
        private String model;           // sora-2 / sora-2-all / sora-2-pro
        private String orientation;     // portrait / landscape
        private String prompt;          // 提示词
        private String size;            // small / large
        private Integer duration;       // 10 / 15 / 25
        private Boolean watermark;      // 水印
        @JsonProperty("private")
        private Boolean privateMode;    // 是否私密
    }

    @Data
    public static class VideoCreateResponse {
        private String id;              // 任务ID
        private String status;          // pending / in_progress / completed / failed
        private Long statusUpdateTime;
    }

    @Data
    public static class VideoQueryResponse {
        private String id;
        private String status;
        private String videoUrl;
        private String enhancedPrompt;
        private Long statusUpdateTime;
    }
}
```

---

## 七、前端项目规划

### 7.1 项目结构（已确定）

| 项目 | 框架 | 用途 | 路径 |
|------|------|------|------|
| yudao-ui-admin-vue3 | Vue3 + Element Plus | 管理员后台（复用） | `/yudao-ui/yudao-ui-admin-vue3` |
| **e-commerce-video-generation-ui** | Vue3（新建） | 普通用户/会员前端 | `/yudao-ui/e-commerce-video-generation-ui` |

### 7.2 管理员后台（yudao-ui-admin-vue3）

复用现有项目，新增视频管理页面：
- 视频任务管理（查看所有用户的视频）
- 用户管理
- 充值订单管理
- 数据统计
- 模型配置

### 7.3 用户端前端（e-commerce-video-generation-ui）【新建】

基于 Vue3 新建独立项目，功能包括：
- 用户注册/登录
- 视频生成页面（文生视频、图生视频）
- 我的视频列表
- 钱包充值
- 会员中心
- 消费记录

**技术栈建议**：
- Vue 3 + TypeScript
- Element Plus 或 Ant Design Vue
- Vite 构建
- Pinia 状态管理
- 可参考 yudao-ui-admin-vue3 的请求封装和权限处理

---

## 八、复用现有功能的修改方式

### 8.1 复用 AI 模块的模型管理

**添加平台枚举** - `AiPlatformEnum.java`：
```java
YUNWU("Yunwu", "云雾API"),
```

### 8.2 计费模式：积分制（待细化）

> **说明**：采用积分制计费，具体的充值套餐、兑换比例、消耗规则等待后期讨论确定。

**基本思路**：
1. 用户充值或购买套餐 → 兑换成等值积分
2. 生成视频 → 消耗积分
3. 积分不足时无法生成

**待确定事项**（后期讨论）：
- [ ] 充值套餐设计（金额档位、赠送比例）
- [ ] 积分兑换比例（1元 = ?积分）
- [ ] 不同模型的积分消耗（sora-2 / sora-2-all / sora-2-pro）
- [ ] 不同时长的积分消耗（10秒 / 15秒 / 25秒）
- [ ] 会员等级折扣（VIP 消耗更少积分？）
- [ ] 积分有效期（是否过期？）
- [ ] 免费体验积分（新用户赠送？）

**复用 member 模块的积分系统**：
- `MemberPointRecordDO` - 积分变动记录
- `MemberPointRecordService` - 积分操作服务

**Phase 1 暂不实现计费**，先完成 Sora 功能接入和测试。

### 8.3 复用会员模块

**积分业务类型** - `MemberPointBizTypeEnum.java`（Phase 2 添加）：
```java
VIDEO_RECHARGE(20, "视频积分充值"),
VIDEO_GENERATE(21, "视频生成消耗"),
```

### 8.4 复用文件存储

```java
@Resource
private FileApi fileApi;

private String saveVideo(String remoteVideoUrl) {
    byte[] videoBytes = HttpUtil.downloadBytes(remoteVideoUrl);
    return fileApi.createFile(videoBytes);
}
```

---

## 九、异步任务处理流程

```
1. 用户提交生成请求
   ↓
2. 检查余额 → 冻结金额
   ↓
3. 调用 SoraApi.createVideo()
   ↓
4. 保存任务到数据库（状态：IN_PROGRESS）
   ↓
5. 返回任务ID给前端
   ↓
6. 定时任务轮询（AiVideoSyncJob，每分钟）
   ↓
7. 调用 SoraApi.queryVideo() 查询状态
   ↓
8. 完成后：
   - 下载视频 → 上传到自己的存储
   - 扣除冻结金额（或失败则解冻）
   - 更新数据库状态
```

---

## 十、开发优先级与里程碑

### 开发策略（已确定）

**先 admin 后 app 的开发顺序**：
1. 先在 `controller/admin/video/` 实现完整的 Sora 视频生成功能
2. 使用管理后台（yudao-ui-admin-vue3）进行完整测试
3. 测试通过后，在 `controller/app/video/` 实现 C 端接口
4. 开发用户端前端（e-commerce-video-generation-ui）

### Phase 1 - 后端 Admin 接口 + 管理后台测试

**后端**：
- [ ] 开启必要模块（system/infra/member/pay/ai）
- [ ] 创建 ai_video 表
- [ ] 添加平台枚举 YUNWU
- [ ] 实现 SoraApi 客户端
- [ ] 实现 AiVideoService
- [ ] 实现 AiVideoController（admin 接口）
- [ ] 实现 AiVideoSyncJob（定时同步任务）

**管理后台前端**（yudao-ui-admin-vue3）：
- [ ] 视频生成测试页面
- [ ] 视频任务列表页面
- [ ] 视频详情/播放页面

**验收标准**：
- [ ] 文生视频（sora-2）测试通过
- [ ] 图生视频（sora-2-all）测试通过
- [ ] 高清视频（sora-2-pro）测试通过
- [ ] 任务状态同步正常
- [ ] 视频下载并存储到自有云存储

### Phase 2 - 后端 App 接口 + 用户端前端

**后端**：
- [ ] 实现 AppAiVideoController（app 接口）
- [ ] 集成积分模块（积分检查、消耗）
- [ ] 集成会员模块（等级权益）

**用户端前端**（e-commerce-video-generation-ui 新建）：
- [ ] 项目初始化（Vue3 + TypeScript + Vite）
- [ ] 用户登录/注册页面
- [ ] 视频生成页面
- [ ] 我的视频列表
- [ ] 积分充值页面
- [ ] 会员中心

### Phase 3 - 积分计费系统（待讨论后实现）

- [ ] 确定充值套餐和积分兑换比例
- [ ] 确定各模型/时长的积分消耗规则
- [ ] 实现积分充值功能
- [ ] 实现积分消耗逻辑
- [ ] 会员等级折扣
- [ ] 消费记录与账单
- [ ] 数据统计大屏

### Phase 4 - 扩展（持续）

- [ ] 接入更多视频生成模型
- [ ] 模板市场
- [ ] 批量生成
- [ ] API 开放给第三方

---

## 十一、关键配置

### application.yml

```yaml
yudao:
  ai:
    sora:
      enable: true
      base-url: https://yunwu.ai
      api-key: sk-lLqS0EeXrwzieSmF6KqOBwyHAalENKG9FFnMKn0NG7NtlOkW
```

### 定时任务配置

在管理后台添加：
- 任务名称：AI 视频同步
- 处理器：aiVideoSyncJob
- Cron：`0 */1 * * * ?`（每分钟）

---

## 十二、总结

### 复用度分析

| 功能 | 复用方式 | 工作量 |
|------|----------|--------|
| 用户认证 | 100% 复用 | 无 |
| 会员体系 | 100% 复用 | 无 |
| 支付充值 | 100% 复用 | 无 |
| 文件存储 | 100% 复用 | 无 |
| 模型管理 | 80% 复用 | 小 |
| 视频生成 | 新建 | 中 |

### 新增代码量

- 后端：~1500 行
- 管理后台前端（新增页面）：~1000 行
- 用户端前端（新建项目）：~3000 行
- SQL：~50 行

### 预计时间

| 阶段 | 内容 | 时间 |
|------|------|------|
| Phase 1 | 后端 admin 接口 + 管理后台测试 | 1 周 |
| Phase 2 | 后端 app 接口 + 用户端前端 | 1-2 周 |
| Phase 3 | 完善优化 | 1 周 |
| **总计** | **MVP 上线** | **2-3 周** |
