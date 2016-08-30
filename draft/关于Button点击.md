# 关于Button点击

## Selected

在使用按钮selected属性用作网络请求的过程中, 由于网络访问的不确定性, 我们无法判断当前点击请求是否发送成功, 如果将操作反馈滞后, 有可能导致等待过长, 如

````
- (IBAction)normalButtonDidTap:(UIButton *)sender {
    [self sendNetEventWithSelected:sender.selected success:^(){
   	    sender.selected = !sender.selected;
   	    // 刷新数据
    } failure:^(NSError *error) {
    }];
}
````

一般都是直接采用以下方法, 先改变按钮selected状态, 假设网络请求会成功, 请求成功或失败后刷新数据, 保证本地数据不会有异于远程数据

````
- (IBAction)normalButtonDidTap:(UIButton *)sender {
    sender.selected = !sender.selected;
    [self sendNetEventWithSelected:sender.selected success:^(){
    	// 刷新数据
    } failure:^(NSError *error) {
        sender.selected = !sender.selected;
        // 刷新数据
    }];
}
````
在快速点击时也会引发网络堵塞, 上一次的请求返回, 当前这次已经返回了, Button的selected状态会一直变化

```
- (IBAction)normalButtonDidTap:(UIButton *)sender {
    sender.selected = !sender.selected;
    [self.operation cancel];
    self.operation = [self sendNetEventWithSelected:sender.selected success:nil failure:^(NSError *error) {
        sender.selected = !sender.selected;
    }];
}
```




