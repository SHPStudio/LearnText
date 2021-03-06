# WebHooks
## 简介
 它是一种事件的机制，可以订阅在Github.com上的事件，当这些事件被触发后，例如git push事件，github会发送http post请求到webHook配置的url上并且会带有相应数据。所以可以用来做很多事情，例如更新外部issue跟踪器，触发CI构建、更新备份镜像，甚至可以部署到生产服务器上，没有做不到，只有你想不到。
 webhook可以设置在`组织`或者`特殊的仓库`上。
 **注意** 每个事件只能最多设置`20`个webhook。

## 事件
**默认只订阅了push事件**

常用事件分类
- `*` 所有支持的事件，通配符事件，任何事件都会触发
- `create` 当创建一个分支(branch)或者标签(tag)时触发
- `delete` 当删除一个分支或标签时触发
- `fork` 当有人fork时触发
- `issues` 当有issue操作时触发，例如开启、编辑、删除等
- `push` 默认事件,当分支有push时或者标签有push时都会触发
- `watch` 当有人点星仓库时触发

## 事件数据
每个事件都有特定的数据格式来标识该事件的相应信息。事件数据大体包括几项数据:
- `sender` 触发了事件的用户
- 事件在哪里发生的 可能是组织(organization)或者仓库(repository)
- `installation` 如果一个webhook的数据是准备给GITHUB APP的，那么还会包含一个instllation事件的数据。
**注意** 传输的数据最大为25mb，如果事件生成的数据太大将不会触发webhook。例如当很多分支同时push时触发create事件产生的数据将大于25mb。

## 传输数据header
在触发事件github向配置的url发送post请求传输数据时会有一些特殊的`headers`
- `X-GitHub-Event` 触发事件的类型
- `X-Github-Delivery` 传输的标识GUID
- `X-Hub-Signature` 响应体HMAC16进制摘要。这个特殊的header在有密钥(secret)配置下才会发送，生成方式是使用`sha1`加密hash算法以`secret`密钥作为HMAC计算得出。
- `User-Agent`将会带有`GitHub-Hookshot/`前缀

数据示例：
```
POST /payload HTTP/1.1
Host: localhost:4567
X-GitHub-Delivery: 72d3162e-cc78-11e3-81ab-4c9367dc0958
X-Hub-Signature: sha1=7d38cdd689735b008b3c702edd92eea23791c5f6
User-Agent: GitHub-Hookshot/044aadd
Content-Type: application/json
Content-Length: 6615
X-GitHub-Event: issues
{
  "action": "opened",
  "issue": {
    "url": "https://api.github.com/repos/octocat/Hello-World/issues/1347",
    "number": 1347,
    ...
  },
  "repository" : {
    "id": 1296269,
    "full_name": "octocat/Hello-World",
    "owner": {
      "login": "octocat",
      "id": 1,
      ...
    },
    ...
  },
  "sender": {
    "login": "octocat",
    "id": 1,
    ...
  }
}
```

