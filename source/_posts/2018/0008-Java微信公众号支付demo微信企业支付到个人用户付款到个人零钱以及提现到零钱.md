---
title: Java微信公众号支付demo微信企业支付到个人用户付款到个人零钱以及提现到零钱
index_img: /img/cover/08.jpg
categories:
  - 支付
tags:
  - 微信支付
abbrlink: dba677ad
date: 2018-02-08 10:00:56
---
微信提现到零钱（微信内部交易，不需要手续费）

价格格式错误：可能是价格经过计算后.doubleValue了一下 最后价格是 200.00

企业付款签名错误：和支付的时候签名的参数有出入（搞了半天，最后到看了一段php代码后做了一下调整。OK了)

参考PHP博客：http://blog.csdn.net/sinat_35861727/article/details/72843383
```java
Map<String,String> data = new HashMap<>();
data.put("mch_appid",Constant.WX_APPID);
data.put("mchid",Constant.WXPAY_MCHID);
data.put("nonce_str",withdrawApply.getNonceStr());
data.put("partner_trade_no",withdrawApply.getId()+"");
data.put("openid",withdrawApply.getWxOpenid());
data.put("check_name","NO_CHECK");
String price = withdrawApply.getPrice().multiply(new BigDecimal(100)).intValue()+""; //价格一定要保证int类型字符串  不能又小数点
data.put("amount",price);
data.put("desc","用户"+withdrawApply.getUid()+"提现");
data.put("spbill_create_ip",getIpAddress());
String sign = WXPayUtil.generateSignature(data, Constant.WXPAY_KEY);  
data.put("sign",sign);
String xml = WXPayUtil.mapToXml(data);
String url = "https://api.mch.weixin.qq.com/mmpaymkttransfers/promotion/transfers";
String certificate_path = "conf"+File.separator+"apiclient_cert.p12";   //证书路径
String request = CommonUtil.request(url, xml, certificate_path);
enerateSignature（）：
```
```java
/**
* 生成签名. 注意，若含有sign_type字段，必须和signType参数保持一致。
*
* @param data 待签名数据
* @param key API密钥
* @param signType 签名方式
* @return 签名
*/
public static String generateSignature(final Map<String, String> data, String key, SignType signType) throws Exception {
    Set<String> keySet = data.keySet();
    String[] keyArray = keySet.toArray(new String[keySet.size()]);
    Arrays.sort(keyArray);
    StringBuilder sb = new StringBuilder();
    for (String k : keyArray) {
    if (k.equals(WXPayConstants.FIELD_SIGN)) {
        continue;
    }
    if (data.get(k).trim().length() > 0) // 参数值为空，则不参与签名
        sb.append(k).append("=").append(data.get(k).trim()).append("&");
    }
    sb.append("key=").append(key);
    if (SignType.MD5.equals(signType)) {
        return MD5(sb.toString()).toUpperCase();
    }else if (SignType.HMACSHA256.equals(signType)) {
        return HMACSHA256(sb.toString(), key);
    }else {
        throw new Exception(String.format("Invalid sign_type: %s", signType));
    }
}
```
```java
/**
* 生成 MD5
*
* @param data 待处理数据
* @return MD5结果
*/
public static String MD5(String data) throws Exception {
    MessageDigest md = MessageDigest.getInstance("MD5");
    byte[] array = md.digest(data.getBytes("UTF-8"));
    StringBuilder sb = new StringBuilder();
    for (byte item : array) {
        sb.append(Integer.toHexString((item & 0xFF) | 0x100).substring(1, 3));
    }
    return sb.toString().toUpperCase();
}
```
```java
/**
* 生成 HMACSHA256
* @param data 待处理数据
* @param key 密钥
* @return 加密结果
* @throws Exception
*/
public static String HMACSHA256(String data, String key) throws Exception {
    Mac sha256_HMAC = Mac.getInstance("HmacSHA256");
    SecretKeySpec secret_key = new SecretKeySpec(key.getBytes("UTF-8"), "HmacSHA256");
    sha256_HMAC.init(secret_key);
    byte[] array = sha256_HMAC.doFinal(data.getBytes("UTF-8"));
    StringBuilder sb = new StringBuilder();
    for (byte item : array) {
        sb.append(Integer.toHexString((item & 0xFF) | 0x100).substring(1, 3));
    }
    return sb.toString().toUpperCase();
}

```