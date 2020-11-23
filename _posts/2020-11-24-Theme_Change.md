---
layout : post
title : "블로그 테마 변경 - Jekyll dark theme"
date : 2020-11-24 +0900
description : Zekyll theme change
tag : [Theme]
---

### Zekyll Dark Theme

 기존에 쓰던 Bef theme(Zekyll base)의 경우, 항상 이미지를 넣어야 했던 점이 불편했다. 아무래도 기존에 다른 사람들이 사용하던 것 처럼 못쓸까? 해서 어떻게 다른 것을 좀 찾아보니 좋은 형태의 테마를 발견한 것 같다.

 덕분에 이전에 세팅해둔게 다 날아갔지만, 아마 여기서 이제 안바꿀 것 같다. 내 마음대로 개조를 조금씩 추가로 해 봐야겠다.

 아마도 지금 아니면 더이상 깃허브 블로그의 테마를 찾긴 힘들 것 같으니....

 테마 링크는 아래를 따라가면 된다! (정확한 테마 이름은 Dark Clean Theme)

[Zekyll Dark Theme](https://github.com/streetturtle/jekyll-clean-dark)

 일단 이전에 Bef로 고생하면서 배운 html, css를 여기서 좀 써먹어봐야겠다.



### 댓글창 추가

 이전에도 적어놨는데, 계획만 적어놓고 아무것도 안해놨길래 추후 기록삼아 다시 기록한다.

 일단, utterance에는 이미 등록을 해놨으니, 다시 코드를 얻으러 가자.

[https://utteranc.es/](https://utteranc.es/)

위 링크로 접속해서, repo에 자신의 케이스를 적고(나의 경우 ReaperMaKNaE/reapermaknae.github.io), 아래에서 Blog Post <-> Issue Mapping에서 원하는걸 고르는데 난 pathname으로 했다.

 다음 Label은 대충 comment로 적고, Theme은 내가 dark theme이기에 dark로 했다.

 그리고 생긴 아래 코드.

```python
<script src="https://utteranc.es/client.js"
	repo="ReaperMaKNaE/reapermaknae.github.io"
	issue-term="pathname"
	label="Comment"
	theme="github-dark"
	crossorigin="anonymous"
	async>
</script>
```

 이걸 post가 존재하는 곳에 찾아 적당한 위치에 넣으면 된다.

 (나의 경우는 ./_layouts/post.html이었다. 적당한 위치는 아래 빨간 박스.)

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201124-1.png)

 기존에 social이나 disqus같은 것들은 내가 다 의미없다고 생각하는 것들이라 다 삭-제 했다. 또 뭐 지울거 없는지는 추후 조금씩 업데이트 예정.