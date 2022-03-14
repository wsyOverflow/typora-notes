## Summary

记录一些遇到的问题，以及解决过程



## Closest-hit shader is never triggered

这个问题让我挣扎了好久，还在 khronos 论坛上发了求助帖 [[1]](#[1])。

一个非常简单的 ray tracing，应该是必会相交的，就是不会触发 closest-hit，总是 miss。检查了代码很久很久，使用 nsight graphics 看参数，都没发现任何问题。最奇葩的是，使用 nsight graphics 捕获我代码的 c++ trace，重放的时候却是正常的，而跑我自己的代码就有问题。挣扎了好久，怀疑人生。

最后发现，是创建加速结构没有等待完成，但是 vulkan 验证层也没有任何提示，nsight graphics 里可视化的加速结构都一切正常，所以前面查找问题完全没有想到这点。

不过，这个挣扎的过程让我对 graphics debug 更熟练了，自从加入了 ray tracing 就再也不能使用 renderdoc，只能使用 nsight graphics，但 nsight 不支持 debug shader，不过 nsight 可以实时更改 shader 代码，因此可以配合 debugprintf 进行 debug，感觉比 renderdoc 好用。





































## Reference

<a name="[1]">[1]</a> https://community.khronos.org/t/miss-shader-is-always-triggered-and-closest-hit-is-never-triggered/108361/2