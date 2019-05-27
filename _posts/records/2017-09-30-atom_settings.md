---
category: record
---
### 아톰에서 markdown-preview 단축키(`ctrl+shift+m`) 안먹힐 때

`ctrl+.`를 눌러서 확인해 보면 emmet의 라인합치기 단축키와  겹쳐서 emmet 을 설치했을 경우 해당 문제가 발생하는 것 같다(추측)  

해결 방법으로는: settings -> open config folder 클릭 -> keymap.cson 파일 열기 -> 아래내용 입력 및 저장

---아래---
```
'.editor:not(.mini)':
  'ctrl-shift-M': 'markdown-preview:toggle'
  ```
