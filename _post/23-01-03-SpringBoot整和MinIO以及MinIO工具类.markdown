---
layout:  post
title:   SpringBoot整和MinIO以及MinIO工具类
date:   23-01-03 18:32:23 修改
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-Springboot
-spring boot

---


## 安装

```java
docker run -p 9000:9000 -p 9090:9090      --net=bridge      --name minio      -d --restart=always      -e "MINIO_ACCESS_KEY=minioadmin"      -e "MINIO_SECRET_KEY=minioadmin"      -v D:/minio/data:/data      -v D:/minio/config:/root/.minio      minio/minio server      /data --console-address ":9090" -address ":9000"
```

### MinIO坐标

```java
<!-- MinIO -->
<dependency>
    <groupId>io.minio</groupId>
    <artifactId>minio</artifactId>
    <version>8.2.1</version>
</dependency>
```

### 配置文件

```java
# MinIO 配置
minio:
  endpoint: http://192.168.1.61:9000      # MinIO服务地址
  fileHost: http://192.168.1.61:9000      # 文件地址host
  bucketName: imooc                      # 存储桶bucket名称
  accessKey: imooc                         # 用户名
  secretKey: imooc123456                     # 密码
  imgSize: 1024                           # 图片大小限制，单位：m
  fileSize: 1024                          # 文件大小限制，单位：m
```

### 文件类型枚举

```java
public enum ContentType {
   
    DEFAULT("default","application/octet-stream"),
    JPG("jpg", "image/jpeg"),
    TIFF("tiff", "image/tiff"),
    GIF("gif", "image/gif"),
    JFIF("jfif", "image/jpeg"),
    PNG("png", "image/png"),
    TIF("tif", "image/tiff"),
    ICO("ico", "image/x-icon"),
    JPEG("jpeg", "image/jpeg"),
    WBMP("wbmp", "image/vnd.wap.wbmp"),
    FAX("fax", "image/fax"),
    NET("net", "image/pnetvue"),
    JPE("jpe", "image/jpeg"),
    RP("rp", "image/vnd.rn-realpix"),
    MP4("mp4", "video/mp4");

    private String prefix;

    private String type;

    public static String getContentType(String prefix){
   
        if(StrUtil.isEmpty(prefix)){
   
            return DEFAULT.getType();
        }
        prefix = prefix.substring(prefix.lastIndexOf(".") + 1);
        for (ContentType value : ContentType.values()) {
   
            if(prefix.equalsIgnoreCase(value.getPrefix())){
   
                return value.getType();
            }
        }
        return DEFAULT.getType();
    }

    ContentType(String prefix, String type) {
   
        this.prefix = prefix;
        this.type = type;
    }

    public String getPrefix() {
   
        return prefix;
    }

    public String getType() {
   
        return type;
    }
}
```

### MinIO工具类

