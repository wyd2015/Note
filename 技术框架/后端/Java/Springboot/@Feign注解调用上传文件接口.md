# Feign调用上传文件接口

## 业务场景

- 接口风格：Restful形式；
- 功能：以图搜图

- 使用方式：Feign

## 代码

1. `ImageSearchAPI.java`

   ```java
   @FeignClient(name = "image-search", url = "${imgSearch.url:}")
   public interface ImageSearchAPI {
       @PostMapping(consumes = MediaType.MULTIPART_FORM_DATA_VALUE, 
               produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
       String searchImages(@RequestPart("file") MultipartFile file);
   }
   ```

2. `ImageSearchService.java`

   ```java
   @Slf4j
   @Service
   public class ImageSearchService {
       @Autowired
       private ImageSearchAPI imageSearchAPI;    
       
       /**
        * 以图搜图
        * @Param: image
        * @Return: trs.cloud.common.status.RestResult
        * @Date: 2020/4/23 20:29
        */
       public RestResult searchImages(MultipartFile image){
           RestResult<Object> result = new RestResult<>();
           String json = imageSearchAPI.searchImages(image);
           ImageSearchResult searchResult = null;
           try {
               searchResult = JSONUtil.toObject(json, ImageSearchResult.class);
               if(searchResult.getStatus() != 1){
                   result.setStatus(searchResult.getStatus());
                   result.setMessage(searchResult.getMsg());
                   return result;
               }
               result.setData(searchResult.getDatas());
           } catch (Exception e) {
               log.error("根据图片[{}]以图搜图检索结果json转换异常：", image.getOriginalFilename(), e);
           }
           return result;
       }
       
   }
   ```

3. `ImageSearchController.java`

   ```java
   @RestController
   @Api(value = "以图搜图")
   @RequestMapping("/search/image")
   public class ImageSearchController {
       @Autowired
       private ImageSearchService imageSearchService;
       
       /**
        * 以图搜图
        * @Param: file
        * @Return: trs.cloud.common.status.RestResult
        * @Date: 2020/4/23 20:16
        */
       @ApiOperation(value = "上传文件开始搜索", httpMethod = "POST")
       @PostMapping(consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
       public RestResult searchImages(
           @ApiParam(value = "上传图片文件", required = true) @RequestParam("file") MultipartFile file){
           return imageSearchService.searchImages(file);
       }
   }
   ```

   