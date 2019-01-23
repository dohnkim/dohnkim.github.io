---
layout: posts
title: github pages에서 latex를 직접 표지하기
author: 김동훈
---

Github pages에서 Jekyll을 사용하고 있다면,

\_includes/head.html의 적당한 곳에 다음의 내용을 추가하면 된다.

``

```
<script type="text/x-mathjax-config">
    MathJax.Hub.Config({
      tex2jax: {
        skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
        inlineMath: [['$','$']]
      }
    });
  </script>
  <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
```
