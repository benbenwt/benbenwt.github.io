# 压力测试
在Runner选择需要测试的接口，制定请求次数，并编写test断言即可。
## 常用断言
```
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});
```

```
pm.test("Response time is less than 200ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(200);
});
```

```
pm.test("Body matches string", function () {
    pm.expect(pm.response.text()).to.include("string_you_want_to_search");
});
```
