---
title: Java中FTP和SFTP的简单使用
index_img: /img/cover/07.jpg
categories:
  - FTP和SFTP
tags:
  - SFTP
  - FTP
abbrlink: cc067b24
date: 2018-01-14 21:57:52
---
    FTP和SFTP的区别和工作原理：
    很详细：http://blog.csdn.net/cuker919/article/details/6403925
可以根据自己的实际情况选择使用FTP或者SFTP，使用FTP时要安装FTP，而SFTP不需要安装。

效率：FTP>SFTP 

安全：SFTP>FTP

这儿只是比较SFTP 读取image和URL读取image效率：适用文件服务和系统服务是不同服务器。
```java
public class SFTP {
    public static void testSftp() throws Exception{
        ChannelSftp sftp = ChannelSFTPImpl.getSFTP();
        String directory = "/data/apps/c2m/release/images-src";
        String downloadFile = "/2S8959_20180114162259.jpg";
        long s = System.currentTimeMillis();
        InputStream is = sftp.get(directory+downloadFile);
        BufferedImage image = ImageIO.read(is);
        long e = System.currentTimeMillis();
        System.out.println(e-s);
    }
    public static void testUrl() throws Exception{
        String str = "http://h5.jinzhiyun.net/images-src/2S8959_20180114162259.jpg";
        URL url = new URL(str);
        long s = System.currentTimeMillis();
        BufferedImage image = ImageIO.read(url);
        long e = System.currentTimeMillis();
        System.out.println(e-s);
    }
    public static void main(String[] args) throws Exception {
        testSftp();  //SFTP      198ms
        testUrl();   //URL        497ms
    }
}
```
//执行结果
![xx](20180114221311479.png)

ChannelSFTPImpl.java
```java
public class ChannelSFTPImpl {
    private static final String host = "";
    private static final int port = 22;
    private static final String username = "";
    private static String password = "";
    public static ChannelSftp sftp = null;
    private  ChannelSFTPImpl() {
    }
    public static ChannelSftp getSFTP() {
        if(null == sftp){
            MySFTP sf = new MySFTP();
            sftp = sf.connect(host, port, username, password);
        }
        return sftp;
    }
}
```
MySFTP.java   connect()
```java
public ChannelSftp connect(String host, int port, String username, String password) {
    ChannelSftp sftp = null;
    try {
        JSch jsch = new JSch();
        jsch.getSession(username, host, port);
        Session sshSession = jsch.getSession(username, host, port);
        sshSession.setPassword(password);
        Properties sshConfig = new Properties();
        sshConfig.put("StrictHostKeyChecking", "no");
        sshSession.setConfig(sshConfig);
        sshSession.connect();
        com.jcraft.jsch.Channel channel = sshSession.openChannel("sftp");
        channel.connect();
        sftp = (ChannelSftp) channel;
    } catch (Exception e) {
    }
    return sftp;
}
```
FTP：从FTP服务器下载文件：
```java
public class FtpUtil {
    public static void main(String[] args) {
        String url = "";
        int port = 21;
        String username = "";
        String password = "";
        String remotePath = "/data/apps/c2m/release/images-src";
        String fileName = "";
        downFile(url, port, username, password, remotePath, fileName);
    }

    /**
     * Description: 从FTP服务器下载文件
     */
    public static boolean downFile(String url, int port, String username, String password, String remotePath, String fileName) {
        boolean success = false;
        FTPClient ftp = new FTPClient();
        int reply;
        try {
            ftp.enterLocalPassiveMode();//本地模式
            ftp.connect(url, port);
            ftp.login(username, password);
            ftp.setFileType(FTPClient.BINARY_FILE_TYPE);//文件类型为二进制文件
            reply = ftp.getReplyCode();
            if (!FTPReply.isPositiveCompletion(reply)) {
                ftp.disconnect();
                return success;
            }
            ftp.changeWorkingDirectory(remotePath);
            FTPFile[] fs = ftp.listFiles();
            for (FTPFile ff : fs) {
                if (ff.getName().equals(fileName)) {
                }
            }
            ftp.logout();
            success = true;
        } catch (SocketException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (ftp.isConnected()) {
                try {
                    ftp.disconnect();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return success;
    }
}
```