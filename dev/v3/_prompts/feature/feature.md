---
description: visit-admin 서비스에서 수행할 작업 내용, when start new chat use this rule for checkout project summary 
globs:
alwaysApply: false
---

> 해당 파일은 예제용 파일 
> 해당 템플릿? 내용 참고해서 자유롭게 

# Visit-Admin 작업 내용

## 현재 작업 (ISH-407)
- 방문 설치 업무 기록 
- 메인 페이지에서 "완료"인 작업들을 클릭한 경우 전환되는 페이지 

### view 모드 
<img width="271" height="584" alt="View Mode Design" src="./images/view-mode-design.png" />

- 완료된 작업들을 API 를 통해서 조회해서 업무 사진 내용 섹션 과 메모 섹션을 노출
- 업무 기록 수정 버튼을 클릭시 edit 모드로 변경 

### API 
- 방문 설치 업무 기록 조회 API 
URL : /public/visit-admin/visit-installation-task/{id}/logs
response : 
```
{
  "success": {
    "downloadUrls": [
      "https://live-ishopcare-server-tossplace/i-partners/6e25fafb-2504-462e-b6e8-82093671bb18/c6281714-81cb-4108-84cf-4d9dd94e65ee.jpeg"
    ],
    "taskMemo": "프론트 설치 완료"
  }
}
```



### edit 모드 
<img width="271" height="584" alt="Edit Mode Design" src="./images/edit-mode-design.png" />

- edit 모드는 입력된 이미지를 삭제할 수 있고 추가할 수 있음 최대 5개 
  - TaskNoteSectionInprogress 부분을 참고해서 이미지 업로드 thubnail 등을 참고해서 재사용하기
- 저장 버튼 클릭시 "업무 기록을 수정했어요." toast 메시지를 띄우고 view 모드로 전환 

### API 
- 방문 설치 업무 기록 수정 API 
- 해당 API 는 
URL : /public/visit-admin/visit-installation-task/{id}/log
request : updateTask API 로 정의 
```
{
  "taskStatus": "진행중",
  "taskMemo": "프린터 설치 완료 / 장비 추가 구매 하셔서 재방문 필요",
  "files": [
    {
      "downloadUrl": "https://live-ishopcare-server-tossplace/i-partners/6e25fafb-2504-462e-b6e8-82093671bb18/c6281714-81cb-4108-84cf-4d9dd94e65ee.jpeg",
      "fileName": "c6281714-81cb-4108-84cf-4d9dd94e65ee.jpeg"
    }
  ]
}
```
response : 
```
{
  "success": {
    "downloadUrls": [
      "https://live-ishopcare-server-tossplace/i-partners/6e25fafb-2504-462e-b6e8-82093671bb18/c6281714-81cb-4108-84cf-4d9dd94e65ee.jpeg"
    ],
    "taskMemo": "프론트 설치 완료"
  }
}
```


## 주의 할 것 
- 현재 중복되는 부분이 있는데 이를 재사용하기 위해 고려가 필요함 
- 너무 많은 변경 사항이 있는 경우 순차적으로 진행할것 
- 나의 코드 철학 기반으로 설계 및 코드 작성할 것 
- 설계 먼저를 진행하고, 논의가 마무리 되면 그때 개발 진행하기, 이떄도 한번에 다 작성하는 것이 아닌 순착적으로 ㅐ발을 펲어 프로그래밍으로 진행하기 
- 최대한 기존의 작업을 유지하고, 어쩔 수 없는 변경 사항은 허용 
- think mode 로 실행하기 
