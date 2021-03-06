# 账户管理

标签：``java-sdk`` ``设置账户``

----

java sdk提供账户管理接口，支持以下功能：

- **账户加载**: 从指定路径加载账户，同时支持`pem`和`p12`格式的账户文件加载

- **账户生成**: 随机生成账户公私钥对

- **账户保存**: 保存账户信息

- **提供加载`p12`和`pem`文件解析的接口**

```eval_rst
.. note::
    java sdk提供了接口级别的账户生成方法，工具级别的账户生成脚本请参考 `get_account.sh脚本 <../../app_dev/account.html>`_ .
```



## 1. 账户加载

java sdk的`org.fisco.bcos.sdk.crypto.CryptoSuite`提供账户加载功能，默认从配置文件的`[account]`配置项加载交易发送账户，具体请参考[这里](./configuration.html#id6).

从指定`pem`账户文件加载交易发送账户的示例如下(client初始化方法请参考[快速入门](./quick_start.html#id4))：

```java
// 从pemAccountFilePath指定路径加载pem账户文件，并将其设置为交易发送账户
public void loadPemAccount(Client client, String pemAccountFilePath)
{
    // 通过client获取CryptoSuite对象
    CryptoSuite cryptoSuite = client.getCryptoSuite();
    // 加载pem账户文件
    cryptoSuite.loadAccount("pem", pemAccountFilePath, null);
}
```

从指定的`p12`账户文件加载交易发送账户的示例如下：

```java
// 从p12AccountFilePath指定的路径加载p12账户文件，并将其设置为交易发送账户
public void loadP12Account(Client client, String p12AccountFilePath, String password)
{
    // 通过client获取CryptoSuite对象
    CryptoSuite cryptoSuite = client.getCryptoSuite();
    // 加载pem账户文件
    cryptoSuite.loadAccount("p12", p12AccountFilePath, password);
}
```



## 2. 账户生成

java sdk的`org.fisco.bcos.sdk.crypto.CryptoSuite`提供了账户生成功能。

随机生成非国密账户示例如下：

```java
// 创建非国密类型的CryptoSuite
CryptoSuite cryptoSuite = new CryptoSuite(CryptoType.ECDSA_TYPE);
// 随机生成非国密公私钥对
CryptoKeyPair cryptoKeyPair = cryptoSuite.createKeyPair();
// 获取账户地址
String accountAddress = cryptoKeyPair.getAddress();
```

随机生成国密账户示例如下：

```java
// 创建国密类型的CryptoSuite
CryptoSuite cryptoSuite = new CryptoSuite(CryptoType.SM_TYPE);
// 随机生成国密公私钥对
CryptoKeyPair cryptoKeyPair = cryptoSuite.createKeyPair();
// 获取账户地址
String accountAddress = cryptoKeyPair.getAddress();
```



## 3. 账户保存

当没有自定义加载账户，也没有通过配置文件配置账户信息时(账户配置请参考[这里](./configuration.html#id6))，java sdk会随机生成账户发送交易，java sdk的`org.fisco.bcos.sdk.crypto.CryptoSuite`提供账户保存功能，可将随机生成的账户保存在指定路径。

以`pem`的格式保存账户文件到指定路径的示例如下：

```java
// 将随机生成的账户信息保存在pemFilePath指定的路径
public void saveAccountWithPem(CryptoKeyPair cryptoKeyPair, String pemFilePath)
{
    // 以pem的格式保存账户文件到pemFilePath路径
    cryptoKeyPair.storeKeyPairWithPem(pemFilePath);
}

// 将随机生成的账户信息保存在账户配置${keyStoreDir}指定的目录下
public void saveAccountWithPemToKeyStoreDir(CryptoKeyPair cryptoKeyPair)
{
    // 账户文件名为${accountAddress}.pem
    cryptoKeyPair.storeKeyPairWithPemFormat();
}
```

以`p12`的格式保存账户文件到指定路径的示例如下：

```java
public void saveAccountWithP12(CryptoKeyPair cryptoKeyPair, String p12FilePath, String password)
{
    // 以p12的格式将账户文件保存到p12FilePath路径
    cryptoKeyPair.storeKeyPairWithP12(p12FilePath, password);
}

// 将随机生成的账户信息保存在账户配置${keyStoreDir}指定的目录下
public void saveAccountWithP12ToKeyStoreDir(CryptoKeyPair cryptoKeyPair, String password)
{
    // 账户文件名为${accountAddress}.p12
    cryptoKeyPair.storeKeyPairWithP12Format(password);
}
```
java sdk随机生成的账户信息可通过如下方法获取：

```java
public CryptoKeyPair getCreatedCryptoKeyPair(Client client)
{
    return client.getCryptoSuite().getCryptoKeyPair();
}
```



## 4. `p12`和`pem`文件解析接口

java sdk的`org.fisco.bcos.sdk.crypto.keystore.KeyManager`提供`p12`和`pem`文件解析接口。

从指定文件加载`pem`文件示例如下：

```java
public KeyManager loadPem(String pemFilePath)
{
    return new PEMManager(pemFilePath);
}
```

从指定文件加载`pem`文件示例如下：

```java
public KeyManger loadP12(String p12FilePath, String password)
{
    return new P12Manager(p12FilePath, password);
}
```

`pem`和`p12`解析后生成的`KeyManager`对象提供了访问公私钥信息的接口如下：

```java
// 获取公私钥对信息
public KeyPair getKeyPair();
```

此外，`org.fisco.bcos.sdk.crypto.CryptoSuite`也提供了将`java.security.KeyPair`类型的公私钥信息转换为`CryptoKeyPair`的功能，示例如下：

```java
// KeyManager中维护的是非国密公私钥信息
public CryptoKeyPair loadKeyManager(KeyManager keyManager)
{
    KeyPair keyPair = keyManager.getKeyPair();
    CryptoSuite cryptoSuite = new CryptoSuite(CryptoType.ECDSA_TYPE);
    return cryptoSuite.createKeyPair(keyPair);
}

// KeyManager中维护的是国密公私钥信息
public CryptoKeyPair loadGMKeyManager(KeyManager GMKeyManager)
{
    KeyPair keyPair = GMKeyManager.getKeyPair();
    CryptoSuite cryptoSuite = new CryptoSuite(CryptoType.SM_TYPE);
    return cryptoSuite.createKeyPair(keyPair);
}
```