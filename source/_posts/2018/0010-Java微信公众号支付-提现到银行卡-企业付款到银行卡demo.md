---
title: Java微信公众号支付-提现到银行卡-企业付款到银行卡demo
index_img: /img/cover/10.jpg
categories:
  - 支付
tags:
  - 微信体现
abbrlink: 936960f9
date: 2018-03-06 16:57:36
---
java 微信公众号支付，微信公众号提现到银行卡，企业付款到银行卡demo

开发文档：https://pay.weixin.qq.com/wiki/doc/api/tools/mch_pay.php?chapter=14_1#

开通条件：
    
    1、商户号已入驻90日
    2、商户号有30天连续正常交易
    3、 登录微信支付商户平台-产品中心，开通企业付款。

开始之前认真看一遍开发文档还是很有用的。
![](1.jpg)

#### 1、 调用获取RSA公钥API获取RSA公钥，落地成本地文件（PKCS#1）假设wx_public_key.pem
```java
String url = "https://fraud.mch.weixin.qq.com/risk/getpublickey";
Map<String,String> map = new HashMap<>();
map.put("mch_id",Constant.WXPAY_MCHID);
map.put("nonce_str", WXPayUtil.generateNonceStr());
map.put("sign_type","MD5");
map.put("sign", WXPayUtil.generateSignature(map,Constant.WXPAY_KEY));
String certificate_path = "conf"+File.separator+"apiclient_cert.p12";
String request = CommonUtil.request(url, WXPayUtil.mapToXml(map), certificate_path);
return request;
```
#### 2、 RSA公钥格式PKCS#1转PKCS#8

    1）、openssl安装：http://blog.csdn.net/shiyong1949/article/details/78212971?locationNum=10&fps=1
    2）、命令：openssl rsa -RSAPublicKey_in -in <filename> -pubout
    3）、替换原来wx_public_key.pem文件的内容为转换后的PKCS#8（最终需要用的文件）
如下：
![](2.jpg)
#### java   部分demo
```java
String request = "";
Map<String,String> data = new HashMap<>();
if(WithDrawApplyTypeEnum.WxWalletWithdraw.getKey()==withdrawApply.getType().intValue()){    //提现到零钱
    data = getWithdrawWalletMap(withdrawApply,withdrawApply.getPlatform().intValue());
    request = CommonUtil.request(WXPayConstants.WITHDRAW_WALLET_URL, WXPayUtil.mapToXml(data), Constant.APICLIENT_CERT_P12);
}else if(WithDrawApplyTypeEnum.WxBankWithdraw.getKey()==withdrawApply.getType().intValue()){   //提现到银行卡
    data = getWithdrawBankMap(withdrawApply);
    request = CommonUtil.request(WXPayConstants.WITHDRAW_BANK_URL, WXPayUtil.mapToXml(data), Constant.APICLIENT_CERT_P12);
}
Map<String, String> result = WXPayUtil.xmlToMap(request);
if("SUCCESS".equals(result.get("return_code"))){
    if("FAIL".equals(result.get("result_code"))){
        flag = false;
        msg = result.get("err_code_des");
    }
}else{
    flag = false;
    msg = Message.WX_SERVER_EXCEPTION;
}
if (!flag){
    throw new RuntimeException(msg);
}
```
注：request 方法和generateSignature等方法在之前发的微信付款到零钱博客中有介绍到。

提现部分demo

getWithdrawBankMap
```java
public Map<String,String> getWithdrawBankMap(WithdrawApply withdrawApply) throws Exception{
    Map<String,String> data = new HashMap<>();
    data.put("mch_id",Constant.WXPAY_MCHID);
    data.put("partner_trade_no",withdrawApply.getTradeNo());
    data.put("nonce_str",withdrawApply.getNonceStr());
    String rsa ="RSA/ECB/OAEPWITHSHA-1ANDMGF1PADDING";
    PublicKey pub= RSAUtil.getPubKey(Constant.WX_PUBLIC_KEY,"RSA");
    byte[] estr=RSAUtil.encrypt(withdrawApply.getBankCard().getBytes(),pub,2048, 11,rsa);   //对银行账号进行加密
    byte[] name=RSAUtil.encrypt(withdrawApply.getCardHolder().getBytes(),pub,2048, 11,rsa);   //对银行账号进行加密
    String bankno = Base64Util.encode(estr);//并转为base64格式
    String nameStr = Base64Util.encode(name);
    data.put("enc_bank_no",bankno);
    data.put("enc_true_name",nameStr);
    data.put("bank_code",withdrawApply.getBankId()+"");
    String price = withdrawApply.getPrice().multiply(new BigDecimal(100)).intValue()+"";
    data.put("amount",price);
    data.put("desc","[灵定]提现到银行卡");
    String sign = WXPayUtil.generateSignature(data, Constant.WXPAY_KEY);
    data.put("sign",sign);
    return data;
}
```

