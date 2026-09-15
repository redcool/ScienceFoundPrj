# Ignis 使用速查(独立文档,可拷到任何项目)

> 用途:用 ignis CLI 生成图片/视频/音频(公司内部平台,走个人积分)。
> 本文件自包含,不依赖任何项目;复制到新项目即可直接用。
> 官方完整 skill:https://ignis-cli-config.funplus-marketing.ai/skill.md

## 一句话

`ignis` 是公司内部的多模态生成 CLI:云端出图(gpt-image / seedream / nano-banana /
midjourney)、出视频(seedance 最高 4K)、出音频(suno),支持直连生成和 Agent 全通路。

## 1. 安装(全局一次)

```bash
npm install -g ignis-agent-cli@latest
ignis --version          # 验证
```

升级:`ignis update --channel latest`

## 2. 配置(拿到 config 链接或 token 后一次完成)

CLI 读取 `%USERPROFILE%\.ignis\config.json`。两种初始化方式:

```bash
# 方式 A:有 config_url(官方推荐,一次性消费)
ignis login --config-url <config_url>

# 方式 B:手写配置文件
```

config.json 内容(手动方式):
```json
{
  "baseUrl": "https://ignis-cli.funplus-marketing.ai",
  "cliToken": "ignis_ct_xxxxxxxxxxxxxxxxxxxxxxxx",
  "sessions": {}
}
```
存放位置:Windows 为 `C:\Users\<你>\.ignis\config.json`(无则自建目录)。

### ⚠️ Windows/受限环境技巧

CLI 硬编码读 `%USERPROFILE%`。如果无法写用户目录(沙箱/权限限制),
把 USERPROFILE 重定向到工作区内目录再调用:

```powershell
$env:USERPROFILE = "D:\path\to\your\fake-home"   # 内含 .ignis\config.json
ignis whoami
```

### Token 格式提醒

- 有效 cliToken 形如 `ignis_ct_kun.su_…`(带 `ignis_ct_` 前缀)。
- 纯 UUID(如 `97a3b6d8-…`)**不是** cliToken,服务端会返回 `INVALID_TOKEN`。
- 验证:`ignis whoami`(显示账号/后端/过期时间)、`ignis limits`(配额)。

## 3. 常用命令速查

```bash
# 看模型 / 看生成哲学(写提示词前先读)
ignis imagine --models
ignis philosophy image        # 或 video / audio / model

# 出图(同步等结果)
ignis imagine "一只橘猫坐在窗台上,阳光洒在毛发上,写实风格" --model gpt-image --ar 16:9 --out ./outputs

# 出视频(异步提交 → 等待下载)
ignis imagine "镜头缓缓推近,角色抬头看天" --model seedance --resolution 1080p --duration 5 --no-wait
ignis imagine --wait <cli_img_任务ID> --out ./outputs

# 批量出图(requests.json 一次 ≤10 条)
ignis imagine --batch requests.json --out ./outputs

# 资产管理(上传本地文件 → file_id,供后续引用)
ignis upload ./photo.png
ignis assets --query 关键词

# Agent 全通路(需要平台技能/多步/抽帧/字幕时)
ignis skills --query <关键词>
ignis ask "目标"
ignis ask --async "目标" && ignis result <turn_id>

# 任务与诊断
ignis task list
ignis task wait <任务ID>
ignis whoami / limits / doctor
```

## 4. 图片模型一览(2026-09 实测)

| 模型 | 能力 | 备注 |
|---|---|---|
| `gpt-image` | create / edit | 画质高,实测出图 2560×1440;支持 1:1~16:9 |
| `seedream` | create / edit | 支持拆图层 |
| `nano-banana` / `nano-banana-2` | create / edit | 21:9~9:16 全画幅 |
| `midjourney` | create | 吃英文 tag 流(提示词写法与其他模型不同) |
| `dreamina` / `z-image` | create | 备选 |

视频:`seedance`(720p/1080p/4K,4-15s,prompt/image/transition/omni 模式)、`seedance-2-5`、`kling`、`sora`、`pixverse`。
音频:`suno` / `suno-similar` / `suno-bgm`。

## 5. 出图提示词要点(官方 philosophy 摘要)

prompt 是"给摄影师交代活儿的一段话",一两句连贯的话说清:
**主体 + 动作/状态 + 环境 + 光线 + 相机或画风**。

- ✅ 好例:"女骑士在城墙缺口拔剑回头的瞬间,身后是燃烧的攻城塔,黎明逆光勾出发丝,35mm 低角度仰拍。"
- ❌ 禁:质量词咒语(8K/best quality)、逗号 tag 流、权重括号、`不要XX` 式 negative;形容词堆叠。
- 编辑图:只说改什么+保什么,不复述整图。
- 例外:只有 `midjourney` 吃 tag 流写法。

## 6. 注意

1. **走个人积分**,每次生成有成本;`ignis limits` 看配额。
2. 出图约 1-2 分钟,视频约 5-10 分钟;视频用 `--no-wait` + `--wait` 轮询不阻塞。
3. 输出自动下载到 `--out` 指定目录,默认在本地当前目录。
4. 不要提交含 cliToken 的 config 到版本控制。

## 汇报验证口令

首次接入后运行:
```bash
ignis whoami   # 期望:账号 kun.su@… / 后端 ignis-cli.funplus-marketing.ai / 过期时间
ignis imagine --models
```