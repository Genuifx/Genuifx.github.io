---
title: 测试rel=noopener
date: 2018-08-21 12:32:10
---
<h1 id="h1">原来的页面被串改了：）</h1>
<script>
	if (window.opener) {
		opener.location = location.origin+'/2018/08/21/about-rel-noopener/#hax';
		// Just `opener.location.hash = '#hax'` only works on the same origin.
	} else {
		document.querySelector('#h1').innerHTML = '原页面是安全的<code>window.opener</code> was <code>null</code>;';
    }
    
</script>