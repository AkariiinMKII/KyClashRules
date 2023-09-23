# 关于 KyCloudinProviders.yml

## 说明

### 关于 [KyCloud](https://github.com/AkariiinMKII/SubConfig/tree/KyCloud) 分支的 [KyCloudinProviders.yml](https://raw.githubusercontent.com/AkariiinMKII/SubConfig/KyCloud/ProviderTemplates/KyCloudinProviders.yml)

- 此文件为适用于 KyCloud 的配置文件模板，可根据需求自行 fork 并修改配置内容
- 由于此配置使用了 [Proxy Providers](https://dreamacro.github.io/clash/configuration/outbound.html#proxy-providers) 、 [Rule Providers](https://dreamacro.github.io/clash/premium/rule-providers.html) 、 [filter](https://github.com/Dreamacro/clash/pull/2518) 、 [regexp2](https://github.com/Dreamacro/clash/pull/2802) 等功能，需要 [Clash for Windows v0.20.28](https://github.com/Fndroid/clash_for_windows_pkg/releases/tag/0.20.28) 及以上版本
- 由于 KyCloud 提供了 [subconverter](https://github.com/tindy2013/subconverter) 订阅转换服务，此配置模板会使用该服务重命名节点以优化可读性，实际应用时可以选择自建服务后端、公共服务后端或 KyCloud 提供的服务后端
- 推荐使用 [FastGit](https://doc.fastgit.org/zh-cn/guide.html) 代理地址 `https://raw.fgit.cf/AkariiinMKII/SubConfig/KyCloud/ProviderTemplates/KyCloudinProviders.yml` 作为订阅地址

### 关于 [KyCloudwithBackupNodes](https://github.com/AkariiinMKII/SubConfig/tree/KyCloudwithBackupNodes) 分支的 [KyCloudinProviders.yml](https://raw.githubusercontent.com/AkariiinMKII/SubConfig/KyCloudwithBackupNodes/ProviderTemplates/KyCloudinProviders.yml)

- 此文件为基于 KyCloud 分支添加备用订阅的版本，以应对机场临时瘫痪，软件版本要求同上
- 使用方法同上，只需将订阅地址改为 `https://raw.fgit.cf/AkariiinMKII/SubConfig/KyCloudwithBackupNodes/ProviderTemplates/KyCloudinProviders.yml` 并修改 parser 脚本中的对应内容

## 使用

### 使用 Clash for Windows 进行自动配置

需要配合 Clash for Windows 的 [parser](https://docs.cfw.lbyczf.com/contents/parser.html#%E8%BF%9B%E9%98%B6%E6%96%B9%E6%B3%95-javascript) 功能使用，脚本如下

```yaml
parsers: # array
  - reg: '\/SubConfig\/.+\/KyCloudinProviders'
    code: |
      module.exports.parse = async (raw, { axios, yaml, notify, console }) => {

        // 请填写 KyCloud 订阅信息
        const subcon = 'xxxxxx' // 将单引号中的 xxxxxx 改为 subconverter 后端服务地址，需要包含 http:// 或 https:// ，如使用非默认端口需添加端口号，例如 https://api.subconverter.com 或 http://127.0.0.1:25500
        const suburl = 'xxxxxx' // 将单引号中的 xxxxxx 改为 KyCloud 的完整 Clash 个人订阅地址，需要包含 https://

        // 如用于包含备用订阅的模板，请删除以下三行句首的注释符，并根据说明填写信息
        //const backupsub = 'xxxxxx' // 将单引号中的 xxxxxx 改为备用代理订阅地址，格式要求同 KyCloud 个人订阅
        //const backupsubencode = encodeURIComponent(backupsub)
        //raw = raw.replace(/\%BACKUP_SUB\%/gm,`${backupsubencode}`)

        // 如需预置 dns 字段,请删除以下两行句首的注释符
        //const setdns = await axios.get('https://raw.fgit.cf/AkariiinMKII/SubConfig/CommonFiles/OtherTemplates/DNS.yml')
        //raw = `${setdns.data}${raw}`

        // 开始处理配置文件
        const subconaddr = subcon.match(/^(http:\/\/.*?|https:\/\/.*?)(?:\/|$)/)[1]
        const subaddr = suburl.match(/\/\/(.*?)\//)[1]
        const subid = suburl.match(/sid=(.*?)(?:&|$)/)[1]
        const subtoken = suburl.match(/token=(.*?)(?:&|$)/)[1]
        raw = raw.replace(/http:\/\/\%SUBCON_BACKEND\%/gm,`${subconaddr}`)
        raw = raw.replace(/\%SUB_ADDRESS\%/gm,`${subaddr}`)
        raw = raw.replace(/\%SUB_ID\%/gm,`${subid}`)
        raw = raw.replace(/\%SUB_TOKEN\%/gm,`${subtoken}`)

        // 如需在 Profile 选项卡中显示订阅信息，请删除以下五行句首的注释符
        //let { headers:{"subscription-userinfo": subinfo = ""}={}, status } = await axios.head(suburl)
        //subinfo = subinfo.replace(/;*$/g,'')
        //if (status === 200 && subinfo) {
        //  return `# ${subinfo};\n${raw}`
        //}

        return raw
      }
```

- 可以使用 parser 对本配置文件进行多次编辑（如追加规则条目），但需要保证以上脚本最后执行，即放置在 parser 队列末尾
- 不推荐使用脚本预置 `dns` 字段，如有需要可使用 [Mixin](https://docs.cfw.lbyczf.com/contents/mixin.html) 功能添加。[字段内容](https://raw.githubusercontent.com/AkariiinMKII/SubConfig/CommonFiles/OtherTemplates/DNS.yml)
- 如果打开了 Clash for Windows 的 [TUN Mode](https://docs.cfw.lbyczf.com/contents/tun.html) 开关，预置在配置文件中的 `dns` 字段会被覆盖，必须通过 Mixin 功能添加才能使字段生效
- 使用 [diff](https://docs.cfw.lbyczf.com/contents/diff.html) 或者 [parser](https://docs.cfw.lbyczf.com/contents/parser.html#%E8%BF%9B%E9%98%B6%E6%96%B9%E6%B3%95-javascript) 也可以在原始订阅中预置 `dns` 字段，但不推荐，理由同上。预置用脚本如下

```yaml
parsers: # array
  - url: xxxxxx # 将 xxxxxx 改为 KyCloud 的 Clash 个人订阅地址
    code: |
      module.exports.parse = async (raw, { axios, yaml, notify, console }) => {
        const setdns = await axios.get('https://raw.fgit.cf/AkariiinMKII/SubConfig/CommonFiles/OtherTemplates/DNS.yml')
        if (setdns.status === 200 && setdns.data) {
          return `${setdns.data}\n${raw}`
        }
        return raw
      }
```

- 不同区 DNS 服务状态也不同，尤其是 doh 地址，请根据实际情况调整
