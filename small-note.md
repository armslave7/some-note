# chrome 设置用户目录

在chrome的快捷方式，或者可执行文件中，添加运行参数即可指定chrome用户目录和缓存目录

```--disk-cache-dir=```指定缓存目录
```--user-data-dir=```指定用户目录

例如：

	"C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --disk-cache-dir="C:\Cache\Chrome"

指定chrome缓存目录在"C:\Cache\Chrome"目录下。