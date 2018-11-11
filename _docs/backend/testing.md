---
title: 레거시에 테스트를 끼얹어보자
permalink: /docs/testing/
---

## 레거시를 개선하기 위해 가장 중요한 점
- 테스트
    - 있으면 개발자에게 심리적 안정감을 줌


### 어떻게 시작할 것인가?

레거시라고 표현했지만 여기서는 Codeigniter에서 테스트를 추가하는 방식을 기준으로 설명하였습니다


#### Composer를 이용한 Codeception 설치

[Codeception Quickstart Guide](https://codeception.com/quickstart){:target="_blank"}

```sh
$ composer require codeception/codeception --dev
$ ./vendor/bin/codecept bootstrap
```

#### Codeception의 기본 환경설정


```yaml
### bootstrap으로 생성된 기본 설정
paths:
    tests: tests
    output: tests/_output
    data: tests/_data
    support: tests/_support
    envs: tests/_envs
actor_suffix: Tester
extensions:
    enabled:
        - Codeception\Extension\RunFailed

### codeception 부트스트래핑을 위한 추가 설정

settings:
    bootstrap: _bootstrap.php
    colors: true
    memory_limit: 1024M
    strict_xml: true
    
###코드 커버리지 측정을 위한 설정
coverage:
    enabled: true
    include:
        - app/Services/*
        - htdocs/application/libraries/*
        - htdocs/application/helpers/*
```



#### Codeception 에서 CI의 instatnce를 가져오기

1. tests/_bootstrap.php  
    이 파일은 기본적으로 없기 때문에 생성해야한다.  
    상단에서 설정한 파일의 이름과 일치하게 생성.  
    ```php 
    <?php
    
    // ci redirect 함수가 동작할 경우 테스트가 종료되어버리는 관계로 강제로 등록해준다
    function redirect()
    {
        return true;
    }
    
    //cli 에서 codeception 인자값으로 인해 route 동작이 이상하게 되는 것 제거
    $_SERVER['argv']=[]; 
    
    //default controller가 실행된 결과가 출력되는 것을 지워버린다
    ob_start();
    require_once __DIR__ . "/../public/index.php";
    ob_clean();
    
    
    //여기부터는 본인의 환경에 맞게 추가하거나 삭제하시면 됩니다
    
    //등록된 캐시 삭제
    get_instance()->cache->clean();
    
    //테스트 실행시 체크할 안전사항 추가
    if(get_instance()->db->hostname!=="localhost")
        throw new Exception("테스트는 로컬 디비로만 가능합니다");
    
    
    
    ```

2. 테스트 구분하기
    - 이 부분은 프로젝트의 성격에 따라서 내부적으로 협의 후 결정하시기 바랍니다.
    - 여기서는 제가 쓰는 스타일을 기준으로 작성했습니다.
    
    - codeception의 테스트는 크게 3가지로 구분됩니다
      - acceptance
        - acceptance 테스트를 위한 공간으로 저는 selenium을 통한 UI 테스트를 하기 위한 테스트를 작성합니다.
        - 시나리오 기반으로 테스트를 하기위해 준비중입니다.
      - functional
        - 기능 테스트를 위한 공간으로 현재는 저는 사용하지 않습니다.
        - 프로젝트가 더 진행될 경우 이곳에서 클래스간의 통합테스트를 작성할 예정입니다.
      - unit
        - 단위 테스트를 위한 공간으로 저는 주로 이곳에서 작업합니다 
        - 현재 Mock을 이용한 유닛테스트를 작성하는 것과 하지 않는 것(사실상 통합테스트)가 혼재하고 있습니다
        - 앞으로 클래스의 Mock을 추가하여 unit테스트를 위한 공간으로 활용할 생각입니다.


3. 테스트 생성하기
    
    ```sh 
    $ ./vendor/bin/codecept generate:test unit Helper
    ```
    
    ```php
    <?php
    class HelperTest extends \Codeception\Test\Unit
    {
    
    
        /**
         * @var CI_Controller
         */
        protected $ci;
        
        public function __construct(?string $name = null, array $data = [], string $dataName = '')
        {
            parent::__construct($name, $data, $dataName);
            
            $this->ci=&get_instance();
            $this->ci->load->helper('custom_helper');
        }
    
    
        public function testSearch()
        {
            
            $result = test_function('user_args');
            $this->assertObjectHasAttribute('id',$result);
        }
    
    
    }
    
    ```
    
4. 테스트 실행하기
    ```sh
    $ ./vendor/bin/codecept run 
    ```
    


5. PhpStorm에서 코드 커버리지 확인하기

    
### 테스트를 작성하면서 알아두면 좋은점
- 레거시는 테스트가 힘들다
- CI에서 Controller의 테스트는 굉장히 어렵다
  - 이부분은 Acceptance 테스트에서 Selenium을 통한 테스트로 처리
- 테스트 해야 할 대상
  - 비지니스 로직
    - 내가 만든 클래스
    - 내가 만든 헬퍼
- 테스트 하지 않을 대상
  - 프레임워크가 제공해 주는 기능
    - 모델
    - 컨트롤러
    - 헬퍼
  - 외부 라이브러리
  
#### Codeigniter 에서 조금 더 테스트 가능한 소스를 만들기

- 비지니스 로직을 library 또는 Composer를 이용한 소스로 분할
  - library를 이용한 비지니스 로직 분할 및 테스트
    - Codeigniter에 조금 더 친화적으로
  - Composer를 이용한 소스로 분할
    - 조금 더 모던하게 개발 가능
    - 프레임워크를 바꾸더라도 재사용이 쉬움
 
  


### 실제 작업한 소스
[Codeigniter with CodeCeption](https://github.com/kkame/codeigniter-with-codeception)