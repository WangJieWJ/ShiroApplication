# ref: https://github.com/waylau/apache-shiro-1.2.x-reference/blob/master/I.%20Overview%20%E6%80%BB%E8%A7%88/2.%20Tutorial%20%E6%95%99%E7%A8%8B.md

# Shiro几乎所有的事情都和一个中心组件SecurityManager有关。

# Shiro使用
```
为了保护我们的程序安全，我们或许问自己最多的就是“谁是当前的用户？”，“或者当前登录用户的是否允许做某件事？”
通常我们会在写代码或者设计用户接口的时候问这些问题：程序通常建立在用户基础上，
程序功能展示(和安全)也基于每一个用户。所以，通常我们考虑我们程序安全的方法也建立在当前用户的基础上，
Shiro的API提供了“the current user”概念，即Subject

在几乎所有的环境中，你可以通过如下语句得到当前用户的信息：
Subject currentUser=SecurityUtils.getSubject();
```

# Architecture 架构
![Shiro架构图](shiro.png)

### Subject：
```
就像我们在上一章示例中提到的那样，Subject本质上是当前运行用户特定的'View'(视图)，而单词'User'经常暗指一个人，
Subject可以是一个人，但也可以是第三方服务、守护进程账户、时钟守护任务或者其他--当前和软件交互的任何事件。
Subject实例都和(也需要)一个Security Manager绑定，当你和一个Subject进行交互，这些交互动作被转换成SecirityManager下Subject特定的交互动作。
```

### SecurityManager：
```
SecurityManager是Shiro架构的核心，配合内部安全组件共同体组成安全伞。然而，一旦一个程序配置好了SecurityManager和他的内部对象，
SecurityManager通常独自留下来，程序开发人员几乎花费所有时间都集中在Subject API上。
任何Subject的安全操作中SecurityManager是幕后真正的举重者。
```

### Realms：
```
Realms是Shiro和你的程序安全数据之间的“桥”或者“连接”，它用来实际和安全相关的数据如用户执行身份认证(登陆)的账号和授权(访问控制)进行交互，
Shiro从一个或多个程序配置的Realm中查找这些东西。Realm本质上是一个特定的安全DAO：他封装与数据源连接的细节，得到Shiro所需的相关的数据。
在配置Shiro的时候，你必须指定至少一个Realm来实现认证(authentication)和/或授权(authorization)。SecurityManager可以配置多个复杂的Realm，
但至少有一个是需要的。Shiro提供开箱即用的Realms来连接安全数据源(或叫地址)如LDAP、JDBC、文件配置如INI和属性文件等，
如果已有Realms不能满足你的需求你也可以开发自己的Realm实现。和其他内部组件一样，Shiro  SecurityManager管理如何使用Realms获取实例多代表的安全和身份信息。
```

# Configuration 配置
```
Shiro可以在任何环境下工作，从简单的命令行到大型企业企业级集群项目，因为环境的多样化，可以通过许多途径来配置当前环境的配置方式进行配置。
```

## Programmatic Configuration 在程序中配置
创建一个SecurityManager并使之可用的简单的方法就是创建一个org.apache.shiro.mgt.DefaultSecurityManager 对象并将其写入到代码中，例如
```
Realm realm =//实例化获得一个Realm实例，
SecurityManager securityManager=new DefaultSecurityManager(realm);
//使SecurityManager实例通过静态存储器对整个应用程序可见：
SecurityUtils.setSecurityManager(securityManager);
```
仅仅三行代码，你就可以拥有一个适用于任何程序的功能全面的 Shiro 环境，多么简单。

### SecurityManager Object Graph
如同我们在架构（Architecture ）中讨论过的，Shiro SecurityMangger 本质上是一个由一套安全组件组成的对象模块视图（graph），因为与 JavaBean兼容，所以可以对所有这些组件调用的 getter 和 setter 方法来配置SecurityManager 和它的内部对象视图。

例如，你想用一个自定义的 SessionDAO 来定制 Session Management从而配置一个 SecurityManager 实例，你就可以使用 SessionManager 的 setSessionDAO 方法直接 set 这个 SessionDAO。
```
DefaultSecurityManager securityManager = new DefaultSecurityManager(realm);
SessionDAO sessionDAO = new CustomSessionDAO();
((DefaultSessionManager)securityManager.getSessionManager()).setSessionDAO(sessionDAO);
```
使用这些函数，你可以配置 SecurityManager 视图（graph）中的任何一部分。

虽然在程序中配置很简单，但它并不是我们现实中配置的完美解决方案。在几种情况下这种方法可能并不适合你的程序：

