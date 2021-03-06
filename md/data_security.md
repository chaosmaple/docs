# 数据和安全

## 安全总览

几乎每一位使用 LeanCloud 的用户都会问，如何能够保证自己应用的安全？对安全的关注说明你也位对产品负责、对用户负责、对自己负责、做事态度认真的开发者，这也正是 [LeanCloud 所信守的价值观](http://open.leancloud.cn)。

安全界有个说法——绝对的安全是不存在的。其中关键要点是要梳理清楚「安全边界」，即你对于业务安全范围的一个清晰的界定。业内一个很著名的例子就是 Chrome 浏览器的查看密码功能，查看已存储的密码不需要再输入 Google 账号的密码。

LeanCloud 通过以下三重级别来总体控制每个应用的安全：

- SSL 安全连接（HTTPS）
- Web 安全域名
- ACL 权限控制

数据层面，云端每天会备份一次数据，防止误操作等情况删除重要数据。其他安全设置还包括云端验证和检测，比如实时通信可以支持自定义的云端认证、短信验证的各种安全设置、SDK 中的各种安全细节等。

每个应用都有自己的 AppId，并且需要通过这个唯一的 id 从服务端申请和调用资源。理论上说这个 AppId 应该严格保密，但是实际中它总会泄露。如果说用反编译原生应用的方法来破解 AppId 还要费些周折，那对于 Web 应用，只要查看到页面源码就能找出 AppId。所以我们要需要做哪些防御？关键点是，我们要能够保证其他人把你的 AppId 偷过去之后，他也无法直接使用你的服务器资源。Web 端可以通过 `Web 安全域名` 来对请求来源做限制，可以简单的防御住 Web 的服务器资源盗取。但是 **安全域名** 对 Native 类的应用又是无效的，所以如果你是使用 Native App 的 SDK，那么设置安全域名就不够了，这个时候就要考虑使用「ACL 权限控制」。

注意，这里每次的调整都是对安全边界的一次次评估，不是每个设置每个应用都需要如此操作。

ACL 权限控制是如何管理安全的呢？举个例子：

比如你要做一个账号系统，这个系统中每个用户账号都有头像，所以你会有一个用户上传头像接口。那么，如果你把这个上传头像的功能放在注册成功之前，每个没有经过你的 ACL 权限认证（没有登录）的用户都可以通过这个接口上传头像，所以你这个上传头像的接口是存在滥用的。如果接口 ACL 权限设置为注册成功的某类用户，则用户必须要经过 ACL 权限认证为是属于某个权限的用户（即登录），并且此时他才可以使用这个接口。所以，上传接口如此，其他的类似功能也是同理，但凡是通过 SDK 或者 API 调用的接口操作，你都要确保他们的 ACL 权限控制是在你的控制范围内。这需要你的智慧和设计来保证安全，相信你也一定能做到。

SSL 其实没有什么好说的，所有的数据都采用加密链路，这样做可以保证数据的私密性。

总之，一切安全设计的背后都是需要你考虑清楚你的 App、你的产品的「安全边界」，制定对应的安全策略。当然安全是后话，首先通过使用 LeanCloud 节省大量时间成本、研发成本、机会成本把产品快速迭代起来才是正经事。

## 安全中心

安全中心，是我们为每个应用提供的设置基本安全的入口，位置在 [控制台 / 设置 / 安全中心](/app.html?appid={{appid}}#/security)。

### 服务开关

服务开关，是用来开启或者关闭当前应用所使用的服务，从根本上防止由于 AppId 和 AppKey 泄露后而可能会引发的服务资源被盗取的问题。

![image](images/security/service-switch.png)

### Web 安全域名

如果在前端使用 JavaScript SDK，当你打算正式发布出去的时候，请务必配置 **Web 安全域名**，方法是进入 [控制台 > 设置 > 安全中心 > **Web 安全域名**](/app.html?appid={{appid}}#/security)。

设置「Web 安全域名」后，仅可在该域名下通过 JavaScript SDK 调用服务器资源，域名配置策略与浏览器域安全策略一致，要求域名协议、域和端口号都需严格一致，不支持子域和通配符。所以如果你要配置一个域名，要写清楚协议、域和端口，缺少一个都可能导致访问被禁止。举例说明一下域名的区别：

```
// 跨域
www.a.com:8080
www.a.com

// 跨域 
www.a.com:8080
www.a.com:80

// 跨域  
a.com
www.a.com

// 跨域
xxx.a.com
www.a.com

// 不同协议，跨域         
http:
https:
```

这样就可以防止其他人通过外网其他地址盗用你的服务器资源。但是要注意，**Web 安全域名**所能达到的目的是防御恶意部署，而不是防御伪造脏数据（恶意用户通过绑定 host 方式还是有可能访问到应用的数据），所以要想对数据进行更多细粒度的控制，需要配合 ACL 来使用。

在 WebView 中使用，建议通过 WebView 去加载一个部署好的、有域名的 Web，然后缓存在本地，这样可以通过 **Web 安全域名** 来做限制。

![image](images/security/web-host.png)

### 操作日志

操作日志中会显示应用创建者及所有协作者的重要操作记录，比如删除数据操作的历史、操作用户名、操作 IP 及操作时间等，这个日志的目的是为了遇到问题更好地定位故障缘由，排查可能的恶意操作，防止应用数据被错误地改动。

## 数据

### 自动备份

LeanCloud 目前会每天备份一次应用数据，防止用户误操作删除了重要数据。如果发生误删除，请及时联系我们进行恢复。

### 有效的数据类型

我们已经仔细设计并实现了客户端 SDK，在你使用 iOS 或者 Android SDK 的时候，通常来说你不需要担心数据是如何保存的。只要简单地往 AVObject 添加数据，它们都能被正确地保存。

尽管如此，有些情况下了解数据如何存储在 LeanCloud 平台上还是有一些用处。

在平台内部，LeanCloud 将数据存储为 JSON，因此所有能被转换成 JSON 的数据类型都可以保存在 LeanCloud 平台上。并且，框架还可以处理日期、Bytes 以及文件类型。总结来说，对象中字段允许的类型包括：

类型|说明
---|---
String|字符串
Number|数字
Boolean|布尔类型
Array|数组
Object|对象，或者 Pointer 
Date|日期
Bytes|Base64 编码的二进制数据
File|文件
Null|空值

Object 类型简单地表示每个字段的值都可以由能 JSON 编码的内嵌对象组合而成。凡是对象的键（key） 包含 `$` 或者 `.`，或者同时有 `__type` 键，都是框架内保留用来做一些额外处理的特殊键，因此请不要在你的对象中使用这样的 Key。

我们的 SDK 会处理原生的 Objective-C 和 Java 类型到 JSON 之间的转换。例如，当你保存一个 NSString 对象的时候，它在我们的系统中会被自动转换成 String 类型。

有两种方式可以存储原生的二进制数据。Bytes 类型允许直接在 AVObject 中关联 NSData 或者 bytes[] 类型的数据。这种方式只推荐用来存储小片的二进制数据。当要保存实际文件（例如图片、视频、文档等），请使用 AVFile 来表示 File 类型，并且 File 类型可以被保存到对象字段中关联起来。

### 数据类型锁定

当一个 Class 初次创建的时候，它不包含任何预先定义并继承的 schema。也就是说对于存储的第一个对象，它的字段可以包含任何有效的类型。

但是，当一个字段被保存至少一次的时候，这个字段将被锁定为保存过的数据类型。例如，如果一个 User 对象保存了一个字段 name，类型为 String，那么这个 name 字段将被严格限制为只允许保存 String 类型。如果你尝试保存其他类型到这个字段，我们的 SDK 会返回错误。

一个特例是任何字段都允许被设置为 null，无论它是什么类型。

### 数据管理

[数据管理](/data.html?appid={{appid}}) 是一个允许在你任意的一个应用里更新或者创建对象的一个 Web 界面的管理平台。在这里，你可以看到保存在 Class里的每个对象的原生 JSON 值。

当使用这个平台的时候，请牢记：

* 输入 `null` 将会将值设为特殊的空值 **null**，而非字符串 `"null"`。
* objectId、createdAt 和 updatedAt 不可编辑，它们都是系统自动设置的。
* 下划线开始的 Class 为系统内置数据表，不可删除，并且请轻易不要修改它的默认字段，但是可以向其中添加字段。

### 导入数据

除了 REST API 之外，我们还提供通过 JSON 文件和 CSV 格式文件的导入数据的功能。

要使用 JSON 文件创建一个新 Class，请进入 [控制台 / 存储 / 数据管理](/data.html?appid={{appid}})，点击左侧 Class 导航栏的小齿轮图标，选择 **数据导入**。

<div class="callout callout-info">数据文件的扩展名必须是 `.csv` 或者 `.json` 结尾，我们以此来判断导入数据的类型。</div>

#### JSON 文件格式

JSON 格式要求是一个符合我们 REST 格式的 JSON 对象数组，或者是一个包含了键名为 results、值为对象数组的 JSON 对象。例如：

```json
{ "results": [
  {
    "likes": 2333,
    "title": "讲讲明朝的那些事儿",
    "author": {
      "__type": "Pointer",
      "className": "Author",
      "objectId": "mQtjuMF5xk"
    },
    "isDraft": false,
    "createdAt": "2015-11-25T17:15:33.347Z",
    "updatedAt": "2015-11-27T19:05:21.377Z",
    "publishedAt": {
      "__type": "Date",
      "iso": "2015-11-27T19:05:21.377Z"
    },
    "objectId": "fchpZwSuGG"
  }]
}
```

【日期】示例中，`publishedAt` 是一个日期型字段，其格式要求请参考 [REST API &middot; 数据类型](rest_api.html#datatype_date)。

【密码】导入用户密码需要使用一个特殊的字段 `bcryptPassword`，并且完全遵循 [Stackoverflow &middot; What column type/length should I use for storing a Bcrypt hashed password in a Database?](http://stackoverflow.com/a/5882472/1351961)  所描述的加密算法加密后，才可以作为合法的密码进行导入。

【关系】导入 Relation 关联数据时，需要填写要导入的 Class 名称、导入后的字段名称、关联的 Class 名称等信息，才能完整导入，例如：

```json
{ "results": [
{
  "owningId": "dMEbKFJiQo",
  "relatedId": "19rUj9I0cy"
},
{
  "owningId": "mQtjuMF5xk",
  "relatedId": "xPVrHL0W4n"
}]}
```
其中：

* `owningId`： 将要导入的 Class 表内已经存在的对象的 objectId。
* `relatedId`：将要关联的 Class 里的对象的 objectId。

例如，Post 有一个字段 comments 是 Relation 类型，对应的 Class 是 Comment，那么 owningId 就是已存在的 Post 的 objectId，而 relatedId 就是关联的 Comment 的 objectId。

### CSV格式文件

导入 Class 的 csv 文件格式必须符合我们的扩展要求：

* 第一行必须是字段的类型描述，支持 `int`、`long`、`number`、`double`、`string`、`date`、`boolean`、`file`、`array`、`object`、`geopoint` 等。
* 第二行是字段的名称
* 第三行开始才是要导入的数据

```csv
string,int,string,double,date
name,age,address,account,createdAt
张三,33,北京,300.0,2014-05-07T19:45:50.701Z
李四,25,苏州,400.03,2014-05-08T15:45:20.701Z
王五,21,上海,1000.5,2012-04-22T09:21:35.701Z
```

导入的 `geopoint` 格式是一个用空格隔开字符串：

```csv
geopoint,string,int,string,double,date
location,name,age,address,account,createdAt
20 20,张三,33,北京,300.0,2014-05-07T19:45:50.701Z
30 30,李四,25,苏州,400.03,2014-05-08T15:45:20.701Z
40 40,王五,21,上海,1000.5,2012-04-22T09:21:35.701Z
```

导入的 Relation 数据，比 JSON 简单一些，第一列对应 JSON 的 `owningId`，也就是要导入的 Class 的存在对象的 objectId，第二列对应 `relatedId`，对应关联 Class 的 objectId。例如：

```csv
dMEbKFJiQo,19rUj9I0cy
mQtjuMF5xk,xPVrHL0W4n
```

csv 导入也支持 Pointer 类型，要求类型声明为 `pointer:类名`，其中类名就是该 Pointer 列所指定的 className，列的值只要提供 objectId 即可，例如：

```csv
string,pointer:Player
playerName,player
张三,mQtjuMF5xk
李四,xPVrHL0W4n
```

### 导出数据

我们还支持你可以导出所有的应用数据（包括加密后的用户密码），只要进入 [控制台 / 应用设置 / 数据导出](/app.html?appid={{appid}}#/export) 点击导出按钮即可开始导出任务。我们会在导出完成之后发送下载链接到你的注册邮箱。

导出还可以限定日期，我们将导出在限定时间内有过更新或者新增加的数据。

我们还提供了数据导出的 [RETS API](./rest_api.html#数据导出_API)。


#### 导出用户数据的加密算法

我们通过一个 Ruby 脚本来描述这个用户密码加密算法：

1. 创建 SHA-512 加密算法 hasher
2. 使用 salt 和 password（原始密码） 调用 hasher.update
3. 获取加密后的值 `hv`
3. 重复 512 次调用 `hasher.update(hv)`，每次hv都更新为最新的 `hasher.digest` 加密值
4. 最终的 hv 值做 base64 编码，保存为 password

假设：

<table width="100%" border="0" cellpadding="6">
  <tbody>
    <tr>
      <td nowrap>salt</td>
      <td><pre style="margin:0;"><code>h60d8x797d3oa0naxybxxv9bn7xpt2yiowz68mpiwou7gwr2</code></pre></td>
    </tr>
    <tr>
      <td nowrap>原始密码</td>
      <td><code>password</code></td>
    </tr>
    <tr>
      <td nowrap>加密后</td>
      <td><pre style="margin:0;"><code>tA7BLW+NK0UeARng0693gCaVnljkglCB9snqlpCSUKjx2RgYp8VZZOQt0S5iUtlDrkJXfT3gknS4rRqjYsd/Ug==</code></pre></td>
    </tr>
  </tbody>
</table>

实现代码：

```ruby
require 'digest/sha2'
require "base64"

hasher = Digest::SHA512.new
hasher.reset
hasher.update "h60d8x797d3oa0naxybxxv9bn7xpt2yiowz68mpiwou7gwr2"
hasher.update "password"

hv = hasher.digest

def hashme(hasher, hv)
  512.times do
    hasher.reset
    hv = hasher.digest hv
  end
  hv
end

result = Base64.encode64(hashme(hasher,hv))
puts result.gsub(/\n/,'')
```

非常感谢用户残圆贡献了一段 C# 语言示例代码：

```
        /// 根据数据字符串和自定义 salt 值，获取对应加密后的字符串
        /// </summary>
        /// <param name="password">数据字符串</param>
        /// <param name="salt">自定义 salt 值</param>
        /// <returns></returns>
        public static string SHA512Encrypt(string password, string salt)
        {
            /*
            用户密码加密算法
            
            1、创建 SHA-512 加密算法 hasher
            2、使用 salt 和 password（原始密码） 调用 hasher.update
            3、获取加密后的值 hv
            4、重复 512 次调用 hasher.update(hv)，每次hv都更新为最新的 hasher.digest 加密值
            5、最终的 hv 值做 base64 编码，保存为 password
            */
            password = salt + password;
            byte[] bytes = System.Text.Encoding.UTF8.GetBytes(password);
            byte[] result;
            System.Security.Cryptography.SHA512 shaM = new System.Security.Cryptography.SHA512Managed();
            result = shaM.ComputeHash(bytes);
            int i = 0;
            while (i++ < 512)
            {
                result = shaM.ComputeHash(result);
            }
            shaM.Clear();
            return Convert.ToBase64String(result);
        }
```

## 安全性

对于任何移动应用来说。因为客户端代码运行在一台移动设备上，因此可能会有不受信任的客户强行修改代码并发起恶意的请求。选择正确的方式来保护你的应用非常重要，但是正确的方式取决于你的应用，以及应用存储的数据。

我们提供多种方式使用权限控制来获得安全性。如果你有关于任何保护你应用安全的最佳方式的问题，我们都鼓励你联系我们的客户支持。

### SSL 加密传输

首先，我们所有的 API 请求都通过 [SSL加密传输](http://zh.wikipedia.org/wiki/%E5%AE%89%E5%85%A8%E5%A5%97%E6%8E%A5%E5%B1%82)，保证传输过程中的数据安全性和可靠性。

### 对象级别的权限

![image](images/acl.png)

最灵活的保护你应用数据安全的方式是通过访问控制列表（Access Control List），通常简称为 ACL 机制。ACL 背后的思想是为每个对象关联一系列 User 或者 Role，这些 User 或者 Role 包含了特定的权限。一个 User 必须拥有读权限（或者属于一个拥有读权限的 Role）才可以获取一个对象的数据，同时，一个 User 需要写权限（或者属于一个拥有写权限的 Role）才可以更改或者删除一个对象。

大多数应用都通过 ACL 来规范它们的访问模式。例如：

* 对于私有数据，read 和 write 都可以限制为对象拥有者（owner）所有。
* 一个信息公告板的帖子，作者和属于「版主」角色的成员可拥有 write 权限，通常 public 允许 read 访问（也就是允许公开读取帖子）。
* 高优先级用户或者开发者创建的数据，例如全局的每日广播消息，可以让 public拥有 read 许可，但是严格限制 write 权限给「管理员」角色。
* 一条从一个用户发往另一个用户的消息，可以将读和写的访问许可限制到关联的这两个用户。

使用 LeanCloud SDK，你可以设置一个默认的 ACL 给客户端所有创建的对象。如果你同时开启自动匿名用户创建的功能，你可以保证你的数据拥有严格限制到每个单独用户的ACL权限。请仔细阅读 iOS 和 Android 指南关于选择默认安全策略的章节。

通过设置 Master Key的 REST API，你还是可以绕过 ACL 限制执行任何操作。这可以让开发者更容易地管理数据。例如，你可以通过 REST API 删除一条私有消息，哪怕这条消息设置为拥有者私有。

代码中如何使用 ACL，请阅读相应的 ACL 文档。

### 列级别的权限

这个概念比较简单，通过编辑数据管理页面某个 Class 的列属性，某一列数据可以设置为「只读」。对于 `_User` 数据表，还可以设置为 「只限当前用户读写」，即只能当前登录的用户读写自己的数据。

某一列的数据还可以设置为 「客户端不可见」。设置了之后，当客户端发起查询的时候，返回的结果将不包含相关字段。比如，匿名发帖的应用，你仍然希望发帖的时候，也记录下真实的作者，但不希望将此信息返回给客户端，所以，这时候就可以设置作者字段为「客户端不可见」。

### Class 级别的权限

![image](images/cla_permission.png)

你可以对整个 Class 设置权限，让它只读或者只写。进入 [控制台 > 存储 > 数据](/data.html?appid={{appid}}#/)，选择一个 Class，再点击右侧菜单中的 **其他** > **权限设置**。

你可以为选中的 Class 禁止客户端执行下列操作的能力：

* GET - 通过objectId获取对象。
* Find - 发起一次对象列表查询。
* Update - 保存一个已经存在并且被修改的对象。
* Create - 保存一个从未创建过的新对象。
* Delete - 删除一个对象。
* Add fields - 添加新字段到class

### 应用安全选项

进入 [控制台 > 设置 > 应用选项](/app.html?appid={{appid}}#/permission)，选中或取消选中列表项目即可启用或者关闭对应的功能。

- 用户账号 > **用户注册时，发送验证邮件**<br/>
  是否要求你应用里的注册用户验证邮箱， 默认不启用。如果启用，每次用户注册，都会发送一封邮件到用户提供的邮箱，要求认证，具体请看数据存储指南的用户一节。
- 其他 > **禁止客户端创建 Class**<br/>
  是否禁止客户端动态创建 Class。如果启用，新的 Class 将只能通过云端管理平台来创建，而无法通过 SDK 或者 REST API 方式来动态创建了。
- 聊天、推送 > **禁止从客户端进行消息推送**<br/>
  是否禁止从客户端推送消息，如果启用，这那么通过 SDK 或者 REST API 都被禁止推送消息，只能通过我们管理平台提供的推送界面来推送消息。

如果想彻底关闭一个应用的数据存储、短信发送、消息推送和聊天（实时通信）功能，请进入 [控制台 > 设置 > 安全中心](/app.html?appid={{appid}}#/security) 来进行相应设置。

## 第三方加密

对于 Android 应用，除了代码混淆之外，还可以使用第三方加密工具，隐藏 classes.dex，通过动态加载的方法进一步提高应用的安全性。下面我们简单介绍一下爱加密。

### 爱加密

[爱加密](http://www.ijiami.cn/) 是专为移动开发者提供安全服务的一个平台，可解决开发者面临的应用安全问题。加密的步骤很简单：

1. 提交应用；
2. 下载加密后的 apk 文件；
3. 下载爱加密提供的签名工具，对应用进行签名。

相关步骤还可以见下面的截图。

申请账号，提交应用，下载签名工具：

![image](images/ijiami_2.png)

加密后重新签名：

![image](images/ijiami_1.png)

这样得到的 apk 文件，普通的反编译之后得到的是，

![image](images/decompile.png)


可以看到，代码被隐藏起来了，应用被破解的难度大幅增加了。

我们一直努力提供更多功能给开发者来保护你的应用，也希望大家持续地给我们反馈，感谢。
