# WorkDrop
记录工作中常见问题

## 1、工程里调试私有Pod中的Framework
![image](https://github.com/sun8801/WorkDrop/blob/master/工程里调试私有Pod中的Framework.png)

## 2、`EXC_BAD_ACCESS KERN_INVALID_ADDRESS 0x........` 报错修复
`EXC_BAD_ACCESS KERN_INVALID_ADDRESS` 报错一般都是地址问题，使用了已经释放了的地址空间。
空对象可以调用方法、属性，
但空对象直接取值时会崩溃（`nil->_abc`）`EXC_BAD_ACCESS KERN_INVALID_ADDRESS`
在block中用了weak时，且用`->`取值要注意，保证对象不能为`nil`
如下简单测试代码，要加self为nil判断：
```
__weak typeof(self) weakSelf = self;
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(5 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
__strong typeof(self) self = weakSelf;
if (!self) return;
self->_abc = 1;
});
```
## 3、给Xcode添加自定义工程模板
参考：[Xcode自定义模板](https://xiaovv.me/2018/03/16/Custom-xcode-templates/)

自动拷贝模板至Xcode脚本

```sh
#!/bin/sh

current_dir=$(pwd)
script_dir=$(dirname $0)
dir="$current_dir/$script_dir/XXX_XX.xctemplate"

path=~/Library/Developer/Xcode/Templates/File\ Templates/User\ Interface
mkdir  -p "${path}" && cp -r "$dir" "$path"

if [ $? -eq 0 ]; then
   echo "✅XXX_XX 配置脚本执行成功！"
else
   echo "\033[31m❗️XXX_XX 配置脚本执行失败! \033[0m"
fi

```

## 4、pod install / update 时自动执行脚本
在xx.podspec 中添加

```rb
# 脚本路径
spec.resources = ['Source/Script/*']
# 执行脚本
spec.prepare_command = <<-CMD
    path=./Source//Script/config.sh
    if [ -f "$path" ]; then
      sh $path
    else
      echo "$path not exist"
    fi
    pwd
    printenv
    echo "======end======="
CMD
```