```java
public class RSAUtil {

    public static PublicKey getPubKey(String publicKeyPath, String keyAlgorithm){
        PublicKey publicKey = null;
        InputStream inputStream = null;
        try{
            inputStream = new FileInputStream(publicKeyPath);
            publicKey = getPublicKey(inputStream,keyAlgorithm);
        } catch (Exception e) {

            e.printStackTrace();//EAD PUBLIC KEY ERROR
            System.out.println("加载公钥出错!");
        } finally {
            if (inputStream != null){
                try {
                    inputStream.close();
                }catch (Exception e){
                    System.out.println("加载公钥,关闭流时出错!");
                }
            }
        }
        return publicKey;
    }

    public static PublicKey getPublicKey(InputStream inputStream, String keyAlgorithm) throws Exception {
        try
        {
            System.out.println("b1.........");
            BufferedReader br = new BufferedReader(new InputStreamReader(inputStream));
            System.out.println("b2.........");
            StringBuilder sb = new StringBuilder();
            String readLine = null;
            System.out.println("b3.........");
            while ((readLine = br.readLine()) != null) {
                if (readLine.charAt(0) == '-') {
                    continue;
                } else {
                    sb.append(readLine);
                    sb.append('\r');
                }
            }
            X509EncodedKeySpec pubX509 = new X509EncodedKeySpec(decodeBase64(sb.toString()));
            KeyFactory keyFactory = KeyFactory.getInstance(keyAlgorithm);
            //下行出错  java.security.spec.InvalidKeySpecException: java.security.InvalidKeyException: IOException: DerInputStream.getLength(): lengthTag=127, too big.
            PublicKey publicKey = keyFactory.generatePublic(pubX509);
            return publicKey;
        } catch (Exception e) {
            e.printStackTrace();
            throw new Exception("READ PUBLIC KEY ERROR:", e);
        } finally {
            try {
                if (inputStream != null) {
                    inputStream.close();
                }
            } catch (IOException e) {
                inputStream = null;
                throw new Exception("INPUT STREAM CLOSE ERROR:", e);
            }
        }
    }

    /***
     * decode by Base64
     */
    public static byte[] decodeBase64(String input) throws Exception{
        Class clazz=Class.forName("com.sun.org.apache.xerces.internal.impl.dv.util.Base64");
        Method mainMethod= clazz.getMethod("decode", String.class);
        mainMethod.setAccessible(true);
        Object retObj=mainMethod.invoke(null, input);
        return (byte[])retObj;
    }

    public static byte[] encrypt(byte[] plainBytes, PublicKey publicKey, int keyLength, int reserveSize, String cipherAlgorithm) throws Exception {
        int keyByteSize = keyLength / 8;
        int encryptBlockSize = keyByteSize - reserveSize;
        int nBlock = plainBytes.length / encryptBlockSize;
        if ((plainBytes.length % encryptBlockSize) != 0) {
            nBlock += 1;
        }
        ByteArrayOutputStream outbuf = null;
        try {
            Cipher cipher = Cipher.getInstance(cipherAlgorithm);
            cipher.init(Cipher.ENCRYPT_MODE, publicKey);

            outbuf = new ByteArrayOutputStream(nBlock * keyByteSize);
            for (int offset = 0; offset < plainBytes.length; offset += encryptBlockSize) {
                int inputLen = plainBytes.length - offset;
                if (inputLen > encryptBlockSize) {
                    inputLen = encryptBlockSize;
                }
                byte[] encryptedBlock = cipher.doFinal(plainBytes, offset, inputLen);
                outbuf.write(encryptedBlock);
            }
            outbuf.flush();
            return outbuf.toByteArray();
        } catch (Exception e) {
            throw new Exception("ENCRYPT ERROR:", e);
        } finally {
            try{
                if(outbuf != null){
                    outbuf.close();
                }
            }catch (Exception e){
                outbuf = null;
                throw new Exception("CLOSE ByteArrayOutputStream ERROR:", e);
            }
        }
    }


    public static void main(String[] args) throws Exception{
        String rsa ="RSA/ECB/OAEPWITHSHA-1ANDMGF1PADDING";
        PublicKey pub=RSAUtil.getPubKey(Constant.WX_PUBLIC_KEY,"RSA");
        byte[] estr=RSAUtil.encrypt("6222804263000108304".getBytes(),pub,2048, 11,rsa);   //对银行账号进行加密
        String bankno = Base64Util.encode(estr);//并转为base64格式
        System.out.println(bankno);
    }
}
```