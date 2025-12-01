# 博客技术栈升级指南

## 📦 依赖更新

### 1. Hexo 核心升级
```bash
# 备份当前版本
npm ls hexo

# 升级到最新版本 
npm update hexo
npm install hexo@latest

# 升级相关生成器
npm update hexo-generator-*
```

### 2. 安全更新
```bash
# 检查安全漏洞
npm audit

# 自动修复
npm audit fix

# 手动修复高风险漏洞
npm audit fix --force
```

### 3. 建议升级的关键依赖
```json
{
  "hexo": "^6.3.0",
  "hexo-renderer-marked": "^5.0.0", 
  "hexo-generator-search": "^2.4.3",
  "hexo-deployer-git": "^4.0.0"
}
```

## ⚙️ 配置优化

### 1. 评论系统配置
- 注册 [LeanCloud](https://console.leancloud.app/) 账号
- 创建应用获取 AppId 和 AppKey
- 在主题配置中填入相关信息

### 2. 统计功能配置  
- 不蒜子统计：已启用，无需额外配置
- Google Analytics：建议申请 GA4 账号
- 百度统计：申请百度统计账号并配置

### 3. Mermaid 图表优化
- 已配置素描本风格主题
- 字体设置为手写风格
- 适合技术文章的可视化展示

## 🎯 下一步行动

1. **立即执行**：
   - [x] 启用评论系统配置
   - [x] 启用网站统计  
   - [x] 优化关于页面
   - [x] 配置 Mermaid 主题

2. **需要申请账号**：
   - [ ] LeanCloud 账号（评论功能）
   - [ ] Google Analytics（可选）
   - [ ] 百度统计（可选）

3. **技术升级**（谨慎执行）：
   - [ ] 升级 Hexo 到最新版本
   - [ ] 更新依赖包
   - [ ] 测试网站功能

## ⚠️ 注意事项

- 升级前请**备份整个博客目录**
- 建议在本地测试环境先验证升级效果
- 逐步升级，避免一次性升级所有依赖
- 升级后检查网站生成和部署是否正常
