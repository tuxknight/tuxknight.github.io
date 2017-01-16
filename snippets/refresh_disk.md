---
layout: page
title: " 解决 Vmware 给虚拟机添加磁盘后看不到的问题"
group: snippets
---
{% include JB/setup %}

{% highlight shell linenos %}
echo '- - -' > /sys/class/scsi_host/host0/scan
{% endhighlight %}
