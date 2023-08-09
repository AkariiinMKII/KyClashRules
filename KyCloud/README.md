# 关于 KyCloudProviders(-withFilter).yml

## 说明

### 关于 [KyCloudProviders.yml](https://raw.githubusercontent.com/AkariiinMKII/SubConfig/master/KyCloud/KyCloudProviders.yml)

- 此文件为适用于 KyCloud 的配置文件，可根据需求自行 fork 并修改配置内容
- 由于此配置使用了 [Proxy Providers](https://dreamacro.github.io/clash/configuration/outbound.html#proxy-providers) 和 [Rule Providers](https://dreamacro.github.io/clash/premium/rule-providers.html)，需要搭配 [Premium core](https://dreamacro.github.io/clash/premium/introduction.html) 或使用 Premium core 的 GUI，例如 [Clash for Windows](https://github.com/Fndroid/clash_for_windows_pkg) 、 [ClashX Pro](https://install.appcenter.ms/users/clashx/apps/clashx-pro/distribution_groups/public)
- 推荐使用 [FastGit](https://doc.fastgit.org/zh-cn/guide.html) 代理地址 `https://raw.fgit.cf/AkariiinMKII/SubConfig/master/KyCloud/KyCloudProviders.yml`
- 需要配合 [subconverter](https://github.com/tindy2013/subconverter) 使用，可以选择自建服务后端、公共服务后端或 KyCloud 提供的订阅转换服务后端

### 关于 [KyCloudProviderswithFilter.yml](https://raw.githubusercontent.com/AkariiinMKII/SubConfig/master/KyCloud/KyCloudProviderswithFilter.yml)

- 此文件为使用 [filter](https://github.com/Dreamacro/clash/pull/2518) 功能对 KyCloudProviders.yml 进行简化的版本
- 需要 [Clash for Windows v0.20.21 (Clash Premium 2023.04.13)](https://github.com/Fndroid/clash_for_windows_pkg/releases/tag/0.20.21) 及以上版本
- 使用方法同上，只需将订阅地址改为 `https://raw.fgit.cf/AkariiinMKII/SubConfig/master/KyCloud/KyCloudProviderswithFilter.yml`

## 使用

### 使用 Clash for Windows 进行自动配置（推荐）

需要配合 Clash for Windows 的 [parser](https://docs.cfw.lbyczf.com/contents/parser.html#%E8%BF%9B%E9%98%B6%E6%96%B9%E6%B3%95-javascript) 功能使用，脚本如下

```yaml
parsers: # array
  - reg: '\/SubConfig\/master\/KyCloud\/KyCloudProviders'
    code: |
      module.exports.parse = async (raw, { axios, yaml, notify, console }) => {
        const subcon = 'xxxxxx' // 引号中的 xxxxxx 改为 subconverter 后端服务域名，例如 api.subconverter.com ，不需要包含 http:// 或 https:// ，如使用非默认端口需添加端口号
        const suburl = 'xxxxxx' // 引号中的 xxxxxx 改为 KyCloud 的完整 Clash 个人订阅地址，需要包含 https://
        const subaddr = suburl.match(/\/\/(.*?)\//)[1]
        const subid = suburl.match(/sid=(.*?)&/)[1]
        const subtoken = suburl.match(/token=(.*)/)[1]
        raw = raw.replace(/\%SUBCON_BACKEND\%/gm,`${subcon}`)
        raw = raw.replace(/\%SUB_ADDRESS\%/gm,`${subaddr}`)
        raw = raw.replace(/\%SUB_ID\%/gm,`${subid}`)
        raw = raw.replace(/\%SUB_TOKEN\%/gm,`${subtoken}`)
        //const setdns = await axios.get('https://raw.fgit.cf/AkariiinMKII/SubConfig/master/KyCloud/dns.yml') // 如需预置 dns 字段，请删除句首注释符
        //raw = `${setdns.data}${raw}` // 如需预置 dns 字段，请删除句首注释符
        let { headers:{"subscription-userinfo": subinfo = ""}={}, status } = await axios.head(suburl)
        subinfo = subinfo.replace(/;*$/g,'')
        if (status === 200 && subinfo) {
          return `# ${subinfo};\n${raw}`
        }
        return raw
      }
```

- 可以使用 parser 对本配置文件进行多次编辑（如追加规则条目），但需要保证以上脚本最后执行，即放置在 parser 队列末尾
- 不推荐使用脚本预置 `dns` 字段，如有需要可使用 [Mixin](https://docs.cfw.lbyczf.com/contents/mixin.html) 功能添加
- 如果打开了 Clash for Windows 的 [TUN Mode](https://docs.cfw.lbyczf.com/contents/tun.html) 开关，预置在配置文件中的 `dns` 字段会被覆盖，必须通过 Mixin 功能添加才能使字段生效
- 使用 [diff](https://docs.cfw.lbyczf.com/contents/diff.html) 或者 [parser](https://docs.cfw.lbyczf.com/contents/parser.html#%E8%BF%9B%E9%98%B6%E6%96%B9%E6%B3%95-javascript) 也可以在原始订阅中预置 `dns` 字段，但不推荐，理由同上。预置用脚本如下

```yaml
parsers: # array
  - url: xxxxxx # xxxxxx改为 KyCloud 的 Clash 个人订阅地址
    code: |
      module.exports.parse = async (raw, { axios, yaml, notify, console }) => {
        const setdns = await axios.get('https://raw.fgit.cf/AkariiinMKII/SubConfig/master/KyCloud/dns.yml')
        if (setdns.status === 200 && setdns.data) {
          return `${setdns.data}\n${raw}`
        }
        return raw
      }
```

- 不同区 DNS 服务状态也不同，尤其是 doh 地址，请根据实际情况调整