+ 它需要你确切知道并实例化一个直接实现（direct implementation），然而更好的做法是你并不需要知道这些实现也不需要知道从哪里找到它们。
+ 因为JAVA类型安全的特性，你必须对通过 get* 获取的对象进行强制类型转换，这么多强制转换非常的丑陋、累赘并且会和你的类紧耦合。
+ SecurityUtils.setSecurityManager 方法会将 SecurityManager 实例化为虚拟机的单独静态实例，在大多数程序中没有问题，但如果有多个使用 Shiro 的程序在同一个 JVM 中运行时，各程序有自己独立的实例会更好些，而不是共同引用一块静态内存。
改变配置就需要重新编译你的程序。

然而，尽管有这些不足，在程序中定制的这种方法在限制内存（memory-constrained ）的环境中还是很有价值的，像智能电话程序。如果你的程序不是运行在一个限制内存的环境中，你会发现基于文本的配置会更易读易用。

## INI Configuration 配置
通过ini配置文件来进行配置，
它假定所有对象都是兼容 JavaBean 的 POJO。在设置这些属性时，Shiro 默认使用 Apache 通用的 BeanUtils 来完成这项复杂的工作，所以虽然 INI 值是文本，BeanUtils 知道如何将这些字符串值转换为适合的原始值类型并调用合适的 JavaBeans 的 setter 方法

## Authentication认证
认证：就是身份验证的过程---也就是证明一个用户的真实身份。为了证明用户的身份，需要提供系统理解和相信的身份信息和证据。

需要通过向Shiro提供用户的身份(principals)和证明(credentials)来判断是否和系统要求的匹配。

+ Principals(身份)是Subject的"标识属性"，可以使任何与Subject相关的标识，比如名称(给定名称)、名字(姓名或者昵称)、用户名、安全号码等等。当然像昵称这样的内容不能很好的对Subject进行独特标识，所以最好的身份信息（Principals）是使用在程序中唯一的标识--典型的使用用户名或邮件地址
+ + Primary Principal(最主要的身份)虽然 Shiro 可以使用任何数量的身份，Shiro 还是希望一个程序精确地使用一个主要的身份--一个仅有的唯一标识 Subject 值。在多数程序中经常会是一个用户名、邮件地址或者全局唯一的用户 ID。
+ Credentials(证明) 通常是只有 Subject 知道的机密内容，用来证明他们真正拥有所需的身份，一些简单的证书例子如密码、指纹、眼底扫描和X.509证书等。

最常见的身份/证明是用户名和密码，用户名是所需的身份说明，密码是证明身份的证据。如果一个提交的密码和系统要求的一致，程序就认为该用户身份正确，因为其他人不应该知道同样的密码。

## Authenticating Subjects
Subject验证的过程可以有效地划分为以下三个步骤：
1、收集Subject提交的身份和证明
2、向Authentication提交身份和证明
3、如果提交的内容正确，允许访问，否则重新尝试或者拒绝访问。

### 第一步：收集用户身份和证明
```
//最常用的情况是username/password 对：
UsernamePasswordToken token=new UsernamePasswordToken(username,password);

//"Remember Me"功能是内建的
token.setRememberMe(true);
```
在这里我们使用 UsernamePasswordToken，支持所有常用的用户名/密码验证途径，这是一个 org.apache.shiro.authc.AuthenticationToken 接口的实现，这个接口被 Shiro 认证系统用来提交身份和证明

Shiro并不关心你如何获取这些信息：也许是用户从一个HTML表单中提交的或者其他方式获得的但是这与AuthenticationToken完全无关。

你可以随自己喜欢构造和引用AuthenticationToken实例 -- 这与协议无关

### 第二步：提交身份和证明
当身份和证明被收集并实例化为一个AuthenticationToken(认证令牌)后，我们需要向 Shiro 提交令牌以执行真正的验证尝试：
```
Subject currentUser=SecurityUtils.getSubject();

currentUser.login(token);
```
在获取当前执行的 Subject 后，我们执行一个单独的 login 命令，将之前创建的 AuthenticationToken 实例传给它。

调用 login 方法将有效地执行身份验证。

### 第三部：处理成功或失败
当login函数没有返回信息时表示验证通过了。程序可以继续运行，此时执行SecurityUtils.getSubject()将返回验证后的Subject实例，subject。isAuthenticated()将返回true

但是如果 login 失败了呢？例如，用户提供了一个错误的密码或者因访问系统次数过多而被锁定将会怎样呢？

Shiro拥有丰富的运行期异常AuthenticationException可以精确标明为何验证失败，你可以将 login 放入到 try/catch 块中并捕获所有你想捕获的异常并对它们做出处理。例如：
```
try {
    currentUser.login(token);
} catch ( UnknownAccountException uae ) { ...
} catch ( IncorrectCredentialsException ice ) { ...
} catch ( LockedAccountException lae ) { ...
} catch ( ExcessiveAttemptsException eae ) { ...
} ... 捕获你自己的异常 ...
} catch ( AuthenticationException ae ) {
    //未预计的错误?
}

//没问题，继续
```
如果原有的异常不能满足你的需求，可以创建自定义的AuthenticationExceptions 来表示特定的失败场景。

