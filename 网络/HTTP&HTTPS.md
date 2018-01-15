# Hostname XXX not verified

切换为https时报错，原因是CA证书有多个签发机构，服务端配置的CA证书可能在手机rom中没有

解决方案

```
mClient = new OkHttpClient.Builder()
            .hostnameVerifier(new HostnameVerifier() {
                @Override
                public boolean verify(String hostname, SSLSession session) {
                    return true;
                }
            })
            .build();
```