```java
import io.minio.*;
import io.minio.http.Method;
import io.minio.messages.Bucket;
import io.minio.messages.DeleteObject;
import io.minio.messages.Item;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.multipart.MultipartFile;
import java.io.ByteArrayInputStream;
import java.io.InputStream;
import java.io.UnsupportedEncodingException;
import java.net.URLDecoder;
import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;
import java.util.Optional;

/**
 * MinIO工具类
 */
@Slf4j
public class MinIOUtils {
   

    private static MinioClient minioClient;

    private static String endpoint;
    private static String bucketName;
    private static String accessKey;
    private static String secretKey;
    private static Integer imgSize;
    private static Integer fileSize;


    private static final String SEPARATOR = "/";

    public MinIOUtils() {
   
    }

    public MinIOUtils(String endpoint, String bucketName, String accessKey, String secretKey, Integer imgSize, Integer fileSize) {
   
        MinIOUtils.endpoint = endpoint;
        MinIOUtils.bucketName = bucketName;
        MinIOUtils.accessKey = accessKey;
        MinIOUtils.secretKey = secretKey;
        MinIOUtils.imgSize = imgSize;
        MinIOUtils.fileSize = fileSize;
        createMinioClient();
    }

    /**
     * 创建基于Java端的MinioClient
     */
    public void createMinioClient() {
   
        try {
   
            if (null == minioClient) {
   
                log.info("开始创建 MinioClient...");
                minioClient = MinioClient
                                .builder()
                                .endpoint(endpoint)
                                .credentials(accessKey, secretKey)
                                .build();
                createBucket(bucketName);
                log.info("创建完毕 MinioClient...");
            }
        } catch (Exception e) {
   
            log.error("MinIO服务器异常：{}", e);
        }
    }

    /**
     * 获取上传文件前缀路径
     * @return
     */
    public static String getBasisUrl() {
   
        return endpoint + SEPARATOR + bucketName + SEPARATOR;
    }

    /******************************  Operate Bucket Start  ******************************/

    /**
     * 启动SpringBoot容器的时候初始化Bucket
     * 如果没有Bucket则创建
     * @throws Exception
     */
    private static void createBucket(String bucketName) throws Exception {
   
        if (!bucketExists(bucketName)) {
   
            minioClient.makeBucket(MakeBucketArgs.builder().bucket(bucketName).build());
        }
    }

    /**
     *  判断Bucket是否存在，true：存在，false：不存在
     * @return
     * @throws Exception
     */
    public static boolean bucketExists(String bucketName) throws Exception {
   
        return minioClient.bucketExists(BucketExistsArgs.builder().bucket(bucketName).build());
    }


    /**
     * 获得Bucket的策略
     * @param bucketName
     * @return
     * @throws Exception
     */
    public static String getBucketPolicy(String bucketName) throws Exception {
   
        String bucketPolicy = minioClient
                                .getBucketPolicy(
                                        GetBucketPolicyArgs
                                                .builder()
                                                .bucket(bucketName)
                                                .build()
                                );
        return bucketPolicy;
    }


    /**
     * 获得所有Bucket列表
     * @return
     * @throws Exception
     */
    public static List<Bucket> getAllBuckets() throws Exception {
   
        return minioClient.listBuckets();
    }

    /**
     * 根据bucketName获取其相关信息
     * @param bucketName
     * @return
     * @throws Exception
     */
    public static Optional<Bucket> getBucket(String bucketName) throws Exception {
   
        return getAllBuckets().stream().filter(b -> b.name().equals(bucketName)).findFirst();
    }

    /**
     * 根据bucketName删除Bucket，true：删除成功； false：删除失败，文件或已不存在
     * @param bucketName
     * @throws Exception
     */
    public static void removeBucket(String bucketName) throws Exception {
   
        minioClient.removeBucket(RemoveBucketArgs.builder().bucket(bucketName).build());
    }

    /******************************  Operate Bucket End  ******************************/


    /******************************  Operate Files Start  ******************************/

    /**
     * 判断文件是否存在
     * @param bucketName 存储桶
     * @param objectName 文件名
     * @return
     */
    public static boolean isObjectExist(String bucketName, String objectName) {
   
        boolean exist = true;
        try {
   
            minioClient.statObject(StatObjectArgs.builder().bucket(bucketName).object(objectName).build());
        } catch (Exception e) {
   
            exist = false;
        }
        return exist;
    }

    /**
     * 判断文件夹是否存在
     * @param bucketName 存储桶
     * @param objectName 文件夹名称
     * @return
     */
    public static boolean isFolderExist(String bucketName, String objectName) {
   
        boolean exist = false;
        try {
   
            Iterable<Result<Item>> results = minioClient.listObjects(
                    ListObjectsArgs.builder().bucket(bucketName).prefix(objectName).recursive(false).build());
            for (Result<Item> result : results) {
   
                Item item = result.get();
                if (item.isDir() && objectName.equals(item.objectName())) {
   
                    exist = true;
                }
            }
        } catch (Exception e) {
   
            exist = false;
        }
        return exist;
    }

    /**
     * 根据文件前缀查询文件
     * @param bucketName 存储桶
     * @param prefix 前缀
     * @param recursive 是否使用递归查询
     * @return MinioItem 列表
     * @throws Exception
     */
    public static List<Item> getAllObjectsByPrefix(String bucketName,
                                                   String prefix,
                                                   boolean recursive) throws Exception {
   
        List<Item> list = new ArrayList<>();
        Iterable<Result<Item>> objectsIterator = minioClient.listObjects(
                ListObjectsArgs.builder().bucket(bucketName).prefix(prefix).recursive(recursive).build());
        if (objectsIterator != null) {
   
            for (Result<Item> o : objectsIterator) {
   
                Item item = o.get();
                list.add(item);
            }
        }
        return list;
    }

    /**
     * 获取文件流
     * @param bucketName 存储桶
     * @param objectName 文件名
     * @return 二进制流
     */
    public static InputStream getObject(String bucketName, String objectName) throws Exception {
   
        return minioClient.getObject(GetObjectArgs.builder().bucket(bucketName).object(objectName).build());
    }

    /**
     * 断点下载
     * @param bucketName 存储桶
     * @param objectName 文件名称
     * @param offset 起始字节的位置
     * @param length 要读取的长度
     * @return 二进制流
     */
    public InputStream getObject(String bucketName, String objectName, long offset, long length)throws Exception {
   
        return minioClient.getObject(
                GetObjectArgs.builder()
                        .bucket(bucketName)
                        .object(objectName)
                        .offset(offset)
                        .length(length)
                        .build());
    }

    /**
     * 获取路径下文件列表
     * @param bucketName 存储桶
     * @param prefix 文件名称
     * @param recursive 是否递归查找，false：模拟文件夹结构查找
     * @return 二进制流
     */
    public static Iterable<Result<Item>> listObjects(String bucketName, String prefix,
                                                     boolean recursive) {
   
        return minioClient.listObjects(
                ListObjectsArgs.builder()
                        .bucket(bucketName)
                        .prefix(prefix)
                        .recursive(recursive)
                        .build());
    }

    /**
     * 使用MultipartFile进行文件上传
     * @param bucketName 存储桶
     * @param file 文件名
     * @param objectName 对象名
     * @param contentType 类型
     * @return
     * @throws Exception
     */
    public static ObjectWriteResponse uploadFile(String bucketName, MultipartFile file,
                                                String objectName, String contentType) throws Exception {
   
        InputStream inputStream = file.getInputStream();
        return minioClient.putObject(
                PutObjectArgs.builder()
                        .bucket(bucketName)
                        .object(objectName)
                        .contentType(contentType)
                        .stream(inputStream, inputStream.available(), -1)
                        .build());
    }

    /**
     * 上传本地文件
     * @param bucketName 存储桶
     * @param objectName 对象名称
     * @param fileName 本地文件路径
     */
    public static ObjectWriteResponse uploadFile(String bucketName, String objectName,
                                                String fileName) throws Exception {
   
        return minioClient.uploadObject(
                UploadObjectArgs.builder()
                        .bucket(bucketName)
                        .object(objectName)
                        .filename(fileName)
                        .build());
    }

    /**
     * 通过流上传文件
     *
     * @param bucketName 存储桶
     * @param objectName 文件对象
     * @param inputStream 文件流
     */
    public static ObjectWriteResponse uploadFile(String bucketName, String objectName, InputStream inputStream) throws Exception {
   
        return minioClient.putObject(
                PutObjectArgs.builder()
                        .bucket(bucketName)
                        .object(objectName)
                        .stream(inputStream, inputStream.available(), -1)
                        .build());
    }

    /**
     * 创建文件夹或目录
     * @param bucketName 存储桶
     * @param objectName 目录路径
     */
    public static ObjectWriteResponse createDir(String bucketName, String objectName) throws Exception {
   
        return minioClient.putObject(
                PutObjectArgs.builder()
                        .bucket(bucketName)
                        .object(objectName)
                        .stream(new ByteArrayInputStream(new byte[]{
   }), 0, -1)
                        .build());
    }

    /**
     * 获取文件信息, 如果抛出异常则说明文件不存在
     *
     * @param bucketName 存储桶
     * @param objectName 文件名称
     */
    public static String getFileStatusInfo(String bucketName, String objectName) throws Exception {
   
        return minioClient.statObject(
                StatObjectArgs.builder()
                        .bucket(bucketName)
                        .object(objectName)
                        .build()).toString();
    }

    /**
     * 拷贝文件
     *
     * @param bucketName 存储桶
     * @param objectName 文件名
     * @param srcBucketName 目标存储桶
     * @param srcObjectName 目标文件名
     */
    public static ObjectWriteResponse copyFile(String bucketName, String objectName,
                                                 String srcBucketName, String srcObjectName) throws Exception {
   
        return minioClient.copyObject(
                CopyObjectArgs.builder()
                        .source(CopySource.builder().bucket(bucketName).object(objectName).build())
                        .bucket(srcBucketName)
                        .object(srcObjectName)
                        .build());
    }

    /**
     * 删除文件
     * @param bucketName 存储桶
     * @param objectName 文件名称
     */
    public static void removeFile(String bucketName, String objectName) throws Exception {
   
        minioClient.removeObject(
                RemoveObjectArgs.builder()
                        .bucket(bucketName)
                        .object(objectName)
                        .build());
    }

    /**
     * 批量删除文件
     * @param bucketName 存储桶
     * @param keys 需要删除的文件列表
     * @return
     */
    public static void removeFiles(String bucketName, List<String> keys) {
   
        List<DeleteObject> objects = new LinkedList<>();
        keys.forEach(s -> {
   
            objects.add(new DeleteObject(s));
            try {
   
                removeFile(bucketName, s);
            } catch (Exception e) {
   
                log.error("批量删除失败！error:{}",e);
            }
        });
    }

    /**
     * 获取文件外链
     * @param bucketName 存储桶
     * @param objectName 文件名
     * @param expires 过期时间 <=7 秒 （外链有效时间（单位：秒））
     * @return url
     * @throws Exception
     */
    public static String getPresignedObjectUrl(String bucketName, String objectName, Integer expires) throws Exception {
   
        GetPresignedObjectUrlArgs args = GetPresignedObjectUrlArgs.builder().expiry(expires).bucket(bucketName).object(objectName).build();
        return minioClient.getPresignedObjectUrl(args);
    }

    /**
     * 获得文件外链
     * @param bucketName
     * @param objectName
     * @return url
     * @throws Exception
     */
    public static String getPresignedObjectUrl(String bucketName, String objectName) throws Exception {
   
        GetPresignedObjectUrlArgs args = GetPresignedObjectUrlArgs.builder()
                                                                    .bucket(bucketName)
                                                                    .object(objectName)
                                                                    .method(Method.GET).build();
        return minioClient.getPresignedObjectUrl(args);
    }

    /**
     * 将URLDecoder编码转成UTF8
     * @param str
     * @return
     * @throws UnsupportedEncodingException
     */
    public static String getUtf8ByURLDecoder(String str) throws UnsupportedEncodingException {
   
        String url = str.replaceAll("%(?![0-9a-fA-F]{2})", "%25");
        return URLDecoder.decode(url, "UTF-8");
    }

    /******************************  Operate Files End  ******************************/

}
```