## Remembered VS. Authenticated
Shiro支持在登录过程中执行"remember me",在此值得指出，一个以及驻的Subject(remembered Subject)与
一个正常通过认证的Subject(authenticated Subject)在Shiro是完全不同的。

+ 记住的(Remembered):一个被记住的Subject没有已知身份(也就是说subject.getPrincipals()返回空),
但是他的身份被先前的认证过程记住，并存于先前session中，一个被认为记住的对象在执行subject.isRemembered()返回真。
+ 已验证(Authenticated):一个被验证的Subject是成功验证后(如登录成功)并存在于当前的session中，一个被认为验证过的对象调用subject.isAuthenticated()将返回真。

互斥的：已记住(Remembered) 和 已验证(Authenticated)是互斥的。一个标识值为真另一个就为假，反过来也一样。


## Logging Out 退出
与验证相对的是释放所有已知的身份信息，当Subject与程序不再交互了，你可以使用subject.logout()丢掉所有身份信息
```
currentUser.logout(); //清楚验证信息，使session失效
```
当你调用 logout，任何现存的 session 将变为不可用并且所有的身份信息将消失（如：在 web 程序中，RememberMe 的 Cookie 信息同样被删除）。

Web程序需注意：
因为在 Web 程序中记住身份信息往往使用 Cookies，而 Cookies 只能在 Response 提交时才能被删除，所以强烈要求在为最终用户调用subject.logout() 之后立即将用户引导到一个新页面，确保任何与安全相关的 Cookies 如期删除，这是 Http 本身 Cookies 功能的限制而不是 Shiro 的限制


## Authentication Sequence认证序列
![Shiro架构图](Shiro2.png)

+ 第一步：程序代码调用Subject.login方法，向AuthenticationToken(认证令牌)实例的构造函数传递用户的身份和证明。
+ 第二步：Subject实例，通常是一个DelegatingSubject(或其子类)通过调用securityManager.login(token)将这个令牌转交给程序SecurityManager。
+ 第三步：SecurityManager，基本的"安全伞"组件，得到令牌并通过调用 authenticator.authenticate(token)简单地将其转交其内部的Authenticator实例，
大部分情况下是一个ModularRealmAuthenticator实例，用来支持在验证过程中协调一个或者多个Realm实例。ModularRealmAuthenticator本质上为Apache Shiro（在 PAM 术语中每一个 Realm 称为一个“模块”）提供一个 PAM 类型的范例。
+ 第四步：如程序配置了多个Realm，ModularRealmAuthenticator实例将使用其配置的AuthenticationStrategy开始一个或者多个Realm身份验证的尝试。在Realm被验证调用的整个过程中，AuthenticationStrategy(安全策略)被调用用来回应每个Realm结果，我们将稍后讨论AuthenticationStrategies。

注意：单Realm程序
如果仅有一个Realm被配置，他直接被调用--在单Realm程序中不需要AutheticationStrategy。
+ 第五步：每一个配置的Realm都被检验看成其是否支持提交的AuthenticationToken。如果支持，则该 Realm 的 getAuthenticationInfo 方法随着提交令牌被调用，getAuthenticationInfo 方法为特定的 Realm 有效提供一次独立的验证尝试

### Authenticator

Shiro SecurityManager implementations默认使用一个 ModularRealmAuthenticator 实例， ModularRealmAuthenticator 同样支持单 Realm 和多 Realm

在一个单 Realm 程序中，ModularRealmAuthenticator 将直接执行单独的 Realm，如果配置有两个或以上 Realm，将会使用AuthenticationStrategy 实例来协调如何进行验证，我们将在下面的章节中讨论 AuthenticationStrategy。

如果你希望用自定义的 Authenticator 实现配置 SecurityManager，你可以在 shiro.ini 中做这件事，如：
```
[main]
····
authenticator = com.wangjie.customerAuthenticator;

securityManager.authenticator= $authenticator;
```
尽管在实际操作中，ModularRealmAuthenticator使用于大部分需求

### AuthenticationStrategy
当一个程序中定义了两个或者多个realm时，ModularRealmAuthenticator使用一个内部的AuthenticationStrategy 组件来决定一个验证是否成功。

例如，如果一个Realm 验证一个 AuthenticationToken 成功，但其他的都失败了，那这次尝试是否被认为是成功的呢？是不是所有 Realm 验证都成功了才认为是成功？
又或者一个 Realm 验证成功，是否还有必要讨论其他Realm？AuthenticationStrategy 根据程序需求做出恰当的决定。

