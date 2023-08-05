# 파일 업로드
### 서블릿 파일 업로드
+ spring.servlet.multipart.enabled=false -> 서블릿 컨테이너가 멀티파트와 관련된 처리를 하지 않게 설정
+ 서블릿이 제공하는 Part로 멀티파트 형식을 편리하기 읽을 수 있다.
```java
  @PostMapping("/upload")
  public String saveFileV1(HttpServletRequest request) throws ServletException, IOException {
      log.info("request={}", request);

      String itemName = request.getParameter("itemName");
      log.info("itemName={}", itemName);

      Collection<Part> parts = request.getParts();
      log.info("parts={}", parts);

      for (Part part : parts) {
          log.info("==== PART ====");
          log.info("name={}", part.getName());
          Collection<String> headerNames = part.getHeaderNames();
          for (String headerName : headerNames) {
              log.info("header {}: {}", headerName, part.getHeader(headerName));
          }
          // 편의 메서드
          // content-disposition; filename
          log.info("submittedFilename={}", part.getSubmittedFileName());
          log.info("size={}", part.getSize()); // part body size

          // 데이터 읽기
          InputStream inputStream = part.getInputStream();
          String body = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
          log.info("body={}", body);
          
          // 파일에 저장하기
          if (StringUtils.hasText(part.getSubmittedFileName())) {
              String fullPath = fileDir + part.getSubmittedFileName();
              log.info("파일저장 fullPath={}", fullPath);
              part.write(fullPath);
          }
      }

      return "upload-form";
  }
```
+ 편하지만 HttpServletRequest를 사용해야 하고, 파일 부분만 구분해서 처리하려면 여러가지 코드를 넣어야 하기 때문에 불편하다.

### 스프링의 파일 업로드
+ MultipartFile을 활용해 훨씬 간단하게 작성할 수 있다.
```java
@PostMapping("/upload")
    public String saveFile(@RequestParam String itemName,
                           @RequestParam MultipartFile file, HttpServletRequest request) throws IOException {
        
        log.info("request={}", request);
        log.info("itemName={}", itemName);
        log.info("multiPartFile={}", file);
        
        if (!file.isEmpty()) {
            String fullPath = fileDir + file.getOriginalFilename();
            log.info("파일 저장 fullPath={}", fullPath);
            file.transferTo(new File(fullPath));
        }
        return "upload-form";
    }
```
+ UrlResource에 "file:"을 넣어주면 내부 파일에 접근할 수 있다.

### Tip
+ 바이너리 -> 문자 변환할 때는 항상 인코딩 방식을 정해줘야 한다.(UTF-8처럼)
+ 파일 다운로드는 아래와 같이 하면 된다.
+ 
```java
@GetMapping("/attach/{itemId}")
    public ResponseEntity<Resource> downloadAttach(@PathVariable Long itemId) throws MalformedURLException {
        Item item = itemRepository.findById(itemId);
        String storeFileName = item.getAttachFile().getStoreFileName();
        String uploadFileName = item.getAttachFile().getUploadFileName();

        UrlResource resource = new UrlResource("file:" + fileStore.getFullPath(storeFileName));

        log.info("uploadFileName={}", uploadFileName);

        // 인코딩 타입을 설정해준다.
        String encodedUploadFileName = UriUtils.encode(uploadFileName, StandardCharsets.UTF_8);
        // 다운로드를 위한 규약?같은 것 헤더에 추가해주면 응답할 때 첨부파일임을 인식해 다운로드가 가능하게 해준다.
        String contentDisposition = "attachment; filename=\"" + encodedUploadFileName + "\"";

        return ResponseEntity.ok()
                .header(HttpHeaders.CONTENT_DISPOSITION, contentDisposition)
                .body(resource);
    }
```