### MinIo配置类

```java
@Configuration
@Data
public class MinIOConfig {
   

    @Value("${minio.endpoint}")
    private String endpoint;
    @Value("${minio.fileHost}")
    private String fileHost;
    @Value("${minio.bucketName}")
    private String bucketName;
    @Value("${minio.accessKey}")
    private String accessKey;
    @Value("${minio.secretKey}")
    private String secretKey;

    @Value("${minio.imgSize}")
    private Integer imgSize;
    @Value("${minio.fileSize}")
    private Integer fileSize;

    @Bean
    public MinIOUtils creatMinioClient() {
   
        return new MinIOUtils(endpoint, bucketName, accessKey, secretKey, imgSize, fileSize);
    }
}
```

### 测试

```java
@RestController
@Slf4j
@Api(tags = "测试文件上传")
public class FileController {
   
    @Autowired
    private MinIOConfig minIOConfig;


    @PostMapping("/upload")
    public String testUpload(MultipartFile file) throws Exception {
   
        String fileName = file.getOriginalFilename();
        MinIOUtils.uploadFile(minIOConfig.getBucketName(), fileName, file.getInputStream());
        //返回图片链接
        String imgUrl = minIOConfig.getFileHost()+"/"+minIOConfig.getBucketName()+"/"+fileName;
        return imgUrl;
    }
}
```


![img](https://img-blog.csdnimg.cn/img_convert/df716f8f0df92dd50c83d5b51b1033d1.png)

