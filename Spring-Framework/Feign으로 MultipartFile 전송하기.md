# Feign으로 MultipartFile 전송하기
- Feign 에 consumes 타입을 MediaType.MULTIPART_FORM_DATA_VALUE 로 지정하기
- Feign 에 전달하는 파라미터를 @RequestPart 로 지정하고 MultipartFile 지정하기.
- 실제 이용시 MockMultipartFile 로 전송하기

```java
@FeignClient(value = "MultipartFileSendAPI", url = "${multipart-file-send-url}")
public interface MultipartFileSendAPI {
	@PostMapping(path = "/file", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
	Response sendFile(@RequestPart("file") MultipartFile file);
}

public class SendTest {
    @Test
	void test(@Autowired ApplicationContext applicationContext) throws IOException {
        cloudTrailAdminEventAPI.importEvents(
            new MockMultipartFile(
                "file",
                "test.txt",
                MediaType.MULTIPART_FORM_DATA_VALUE,
                Files.readAllBytes(applicationContext.getResource("classpath:/test.txt").getFile().toPath())));
	}
```
