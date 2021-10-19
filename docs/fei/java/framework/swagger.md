# Swagger

自动生成接口文档的小工具

## 集成`Spring`的版本

* `Swagger`依赖包：`springfox-swagger2`
* `Swagger-ui`依赖包：`springfox-swagger-ui`

> 接口文档的UI地址为 `http://host:port/swagger-ui.html`

```java
// Swagger2 Spring的配置类
@Configuration
@EnableSwagger2
public class SwaggerConfig {

    public Docket createRestApiDoc() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Template Application")
                .termsOfServiceUrl("http://localhost:8081")
                .version("1.0.0")
                .build();
    }
}
```

### 注解说明

```text
作用于controller的class
    @Api(tags = "接口的作用")

作用于controller的class method
    @ApiOperation(value = "方法的作用")
    @ApiImplicitParams({
        @ApiImplicitParam(
            name = "参数名",
            value = "参数的说明",
            required = "参数是否必要",
            paramTypes = "参数的位置,header,query,path,body,form",
            dataType = "参数的类型",
            defaultValue = "参数的默认值"
        )
    })
    @ApiResponses({
        @ApiResponse(
            code = "状态码",
            message = "返回信息"
        )
    })

作用于model的class
    @ApiModel("实体名称")

作用于model的class field
    @ApiModelProperty("字段名称")
```
