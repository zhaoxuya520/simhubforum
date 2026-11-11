当前补丁版改用新长期签名；不承诺覆盖旧签名版本。请先导出应用内数据备份，卸载旧版后安装，并在 eSTK ARA-M 添加新证书授权。

包含双芯片切换事务、身份错配诊断、全球运营商标准模板及配置覆盖编辑澄清。新增标准模板未经过运营商确认，不保证 DITO 或所有运营商注册成功；尚未实机验证。

- 密钥别名：`simhub-longterm`
- eSTK ARA-M 证书 SHA-1：`CAEAEE369CA8933EC6A80050ECAF22D87B155006`
- 证书 SHA-256：`50FE8B357EFB2F2397B1F822012A5D7E48A4FC1CAF5A0F8A4F0239FFADAB3C7E`
- RSA 3072 / SHA256withRSA；证书到期日：2056-11-11。
- PKCS#12 含私钥，使用强随机密码、PBES2/PBKDF2-HMAC-SHA256/AES-256-CBC，600000 次迭代。密码单独交付给仓库所有者，不上传到 GitHub。
- 请保存密钥库和密码，用于以后相同签名更新；公开证书及 signing-info.json 不含密码或私钥。

验证：357 项 JVM 断言、16 项协程流程模拟、9 项预设数据测试、4 项签名策略回归通过。APK 已核验 v2/v3 签名、实际证书与应用内签名白名单一致、签名前后内容不变、16 KB 对齐及 ZIP 完整性。详见 apk-validation.json 和 jvm-tests.json。
