---
title: In vim, change language layout to english when type escape key 
published: true
categories: vim, neovim
tags: vim, hammerspoon, neovim
---

### 1. 개요    
- 환경  
OS: OSX  
VIM: Neovim 0.5  

- 문제점  
vim에서 한글 입력중 normal mode로 변경했을 때 명령을 입력하기 위해 한영 전환을 해야한다.  
처음 배울 때는 그러려니 했는데 쓸수록 불편하여 방법을 찾아보았다.  

### 2. 해결 방법  
- [hammperspoon](https://www.hammerspoon.org) 이라는 앱을 사용하였다.  
hammerspoon은 이벤트를 후킹하여 원하는 액션을 수행한다.  
이벤트와 수행할 액션은 lua 로 작성된 init.lua 스크립트 파일에 정의한다.  

- init.lua 파일  
hammerspoon을 설치 후 'Open Config' 를 클릭하여 설정 파일을 연 후 아래와 같이 작성한다.  

```lua
local inputEnglish = "com.apple.keylayout.US"

-- 영문 입력상태 아니면 영문 입력으로 변경
function change_to_inputEnglish()
    local inputSource = hs.keycodes.currentSourceID()
    if not (inputSource == inputEnglish) then
        hs.keycodes.currentSourceID(inputEnglish)
    end
    hs.eventtap.keyStroke({}, 'escape')
end

-- escape 키 입력시 change_to_inputEnglish 함수를 호출하도록 바인딩 
hs.hotkey.bind({'ctrl'}, '[', change_to_inputEnglish)

-- 정확하진 않지만 키를 바인딩하면 그 키 이벤트는 바인딩된 함수 수행 후 소멸하는 것으로 보인다. 
-- 따라서 키 이벤트를 바인딩하여 사용 후에는 동일한 역할을 하는 다른 키 이벤트를 발생히켜야 한다. 
-- 그렇지 않고 동일한 이벤트를 발생시키면 recursive call 이 되어 무한 루프에 빠지게 된다.  
-- 여기서는 Ctrl-[ 키를 바인딩 하였고 사용 후 동일한 역할을 하는 escape 이벤트를 발생시켰다.  

```