AuthenticationStrategy 是一个 stateless 的组件，在整个验证过程中在被用到4次（在这4次活动中需要必要的 state 时，state 将作为方法参数传递）
+ 在任何 Realms 被执行之前；
+ 在某个的 Realm 的 getAuthenticationInfo 方法调用之前；
+ 在某个的 Realm 的 getAuthenticationInfo 方法调用之后；
+ 在所有的 Realm 被执行之后。
AuthenticationStrategy 还有责任从每一个成功的 Realm 中收集结果并将它们“绑定”到一个单独的 AuthenticationInfo，
这个AuthenticationInfo 实例是被 Authenticator 实例返回的，并且 shiro 用它来展现一个 Subject 的最终身份（也就是 Principals ）

Subject 身份“展示（view）”

如果你在程序中使用多于一个的 Realm 从多个数据源中获取帐户数据，程序可看到的是 AuthenticationStrategy 最终负责 Subject 身份最终“合并（merged）”的视图

Shiro有3个具体的AuthenticationStrategy实现：

<table>
<tr>
    <th>AuthenticationStrategy  class</th>
    <th>Description</th>
</tr>
<tr>
    <td>AtLeastOneSuccessfulStrategy</td>
    <td>如果有一个或多个Realm验证成功，所有的尝试都被认为是成功的，如果没有一个验证成功，则该次尝试失败</td>
</tr>
<tr>
    <td>FirstSuccessfulStrategy</td>
    <td>只有从第一个成功验证的Realm返回的信息会被调用，以后的Realm将被忽略，如果没有一个验证成功，则该次尝试失败</td>
</tr>
<tr>
    <td>AllSuccessfulStrategy</td>
    <td>所有配置的Realm在全部尝试中都成功验证才被认为是成功，如果有一个验证不成功，则该次尝试失败</td>
</tr>
</table>

ModularRealmAuthenticator 默认使用 AtLeastOneSuccessfulStrategy 实现，这也是最常用的策略，然而你也可以配置你希望的不同的策略。
```shiro.ini
[main]
···
authcStrategy = org.apache.shiro.authc.FirstSuccessfulStrategy

securityManager.authenticator.authenticatStrategy = $authcStrategy
```

自定义的 AuthenticationStrategy

如果你希望创建你自己的 AuthenticationStrategy 实现，你可以使用 org.apache.shiro.authc.pam.AbstractAuthenticationStrategy 作为起始点。
AbstractAuthenticationStrategy 类自动实现 '绑定（bundling）'/聚集（aggregation）行为将来自于每个Realm 的结果收集到一个 AuthenticationInfo 实例中。

### Realm 验证的顺序
非常重要的一点是，和Realm交互的ModularRealmAuthenticator按迭代(iteration)顺序执行。

ModularRealmAuthenticator可以访问为SecurityManager配置的Realm实例，
当尝试一次验证时，它将在集合中遍历，支持对提交的 AuthenticationToken 处理的每个 Realm 都将执行 Realm 的 getAuthenticationInfo 方法。

### 隐含的顺序
在使用ShiroINI配置文件形式时，你可以按你希望其处理AuthenticationToken 的顺序来配置 Realm，例如，在shiro.ni 中，Realm 将按照他们在INI文件中定义的顺序执行。
```
blahRealm = com.company.blah.Realm
...
fooRealm = com.company.foo.Realm
...
barRealm = com.company.another.Realm
```

SecurityManager上配置了这三个 Realm，在一个验证过程中，blahRealm, fooRealm, 和 barRealm 将被顺序执行。
这基本上与定义下面这一行语句的效果相同：

```
securityManager.realms = $blahRealm, $fooRealm, $barRealm
```
使用这种方法，你不需要设置 securityManager 的 realm 顺序，每一个被定义的realm 将自动加入到 realms 属性中。

### Explicit Ordering明确的顺序
如果你希望明确定义 realm 执行的顺序，不管他们如何被定义，你可以设置 SecurityManager 的 realms 属性，例如，使用上面定义的 realm，但你希望 blahRealm 最后执行而不是第一个：
```
blahRealm = com.company.blah.Realm
...
fooRealm = com.company.foo.Realm
...
barRealm = com.company.another.Realm

securityManager.realms = $fooRealm, $barRealm, $blahRealm
```
明确 Realm 包含

当你明确的配置 securityManager.realms 属性时，只有被引用的 realm 将为 SecurityManager 配置，也就是说你可能在 INI 中定义了5个 realm，但实际上只使用了3个，如果在 realm 属性中只引用了3个，这和隐含的 realm 顺序不同，在那种情况下，所有有效的 realm 都会用到。

https://github.com/waylau/apache-shiro-1.2.x-reference/blob/master/II.%20Core%20%E6%A0%B8%E5%BF%83/6.%20Authorization%20%E6%8E%88%E6%9D%83.md


