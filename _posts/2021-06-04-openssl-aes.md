---
title: AES-256-CBC with openssl
published: true
categories: openssl
tags: openssl aes-256-cbc base64
---

### 1. Base64 

- encode  
  echo 명령에서 -n 옵션은 LF(LineFeed) 제거  
  windows 에서는 "echo | set -p=" 을 사용  

```bash
> echo -n 'this is a raw string.' | openssl enc -base64 -e 
dGhpcyBpcyBhIHJhdyBzdHJpbmcu

# windows command  
> echo | set -p=this is a raw string.| openssl enc -base64 -e 
```
- decode  
  base64 decoding 시에는 echo 명령에서 -n 옵션은 제외할 것.  

```bash
> echo 'dGhpcyBpcyBhIHJhdyBzdHJpbmcu' | openssl enc -base64 -d
this is a raw string.
```

### 2. Create AES-256-CBC Key, IV

```bash

# options
# -k <password> Key, IV를 생성할 패스워드 
# -P 암호화를 진행하지 않고 생성된 Key, IV, Salt 값 표시 
# -S Salt 값 입력 
# -nosalt Salt 를 적용하지 않음  

# <password> 입력하여 Key, IV 생성  
# 여기서는 <password> 로 "password"를 입력함.  
> openssl enc -aes-256-cbc -k password -P
salt=2308E3BE706F36F3
key=887743585FF5E0F1DDF4B99C18A49ED4CD7AF54449455A20430EA499EB0E85C6
iv =93AAAA3D879F1DD3282938859D878DDC

# <password>, salt 값을 입력하여 Key, IV 생성  
# 첫번째 예제와 동일하게 생성된다.  
> openssl enc -aes-256-cbc -k password -S 2308E3BE706F36F3 -P
key=887743585FF5E0F1DDF4B99C18A49ED4CD7AF54449455A20430EA499EB0E85C6
iv =93AAAA3D879F1DD3282938859D878DDC

# <password> 입력하지만 -nosalt 옵션을 붙여(salt 미사용) Key, IV 생성  
# 항상 동일한 Key, IV가 생성된다. 
> openssl enc -aes-256-cbc -k password -nosalt -P  
key=5F4DCC3B5AA765D61D8327DEB882CF992B95990A9151374ABD8FF8C5A7A0FE08
iv =B7B4372CDFBCB3D16A2631B59B509E94
```

### 3. AES-256-CBC 

- 생성된 Key, IV를 사용하여 Encrypt/Decrypt  

```bash  

# options 
# -e encrypt 처리 
# -a Base64로 입출력 
# -K Key 값 입력 (대문자 주의) 
# -iv IV 값 입력 
# -in 입력 파일 
# -out 출력 파일 

# Encrypt text
> echo -n "This is a sample text." | openssl enc -aes-256-cbc -e -a -K "5F4DCC3B5AA765D61D8327DEB882CF992B95990A9151374ABD8FF8C5A7A0FE08" -iv "B7B4372CDFBCB3D16A2631B59B509E94"
satEIAVWTySBa7Bvu8ChfyEJBELVVn+V6Oeus0EusEo=

# Decrypt text 
> echo "satEIAVWTySBa7Bvu8ChfyEJBELVVn+V6Oeus0EusEo=" | openssl enc -aes-256-cbc -d -a -K "5F4DCC3B5AA765D61D8327DEB882CF992B95990A9151374ABD8FF8C5A7A0FE08" -iv "B7B4372CDFBCB3D16A2631B59B509E94"
This is a sample text.

# Encrypt file  
> openssl enc -aes-256-cbc -e -a -in input.txt -out output.txt -K "5F4DCC3B5AA765D61D8327DEB882CF992B95990A9151374ABD8FF8C5A7A0FE08" -iv "B7B4372CDFBCB3D16A2631B59B509E94"

# Decrypt file  
> openssl enc -aes-256-cbc -d -a -in output.txt -out input.txt -K "5F4DCC3B5AA765D61D8327DEB882CF992B95990A9151374ABD8FF8C5A7A0FE08" -iv "B7B4372CDFBCB3D16A2631B59B509E94"
```

### 4. 참고 사항

- 파일 처리시 유의 사항  
  Windows와 POSIX 의 텍스트 파일 라인 처리 기준이 다르다.
  

```
# 파일 내용이 다음과 같은 경우 
-----------
A
A
-----------

# Windows  
  줄이 바뀔 때 CR(0d) LF(0a) 가 붙는다.   
-----------
41 0d 0a 41 
-----------

# POSIX  
  줄이 끝날 때 LF(0a) 가 붙는다. 
-----------
41 0a 41 0a
-----------
```

