# hexo apollo 主题

## 修改说明

本项目基于apollo主题修改, 新增以下功能

1. 支持`tag`显示
2. 增加`category`页面  
3. 增加`gitalk`支持
4. 增加友链页面

![hexo-theme-apollo](https://pic.rmb.bdstatic.com/14a2b512b64854be45f03ebbf100b6cf.png)

## 文档

- [中文文档](https://github.com/achjqz/hexo-theme-apollo/blob/master/docs/doc-zh.md)
- [Document](https://github.com/achjqz/hexo-theme-apollo/blob/master/docs/doc-en.md)

## 安装

``` bash
npm i hexo-theme-apollo
```

## 启用

修改 `_config.yml` 的 `theme` 配置项为 `apollo`:

```yaml
theme: apollo

# 在归档页面显示所有文章
# 需要上面安装的 hexo-generator-archive 插件支持
archive_generator:
    per_page: 0
    yearly: false
    monthly: false
    daily: false
```

## 更新

``` bash
npm i hexo-theme-apollo@latest
```

## 开发

``` bash
git clone https://github.com/achjqz/hexo-theme-apollo.git apollo
cd apollo
npm install
npm install -g gulp-cli
# 构建css
gulp
```

## License

MIT
