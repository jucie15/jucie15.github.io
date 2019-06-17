---
title: "[GIT] Minimal Mistakes 를 적용한 Github Page 만들기"
categories:
  - Git
tags:
  - git
---

##### [Quick-start-guide](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/)를 많이 보고 참고하였습니다.

#### Refer

- [2.2 Documentation](https://mmistakes.github.io/minimal-mistakes/docs/docs-2-2/)

- [Jekyll 블로그 테마 적용하기 (minimal-mistakes)](https://junhobaik.github.io/jekyll-apply-theme/)

1. ### 사이트 기본 정보 설정하기(_config.yml)

   #### 테마설정

   ```yaml
   minimal_mistakes_skin    : "contrast" # "air", "aqua", "contrast", "dark", "dirt", "neon", "mint", "plum", "sunrise"
   ```

   #### 사이트 기본정보

   ```yaml
   # Site Settings
   locale                   : "ko-KR" # 언어 설정(_data/ui-text.yml 확인)
   title                    : "OnDev Tech Blog" # 사이트 타이틀
   title_separator          : "-"
   name                     : "OnDev"
   description              : "OnDev Team Tech Blog"
   url                      : "jucie15.github.io" # the base hostname & protocol for your site e.g.
   baseurl                  : # the subpath of your site, e.g. "/blog"
   repository               : # GitHub username/repo-name e.g. "mmistakes/minimal-mistakes"
   teaser                   : # path of fallback teaser image, e.g. "/assets/images/500x300.png"
   logo                     : # path of logo image to display in the masthead, e.g. "/assets/images/88x88.png"
   masthead_title           : # overrides the website title displayed in the masthead, use " " for no title
   ```

   #### 사이트 저자정보

   ```yaml
   # Site Author
   author:
     name             : "OnDev"
     avatar           : # path of avatar image, e.g. "/assets/images/bio-photo.jpg"
     bio              : "We are OnDev Team"
     location         : "Somewhere"
     email            :
     links:
       - label: "Email"
         icon: "fas fa-fw fa-envelope-square"
         # url: mailto:your.name@email.com
       - label: "Website"
         icon: "fas fa-fw fa-link"
         # url: "https://your-website.com"
       - label: "Twitter"
         icon: "fab fa-fw fa-twitter-square"
         # url: "https://twitter.com/"
       - label: "Facebook"
         icon: "fab fa-fw fa-facebook-square"
         # url: "https://facebook.com/"
       - label: "GitHub"
         icon: "fab fa-fw fa-github"
         # url: "https://github.com/"
       - label: "Instagram"
         icon: "fab fa-fw fa-instagram"
         # url: "https://instagram.com/"

   ```

   #### 기본 페이지 구성 정보

   ```yaml
   # Defaults
   defaults:
     # _posts
     - scope:
         path: ""
         type: posts
       values:
         layout: single
         author_profile: true
         read_time: true
         comments: true
         share: true
         related: true
     # _pages
     - scope:
         path: "_pages"
         type: pages
       values:
         layout: single
         author_profile: true
     # _docs
     - scope:
         path: "_docs"
         type: docs
       values:
         layout: single
         read_time: false
         author_profile: false
         share: false
         comments: false
         sidebar:
           nav: "docs"
   ```

   ##### 이 외 나머지 부분들은 [Github](https://github.com/mmistakes/minimal-mistakes)에서 작성된 코드를 참고하고 커스텀하면 된다.

2. ### 불필요한 파일 제거

   #####  기존 코드를 참고하여 커스텀 하고 싶다면 /docs안에 구조와 코드를 참고하면 좋다.

   - `.editorconfig`
   - `.gitattributes`
   - `.github`
   - `/docs`
   - `/test`
   - `CHANGELOG.md`
   - `minimal-mistakes-jekyll.gemspec`
   - `README.md`
   - `screenshot-layouts.png`
   - `screenshot.png`

3. ### 네브바 설정(_data/navigation.yml)

   ```yaml
   # main links
   main: # 메인 네브바 설정
     - title: "About"
       url: /about/
     - title: "Post"
       url: /post-by-tags/

   docs: # 컬렉션 별로 옆에 사이드바를 추가해줄 수 있다.
     - title: Getting Started
       children:
         - title: "Quick-Start Guide"
           url: /docs/quick-start-guide/
         - title: "Structure"
           url: /docs/structure/
         - title: "Installation"
           url: /docs/installation/
         - title: "Upgrading"
           url: /docs/upgrading/

   ''''''
   ```



4. ### 디렉토리 설정-컬렉션 추가(_config.yml)

   ```yaml
   # Collections
   collections:
     docs: # <- collection name
       output: true
       permalink: /:collection/:path/

   ```



5. ### 페이지 추가(파일구조)

   #### 기본 페이지들 -> `_pages`

   - #####  404.md

   ```html
   ---
   title: "Page Not Found"
   excerpt: "Page not found. Your pixels are in another canvas."
   sitemap: false
   permalink: /404.html
   ---

   왓더퍽!!

   <script>
     var GOOG_FIXURL_LANG = 'en';
     var GOOG_FIXURL_SITE = '{{ site.url }}'
   </script>
   <script src="https://linkhelp.clients.google.com/tbproxy/lh/wm/fixurl.js">
   </script>

   ```

   - ##### tag-posts.html

   ```
   {% highlight liquid %}
     ---
     layout: archive
     permalink: /post-by-tags/
     title: "Posts by Tag"
     author_profile: true
     ---
     {% include group-by-array collection=site.posts field="tags" %}
     <ul>
       {% for tag in site.tags %}
         <span>
           <a href="#{{ tag | first }}">
             {{ tag | first }}
           </a> &nbsp;&nbsp;&nbsp;
         </span>
       {% endfor %}
     </ul>
     <br/>
     <br/>
     {% for tag in group_names %}
       {% assign posts = group_items[forloop.index0] %}
       <h2 id="{{ tag | slugify }}" class="archive__subtitle">{{ tag }}</h2>
       {% for post in posts %}
         {% include archive-single.html %}
       {% endfor %}
     {% endfor %}
   {% endhighlight %}

   ```

   #### 업로드 할 포스트들 -> `_posts`

   #### 업로드 하기 전 포스트들 -> `_drafts`

6. ### 링크 연결

   - ##### permalink만 설정해주면 여러 폴더에서 알아서 잘 찾음(만약 url에 collection이 있다면 collector 설정을 추가해줘야함 ex) docs/amugae/)

### Todo

1. 카테고리 구성
2. CSS 수정
3. Author 및 사이드 바 추가
