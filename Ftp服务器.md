# Ftp服务器

```java
@PostMapping("/upload")
public Response upload(HttpServletRequest request) {
    //获取参数
    Response res;
    String activeId = request.getParameter("activeId");
    MultipartFile file = ((MultipartHttpServletRequest) request).getFile("file");

    String fileName = "activity_" + activeId + ".xlsx";
    String tmpDir = PropertyUtil.FTP_TMP_PATH + "/" + fileName;
    OutputStream os = null;
    FtpClient ftpClient = null;
    String path = null;
    //将上传的文件写入服务器临时文件夹
    try {
        byte[] bytes = file.getBytes();
        os = new FileOutputStream(new File(tmpDir));
        os.write(bytes);
        
        //连接ftp服务器
        ftpClient = FTPUtil.connectFTP();
        //将服务器文件上传至ftp服务器
        path = FTPUtil.upload(fileName, ftpClient);

    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        //关闭流
        try {
            if (os!=null) {
                os.close();
            }
            if (ftpClient!=null) {
                ftpClient.close();
            }
        }catch (Exception e) {
            e.printStackTrace();
        }

    }

    return ResponseUtil.success(path);
}


@Component
public class FTPUtil {

    private static final int    FTP_PORT = 80;
    private static final String FTP_USER = "yyy";
    private static final String FTP_PWD  = "xxx";
    private static final String FTP_PATH = "supervise";

    public static FtpClient connectFTP() {
        //创建ftp
        FtpClient ftp = null;
        try {
            //创建地址
            SocketAddress addr = new InetSocketAddress(PropertyUtil.FTP_HOST, FTP_PORT);
            //连接
            ftp = FtpClient.create();
            ftp.connect(addr);
            //登陆
            ftp.login(FTP_USER, FTP_PWD.toCharArray());
            ftp.setBinaryType();
        } catch (FtpProtocolException | IOException e) {
            e.printStackTrace();
        }
        return ftp;
    }


    public static String upload(String ftpFile, FtpClient ftp) {
        OutputStream os = null;
        FileInputStream fis = null;
        try {
            // 将ftp文件加入输出流中。输出到ftp上。这个路径输出的目的地
            os = ftp.putFileStream(FTP_PATH + "/" + ftpFile);
            // 获取服务器本地文件
            File file = new File(PropertyUtil.FTP_TMP_PATH + "/" + ftpFile);
            // 创建一个缓冲区
            fis = new FileInputStream(file);
            byte[] bytes = new byte[1024];
            int c;
            // 将读取到的服务器本地文件输出到ftp服务器指定文件内
            while ((c = fis.read(bytes)) != -1) {
                os.write(bytes, 0, c);
            }
        } catch (FtpProtocolException | IOException e) {
            e.printStackTrace();
            return null;
        } finally {
            try {
                if (os != null) {
                    os.close();
                }
                if (fis != null) {
                    fis.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        String imgPath = "http://riftp.riskchinaubi.com:8001/supervise/";
        // 返回上传文件的获取路径，前端使用<a>标签即可访问下载。注意，谷歌浏览器无法访问ftp资源
        return imgPath + ftpFile;
    }

    public static void download(String localFile, String ftpFile, FtpClient ftp) {
        InputStream is = null;
        FileOutputStream fos = null;
        try {
            // 获取ftp上的文件
            is = ftp.getFileStream(ftpFile);
            File file = new File(localFile);
            if (!file.exists() && !file.isDirectory()) {
                //              file .mkdir();
                file.createNewFile();
            }
            byte[] bytes = new byte[1024];
            int i;
            fos = new FileOutputStream(file);
            while ((i = is.read(bytes)) != -1) {
                fos.write(bytes, 0, i);
            }
        } catch (FtpProtocolException | IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (fos != null) {
                    fos.close();
                }
                if (is != null) {
                    is.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    public static boolean isExists(String ftpFile, FtpClient ftp) {
        InputStream is = null;
        // 获取ftp上的文件
        try {
            is = ftp.getFileStream(ftpFile);
            byte[] bytes = new byte[1024];
            if (is != null && is.read(bytes) != -1) {
                return true;
            }
        } catch (FtpProtocolException | IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (is != null) {
                    is.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return false;
    }

    public static void uploadbyH5(String localFile, String ftpFile, FtpClient ftp) {
        OutputStream os = null;
        FileInputStream fis = null;
        try {
            // 将ftp文件加入输出流中。输出到ftp上
            os = ftp.putFileStream(ftpFile);
            File file = new File(localFile);
            // 创建一个缓冲区
            fis = new FileInputStream(file);
            byte[] bytes = new byte[1024];
            int c;
            while ((c = fis.read(bytes)) != -1) {
                os.write(bytes, 0, c);
            }
            System.out.println("upload success!!");
        } catch (FtpProtocolException | IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (os != null) {
                    os.close();
                }
                if (fis != null) {
                    fis.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}

```