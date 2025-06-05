# My profile Demo - Gitlab pages

****
Gitlab Link<br><br>
[GitLab](https://dchahyunho-gitlab.duckdns.org/explore)

****
**Gitlab Pages Demo**
<br><br>
[Profile Demo home](https://dchahyunho-gitlab.duckdns.org/chahyunho-hugo-blog-decb70/)
*****
Gitlab을 서버에 직접 구축해보며... <br> 
Gitlab 에서의 pages는 개인 블로그, 프로필등의 정적사이트를 대상으로 간단히 배포하기 좋은 기능인 것 같다.
<br/>

Gitlab의 pages 환경 구성시 별도로 설정을 커스터 마이징 하지않으면 <br>
접근 주소: https://(username).gitlab.io/(project-name)/  <- 해당 형식대로 page주소가 생성된다.
<br><br>직접 서버를 구축해 Nginx 프록시를 설정하고, GitLab Pages로 생성된 도메인을 서버 주소와 매핑시키는 방식은
복잡하고 비효율적이며, 일반적인 사용 사례에도 어긋난다.
<br>
<br> 그래서 gpt와 gitlab 공식홈페이지의 doc을 검색해보니 서브패스 방식으로 설정할 수 있다는 것을 알았다.
<br><br>
결과적으로는 Gitlab url 주소를 그대로 사용하여 pages url를 생성한다.

출처 [Gitlab doc - Single-domain sites](https://docs.gitlab.com/administration/pages/#dns-configuration-for-single-domain-sites)
```
external_url "http://example.com" # Swap out this URL for your own
pages_external_url 'http://example.io' # Important: not a subdomain of external_url, so cannot be http://pages.example.com

# Set this flag to enable this feature
gitlab_pages['namespace_in_path'] = true


The resulting URL scheme is http://example.io/<namespace>/<project_slug>.
```

<br>

기존 디폴트 pages 주소 생성방식
```
https://<username>.gitlab.io/<project-name>/
```
서브패스[Single-domain sites] 설정 후 
```
https://chahyunho.gitlab.io/my-blog/  #기존 Gitlab 주소와 동일하고 추가적으로 뒤에 프로젝트 명이 뒤에 붙는다.
```

```
https://dchahyunho-gitlab.duckdns.org/chahyunho-hugo-blog-decb70/  # pages설정에서 고유 식별자로 변경도 가능하다.
```

<br>

### * 설정 소스 일부
<br/>

#### Gitlab docker-compose.yml
```
GITLAB_OMNIBUS_CONFIG: |
        external_url 'https://dchahyunho-gitlab.duckdns.org'  # 내부 GitLab도 HTTPS 사용
        gitlab_rails['gitlab_shell_ssh_port'] = ????

        # GitLab Pages 설정
        pages_external_url 'https://dchahyunho-gitlab.duckdns.org'
        gitlab_pages['enable'] = 'true'
        gitlab_pages['namespace_in_path'] = 'true' # <- * Single-domain sites
        gitlab_pages['inplace_chroot'] = 'true'
        gitlab_pages['access_control'] = 'false'
        gitlab_pages['api_secret_key'] = "????????????????"
        gitlab_pages['external_https'] = ['0.0.0.0:4444']
        gitlab_pages['cert'] = "???????????"
        gitlab_pages['cert_key'] = "?????????????"
```
<br>

#### nginx 설정

```
erver {
    server_name dchahyunho-gitlab.duckdns.org;

    location / {
        proxy_pass https://127.0.0.1:????/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # GitLab Pages 프록시 (subpath 방식)
    location /chahyunho-hugo-blog-decb70/ {
        proxy_pass https://127.0.0.1:4444;  # GitLab Pages 기본 포트
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
```