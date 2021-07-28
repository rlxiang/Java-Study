# Thymeleaf

```html
<div>
    <div th:utext="'&lt;!--'" th:remove="tag"></div> 效果是<!--, th:remove="tag" 表示页面不显示html标记
    <div th:utext="'Failed Request URl :'+ ${url}" th:remove="tag"></div>
    <div th:utext="'Exception message :'+ ${exception.message}" th:remove="tag"></div>
    <ul th:remove="tag">
        <li th:each="st: ${exception.stackTrace}" th:remove="tag"><span th:utext="${st}" th:remove="tag"></span></li> 循环语句
    </ul>
    <div th:utext="'--&gt;'" th:remove="tag"></div> 效果是 -->
</div>
```



