# 关于 [KyCloudProviders.yml](https://raw.githubusercontent.com/AkariiinMKII/SubConfig/master/KyCloud/KyCloudProviders.yml)

- 此文件为适用于 KyCloud 的 provider 类型配置文件模板，可根据需求自行 fork 并修改
- 推荐使用 [fastgit](https://doc.fastgit.org/zh-cn/guide.html) 代理地址 https://raw.fastgit.org/AkariiinMKII/SubConfig/master/KyCloud/KyCloudProviders.yml
- 需要配合 [subconverter](https://github.com/tindy2013/subconverter) 使用，可以选择自建服务后端、公共服务后端或 KyCloud 提供的订阅转换服务后端
- 需要配合 [Clash for Windows](https://github.com/Fndroid/clash_for_windows_pkg) 的 [parser](https://docs.cfw.lbyczf.com/contents/parser.html#%E8%BF%9B%E9%98%B6%E6%96%B9%E6%B3%95-javascript) 功能使用，脚本如下

```yaml
parsers: # array
  - reg: '\/SubConfig\/master\/KyCloud\/KyCloudProviders.yml'
    code: |
      module.exports.parse = async (raw, { axios, yaml, notify, console }) => {
        const subcon = 'xxxxxx' //引号中改为 subconverter 后端地址
        const suburl = 'xxxxxx' //引号中改为 KyCloud 的 Clash 个人订阅地址
        const subaddr = suburl.match(/\/\/(.*?)\//)[1]
        const subid = suburl.match(/sid=(.*?)&/)[1]
        const subtoken = suburl.match(/token=(.*)/)[1]
        raw = raw.replace(/\%SUBCON_BACKEND\%/gm,`${subcon}`)
        raw = raw.replace(/\%SUB_ADDRESS\%/gm,`${subaddr}`)
        raw = raw.replace(/\%SUB_ID\%/gm,`${subid}`)
        raw = raw.replace(/\%SUB_TOKEN\%/gm,`${subtoken}`)
        let { headers:{"subscription-userinfo": subinfo = ""}={}, status } = await axios.head(suburl)
        subinfo = subinfo.replace(/;*$/g,'')
        if (status === 200 && subinfo) {
          return `# ${subinfo};\n${raw}`
        }
        return raw
      }
```
