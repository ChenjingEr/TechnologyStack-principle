## MyBatis源码解析(二): 配置解析

#### mapper:

**mapper元素配置：mybatis-config.xml**
```
<configuration>
    ...
    <!-- 使用相对于类路径的资源引用 -->
    <mappers>
        <mapper resource="mapper/BookCURDMapper.xml" />
    </mappers>

    <!-- 使用完全限定资源定位符（URL） -->
    <mappers>
      <mapper url="file:///var/mappers/AuthorMapper.xml"/>
    </mappers>

    <!-- 使用映射器接口实现类的完全限定类名 -->
    <mappers>
      <mapper class="org.mybatis.builder.AuthorMapper"/>
    </mappers>

    <!-- 将包内的映射器接口实现全部注册为映射器 -->
    <mappers>
      <package name="org.mybatis.builder"/>
    </mappers>
    ...
</configuration>
```

**mapper元素解析**
```
 private void parseConfiguration(XNode root) {
    ...
    mapperElement(root.evalNode("mappers"));
    ...
  }

  private void mapperElement(XNode parent) throws Exception {
    if (parent != null) {
      for (XNode child : parent.getChildren()) {
        //package
        if ("package".equals(child.getName())) {
          String mapperPackage = child.getStringAttribute("name");
          configuration.addMappers(mapperPackage);
        } else {
          String resource = child.getStringAttribute("resource");
          String url = child.getStringAttribute("url");
          String mapperClass = child.getStringAttribute("class");
          //resource
          if (resource != null && url == null && mapperClass == null) {
            ErrorContext.instance().resource(resource);
            InputStream inputStream = Resources.getResourceAsStream(resource);
            XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, resource, configuration.getSqlFragments());
            mapperParser.parse();
          } else if (resource == null && url != null && mapperClass == null) {
            //url
            ErrorContext.instance().resource(url);
            InputStream inputStream = Resources.getUrlAsStream(url);
            XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, url, configuration.getSqlFragments());
            mapperParser.parse();
          } else if (resource == null && url == null && mapperClass != null) {
            //class
            Class<?> mapperInterface = Resources.classForName(mapperClass);
            configuration.addMapper(mapperInterface);
          } else {
            throw new BuilderException("A mapper element may only specify a url, resource or class, but not more than one.");
          }
        }
      }
    }
  }
```






    <mappers>
        <mapper resource="mapper/BookCURDMapper.xml" />
        <mapper resource="mapper/PublishingCompanyMapper.xml" />
        <mapper resource="mapper/BookDynamicMapper.xml" />
    </mappers>