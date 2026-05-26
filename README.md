# tdx-api-usage skill

这个分支专门用于分发 `tdx-api-usage` skill。

## 文件说明

- `SKILL.md`：skill 主文件
- `references/api-quick-reference.md`：接口速查表

## 安装方式

### 方法 1：在 Trae 中手动创建

1. 打开 Trae。
2. 进入 `Settings -> Rules & Skills -> Skills`。
3. 点击创建新 Skill。
4. 将本仓库 `skill` 分支中的 `SKILL.md` 内容粘贴进去。
5. 保存即可。

### 方法 2：先拉取这个分支再导入

```bash
git clone -b skill --single-branch https://github.com/acdzh/tdx-api.git
```

然后打开仓库根目录下的 `SKILL.md`，将内容导入到 Trae。

## 使用说明

安装后，在需要查询、调试或编写 TDX A 股股票数据 HTTP API 调用示例时使用该 skill。

不建议把它用于基金净值、基金档案、债券、期货或宏观经济查询。
