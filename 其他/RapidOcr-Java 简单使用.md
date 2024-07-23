# RapidOcr-Java 简单使用

- 主项目：[RapidOCR](https://github.com/RapidAI/RapidOCR)
- 官方文档：[RapidOCR 文档](https://rapidai.github.io/RapidOCRDocs/)
- Java 版本：[RapidOcr-Java](https://github.com/MyMonsterCat/RapidOcr-Java/tree/main)

## 需求

准备实现一个拍照录题的功能，需要 OCR 功能，正好看到了 RapidOCR 项目，开源且支持离线，并且有 Java 实现的版本，降低了学习成本

官方的 demo 可直接使用，十分方便，且有针对特定问题的解决方案

## 准备

- Java 环境
- 官方 demo：[rapidocr-demo](https://github.com/MyMonsterCat/rapidocr-demo/tree/main)，支持 JavaEE 和 SpringBoot
- 服务器（可不需要）
- 前端项目（可不需要）

## 运行

下载 demo 下来后，直接启动即可，默认端口为 18080，建议修改下默认的可上传文件大小，默认为 1MB，既然是测试调高点无所谓

```yaml
spring:
  servlet:
    multipart:
      max-file-size: 10MB
      max-request-size: 10MB
```

默认有两个接口

- GET：`ocr`
- POST：`ocr`

如果请求失败，很可能是路径不对，目前发现两处

```java
@PostMapping
public String ocr(@RequestParam("file") MultipartFile fileUpload) throws IOException {
    ParamConfig paramConfig = ParamConfig.getDefaultConfig();
    paramConfig.setDoAngle(true);
    paramConfig.setMostAngle(true);
    InferenceEngine engine = InferenceEngine.getInstance(Model.ONNX_PPOCR_V3);
    // Windows下运行正常
    // Linux环境下，ocrJava前加入斜杠
    File file = new File(System.getProperty("java.io.tmpdir") + "/ocrJava/test.png");
    fileUpload.transferTo(file);
    file.deleteOnExit();

    OcrResult ocrResult = engine.runOcr(file.getPath(),paramConfig);
    return ocrResult.getStrRes().toString();
}


@SneakyThrows
private static String getResourcePath(String path) {
    // 末尾加入.substring(1)，因为最终路径前多了个斜杠
    return Thread.currentThread().getContextClassLoader().getResource(path).getPath().substring(1);
}
```

## 部署

无特别的注意事项，但在 CentOS 7 及其他低版本的系统下，需要升级某些依赖，很不幸我的服务器使用的就是 CentOS 7，详见 [如何在CentOS7或其他低版本Linux系统上运行](https://github.com/MyMonsterCat/RapidOcr-Java/blob/main/docs/CentOS7.md)，升级较为耗时，大约 3 个小时左右

打包到 Linux 环境执行，需执行 `mvn clean package -P linux-x86_64 -Dlinux-build` 进行打包

## 总结

后续使用时可能需要再调整调整配置，但 demo 已足够能满足基本的需求，后续如遇到问题会再次更新
